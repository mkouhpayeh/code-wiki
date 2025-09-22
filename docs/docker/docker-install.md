# Docker Installation

## Docker Desktop
Docker Machine used Oracle's VirtualBox to create a small Linux-based virtual machine whose only purpose was to run the Docker engine. <br />
Once created, users needed to run a small shell script to connect their Docker CLI with the virtual machine. <br />

1- https://docs.docker.io <br />
2- Install on Windows or Mac: <br />
    - https://www.docker.com 
    - in PowerShell 
      - > docker os 
      - > docker run hello-world <br />
3- Install on Linux  <br />
    - > sudo apt install curl
    - > curl -o /tmp/get-docker.sh https://get.docker.com
    - > sh /tmp/get-docker.sh
    - > sudo docker run hello-world
    - > sudo usermod -aG docker $USER => add to the group list
    - > sudo -s $USER
    - > sudo -u usename sh
    - > grups => see the group list
