---
layout: post
title: "CQRS, PHP and the Governor Framework"
description: desc
headline: headlle
modified: "Wed Jul 23 2014 02:00:00 GMT+0200 (Central Europe Daylight Time)"
category: cqrs
tags: "php,cqrs,event sourcing"
imagefeature: cover1.jpg
mathjax: false
chart: null
comments: true
featured: true
published: true
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

With the basic building blocks already in place it is now the right time to evolve the library even further - since it was taken from the Java world certain parts doesn't make that much sense in PHP the way they are implemented at the moment - for example repository locking and saga management being the hottest issues. After hours and hours of research, reading and googling the plans for further development and improvement slowly started to emerge from the dark: 

[React PHP](http://reactphp.org/) 

Once I came to know Node.JS better I instantly become a big fan and endorser of this technology becase the power and potential it packs is simply amazing. 

