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




