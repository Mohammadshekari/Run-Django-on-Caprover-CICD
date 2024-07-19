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


# Configure Firewall
Some server providers have strict firewall settings. To disable firewall on Ubuntu:
```bash
ufw allow 80,443,3000,996,7946,4789,2377/tcp; ufw allow 7946,4789,2377/udp;
```


# CapRover Setup
## Step 1: CapRover Installation

- You can Follow this [LINK](https://caprover.com/docs/get-started.html)

Just run the following line, sit back and enjoy!
```bash
docker run -p 80:80 -p 443:443 -p 3000:3000 -e ACCEPTED_TERMS=true -v /var/run/docker.sock:/var/run/docker.sock -v /captain:/captain caprover/caprover
```

## Step 2: Connect Root Domain
Let's say you own `mydomain.com`. You can set `*.something.mydomain.com` as an `A-record` in your DNS settings to point to the IP address of the server where you installed CapRover. Note that it can take several hours for this change to take into effect. It will show up like this in your DNS configs:

- **TYPE:** A record
- **HOST:** `*.something`
- **POINTS TO:** (IP Address of your server)
- **TTL:** (doesn't really matter)

To confirm, go to https://mxtoolbox.com/DNSLookup.aspx and enter `randomthing123.something.mydomain.com` and check if IP address resolves to the IP you set in your DNS. Note that `randomthing123` is needed because you set a wildcard entry in your DNS by setting `*.something` as your host, not `something`.

## Step 3: Install CapRover CLI
Assuming you have npm installed on your local machine (e.g., your laptop), simply run (add `sudo` if needed):
```bash
npm install -g caprover
```
Then, run
```bash
caprover serversetup
```
Follow the steps and login to your CapRover instance. When prompted to enter the root domain, enter `something.mydomain.com` assuming that you set `*.something.mydomain.com` to point to your IP address in step #2. Now you can access your CapRover from `captain.something.mydomain.com`. You can read more about hiding the root domain [here](https://caprover.com/docs/best-practices.html#hidden-root-domain).


# Start in Caprover Server
We go to `captain.mydomain.com` or any domain we have set. and After Login we can see the dashboard.

## Install PostgreSQL
Go to `One Click Apps` and Search for `postgreSQL`
for `Version` i tested `15.4` and other fields could be default.
after install its easy to access to the Database HOST,USERNAME,PASSWORD,DATABASE and other...

## Install Django App
for install a Django app, First we have to Check `Has Persistent Data` in the first Step! beacuase we want to use `Media` in Django App.
Description of CapRover about this Checkbox:

**Any database that stores data on disk has to have persistent data enabled.**

**Otherwise all data will be lost when the container restarts (due to crash, host restart and etc...)**

- A photo upload app which does not use third party storages like Amazon S3 to store images. Instead, it **locally stores uploaded images**.
- A webapp that needs to store some user **uploaded files** and plugins locally on disk (like WordPress)

### Project HTTP Settings
First step is `HTTP Settings` that contains SSL / Domain / HTTPS / Connect to Domain and this things thats so easy to understand and work with.

we have to Create Nginx folder for `media` of our django to be available on the Web:

**Connect to Your SSH Server, then**:

```bash
cd /captain/data/nginx-shared/
mkdir YOUR_PROJECT_DIRECTORY_NAME
cd YOUR_PROJECT_DIRECTORY_NAME
mkdir media
```
This will be directory of you Project `media`.


we Should append this code to end of Customize Nginix Config in the `Http Settings` tab (after click to edit nginx Configurations):
```
location /media/ {
    alias /nginx-shared/YOUR_PROJECT_DIRECTORY_NAME/media/;
}
```
this will access to users and project to work with `media`

`Container HTTP Port` should set to Dockerfile EXPOSE port.  

`Force HTTPS by redirecting all HTTP traffic to HTTPS` this is better to be checked!


### Project App Configs
in the `Environmental Variables` we have to set our .env Config as key value Pair

Bulk text Example:
```
SECRET_KEY=xxxxxxxxxxx
POSTGRES_NAME=xxxxxxxx
POSTGRES_USER=xxxxxxxx
POSTGRES_PASS=xxxxxxxx
POSTGRES_HOST=xxxxxxxx
POSTGRES_PORT=xxxxxxxx
```

Then we Have `Persistent Directories` that points to `media` directory of django that we set in the previus part.

Click on `Add Persistent Directories` and Set this Config:

```
Path in App:
/usr/src/app/media
```

This Refers to media folder of Django on Docker.

Then click on `Set specefic host path` and :

```
Path on Host:
/captain/data/nginx-shared/examplegift/media
```

This refers to Where to Access media files that we already created in captain folder!


### Project Deployment

Scroll down to Deploy methods.

Enable your `App Token` and we will use this in the `Github CI/CD Secrets` to access the github to Deploy your Project.

## Captain Definition
create a file named as `captain-definition` and defind your `Dockerfile` location like this:

```
{
  "schemaVersion": 2,
  "dockerfilePath": "./Dockerfile"
}
```

## Dockerfile 
this is a example Dockerfile for Django app:

```
FROM python:3.10-slim-buster

LABEL maintainer="YOUR_ADDRESS@gmail.com"

ENV PYTHONUNBUFFERED=1

WORKDIR /usr/src/app

COPY ./requirements.txt .
COPY ./entrypoint.sh .
COPY ./core .
RUN chmod +x ./entrypoint.sh

RUN pip install --upgrade pip && pip install -r requirements.txt


EXPOSE 8000

# execute our entrypoint.sh file
CMD ["sh","./entrypoint.sh"]
```

## Entry Point
in here, we write the commands to Start our Django application:

```
#!/bin/bash

python manage.py collectstatic --noinput
python manage.py migrate
gunicorn core.wsgi:application --bind "0.0.0.0:8000"
```

## Github Workflows
its time to tell Github that we When wants you to update our Caporver Server.
lets config our Django app Workflow in this address:
**.github/workflows/deploy.yml**

and then we copy this code to config our GitHub Workflow Action:

```
name: Build & Deploy

on:
  push:
    branches: [ "production" ]

  pull_request:
    branches: [ "production" ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x]

    steps:
      - name: Check out repository
        uses: actions/checkout@v4
      - name: Open Web Client Directory
        working-directory: .
        run: |
          ls -la

      - uses: a7ul/tar-action@v1.1.0
        with:
          command: c
          cwd: "./"
          files: |
            core/
            Dockerfile
            entrypoint.sh
            README.md
            requirements.txt
            captain-definition
          outPath: deploy.tar

      - name: Deploy App to CapRover
        uses: caprover/deploy-from-github@v1.0.1
        with:
          server: '${{ secrets.CAPROVER_SERVER }}'
          app: '${{ secrets.APP_NAME }}'
          token: '${{ secrets.APP_TOKEN }}'
```
but somethings is missing and that's our `secrets`.

### Github Secrets

we go to this address:
```https://github.com/YOUR_USERNAME/YOUR_PROJECT_NAME/settings/secrets/actions```
this we click on  `New repository secret` and Add this secrets:

- `APP_NAME` : name you app when you create on caprover.
- `APP_TOKEN` : the token that we have on caprover `Deployment` tab.
- `CAPROVER_SERVER`: your domain like `captain.YOUR-DOMAIN.com`.

** NOTE: CapRover server must be in the format of "https://captain.apps.YOUR-DOMAIN.com". You can set CAPROVER_SERVER as a Global Secret for all your private and/or public projects. **

## Done

Commit changes to your code and push in `production` branch to deploy! 

