# Docker Usage
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
docker build --file server.Dockerfile (Specific file) --tag server-1 .
```

``` shell title="Specific file"
docker build -t app-1 -f server.Dockerfile .
```

``` shell title="run"
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

``` shell title="remove images"
docker rmi 
docker rmi -f #Force to temove images fast
```
  
#### Bind Port
