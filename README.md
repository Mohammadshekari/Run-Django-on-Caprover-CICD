# Run-Django-on-Caprover-CICD
How to Run Django + Postgres + CI/CD on Caprover (ubuntu OS)

# Docker 
## Installation
First we need to install docker on out OS:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```
## Post Installation
Then we need to complete Docker Post Installation to Activate Docker service on OS:
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```
## Verify
Verify that you can run docker commands without sudo:
```bash
docker run hello-world
```
# 2- 
