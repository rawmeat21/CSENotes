https://courses.mooc.fi/org/uh-cs/courses/devops-with-docker-spring-2026/chapter-2/in-depth-dive-into-images

General steps: 

- **Lock your dependencies** — requirements.txt / package.json / pom.xml / go.mod with pinned versions.
- **Pick a base image** — slim official image matching your runtime (python:3.11-slim, node:20-alpine, etc.).
- **Write the Dockerfile** — `FROM <base>`, `WORKDIR /app`, copy dependency manifest and install deps first, then copy the rest of the code, then `CMD`/`ENTRYPOINT`. Deps before code, for layer caching.
- **Add a .dockerignore** — exclude `.git`, `node_modules`, `__pycache__`, `venv`, build artifacts.
- **Build the image** — `docker build -t myproject:latest .`

```bash
$ docker build -t hello-docker .
```


- **Run and test it** — `docker run -p <host>:<container> myproject:latest`; pass env vars/secrets via `-e`/`--env-file`, never bake them into the image.

An example:

Dockerfile:
```dockerfile
# Start from the alpine image that is smaller but no fancy tools
FROM alpine:3.21

# Use /usr/src/app as our workdir. The following instructions will be executed in this location.
WORKDIR /usr/src/app

# Copy the hello.sh file from this directory to /usr/src/app/ creating /usr/src/app/hello.sh
COPY hello.sh .

# Alternatively, if we skipped chmod earlier, we can add execution permissions during the build.
# RUN chmod +x hello.sh

# When running docker run the command will be ./hello.sh
CMD ["./hello.sh"]
```

All instructions in a Dockerfile **except** CMD (and one other) are executed during build time. **CMD** is executed when we call docker run, unless we overwrite it.


```bash
$ docker build -t hello-docker .
 => [internal] load build definition from Dockerfile                                                                                                                                              0.0s
 => => transferring dockerfile: 478B                                                                                                                                                              0.0s
 => [internal] load metadata for docker.io/library/alpine:3.21                                                                                                                                    2.1s
 => [auth] library/alpine:pull token for registry-1.docker.io                                                                                                                                     0.0s
 => [internal] load .dockerignore                                                                                                                                                                 0.0s
 => => transferring context: 2B                                                                                                                                                                   0.0s
 => [1/3] FROM docker.io/library/alpine:3.19@sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b                                                                              0.0s
 => [internal] load build context                                                                                                                                                                 0.0s
 => => transferring context: 68B                                                                                                                                                                  0.0s
 => [2/3] WORKDIR /usr/src/app                                                                                                                                                                    0.0s
 => [3/3] COPY hello.sh .                                                                                                                                                                         0.0s
 => exporting to image                                                                                                                                                                            0.0s
 => => exporting layers                                                                                                                                                                           0.0s
 => => writing image sha256:5f8f5d7445f34b0bcfaaa4d685a068cdccc1ed79e65068337a3a228c79ea69c8                                                                                                      0.0s
 => => naming to docker.io/library/hello-docker
```

During the build we see from the output that there are three steps: [1/3], [2/3] and [3/3]. The steps here represent [layers](https://docs.docker.com/build/guide/layers/) of the image so that each step is a new layer on top of the base image (alpine:3.21 in our case).

Layers have multiple functions. We often try to limit the number of layers to save on storage space but layers can work as a _cache_ during build time. If we just edit the last lines of Dockerfile the build command can start from the previous layer and skip straight to the section that has changed. COPY automatically detects changes in the files, so if we change the `hello.sh` it will run from step 3/3, skipping 1 and 2.

### Switching to shell in our image

```bash
$ docker run -it hello-docker sh
/usr/src/app #
```
We replaced the CMD we defined earlier with `sh` and used `-it` to start the container so that we can interact with it.

### Add a file to your docker image

```shell
$ docker cp ./additional.txt <docker_image_identifier>:/usr/src/app/
```

`/usr/src/app` is just a choice. You are free to change this.

### How to view change to a docker image

```bash
$ docker diff zen_rosalind (name of our docker image)
  C /usr
  C /usr/src
  C /usr/src/app
  A /usr/src/app/additional.txt
  C /root
  A /root/.ash_history
```

### Save changes to the docker image

```bash
$ docker commit zen_rosalind hello-docker-additional
  sha256:2f63baa355ce5976bf89fe6000b92717f25dd91172aed716208e784315bfc4fd
  
$ docker image ls
  REPOSITORY                   TAG          IMAGE ID       CREATED          SIZE
  hello-docker-additional      latest       2f63baa355ce   3 seconds ago    12.9MB
  hello-docker                 latest       444f21cf7bd5   31 minutes ago   12.9MB
```

Technically the command `docker commit` added a new layer on top of the image `hello-docker`, and the resulting image was given the name `hello-docker-additional`.


#### We can just edit the Dockerfile too

```dockerfile
# Start from the alpine image
FROM alpine:3.21

# Use /usr/src/app as our workdir. The following instructions will be executed in this location.
WORKDIR /usr/src/app

# Copy the hello.sh file from this location to /usr/src/app/ creating /usr/src/app/hello.sh.
COPY hello.sh .

# Execute a command with `/bin/sh -c` prefix.
RUN touch additional.txt <--- adding our file

# When running Docker run the command will be ./hello.sh
CMD ["./hello.sh"]
```

Build the Dockerfile with `docker build -t hello-docker:v2 .`

```bash
$ docker run hello-docker:v2 ls
  additional.txt
  hello.sh
```

Note: You can have distinct Dockerfiles under same project directory by naming these files as `Dockerfile.<something>`. When building an image, just specify the used Dockerfile with `--file` or `-f` flag. For example `docker build -t tester -f Dockerfile.testing .`


**Assignment 1.7 solution**

```Dockerfile
FROM ubuntu:24.04

WORKDIR /usr/src/app


RUN apt-get update
RUN apt-get -y install curl

COPY script.sh .

RUN chmod +x script.sh

CMD ["./script.sh"]
```

**Assignment 1.8 solution**

Note: The key to solving this problem is to understand that the last argument in `docker run` can be used to give a command or an argument.

By default `devopsdockeruh/simple-web-service:alpine` doesn't have a CMD. Instead, it uses _ENTRYPOINT_ to declare which application is run. 

The Docker documentation [CMD](https://docs.docker.com/engine/reference/builder/#cmd) says a bit indirectly that if a image has ENTRYPOINT defined, CMD is used to define it the **default arguments**.

```Dockerfile
FROM devopsdockeruh/simple-web-service:alpine
CMD ["server"] <--- pass arguments 
```

```bash
$ docker build -t web-server .
$ docker run web-server
```




