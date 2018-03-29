# [ VEL-813: Spike ]: Ascertain castlabs errors

### Errors Handling

There are two different ways of catching errors with the presto-player

- 1 catching a rejected promise
- 2 listening to the error event provided by the sdk

#### catching a rejected promise

```
promise.then(onFulfilled[, onRejected])
```

Pros:

- This is generally a best practice procedure and its second nature to developers  

Cons: 

- Relies on promises to be handled accurately by the provider which hasn't been the case in some occassions


#### listening to the errors event

```
prestoPlayer.on('error', handler)
```

Pros:

- Creates a single interface to deal with errors that can be subscribed to at any point in time creating a single point of failure


Cons:

- Not all erros may be caught by the event emmitter


### What errors are reported?
The presto player relies on [videoJS](http://videojs.com/) and [shaka-player](https://opensource.google.com/projects/shaka-player) to handle errors. It then wraps around them to communicate them to the host.
Both engines have a very well defined set of errors :

- [Shaka errors](https://shaka-player-demo.appspot.com/docs/api/shaka.util.Error.html)
- [Video JS](https://docs.videojs.com/mediaerror)

Preso also has a defined set of erros, but not as evolved and well documented. You may find that checking directly with shaka or video JS will speed your query.

Pros:
    - should unify error schemas accross platforms

Cons:
    - Some error events haven't been unified resulting in diff behaviours accross platforms (ie dash vs hls)
    - Abstracted interfaces are not well documented, nor linked to its original source (shaka/videoJS)
    - Intermediate handling seems to be doing extra work
    

### Findings

This spike has focused on network and drm erros in particular. We have found that even though errors are notified they may differ accross platforms 

#### Use Case: 404 manifest not found
Test: Provide a fake manifest url to the presto-player

Dash: htpps://dummy/file.mpd
<img width="739" alt="screen shot 2018-04-23 at 12 39 34" src="https://user-images.githubusercontent.com/4896851/39124687-3fd7ffb4-46f4-11e8-92c4-72635c2cb8c7.png">

HLS: htpps://dummy/file.m3u8
<img width="1116" alt="screen shot 2018-04-23 at 12 42 29" src="https://user-images.githubusercontent.com/4896851/39124688-3fef3c06-46f4-11e8-86a8-2a406fcf2227.png">

Result: 
The error has been caught in both scenarios, however their errorcodes are different. 
Since the error handling is inconsistent accross platforms, we might end up with customised solutions which is a smell in itself given that a correct mapping should be supplied by the presto-player.
The presto-player's host shouldn't have to be aware of where the errors come from `shaka vs videojs` but having different behaviours forces an implementation that is vendor aware which is a potential bottleneck.