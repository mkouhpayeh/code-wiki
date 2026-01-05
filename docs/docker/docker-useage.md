# 📚 Docker Usage
- Everything you create within a container stays within the container. Once the container stops, the data gets deleted with it.
- Use only verified images for security. Verified image have a **Docker Official Image** designation on them to let you know that they are scanned by Docker themselves. There are some image scanners:
    - Clair
    - Trivy
    - Dagda
- When some changes applied to the Docker file, you should remove the container and build it again.
- I always recommend putting a version tag when creating images, even if they are local. It's easy, quick, and solves a lot of problems in the future.
- I recommend running containers as a user other than root. You can do this for your own images as well as for containers created from existing images. For containers created from other images, specify the --user option when running Docker Run or Docker Container Create.
- For your own images, you can specify the user command within your Docker file to tell Docker which user to run your application as by default. You can make certain sections of your Docker file run as root and others run as non-root. It is quite flexible.
- You can run multiple containers and connect them all through virtual networks and separate data volumes managed entirely by the Docker engine. This is a great way to implement a three-tier architecture, like the one our web app is using.
- Docker Compose is a tool provided by Docker that makes it really easy to run and connect multiple containers on a single machine. With Compose, you use a single file called the Compose Manifest to define all of your containers and how they relate to each other. Starting those containers together is as easy as running Docker-compose up.
- While you can link containers together with Docker networks, these networks do not span multiple hosts by default. You can also use the Docker CLI to talk to Docker engines running on remote hosts but it is quite cumbersome, especially when authentication comes into play. Docker also does not have built-in solutions for moving containers between hosts or auto-scaling containers to respond to load. Finally, higher-level concerns like securing traffic between containers or configuring load balancing and routing amongst them are outside of Docker's realm of responsibility. At best, this can make using Docker alone for production-type workloads really, really complex. At worst, it can increase security risk, decrease performance, and make your infrastructure more susceptible to downtime. **Container orchestrators** solve these problems. Orchestrators use scheduling techniques, networking magic, and service discovery tools to make scaling, moving, and routing traffic to containers really, really easy. If you've ever used VMware's vCenter or tools like Rundeck, you're already familiar with the basic idea. Many container orchestrators have popped up over the years. Docker's own Swarm product, Mesosphere, HashiCorp's Nomad, and cloud offerings like AWS Elastic Container Service or Azure Container Service are examples of some popular container orchestrators in the world. 
    - Docker Swarm
    - Marathob
    - HashiCorp Nomad
    - Cloud offerings (Amazon Elastic Container Service, Azure Container Service)
    - Kubernetes describes itself as a planet-scale container orchestrator for automating the deployment, scaling, and management of containerized applications.
        - Kubernetes is a distributed system. It is designed from the ground up to run its components and store their data across many machines. This makes it capable of running and connecting hundreds of thousands of containers seamlessly as if they were all running on one machine. This also makes Kubernetes possible to run on almost anything from Raspberry Pis to some of the largest cloud platforms in the world.
        - Kubernetes makes it really easy to group containers together, kind of like Docker Compose. You can also use Kubernetes to scale those container groups up or down to respond to your application's demands without creating more VMs or other hardware. This is typically expensive and sometimes cumbersome but much cheaper to do with Kubernetes.
        - Kubernetes also makes it really easy to secure traffic within your container networks and to or from the outside world. You can use Kubernetes to provide specific rules about how your traffic to or from your containers is routed.
        - Kubernetes can be used as a platform of platforms. 

---

## 1️⃣ Docker CLI
``` shell title="Exit the shell"
ctrl+d
```

``` shell title="Find previous command"
ctrl+r
```

``` shell title="Stop the process"
ctrl+c
```

``` shell title="Clear the screen"
ctrl+l
```

``` shell title="List directory"
ls
```

### Create Docker Container
``` shell title="Help"
docker --help
docker network --help
docker container create --help
```

``` shell title="Create new container"
docker container create hello-world:linux 
```

``` shell title="Show the actively running containers"
docker ps
```

``` shell title="Show the all containers"
docker ps --all
docker ps -aq
```

``` shell title="Start the container"
docker container start dockerID
docker container start --attach
```

``` shell 
docker logs three_char_con_id
```

``` shell title="docker run => create, start, attach"
docker run hello-world:linux
```

``` shell title="t: the tag insead of ID and .:  the path of all images to include"
docker build -t app-1 . 
```

``` shell title="buildx: use BuildKit"
docker buildx build -t app-1 --load .
```

``` shell title="Use specific docker file"
docker build --file server.Dockerfile --tag server-1 .
docker build -t app-1 -f server.Dockerfile .
```

``` shell title="-d: Move to background"
docker run -d 
docker kill four_char_con_id
docker exec four_char_con_id date
```

``` shell title="tty: allocates a pseudo-tty, interactive: keep stdin open even if not attached, tell Docker to enable keystrokes"
-starts an interactive Bash shell within a container starting with ID 2bf with a pseudo-TTY allocated 
docker exec --interactive --tty  four_char_con_id date bash
```

### Performance Container
``` shell title="Show all images"
docker images
```

``` shell title="Smartly removes useless data"
docker system prune ID/tag
```

``` shell title="Snapshot of the container's performance"
docker stats [containerName]
```

``` shell title="Show what's running inside of the container without having to exec"
docker top [containerName]
```

``` shell title="Show advanced information about a container in JSON format. Its searchable. to quit press 'q'"
docker inspect ID/tag | less
```

``` shell title="Show contexts"
docker context ls
```

``` shell title="Switch to the context created by Docker Desktop"
docker context use default
```

- The context can be overridden with the DOCKER_CONTEXT environment variable and the endpoint can be overridden with DOCKER_HOST.
``` shell
$DOCKER_CONTEXT ; echo $DOCKER_HOST
```

``` shell title="in Windows PowerShell"
$env:DOCKER_CONTEXT ; echo $env:DOCKER_HOST
```

``` shell
export DOCKER_CONTEXT=xxx
unset DOCKER_CONTEXT
```

### Interact Docker Container
#### Stop/Remove

``` shell
docker stop four_char_con_id
```

``` shell title="Force to terminate"
docker stop -t 0 four_char_con_id
```

``` shell title="Work as a loop, provide IDs from left and applies to the right command"
docker ps -aq | xargs docker rm -f
```

``` shell title="Remove container"
docker rm four_char_con_id
```

``` shell title="Remove images, -f: Force to remove images fast"
docker rmi 
docker rmi -f 
```
  
#### Bind Port
``` shell title="outside port:inside port => browse 5001"
docker run -p 5001:5000
```

#### Save Data
- Everything you create within a container stays within the container. Once the container stops, the data gets deleted with it.
- We can use the **volume mounting** feature to work around this. Like the port binding feature, this allows Docker to map a folder on our computer to a folder in the container.

``` shell title="--volum: map temp folder in the container to the /tmp/container path => the file should exists otherwise assumes as a directory"
docker run -v /tmp/container:/temp
cat /tmp/container/file
```

### Docker Hub
- A container image registry is a place for storing and tracking container images. Container images are tracked by their tags, a string combining the name of the image, and optionally it's version with a semicolon.
- Container images that do not have a version automatically get tagged with the version called latest. Like downloading software on Homebrew or GitHub, this naming scheme makes it really easy to download specific versions of images.
- We'll see what I mean here in a moment. Docker Hub is a default registry used by the Docker client. This is a publicly accessible registry that anyone can push images to.
- Whenever you pull images or whenever Docker needs to pull an image from the from line of a Docker file, it will fetch an image from Docker Hub.

``` shell
docker login
```

``` shell title="Rename docker images"
docker tag currentTagName dockerHubUsername/nameInDockerHub:versionNo
```

``` shell title="Push the image to Hub"
docker push dockerHubUsername/nameInDockerHub:versionNo
```

``` shell title="Remove the image from Hub"
remove from the Docker hub website
```

#### Container Registery
- Self-hosting image registries:
    - JFrog's Artifactory
    - Sonatype's Nexus
    - Red Hat's Quay.io
    - Project Harbor


### Commmon Steps 
``` shell 
docker build -t app-1 -f server.Dockerfile .
docker run -d --name app-1 app-1
docker ps
docker rm -f app-1
docker images
docker rmi tag-1 tag-2 tag-3
```

#### nginx Sample
``` shell 
docker run --name website -v "$PWD/website:usr/share/nginx/html" -p 8080:80 --rm nginx
```
