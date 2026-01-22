# How to self host n8n on a VPS

This tutorial will show you how you can self host n8n on an Ubuntu server, using 
- [Hetzner](https://hetzner.cloud/?ref=mZziDsGU2VVp) (affilate link, you get 20€ in credits to get started) as the server provider,
- [Docker](docker.com) for easy installation,
- [PostgreSQL](https://www.postgresql.org/) as the database and
- [Caddy](https://caddyserver.com/) as our reverse proxy.

> if you're just here for unlimited n8n workflows, you can save time and money by deploying on [Sliplane](https://sliplane.io/n8n-hosting?utm_source=gh-selfhosting-n8n) 

## Step 1: Create an SSH key

On your local computer open a terminal and run:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Note: replace `your_email@example.com` with your actual email address.

Add the key to your ssh agent

```bash
ssh-add ~/.ssh/[KEYNAME]
```

Note: replace `[KEYNAME]` with the actual name of the ssh key.

## Steps 2: Buy a server

Go to hetzner.com and sign up for their cloud console. If you signup through our affiliate link, you get 20€ in credits: https://hetzner.cloud/?ref=mZziDsGU2VVp

Create a new project and then create a new server inside the new project.

- use a shared server to get started for little money
- use Ubuntu as the OS
- enable IPv4 and IPv6
- add the public SSH key you created before
- enable backups

## Step 3: Setup a firewall

The Hetzner firewall is in front of your server and can block incoming traffic, before it reaches your server. 
You could additionally use [ufw](https://help.ubuntu.com/community/UFW), but keep in mind that [Docker can bypass ufw](https://docs.docker.com/engine/network/packet-filtering-firewalls/#docker-and-ufw).

Navigate to firewalls tab in the Hetzner dashboard and create a new firewall with 3 rules, limiting inbound connections:
- allow any IPv4 and any IPv6 to connect via TCP on port 22 (this is for SSH access)
- allow any IPv4 and any IPv6 to connect via TCP on port 443 (this is for https traffic)
- allow any IPv4 and any IPv6 to connect via TCP on port 80 (this is for http traffic, we will keep port 80 open to redirect http to https)

Apply the firewall to the server you created.

## Step 4: Update DNS records

Make sure you have a free domain or subdomain ready, that you can use to point to the Hetzner server. A domain is required in order to access the service via https.

You will find both IPv4 and IPv6 addresses of your newly created server in the Hetzner dashboard.

Go to your DNS provider and add these records to the domain, that you want to use:

- `A` record pointing to your IPv4 address
- `AAAA` record pointing to your IPv6 address (without the subnet /64)

## Step 5: Setup the server

Connect to the server via SSH, open a terminal and run:

```bash
ssh root@[SERVER_IP]
```

Replace `[SERVER_IP]` with the IPv4 address of your server.

On the server update packages first by running

```bash
apt update
apt upgrade -y
```

Next install Docker following the instructions from the [official Docker documentation](https://docs.docker.com/engine/install/ubuntu/)

After Docker is installed, you can clone the contents of this repository, by running

```bash
git clone https://github.com/sliplane-support/selfhosting-n8n-example.git
```

A new folder will be created on your VPS named `selfhosting-n8n-example`. Navigate inside the directory with 

```bash
cd selfhosting-n8n-example
```

Once inside the directory, update the content of the `Caddyfile` which is located in the `conf` directory. You can open the file using

```bash
nano conf/Caddyfile
```

Replace the example domain with your actual domain, that you are going to use, save the changes and exit the file editor.

Next, rename the `.env.example` file to `.env` by running

```bash
mv .env.example .env
```

You can open the `.env` file with `nano` as well, in order to overwrite the default password.

The last step is to spin up all services using 

```bash
docker compose up -d
```

This will download and run the services defined in the docker-compose.yml file.

Once the services are up and running, you can access n8n in a browser using your custom domain.




