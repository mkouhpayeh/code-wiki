# Docker Usage
- ctrl+d (exit the shell)
- ls (list)
  
## Docker CLI

### Create Docker Container
- docker --help
- docker network --help
- docker container create --help
- docker container create hello-world:linux (Create new container)
- docker ps (Shows the actively running containers)
- docker ps --all
- docker ps -aq (Shows only IDs)
- docker container start dockerID (Starts the container)
- docker logs three_char_con_id
- docker container start --attach
<br />
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
