Run a container:

```bash
docker container run <name of image> # or use: docker run <name of image>
```

Containers are instances of images.

Think of a container as a ready-to-eat meal that you can simply heat up and consume. An image, on the other hand, is the recipe _and_ the ingredients for that meal. 

You need an image and a container runtime (Docker engine) to create a container. The image provides all the necessary instructions and dependencies for the container to run, just like a recipe provides the steps and ingredients to make a meal.

### Image[​](http://localhost:3000/part-1/section-1#image)

A Docker image is a file. An image never changes; you can not edit an existing file. Creating a new image happens by starting from a base image and adding new **layers** to it.

List all your images with: `docker image ls`

Where do images come from? This image file is built from an instructional file named **Dockerfile** that is parsed when you run `docker image build`.

Dockerfile is the instruction set for building an image.

#### Where do the images come from?

When running a command such as `==docker run hello-world==`, Docker will automatically search [Docker Hub (opens in a new tab)](https://hub.docker.com/) for the image if it is not found locally.

We can search for images in the Docker Hub with `docker search`.

```bash
$ docker search hello-world

  NAME                         DESCRIPTION    STARS   OFFICIAL
  hello-world                  Hello World!…  2387     [OK]
  rancher/hello-world          This contain…     6
  atlassian/hello-world                          0
  ...
```

`hello-world` is an official image. [Official images (opens in a new tab)](https://docs.docker.com/docker-hub/official_images/) are curated and reviewed by Docker, Inc. and are usually actively maintained by the authors. They are built from repositories in the [docker-library (opens in a new tab)](https://github.com/docker-library).

There are also other Docker registries competing with Docker Hub, such as [Quay (opens in a new tab)](https://quay.io/). By default, `docker search` will only search from Docker Hub, but to search a different registry, you can add the registry address before the search term, for example, `docker search quay.io/hello`.

```awk
docker pull quay.io/podman/hello <--- pull from a repo other than docker hub
```

#### Versions of image and how to rename images

```bash
$ docker pull ubuntu
  Using default tag: latest <--- using latest (we can change this)
  latest: Pulling from library/ubuntu
```

Images can be tagged to save different versions of the same image. You define an image's tag by adding `:<tag>` after the image's name.

```bash
$ docker pull ubuntu:25.10
  25.10: Pulling from library/ubuntu
  c2ca09a1934b: Downloading [============================================>      ]  34.25MB/38.64MB
```
Images are composed of different layers that are downloaded in parallel to speed up the download.

We can also tag images locally for convenience, for example, `docker tag ubuntu:25.10 ubuntu:questing_quokka` creates the tag `ubuntu:questing_quokka` which refers to `ubuntu:25.10`.

Tagging is also a way to "rename" images. Run `docker tag ubuntu:25.10 fav_distro:questing_quokka` and check `docker image ls` to see what effects the command had.

**An image name may consist of 3 parts plus a tag. Usually like the following: `registry/organisation/image:tag`. But may be as short as `ubuntu`, then the registry will default to Docker hub, organisation to _library_ and tag to _latest_.**

#### How to build images

Dockerfile - contains the build instructions for an image. You define what should be included in the image with different instructions.


### Container[​](http://localhost:3000/part-1/section-1#container)

Containers contain the application and what is required to execute it (dependencies); and you can start, stop and interact with them. They are **isolated** environments in the host machine with the ability to interact with each other and the host machine itself via defined methods (TCP/UDP).

List all your containers with `docker container ls`. Use `-a` option to see every container you ever ran.

### Docker CLI

We are using the command line to interact with the "Docker Engine" that is made up of 3 parts:

- command line interface (CLI) client
- a REST API
- Docker daemon

When you run a command, e.g. `docker container run`, behind the scenes the CLI client sends a request to the **Docker daemon** through the REST API. The Docker daemon takes care of images, containers and other resources.


### How to remove Docker image?

You cannot remove an image directly. You need to delete all of the containers first. 

1. `docker container rm <container ID>...`. For deleting a lot of containers, use `docker container prune`. Delete ALL containers for the image.
2. Then, run  `docker image rm <name of image>`.


### Download image without running: 

```
docker image pull <name>
```

### Run container in background:

The `-d` flag starts a container _detached_, meaning that it runs in the background.
```
docker run -d nginx
```

### What if you want to remove a running container?

```
$ docker container stop <name>
$ docker container rm <name>
```

Notes:

1. `docker run` requires an image. Don't try running existing containers with this.
2. `docker exec` works on running containers only.
3. Use `docker start` to start a container in background by default (yeah it's true).


### Managing Container Lifecycles

| Command                      | Action                                    |
| ---------------------------- | ----------------------------------------- |
| `docker run <image>`         | Create & start a brand-new container      |
| `docker stop <container>`    | Stop a running container                  |
| `docker start <container>`   | Start an existing stopped container       |
| `docker restart <container>` | Stop and immediately restart a container  |
| `docker ps`                  | List all currently running containers     |
| `docker ps -a`               | List all containers (running and stopped) |











