# P10
This repo is a case study as part of Openclassrooms' formation. The objective is to learn to deploy an application. Here we deploy the app develop in P8.

# Push app image to DockerHub

## Build the app loccally

- Pull the branch deploiement of P8 repo :
`git pull https://github.com/Nicolasdvl/P8.git deploiement`  

- Create and complete the .env files (cf. P8 repo).

- Then build the app: 
`docker-compose -f docker-compose.prod.yml up -d --build`  
`docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput`  
`docker-compose -f docker-compose.prod.yml exec web python manage.py collectstatic --noinput`  
- Check if the app run correctly by open 'http://localhost:8000' in your browser.

## Push the app image

- Create a repo on DockerHub

- Connect to DockerHub
`docker login -u <user_name>`  

- Push
`docker push <user_name>/<repo_name>`  

# Deployment

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
    command: gunicorn config.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - static_volume:/home/app/web/staticfiles
    expose:
      - 8000
    env_file:
      - ./.env.prod
    depends_on:
      - db
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
DEBUG=False
DJANGO_KEY=<DJANGO KEY>
DJANGO_ALLOWED_HOSTS=*
SQL_ENGINE=django.db.backends.postgresql
SQL_DATABASE=<DB NAME>
SQL_USER=<USER>
SQL_PASSWORD=<PASSWORD>
SQL_HOST=db
SQL_PORT=5432
DATABASE=postgres
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

## Building app

`docker-compose -f docker-compose.prod.yml up -d --build`  
`docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput`  
`docker-compose -f docker-compose.prod.yml exec web python manage.py collectstatic --noinput`  
`docker-compose -f docker-compose.prod.yml exec web python manage.py insert_data`

- Check if the app run correctly by open 'http://SERVER_IP' in your browser.

