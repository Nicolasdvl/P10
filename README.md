# P10
This repo is a case study as part of Openclassrooms' formation. The objective is to learn to deploy an application. Here we deploy the app develop in P8.

## Host
The app is deploy in Scaleway : https://www.scaleway.com/fr/  
An instance (DEV1-S) with Docker image was create to deploy the app.

## Connection to Scaleway CLI
`ssh -i <public_key> root@<ip>`  

## Create user 

- Create user  
`adduser purbeurre`  

- Add user to sudo group
`usermod -aG sudo purbeurre`  

- Connect to user 
`su purbeurre`  

## Create app directory

`cd home/purbeurre`  
`mkdir app`  
`cd app`

## Define docker-compose file

Create file with `touch docker-compose.prod.yml` then edit with `nano docker-compose.prod.yml`  
Paste the following lines : 
```
version: '3.8'

services:
  web:
    image: nicolasdvl/purbeurre:latest
    command: newrelic-admin run-program gunicorn config.wsgi:application --bind 0.0.0.0:8000 --reload
    volumes:
      - static_volume:/home/app/web/staticfiles
    expose:
      - 8000
    environment:
      - NEW_RELIC_CONFIG_FILE=newrelic.ini
    env_file:
      - ./.env.prod
    depends_on:
      - db
      - redis
  db:
    image: postgres:13.0-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    env_file:
      - ./.env.prod.db
  nginx:
    image: nginx:latest
    volumes:
      - ./nginx/:/etc/nginx/conf.d/:ro
      - static_volume:/home/app/web/staticfiles
    ports:
      - 80:80
    depends_on:
      - web
  redis:
    image: redis:alpine
  celery:
    image: nicolasdvl/purbeurre:latest
    command: celery -A config worker -l info
    volumes:
      - ./app/:/usr/src/app/
    env_file:
      - ./.env.prod
    depends_on:
      - redis
  celery-beat:
    image: nicolasdvl/purbeurre:latest
    command: celery -A config beat -l info
    volumes:
      - ./app/:/usr/src/app/
    env_file:
      - ./.env.prod
    depends_on:
      - redis
  agent:
    container_name: newrelic-infra
    build:
      context: ./newrelic-infra-setup
      dockerfile: newrelic-infra.dockerfile
    cap_add:
      - SYS_PTRACE
    network_mode: host
    pid: host
    privileged: true
    volumes:
      - "/:/host:ro"
      - "/var/run/docker.sock:/var/run/docker.sock"
    env_file:
      - ./.env.prod
    restart: unless-stopped
volumes:
  postgres_data:
  static_volume:

```
Here we define 3 services : 
- web include the app image we push in DockerHub.  
- db include official postgresql image
- nginx include nginx official image

## Define '.env' files

- Define '.env.prod' file : 
```
DEBUG=0
DJANGO_KEY=<DJANGO KEY>
DJANGO_ALLOWED_HOSTS=*
SQL_ENGINE=django.db.backends.postgresql
SQL_DATABASE=<DB NAME>
SQL_USER=<USER>
SQL_PASSWORD=<PASSWORD>
SQL_HOST=db
SQL_PORT=5432
DATABASE=postgres
NRIA_LICENSE_KEY=<NEW RELIC LICENSE KEY>
SENTRY_DSN=<SENTRY DSN>
```

- Define'.env.prod.db' file : 
```
POSTGRES_USER=<USER>
POSTGRES_PASSWORD=<PASSWORD>
POSTGRES_DB=<DB NAME>
```
## Configure Nginx

- Create 'nginx' folder  
`mkdir nginx`

- Define 'nginx/nginx.conf'
```
upstream config {
    server web:8000;
}

server {

    listen 80;

    location / {
        proxy_pass http://config;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

    location /static/ {
        alias /home/app/web/staticfiles/;
    }
}
```
## Configure NewRelic

- Create 'newrelic-infra-setup' folder  
`mkdir newrelic-infra-setup`  

- Define 'newrelic-infra-setup/newrelic-infra.dockerfile'
```
FROM newrelic/infrastructure:latest
ADD newrelic-infra.yml /etc/newrelic-infra.yml
```

- Define 'newrelic-infra-setup/newrelic-infra.yml'
```
license_key: NRIA_LICENSE_KEY
```  
newrelic.ini is already includ in the app image.

## Celery

A service named celery is used in 'docker-compose.prod.yml' to create a cron task. This task update the database weekly. No configuration is required, it's define in the app image.

## Sentry

Sentry configuration just need your dsn in .env.prod

## Building app

- Build : 
`docker-compose -f docker-compose.prod.yml up -d --build`  

- Check if the app run correctly by open 'http://SERVER_IP' in your browser.

- Populate the database :
`docker-compose -f docker-compose.prod.yml exec web python manage.py insert_data`
