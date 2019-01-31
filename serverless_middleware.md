# Handling complexity in lambda functions
<p style="font-size:10px; text-align:right; margin: 0;">1st Feb 2019</p>
<p style="font-size:10px; text-align:right">5min</p>

# TLDR;



I have had the luck and pleasure of working with lambda functions recently and boy I am having fun! The idea of having an event driven execution environment is both daring and exciting.

If you are new to the FaaS world, don't worry, the community has already prepared a curated list of reads for you [here](https://github.com/anaibol/awesome-serverless) and [here](https://github.com/pmuens/awesome-serverless). Have a browse, drink some coffee.

As you can see, (because you checked at least one of the links, didn't you?) A huge weight has been lifted. We can now develop, run and manage applications without the intricacy of building and maintaining an infrastructure. But with new beginnings, new challenges arise. One of such problems and the reason of this blog is function complexity.

## How complex is too complex?

Generally speaking, when writing lambda functions you would organise them into a series of dedicated modules following the [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle). If you would rather do otherwise, please read this excellent blog comparing [monolithic vs single purposed functions](https://hackernoon.com/aws-lambda-should-you-have-few-monolithic-functions-or-many-single-purposed-functions-8c3872d4338f). Isolating application from business logic reduces code complexity, simplifies reasoning and will help you debug your code more efficiently. Your colleagues and your future self will thank you.

Take for example some sort of CRUD api, we would like to make sure that a payload is properly formatted before handling any data. The payload validation could emit an error that would communicate a status to the consumer. If the validation was successful it would then move onto executing some business logic. For both scenarios we would also want to set up some percentile tracing and error loggings. While all of these sound great, we have essentially mapped out a request lifecycle with various [cross-cutting concerns](https://en.wikipedia.org/wiki/Cross-cutting_concern):

- Data validation.
- Error detection and correction.
- Logging.
- Monitoring.

<div align="center" style="padding: 15px">
    <img src="https://media.giphy.com/media/W96QyV2ACacGk/giphy.gif" height=200>
</div>

Following the serverless mantra, we should outsource anything that isn't related to the core business, so wouldn't it be great if we could just focus on the business logic and outsource these application specific problems too? 
Of course this isn't a new concept and we have been using it in web frameworks like [hapijs](http://hapijs.com) and [express](https://expressjs.com) for quite sometime now.

## Lambda Middleware

A lambda [middleware](https://en.wikipedia.org/wiki/Middleware) is essentially a function that contributes to managing cross-cutting concerns in a consistent manner. It provides a clear separation of application and business logic and outlines an easy to reason configuration cycle. It is the glue that when adopted by everyone in a team becomes the lingua franca that supports and creates features. But do we really need a middleware? Well, look at the current state of your lambdas and ask yourself a few questions:

 - What does the code look like?
 - can you envision anyone supporting it in at least 2 years?
 - How much business logic expertise do you need? (if specialised then separation is imperative)
 - Is logic intertwined and hard to reason about?
 - Does the project have a clear path or do you see it going in different directions?
 - How will the project look like the day the expert leaves?
 - Are best practices enforced and maintained?

<div align="center" style="padding: 15px">
    <img src="https://media.giphy.com/media/3sZlRwZfxAI8g/giphy.gif" height=200>
</div>

# Enter Lambcycle

[Lambcycle](https://github.com/juliantellez/lambcycle) 🐑 🛵 is a declarative middleware for lambda functions. It defines a configurable event cycle and allows you to focus on your application's logic. It has a "Feature as Plugin" approach, so you can easily create your own plugins or reuse your favorite packages with very little effort.

<div align="center" style="padding: 15px">
    <img src="https://raw.githubusercontent.com/juliantellez/lambcycle/develop/assets/lifecycle.svg?sanitize=true" width=400>
</div>

In practice there is no such a thing as an un-opinionated framework, every idea is an opinion and a way of working with your application. Lambcycle is created around the idea that configuration is better than code and that business logic should be as isolated as possible. It's main goal is to shift the conversation around predictable cycles while promoting reusability and consistent error handling across all functions and teams.

Lambcycle is built with [developer experience "DX"](https://hackernoon.com/the-best-practices-for-a-great-developer-experience-dx-9036834382b0) in mind and ships with [type](https://www.typescriptlang.org) definitions, to promote a smoother adoption 🚀 (VScode only).

<div align="center" style="padding: 15px">
    <img src="https://user-images.githubusercontent.com/4896851/51274743-db4db500-19c7-11e9-903c-cb50d127d933.gif" width=600>
</div>

# Summary

