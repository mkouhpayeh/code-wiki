# Docker Usage
- Everything you create within a container stays within the container. Once the container stops, the data gets deleted with it.
  
``` shell title="exit the shell"
ctrl+d
```

``` shell title="list directory"
ls
```
  
## Docker CLI

### Create Docker Container
``` shell
docker --help
```

``` shell
docker network --help
```

``` shell
docker container create --help
```

``` shell title="Create new container"
docker container create hello-world:linux 
```

``` shell title="Show the actively running containers"
docker ps
```

``` shell title="Show the actively running containers"
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

``` shell title="docker run = create, start container"
docker run hello-world:linux
```

``` shell title="t: the tag insead of ID and .:  the path of all images to include"
docker build -t app-1 . 
```

``` shell title="buildx: use BuildKit"
docker buildx build -t app-1 --load .
```

``` shell title="use specific file"
docker build --file server.Dockerfile --tag server-1 .
```

``` shell title="Specific file"
docker build -t app-1 -f server.Dockerfile .
```

``` shell title="-d: move to background"
docker run -d 
docker kill four_char_con_id
docker exec four_char_con_id date
```

``` shell title="tty: allocates a pseudo-tty, interactive: keep stdin open even if not attached"
docker exec --interactive --tty  four_char_con_id date bash
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
docker ps -aq | xargs docker rm
```

``` shell title="remove container"
docker rm four_char_con_id

``` shell title="remove images, -f: Force to remove images fast"
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

``` shell title="rename docker images"
docker tag currentTagName dockerHubUsername/nameInDockerHub:versionNo
```

``` shell title="push the image to Hub"
docker push dockerHubUsername/nameInDockerHub:versionNo
```

``` shell title="remove the image from Hub"
remove from the Docker hub website
```

### Commmon Steps 
``` shell 
docker build -t app-1 -f server.Dockerfile .
docker run -d --name app-1 app-1
docker ps
docker rm -f app-1
```
