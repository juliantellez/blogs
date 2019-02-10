# Handling complexity in lambda functions
<p style="font-size:10px; text-align:right; margin: 0;">6st Feb 2019</p>
<p style="font-size:10px; text-align:right">8min</p>

### TLDR;
> Middlewares can handle the complexity of your lambdas while isolating business logic and cross-cutting concerns in reusable components that can be modeled by event cycles.


I have had the luck and pleasure of working with lambda functions recently and boy I am having fun! The idea of having an event driven execution environment is both daring and exciting.

If you are new to the FaaS world, don't worry, the community has already prepared a curated list of reads for you [here](https://github.com/anaibol/awesome-serverless) and [here](https://github.com/pmuens/awesome-serverless). Have a browse, drink some coffee.

As you can see, (because you checked at least one of the links, didn't you?) A huge weight has been lifted. We can now develop, run and manage applications without the intricacy of building and maintaining an infrastructure. But with new beginnings, new challenges arise. One of such problems and the reason of this blog is function complexity.

## How complex is too complex?

Generally speaking, when writing lambda functions you would organise them into a series of dedicated modules following the [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle). If you would rather do otherwise, please read this excellent blog comparing [monolithic vs single purposed functions](https://hackernoon.com/aws-lambda-should-you-have-few-monolithic-functions-or-many-single-purposed-functions-8c3872d4338f). Isolating application from business logic reduces code complexity, simplifies reasoning and will help you debug your code more efficiently. Your colleagues and your future self will thank you.

Take for example some sort of CRUD API, we would like to make sure that a payload is properly formatted before handling any data. The payload validation could emit an error that would communicate a status to the consumer. If the validation was successful it would then move onto executing some business logic. For both scenarios we would also want to set up some percentile tracing and error loggings. While all of these sound great, we have essentially mapped out a request lifecycle with various [cross-cutting concerns](https://en.wikipedia.org/wiki/Cross-cutting_concern):

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
 - Can you envision anyone supporting it in at least 2 years?
 - How much business logic expertise do you need? (if specialised then separation is imperative)
 - Is logic intertwined and hard to reason about?
 - Does the project have a clear path or do you see it going in different directions?
 - How will the project look like the day the expert leaves?
 - Are best practices enforced and maintained?

<div align="center" style="padding: 15px">
    <img src="https://media.giphy.com/media/3sZlRwZfxAI8g/giphy.gif" height=200>
</div>

## Enter Lambcycle

[Lambcycle](https://github.com/juliantellez/lambcycle) 🐑 🛵 is a declarative middleware for lambda functions. It defines a configurable event cycle and allows you to focus on your application's logic. It has a "Feature as Plugin" approach, so you can easily create your own plugins or reuse your favorite packages with very little effort.

In practice there is no such a thing as an un-opinionated framework, every idea is an opinion and a way of working with your application. Lambcycle favours configuration over code and believes that business logic should be as isolated as possible. 

<div align="center" style="padding: 15px">
    <img src="./assets/lambcycle-lifecycle.png" width=400>
</div>

Lambcycle enhances lambda functions with a few extension points (see graph), each of which can be used to interact with the event in a decomposed manner.

- The first extension point is `Request` which occurs immediately after the lambda is called. You can use this step for parsing, validation, etc...
Note: If you are thinking of auth, please consider a [lambda authoriser](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html) instead. 

- The `Pre Handler` extension comes in handy when you need to adapt data to fit an interface. Its also a great place for fetching secrets.

- The `Handler`, where your beautiful business logic lives.

- Next up is the `Post Handler`, use this extension to validate and/or cache the output of your business logic.

- `Error` is an implicit extension for logging and tracing.

- And finally `Pre Response`, your chance to format a response to the consumer (which could be data or an error).

## Using the middleware

```javascript
import Joi from 'joi'
import lambcycle from 'lambcycle'
import joiPlugin from 'lambcycle/dist/plugin-joi'
import bodyParser from 'lambcycle/dist/plugin-body-parser'
import realTimeEventTracker from './bugFreeBizLogic'

const businessLogic = async(event, context) => {
    const {error, data} = await realTimeEventTracker(event.data)

    if(error) {
        throw error
    }

    return data;
};

const schema = Joi.object().keys({
    assetId: Joi.string().required(),
    competition: Joi.string().required(),
    sport: Joi.string(),
    tournamentCalendar: Joi.date(),
}).required()

const handler = lambcycle(businessLogic)
.register([
    bodyParser({type: 'json'}),
    joiPlugin(schema)
])

export default handler
```

The example above leverages [express' body parser](https://github.com/expressjs/body-parser) and [hapi's joi validation](https://github.com/hapijs/joi). We have configured both plugins with a parsing type and a validation schema respectively. By abstracting these two we have rid of complexity and shift the focus to the business logic. Note that a more robust lambda would include error tracing, and response plugins.

## Crafting a Plugin

The possibilities are endless when it comes to plugins! Do you have something in mind? [Contributions](https://github.com/juliantellez/lambcycle/blob/develop/contributing.md) are more than welcome ❤️

```javascript
import * as Sentry from '@sentry/node';
import MyAwesomeIntegration from './MyAwesomeIntegration'

const sentryPlugin = (config) => {
    Sentry.init({
        dsn: `https://${config.key}@sentry.io/${config.project}`,
        integrations: [new MyAwesomeIntegration()]
    });

    return {
        config,
        plugin: {
            onPreResponse: async (handlerWrapper, config) => {
                Sentry.captureMessage('some percentile log perhaps?')
            },
            onError: async (handlerWrapper, config) => {
                Sentry.captureException(handlerWrapper.error);
            }
        }
    }
}

export default sentryPlugin;
```
In your lambda ....

```javascript
import lambcycle from 'lambcycle'
import sentryPlugin from './sentryPlugin'

const myApplicationLogic = async (event, context) => {
    return await someLogic()
}

const handler = lambcycle(myApplicationLogic)
.register([
    sentryPlugin({
        key: process.env.SENTRY_KEY,
        project: process.env.SENTRY_PROJECT,
    })
]);

export default handler;
```

# DX
Lambcycle has been built with [developer experience "DX"](https://hackernoon.com/the-best-practices-for-a-great-developer-experience-dx-9036834382b0) in mind and ships with [type](https://www.typescriptlang.org) definitions, for consistency and auto-completion 🚀 (VScode only).

<div align="center" style="padding: 15px">
    <img src="https://user-images.githubusercontent.com/4896851/51274743-db4db500-19c7-11e9-903c-cb50d127d933.gif" width=600>
</div>

## Conclusion

It is a brave new world and serverless is here to stay! Promoting reusable components and consistent error handling will help you and your team create and support features in an more controlled and organised fashion. Lambcycle's main goals are to shift the conversation around a predictable unidirectional cycle and encouraging component reusability across features. Its worth mentioning that for more complex scenarios you should be looking at [Step functions](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html). Let me know if you would like to know more about them in the comments, in the meantime check out these [use cases](https://aws.amazon.com/step-functions/use-cases/)!

Special thanks to the legendary [burning monk](https://theburningmonk.com/) for the in-depth review of this article.
