---
title: "Initial Architecture"
date: 2019-12-06T06:11:18+01:00
draft: true
---

## Intro

The [core idea](#) of the project is simple. Allow the user to:

- link one or more bank accounts;
- periodically sync their transactions and remaining balances (in the background);
- occasionally send an SMS.

One requirement I think is important, is having the experience be super fast. Not "good enough" and "probably fine since everybody has fiber". I mean stuff like "I tested this on 3G with 25% packet loss, and it wasn't completely awful".

There are other aspects, and other functionality, but these are the main ones deserving some thought to the architecture.

Nuance:

- I want the transaction sync to happen in the background. However, I also want to be able to trigger it on-demand, for a quick feedback loop (when a user is e.g. visiting the dashboard, and clicks refresh)

## First version

- Tech stack
	- ⚠️ frontend SPA in react
	- ASP.NET Core 3.0
	- Auth0
	- React
	- Mongo
	- RabbitMQ
- Marketing stack
	- MailChimp
	- The Unicorn Platform 
- Infrastructure
	- DigitalOcean
	- Docker Compose
	- Traefik
	- Azure DevOps
	- Cron
	- Netlify

## Justification

### The tech stack

Having a SPA is not driven by project requirements at all. A big red flag for me. I'm indulging myself and scratching an itch in trying something new.
 
Backend 3.0 is a bit too cutting edge but otherwise, should be a solid choice. I know how to use it, I've used it before, it is a very productive environment for me. Safe choice. Battled the urge to try something more esoteric (like serverside Swift), and happy with the results;

### Netlify

The frontend app is a React app that is deployed by Netlify on every push.

### DigitalOcean

Midway through setting up my ECS cluster, I realized this is going to be too much overhead.

Going to start with a single DigitalOcean droplet and a Docker Compose file. It's probably months until I'm even close to having somebody else use this. So I'll cross that bridge when I get to it. Same with monitoring - currently I don't really care if something's up or down.

The docker compose runs:

- the backend API
- a RabbitMQ queue
- a cron task
- a background service container

### API + queue + cron

All background service logic is implemented as [IHostedService](#) in the main project. This lets me, for example, trigger a transaction sync on-demand, without waiting on the cron job.

The cron container runs cron in the foreground, and periodically invokes a special-purpose CLI app. This app shares code with the main project, and is a generic host for the same IHostedService classes. For example, here is a line in the cronjob file:

    0 * * * * /app/transaction_sync.sh
     
Where /app/transaction_sync.sh invokes:

    dotnet ServiceRunner.dll --service UserTxSyncEnqueuer

#### Azure DevOps

Finally, I set up an Azure DevOps pipeline to build the project and execute tests, and e-mail me if a test fails.

##### Not a bit premature?

Absolutely. Again, I'm scratching an itch, and indulging myself. 

Hopefully writing this down will make me more disciplined going forward, and let me avoid unnecessary tech tasks.

## docker-compose.yml

Here's what the docker compose file looks like at the moment.

	version: "3.3"
	
	services:
	
	  traefik:
	    image: "traefik:v2.0.0-rc3"
	    restart: always
	    container_name: "traefik"
	    command:
	      - "--providers.docker=true"
	      - "--providers.docker.exposedbydefault=false"
	      - "--entrypoints.web.address=:80"
	      - "--entrypoints.websecure.address=:443"
	      - "--entrypoints.mongoadmin-secure.address=:1234"
	      - "--certificatesresolvers.mytlschallenge.acme.tlschallenge=true"
	      - "--certificatesresolvers.mytlschallenge.acme.email=janis@verygoodfinances.com"
	      - "--certificatesresolvers.mytlschallenge.acme.storage=/letsencrypt/acme.json"
	    ports:
	      - "443:443"
	      - "80:80"
	    volumes:
	      - "./letsencrypt:/letsencrypt"
	      - "/var/run/docker.sock:/var/run/docker.sock:ro"
	
	  backend:
	    image: "jkirsteins/budgetback:latest"
	    restart: always
	    environment:
	      - "FP__AUTH0__DOMAIN=..."
	      - "FP__AUTH0__AUDIENCE=..."
	      - "FP__AUTH0__CLIENTSECRET=..."
	      - "FP__SALTEDGE__APPID=..."
	      - "FP__SALTEDGE__SECRET=..."
	      - "FP__SALTEDGE__REDIRECTBASEURI=..."
	      - "FP__FRONTEND__URL=..."
	      - "FP__FRONTEND__CORSORIGINS=..."
	      - "FP__FRONTEND__COOKIEDOMAIN=..."
	      - 'FP__MONGO__CONNECTIONSTRING=...'
	      - "ASPNETCORE_URLS=http://0.0.0.0:5000"
	      - "ASPNETCORE_ENVIRONMENT=Production" 
	      - "ASPNETCORE_FORWARDEDHEADERS_ENABLED=true"
	    container_name: "backend"
	    expose:
	      - 5000
	    labels:
	      - "traefik.enable=true"
	      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
	      - "traefik.http.routers.backend-http.rule=Host(`api.verygoodfinances.com`)"
	      - "traefik.http.routers.backend-http.entrypoints=web"
	      - "traefik.http.routers.backend-http.middlewares=redirect-to-https@docker"
	      - "traefik.http.routers.backend.entrypoints=websecure"
	      - "traefik.http.routers.backend.rule=Host(`api.verygoodfinances.com`)"
	      - "traefik.http.routers.backend.tls.certresolver=mytlschallenge"
	
	

