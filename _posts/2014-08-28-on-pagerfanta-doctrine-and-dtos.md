---
layout: post
published: false
title: "On Pagerfanta, Doctrine and DTOs"
mathjax: false
featured: false
comments: true
categories: 
  - PHP
tags: "symfony,doctrine,pagerfanta,dto,php"
---

Let's suppose we have an entity repository returning an array of Doctrine entities feeding a DTO assembler that created the output data, or the "view" model.
Pagination is of course needed since we have plenty of data. As I found out this scenario it is not that easy to implement with the tools in hand.

There are of course another possibilites - using a dedicated repository and native SQL to create our DTO objects directly but for the sake of this article let's pretend we are stuck with what we have for whatever reasons.

We will assume a simple invocing application as the basis of our example and our simplified domain model looks the following

{% highlight php %}
/**
 * @ORM\Entity(repositoryClass="InvoiceRepository")
 */
class Invoice 
{
	/**
     * @ORM\Id
     * @ORM\Column(name="id", type="string")
     */
	private $id;
    
    /**
     * @ORM\Column(name="due_date", type="datetime")
     */
	private $dueDate;
	
    // and so on
}
{% endhighlight %}

and the repository which among others defines a method for finding all my overdue invoices. For now we will stick with a doctrine entity repository.

{% highlight php %}
class InvoiceRepository extends EntityRepository
{

	public function findOverdueInvoices()
  	{
		$dql = "SELECT i FROM Invoice i WHERE i.dueDate >= CURRENT_TIMESTAMP()";
		$query = $this->_em->createQuery($dql);
        
		return $query->getResult();
  	}
}
{% endhighlight %}

This repository method can be used in multiple places 
- an email notification service that sends out notification emails 

{% highlight php %}
class OverdueInvoiceNotificationService
{
	private $repository;

	public function sendNotifications()
	{
		$invoices = $this->repository->findOverdueInvoices();
		
		foreach ($invoices as $invoice) {
			// send out email notifications
		}
	}
}
{% endhighlight %}

- a REST api feeding the GUI of our application

Since sending domain objects directly to the GUI is not a good idea we have an application facade that returns DTOs created from the domain object. This coud like like 

{% highlight php %}
class InvoiceFacade 
{
	private $repository;
	private $dtoAssembler;

	public function getOverdueInvoices()
	{
		$dtos = array();
		$invoices = $this->repository->findOverdueInvoices();
		
		foreach ($invoices as $invoice) {
			$dtos[] = $this->dtoAssembler->createDto($invoice);
		}

		return $dtos;
	}
}
{% endhighlight %}

So far so good but since now we are thrashing out a gazillion invoices for each request into the GUI some server side pagination would come handy to reduce the response size and improve our GUI response time. 

Time to refactor certain parts

{% highlight php %}
class InvoiceFacade 
{
	private $repository;
	private $dtoAssembler;

	public function getOverdueInvoices($page, $rows)
	{
		$dtos = array();
		$invoices = $this->repository->findOverdueInvoices();
		
		foreach ($invoices as $invoice) {
			$dtos[] = $this->dtoAssembler->createDto($invoice);
		}

		$adapter = new ArrayAdapter($dtos);
		$pagerfanta = new Pagerfanta($adapter);
		$pagerfanta->setMaxPerPage($rows)
                	->setCurrentPage($page);

		return $pagerfanta;
	}
}
{% endhighlight %}

{% highlight php %}
class InvoiceController
{
	/**
	 * @Rest\QueryParam(name="page", requirements="\d+", default="1")
 	 * @Rest\QueryParam(name="rows", requirements="\d+", default="25")
     */
     
	public function getOverdueInvoicesAction(ParamFetcher $paramFetcher)
	{
    	return $this->getFacade()->getOverdueInvoices($paramFetcher->get('page'),
			$paramFetcher->get('rows'));
	}
}
{% endhighlight %}

Cool, now the GUI gets max 25 rows on each request but we did not make any improvement on the backend side since we are still fetching all the rows from the database, transforming them into DTOs and then throwing the majority of the result set away. That means we have to find a way to start limiting this ad the database level so we need to get back to the repository. 

We could modify the repository and hack in the DoctrineORMAdapter but that would break our emailing service since it expects all the rows not a paginator.

An intermediate solution that could actually work would be modifying the repository to return a QueryBuilder insted of the results.
That would give us

{% highlight php %}
class InvoiceRepository extends EntityRepository
{

  public function findOverdueInvoices()
  {
	return $this->createQueryBuilder("i");
  }

}
{% endhighlight %}

With a minor change the notification service is running as expected.

class OverdueInvoiceNotificationService
{
	private $repository;

	public function sendNotifications()
	{
		$invoices = $this->repository->findOverdueInvoices()->getResult();
		
		foreach ($invoices as $invoice) {
			$this->notify($invoice);
		}
	}
}
{% endhighlight %}

But things do not work very well on the other side. While this code may look OK at the first - we are losing pagination information by returning just the array of DTOs. Returning the Pagerfanta directly will serialize the domain objects and not the DTOs. 

{% highlight php %}
class InvoiceFacade 
{
	private $repository;
	private $dtoAssembler;

	public function getOverdueInvoices()
	{
		$dtos = array();
		$qb = $this->repository->findOverdueInvoices();
		$adapter = new DoctrineORMAdapter($qb);
		
		$pagerfanta = new Pagerfanta($adapter);
		$pagerfanta->setMaxPerPage($rows)
                	->setCurrentPage($page);
		
		foreach ($invoices as $pagerfanta->getCurrentPageResults()){
			$dtos[] = $this->dtoAssembler->createDto($invoice);
		}

		return $dtos;
	}
}
{% endhighlight %}

Fortunatelly there is an elegant solution for this - we will create a new implementation of the PagerfantaAdapter called DelegatingCallbackAdapter and as can be guessed from its name it will serve the following purpose

- it will delegate the pagination to another adapter
- and will apply a callback for each paginated result.

The implementation

{% highlight php %}
class DelegatingCallbackAdapter implements AdapterInterface
{

    /**
     * @var AdapterInterface 
     */
    private $delegate;

    /**
     * @var \Closure
     */
    private $callback;

    function __construct(AdapterInterface $delegate, \Closure $callback)
    {
        $this->delegate = $delegate;
        $this->callback = $callback;
    }

    public function getNbResults()
    {
        return $this->delegate->getNbResults();
    }

    public function getSlice($offset, $length)
    {
        $slice = $this->delegate->getSlice($offset, $length);

        return array_map($this->callback, $slice);
    }
}
{% endhighlight %}

Now we can rewrite our facade service in the following manner:

{% highlight php %}
class InvoiceFacade 
{
	private $repository;
	private $dtoAssembler;

	public function getOverdueInvoices()
	{
		$qb = $this->repository->findOverdueInvoices();

		$self = $this;
		$callback = function($item) use ($self) {
			return $self->dtoAssembler->createDto($item);
		};

		$adapter = new DelegatingCallbackAdapter(new DoctrineORMAdapter($qb), $callback);		
		$pagerfanta = new Pagerfanta($adapter);
		$pagerfanta->setMaxPerPage($rows)
                	->setCurrentPage($page);
				

		return $pagerfanta;
	}
}
{% endhighlight %}


http://www.whitewashing.de/2013/03/04/doctrine_repositories.html