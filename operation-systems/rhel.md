# RHEL

## Install Docker in RHEL

```text
sudo yum remove docker docker-common docker-selinux docker-engine
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce

sudo usermod -aG docker $USER
newgrp docker

sudo systemctl enable docker.service
sudo systemctl start docker.service
```



