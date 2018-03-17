# Clone private git repo with dockerfile

![img](https://e003241e1ca22fe3a801-e6bf09af38ee59e24e7cffd16c43a1d9.ssl.cf2.rackcdn.com/68-deploy-with-git-docker.png)

Docker images are great, not only they are the basis of containers, but they are stateles and predictable. Sometimes however there are some confusing moments. Accessing dependencies from private repositories is one of such moments.

```
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

This means that the docker's container doesn't have a means of identifying against an SSH server. So let's see how do we go about solving this problem.

## First Approach: Pass the ssh key as a build arg
We know that we would like any dev or machine to be able to access the private repo. A basic approach would be to pass the ssh-key as a build argument to the docker container.

```
# [...]
ARG SSH_KEY
RUN mkdir -p /root/.ssh/
RUN echo "${SSH_KEY}" >> /root/.ssh/id_rsa && chmod 600 /root/.ssh/id_rsa

# Accept github domain
RUN touch /root/.ssh/known_hosts
RUN ssh-keyscan github.com >> /root/.ssh/known_hosts

# [...]
RUN rm /root/.ssh/id_rsa
```

```
SSH_KEY=$(cat ~/.ssh/id_rsa)
docker build [...args] --build-arg SSH_KEY=$SSH_KEY .
```

The above works pretty well. However, the key can be easily retrieved from the images history `docker history [IMAGE ID]`. I know what you are thinking...

![not great](http://library.ryerson.ca/wp-content/uploads/not-great-bob.jpg)

## Second Approach: squash your builds?
Docker builds can be conviniently squashed, this removes deleted files and reduces multiple build layers to a single one between the origin and the HEAD of the build.

```
docker build --squash [...]
```

So this seems like a convinient approach, but one has to make sure to always squash the builds. Also, reducing the caching layer could make the builds slower. Not to mention that in both approaches the key may be available in the terminal's history.


## Third Approach: Add the ssh key to the context of the build

So we know we would like to have an agnostic configuration, but we would also like to keep our secrets, well ... secret. At this point you may be thinking of several approaches:

- You could fetch the key from a local server

```
# [...]
RUN mkdir -p /root/.ssh/
RUN wget -O /root/.ssh/id_rsa http://192.168.XX.X:8080/id_rsa
```

- Or you could fetch from an s3 bucket 

```
RUN aws s3 cp s3://my-secrets/ /root/.ssh/
```

But it seems counter productive to make your images dependent on external configurations. It makes them less predictable and error prone, it may seem simple enough to reason about a single config, but controlling which secrets are exposed to the images when running multiple Dockerfiles can make your recipes hard to control.

A simple alternative 

```
# [...]
RUN mkdir -p /root/.ssh/
ADD id_rsa /root/.ssh/id_rsa
# [...]
RUN rm -f /root/.ssh/
```

```
cp ~/.ssh/id_rsa id_rsa
docker build [...args] --build-arg SSH_KEY=$SSH_KEY .
rm id_rsa
```

### Cons:
- You need to make sure you tidy the workspace, small price for security


## Bonus:
When building images for production environments. You probably want to make sure you manage sensitive data with [Docker Secrets](https://docs.docker.com/engine/reference/commandline/secret_create/)
