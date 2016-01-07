---
layout: post
category : programming
tags : [Hystrix, monitoring, java]
---
{% include JB/setup %}

When you observe the [Hystrix's dashboard](https://github.com/Netflix/Hystrix/tree/master/hystrix-dashboard) (which is sooo cool by the way) you will find one statistic labelled as "Bad Request" - the yellow number on the dashboard. Unfortunately it's not that easy to find out whether you should be worried by the yellow-coloured statistic. 

### So what on Earth is it?

According to the documentation a Bad Request is a request handled by a Hystrix command which did not throw an exception but is not seen as a proper request. Pretty simple, right? But when can you actually get a result which Hystrix will treat as a Bad Request? One situation is when you use the Hystrix Command's ability to ignore certain exceptions. If you ignore a concrete exception and the exception is thrown, Hystrix will not fire the fallback method but will not treat the result as a success neither - it will be classified as a Bad Request.

### How can it harm me?

Well, it can't cause physical pain of course, but it can become a bit of a nuisance. One of the properties of a Bad Request is that it is not taken into consideration when making decisions on the Circuit Breaker, whether Hystrix is considering opening or closing one. This means that once a Circuit Breaker opens and the one request which is made to check whether it can be closed results in a Bad Request the Breaker will remain open.

This has actually happened at a project I've been working with recently. Many of our requests were treated as Bad Requests - we've been ignoring any `HttpClientNotFoundException`. The Circuit Breaker opened during a short hiccup of the remote service. It could not close itself afterwards, even though the remote resource was working fine. Every request made to check the service ended with a 404, and a Bad Request was not treated as a success so the Breaker was kept open. If we were lucky and get one 200 status the Circuit would close.

This shows that you have to be careful when letting a Hystrix Command to ignore certain exceptions.

I hope this helps to clarify things a bit on the topic of Hystrix's Bad Requests.
