---
layout: post
title: "CQRS, PHP and the Governor Framework"
description: ""
headline: ""
modified: "2014-08-20"
category: cqrs
tags: "php,cqrs,event sourcing"
imagefeature: cover1.jpg
mathjax: false
chart: null
comments: true
featured: true
published: true
categories: 
  - CQRS
---

Topics like Command Query Responsibility Segregation and Event Sourcing have been around for quite a time, and have spawned a number of fremworks and libraries mostly in the C# and Java communities. So what about PHP?

Sadly not that much. While searching for CQRS on packagist will give you a dozen packages with [Lite-CQRS](https://github.com/beberlei/litecqrs-php) leading the pack a serious CQRS library is/was still missing. And since a project I started working on last year was in a desperate need for one. And after a couple of days experimenting with Lite CQRS I came to a conclusion that something more robust was needed for that particular project. I have spent a copule of more days checking out different non PHP libraries when I fell in love with the [Axon Framework](http://www.axonframework.org/) written in Java. This is what I need but in a different language. So came the crazy idea to take the Axon and port it to a PHP library. 

Hence the [Governor Framework](https://github.com/davidkalosi/GovernorFramework) was born. Since the first commit on the 11th of February 2014 I have made quite a progress and in the current stage almost all of the Axon building blocks are ported ,functional and tested to a certain extent. So what exactly is the library able do at the moment?

- Define aggregates and entities.
- Define command buses and gateways as well as event buses and clusters
- Forwarding the event bus to AMQP 
- Store aggregates in various repository implementations (doctrine, hybrid, event sourcing)
- Define commands and events with their respective handlers.
- Create event stores, currently filesystem and doctrine backends are supported with a MongoDB backend in the works
- Define and manage Sagas - currently only doctrine repository is implemented.
- Integrate the library to Symfony as a Bundle.

So in essence one can write an event sourced application inside Symfony 2 with all the bells and whistles that such an app requires. 

With the basic building blocks already in place it is now the right time to evolve the library even further - since it was taken from the Java world certain parts doesn't make that much sense in PHP the way they are implemented at the moment - for example repository locking and saga management being the hottest issues. 

The command dispatching process also needs to be refactored in order to provide async command dispatching abilities. 

All of this can be summed up in the following roadmap:

- Command handling logic will be moved into a pool of background worker processes with support for both synchronous and asynchronous processing. One command per process.
- Because of this the repository locking has to be implemented so it can cope with multiple processes.

I have spent many hours of research and googling while trying to find the best way to accomplish theese tasks. I have [considered](http://zeromq.org/) a [number](http://reactphp.org/) of [options](https://www.rabbitmq.com/) what library or technology to use since all of them have it's strengths and weaknesses. 

In the end I have settled for [Redis] (http://redis.io/). With Redis we can implement all the components with a single piece of technology like 

- Command buses (lists - PUSH/POP)
- Event buses (Pub/Sub)
- Repository lock maps (hashes maintaining aggregate versions)

I won't commit to any deadlines at this point since there is a ton of POC work ahead to show that we are heading into the right direction - I will report on progress on this blog periodically as the new features will be ready.

Wish me luck ;)
