# Docker Usage
- ctrl+d (exit the shell)
- ls (list)
  
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
``` shell title="Shows the actively running containers"
docker ps
```
``` shell title="Shows the actively running containers"
docker ps --all
docker ps -aq
```
``` shell title="Starts the container"
docker container start dockerID
docker container start --attach
```
``` shell 
docker logs three_char_con_id
```

- docker run hello-world:linux (docker run = create, start and docker)
- docker build -t app-1 (The tag insead of ID) . (the path of all images to include)
- docker buildx (Uses BuildKit) build -t app-1 --load .
- docker build --file server.Dockerfile (Specific file) --tag server-1 .
- docker build -t app-1 -f server.Dockerfile .
- docker run -d 
- docker kill four_char_con_id
- docker exec four_char_con_id date
- docker exec --interactive --tty  four_char_con_id date bash


### Interact Docker Container
#### Stop/Remove
- docker stop four_char_con_id
- docker stop -t 0 four_char_con_id (Force to terminate)
- docker ps -aq | xargs docker rm (Works as a loop, provide IDs from left and applies to the right command)
- docker rm four_char_con_id (Removes container)
- docker rmi (Removes images)
- docker rmi -f (Forces to temove images fast)
  
#### Bind Port
