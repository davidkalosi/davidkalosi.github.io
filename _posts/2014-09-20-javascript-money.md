---
layout: post
published: true
title: Money object and Javascript
mathjax: false
featured: true
comments: true
modified: "2014-09-20"
categories: 
  - Node.JS
tags: "node,javascript,money,currency,fowler,ddd,value object"
---

Everybody who ever had to work with monetary values should know Fowlers Money pattern as well as why storing money as a floating point number is a bad bad idea.

I won't go into any more details on this subject since there are good resources all over the internet. There are also libraries addressing this issue for most of the languages. For example [this](https://github.com/mathiasverraes/money) and [this](https://github.com/JodaOrg/joda-money). 

What I wanted to write about is that the lack of a solid library in Javascript made me write [JS Money](https://github.com/davidkalosi/js-money). 

It does everything what it is supposed to do 

- Basic arithmetics (add/subtract/multiply/divide)
- Comparison and equality checks
- Currency support
- Funds allocation

Like all the other ones it is super easy to use. Simply install with npm and then

{% highlight javascript %}
var Money = require('js-money');

var fiveEur = new Money(500, Money.EUR);
var tenEur = fiveEur.multiply(2);

{% endhighlight %}

For detailed docs just head over to [Github](https://github.com/davidkalosi/js-money)

So from know on no lost cents in Javascript as well :)
