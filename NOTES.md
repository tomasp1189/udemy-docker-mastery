# Docker Mastery: with Kubernetes +Swarm from a Docker Captain

No need to adapt apps for containers. Easy to adpot.
landscape.cncf.io

An image is the application we want to run, a container is an instance of that image running as a process. You can have many container running off the same image.

## USEFUL COMMANDS

- `sudo lsof -i -n -P | grep TCP`
- `docker container inspect --format '{{ .NetworkSettings.IPAddress }}' <containerName>`
- `ifconfig en0`
- `docker container run <stuff> nginx:alpine`
- `docker exec -it <container> sh (connect to shell inside it)`
- `docker container —rm`
- `docker image ls`
- `docker pull <image>`
- `docker history <image>`
- `docker image inspect <image>`
- `docker image tag <tag>`
- `docker image push <image>`
- `docker login <server>`
- `docker image build -t <name> <dockerfile/path>`
- ubunto: `apt-get update && apt-get install <package>`
- centos: `yum update <package>`
- alpine: `apk add --update <package>`
- `docker image prune`
- `docker system prune`

## Creating and Using Containers Like a Boss

### Whats going on in containers

- `docker container top` - process list in one container
- `docker container inspect` - details of one container config
- `docker container stats` - performance stats for all containers

### Getting a Shell inside containers

- `docker container run -it` - start new container interactively
- `docker container exec -it` - run additional command in existing container
- (No SSH needed, docker cli is a great substitute for adding SSH to containers)
- `-t` (pseudo-TTY option), simulates a real terminal, like what SSH does
- `-i` (interactive option), keeps STDIN open even if not attached, keeps session open to receive - terminal input
- bash shell, if run with `-it`, it will give you a terminal inside the running container

### Docker Network concepts

- Review of `docker container run -p`
- For local dev/testing, networks usually just work
- Quick por check with `docker container port <container>`
- Learn concepts of Docker Networking
- Understand how network packets move around docker
- “Batteries included, but removable”
- Defaults work well in many cases, but easy to swap out parts to customise it
- `-p` (`—publish`), remember publishing ports is always in HOST:CONTAINER format
- `—format`, a common option for formatting the output of commands using “Go templates”

### CLI Management

- Show networks `docker network ls`
- Inspect a network `docker network inspect`
- Create a Network `docker network create —driver`
- Attach a network to container `docker network connect`
- Detach a network from container `docker network disconnect`
- `—network host` It gains performance by skipping virtual networks but sacrifices security container model
- `—network none` removes eth0 and only leaves you with localhost interface in container
- `docker network connect` Dynamically creates a Nic in a container on an existing virtual network

### Default Security

- Create your apps so frontend /backed sit on the same Docker network.
- Their inter-communication never leaves host.
- All externally exposed ports closed by default.
- You must manually expose via `-p`, which is better default security.
- This get even better later with Swarm and Overlay Networks.

### Docker DNS

- Docker daemon has a built-in container use by default.
- DNS Default Names, docker defaults the host names to the containers name, but you can also set aliases.
- Containers shouldn’t rely on IP’s for inter-communication.
- DNS for friendly names is built-in if you use custom networks.
- USE CUSTOM NETWORKS!
- This gets way easier with Docker Compose.

## Container Images, Where To Find Them and How To Build Them

- All about images, the building block of containers.
- What's in an image (and what isn't).
- Use Docker Hub registry.
- Managing our local image cache.

### What's in an image (and what isn't)

- App binaries and dependencies.
- Metadata about the image data and how to run the image.
- Official definition: "An Image is an ordered collection of root filesystem changes and the corresponding, execution parameters for use within a container runtime.".
- Not a complete OS. No kernel, kernel modules (e.g. drivers).
- Small as one file (your app binary) like a golang static binary.
- Big as a Ubuntu distro with apt, and Apache, PHP, and more isntalled.

### The Might Hub: Using Docker Hub Registry Images

- Official repos are maintained by a Docker team.
- Docker hub, "the apt package system for Containers".
- Official images are the better option.
- Keep in mind Alpine images are smaller than Debian.

### Images and their Layers: Discover the Image Cache

- Docker uses cache to store image layers (SHA key) this way if you reuse the same layers in different images, they actually point to the same layers and only change when layers are actually different.
- `docker history <image>` shows layers of changes made in image.
- `docker image inspect <image>` returns JSON metadata about the image.
- Images are made up of file system changes and metadata.
- Each layer is uniquely identified and only stored once on a host.
- This saves storage space on host and transfer time on push/pull.
- A container is just a single read/write layer on top of image.

### Image Tagging and Pushing to Docker Hub

- `docker image tag <tag>` assign one or more tags to an image.
- `<user>/<repo>:<tag>`, default tag is `latest` if not specified.
- `<tag>` not quite a version, not quite a branch but a little bit of both. Like in git tags.
- **"latest" tag** is just the default tag, but image owners _should_ assing it to the _newest stable version_.
- `docker image push <image>` uploads changes layers to an image registry (default is Hub).
- `docker login <server>` Defaults to logging in Hub, but you can override by adding server url.
- Tagging doesnt necessarily change image id.

### Building Images: The Dockerfile Basics

- **Package Manager** PM's like apt and yum are one of the reasons to build containers `FROM` Debian, Ubuntu, Fedora or CentOS.
- **Environment Variables** one reason they were chosen ar preferred way to inject key/value is they work everywhere, on every OS and config.
- Command order matters. After a change in a line of the docker file it will rebuild everything after the change. Anything before the change will just be pulled from the cache. Because of this, keep what changes the least at the top of the file and what changes the least at the bottom.

### Building Images: Extending Official Images

- `docker image build -t <name> <dockerfile/path>`
- If you need to change working directory to run commands its always better to use `WORKDIR` instead of `RUN cd /some/path`.
- `COPY` command used to copy src code from your local machines or build servers into your container images.
- Which flag would you pass to specify a docker build referencing a file other than the default 'Dockerfile'? `-f`.

### Using Prune to keep you Docker System Clean

- `docker image prune` to clean up just "dangling" images
- `docker system prune` will clean up everything
- The big one is usually `docker image prune -a` which will remove all images you're not using. Use `docker system df` to see space usage.

## Container Lifetime & Persistent Data: Volumes

### Container Lifetime & Persistent Data

- Containers are **usually** immutable and ephemeral.
- "immutable infrastructure": only re-deploy containers, they never change.
- This is the ideal scenario, but what about databasesm or unique data?
- Docker gives us features to ensure these "seperation of concern".
- This is known as "persistent data"
- Two ways: Volumes and Bind Mounts
- Volumes: make special location outsidce of container UFS
- Bind Mounts: link container path to host path

### Persistent Data: Data Volumes

- **named volumes** friendly way to assing volumes to containers
- `docker volume create` required to do this before `docker run` to use custom drivers and labels

### Persistent Data: Bind Mounts

- Maps a host file or directory to a container file or directory.
- Basically just two locations pointing to the same files.
- Again, skips UFS, and host files overwrite any in container.
- Can't use in Dockerfile, mus be at `container run`.
- `... run -v /Users/bret/stuff:/path/container`

## ASIDES

### NGINX

#### _What is NGINX?_

Nginx (pronounced "engine-x") is an open source reverse proxy server for HTTP, HTTPS, SMTP, POP3, and IMAP protocols, as well as a load balancer, HTTP cache, and a web server (origin server). The nginx project started with a strong focus on high concurrency, high performance and low memory usage. It is licensed under the 2-clause BSD-like license and it runs on Linux, BSD variants, Mac OS X, Solaris, AIX, HP-UX, as well as on other \*nix flavors. It also has a proof of concept port for Microsoft Windows.
