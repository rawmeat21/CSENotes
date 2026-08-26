First read this to get context about out `yt-dlp` image. 

https://courses.mooc.fi/org/uh-cs/courses/devops-with-docker-spring-2026/chapter-2/defining-start-conditions-for-the-container

Now, one of the problems as you see is the videos get downloaded inside the container. What's the fix? 

### Volumes

We can use Docker [volumes](https://docs.docker.com/storage/volumes/) to make it easier to store the downloads outside the container's ephemeral storage. With [bind mount](https://docs.docker.com/storage/bind-mounts/) we can mount a file or directory from our own machine (the host machine) into the container.

Let's start a container with the `-v` option.

```bash
docker run -v "$(pwd):/mydir" yt-dlp https://www.youtube.com/watch?v=saEpkcVi1d4
```

We mount **our current folder** as `/mydir` in the container, overwriting everything that we have put in that folder in our Dockerfile.

A Docker volume is essentially a **shared directory or file** between the host machine and the container. When a program running inside the container modifies a file within this volume, the changes are preserved even after the container is shut down and removed, as the file resides on the host machine.

Additionally, volumes facilitate file sharing between containers, enabling programs to access and load updated files seamlessly.

**What about files?**

If we wish to create a volume with only a single file we could also do that by pointing to it. For example `-v "$(pwd)/material.md:/mydir/material.md"` this way we could edit the file material.md locally and have it change in the container (and vice versa).


**Note: `-v` option will create a directory at the given path if the specified file does not exist on your filesystem.**


**Solution to Assignment 1.9**

```bash
docker run -v "$(pwd)/logs:/usr/src/app/text.log" devopsdockeruh/simple-web-service:ubuntu
```

Don't do: 

```bash
docker run -v "$(pwd):/usr/src/app" devopsdockeruh/simple-web-service:ubuntu
```

This will cause `/use/src/app/server` to be removed and your image will not be able to run.

