```
<!--metada
{
    "slug": create-a-serverless-blog,
    "title": "Create a Serverless Blog",
    "subTitle": "TODO",
    "description": "TODO",
    "language": "en",
    "coverImage": <string>,
    "tags": ["serverless"]
}
-->
```


# Serverless Blog

A Blog? really? in this day an age? I hear you say. But there are many valid reasons to [start blogging in 2018](https://problogger.com/start-a-blog-in-2018/). I have recently been working with the serverless model and wanted to document my progress. And then it hit me, why not create a small blog powered by lambda functions? Here's what happened...

## Building Blocks
I first decided to jot down a rough sketch of what I wanted to accomplish, these were my inital thoughts:

 - O.K, Let's write the posts with mark down.
 - Let's version them with git. Great!
 - Let's hook some triggers so they react to changes in the master branch! Nice :)
 - Let's convert these blogs into plain HTML and store them in s3
 - Also, let's include some metadata so there's meaningful info attached to the posts.
 - Oh farts!, now I need a database.
 - Ok let's include some lambda CRUDs to interact with dynamoDb.
 - and let's make a small API public so my client app can fetch the data.
 - As a safe measure, let's provision the orchestration with cloudformation. 

 
<img src='https://user-images.githubusercontent.com/4896851/39407041-e62dcd4a-4bb7-11e8-8f58-a7d49df5093e.png' width=450 height=450 />

You may have noticed that I am relying heavily on AWS to accomplish this task, this is not to say that you 

### - BlogPost

```
BlogPost: {
    uuid: <string>,
    slug: <string>,
    title: <string>,
    subTitle: <string>,
    description: <string>,
    createdAt: <date>,
    updatedAt: <date>,
    language: <string>,
    content: <HTML>,
    coverImage: <string>,
    tags: [ <string> ]
}
```

### - Blogs

```
Blogs: [ <BlogPost > ]
```


```
aws dynamodb create-table \
    --cli-input-json file://input.json
```
