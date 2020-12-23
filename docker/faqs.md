# FAQs

## Install Docker

### On Ubuntu

Ref: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

```text
sudo apt-get update

# Uninstall old versions
sudo apt-get remove docker docker-engine docker.io containerd runc

# Install 
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
    
# Add Docker’s official GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Verify that you now have the key with the fingerprint 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88, by searching for the last 8 characters of the fingerprint.
sudo apt-key fingerprint 0EBFCD88

# Set up the stable repository
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   
# INSTALL DOCKER ENGINE
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Verify that Docker Engine is installed correctly
sudo docker run hello-world

# Post-installation steps for Linux
sudo usermod -aG docker $USER

# Now you can log out and log into the machine and try without sudo
docker run hello-world
```

## How to display all containers with IPs

By default, `docker ps` can't display containers' IPs:

```text
$ docker ps
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS                             NAMES
95875cd6c516        rancher/k3d-proxy:v3.1.3   "/bin/sh -c nginx-pr…"   15 minutes ago      Up 15 minutes       80/tcp, 0.0.0.0:52434->6443/tcp   k3d-demo-serverlb
182f545be7e2        rancher/k3s:v1.18.9-k3s1   "/bin/k3s server --t…"   15 minutes ago      Up 15 minutes                                         k3d-demo-server-2
f984bb22535f        rancher/k3s:v1.18.9-k3s1   "/bin/k3s server --t…"   16 minutes ago      Up 16 minutes                                         k3d-demo-server-1
9a63651a904e        rancher/k3s:v1.18.9-k3s1   "/bin/k3s server --c…"   16 minutes ago      Up 16 minutes                                         k3d-demo-server-0
```

But it's quite flexible to custom the output. For example:

```text
$ docker ps -q | xargs docker inspect --format '{{ .Name }} - {{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}'
/k3d-demo-serverlb - 172.25.0.5
/k3d-demo-server-2 - 172.25.0.4
/k3d-demo-server-1 - 172.25.0.3
/k3d-demo-server-0 - 172.25.0.2
```

