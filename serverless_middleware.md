I have had the luck and pleasure of working with lambda functions recently and boy I am having fun! The idea of having an event driven execution environment is both daring and exciting.

If you are new to the FaaS world, don't worry the community has already prepared a curated list of reads for you [here](https://github.com/anaibol/awesome-serverless) and [here](https://github.com/pmuens/awesome-serverless). Have a browse, get some coffee.

As you can see, (because you checked at least one of the links, didn't you?) A huge weight has been lifted. We can now develop, run and manage applications without the complexity of building and maintaining an infrastructure. But with new beginnings, new challenges arise. One of such problems and the purpose of this blog is function complexity.

## How complex is too complex?

Take for example some sort of CRUD api, you would like to make sure that your payload is properly formatted before handling any data. The payload validation could emit an error that would communicate a 500 to the consumer. While all of these sounds great, your lambda didn't get to execute any business logic. This scenario screams configuration and isolation.

![](https://media.giphy.com/media/W96QyV2ACacGk/giphy.gif)

I have come to realise that serverless is best approached as a mindset:

"outsource anything that isn't related to your core business".

For the lambda context this means a cleaner distinction between cross-cutting concerns such as.

- Error detection and correction.
- Data validation.
- Logging.
- Monitoring.

Wouldn't it be great if you could just focus on your business logic and outsource these application specific problems too?

This of course isn't a new concept and we have been using such approach in web frameworks like [hapijs](http://hapijs.com) or [express](https://expressjs.com) for sometime now.

![](https://media.giphy.com/media/3sZlRwZfxAI8g/giphy.gif)

## Enter Middleware

Serverless is perfect for fanning-out and scaling tasks that can be run in parallel

- why do you still need middlewares in Lambda

* what inspired you to create the project

Lambcycle is a declarative lambda middleware. Its main purpose is to let you focus on the specifics of your application by providing a configuration cycle.

debugging serverless is hard

when things work is fantastic but when they dont is a nightmare

- Developers First
  Lambcycle is built with the developer in mind.

- Predictability
  predictable life cycle

configuration over code

clean separation between business logic and the framework

reduce errors , reusability
consistent error handling across all functions and teams

- how we're using it at DAZN, etc.

I feel that some features require a cleaner isolation from your application logic.
I have found that in practice there is no such a thing as an unoppinionated framework, every feature is an opinion and a way of working with your application.
Lambcycle is created around the idea that configuration is better than code and that application logic should be as isolated as possible. With a long list of chained events you may find some hard to the bug problems that are simply a flow ordering issue. You could also choose to extract all of this in step functions or have authorizers that sit before your handler, and I recommend you to do it, but remember that not every problem is a nail and not every solution a hammer.
