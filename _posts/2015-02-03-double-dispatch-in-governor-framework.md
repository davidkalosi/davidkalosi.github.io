---
layout: post
published: true
title: Double dispatch in Governor Framework
mathjax: false
featured: true
comments: true
categories: 
  - CQRS
  - PHP
---

There is a high probability that everyone reading this article already knows what double dispatch is and when and why to use it.

If not this [article](http://lostechies.com/jimmybogard/2010/03/30/strengthening-your-domain-the-double-dispatch-pattern/) covers it pretty well.

What I want to focus on today is how to utilise this pattern in Governor Framework on this simple example.

{% highlight php startinline=true %}
class DiscountPolicy
{
   public function calculate(Cart $cart) 
   {
     // actual logic here
   }
}

class Cart extends AnnotatedAggregateRoot
{
  /**
   * @AggregateIdentifier
   */
  private $identifier;
   
  /**
   * @CommandHandler
   */
  public function applyDiscount(ApplyCartDiscountCommand $command)
  {
	// need to access DiscountPolicy 
  }
}

{% endhighlight %}

The goal is to access the logic in the DiscountPolicy in the command handler.
At a first glance one would say inject the service into the aggregate. That seems tempting but it is not such a great idea as one would say.
Read [here](http://lostechies.com/jimmybogard/2010/04/14/injecting-services-into-entities/).

Instead of introducing a class level dependency that is required only in specific scenario we will inject the service as an argument.
Governor provides a ParameterResolver mechanism that can resolve method parameters and one of the implementations will inject a service from the Symfony DIC.

{% highlight php startinline=true %}

class Cart extends AnnotatedAggregateRoot
{
  /**
   * @AggregateIdentifier
   */
  private $identifier;

  
  /**
   * @CommandHandler
   * @Resolve(parameter=“policy”, resolver = @Inject(“cart.discount_policy”))
   */
  public function applyDiscount(ApplyCartDiscountCommand $command, DiscountPolicy $policy)
  {
        $discount = $policy->calculate($this);
        
        $this->apply(new CartDiscountAppliedEvent($command->getIdentifier(), $discount);
  }
}
{% endhighlight %}