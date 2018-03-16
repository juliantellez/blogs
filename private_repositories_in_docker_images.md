# Clone private git repo with dockerfile

Docker images are great, not only they are the basis of containers, but they are stateles and predictable. Sometimes however there are some confusing moments.
One of such problems is using dependencies from private repositories.

TODO: expand these approaches, make them more witty ?

## First Approach: Pass the ssh key as a build arg

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

### Cons:
- The key can be easily retrieved from the images history `docker history [IMAGE ID]`


## Second Approach: Add the ssh key to the context of the build

```
docker build --squash [...]
```

### Cons:
- One has to make sure to always squash the builds
- Reducing the caching layer could make the builds slower
- In both approaches the key will be available in the terminal's history `cat ~/.bash_history`

## Third Approach: Add the ssh key to the context of the build
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
