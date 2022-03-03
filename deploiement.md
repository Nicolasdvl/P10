# Deploiement 
## Démarche

- Déployer l'application sur scaleway.

### Connection ssh to scaleway CLI

`ssh -i <public_key> root@<ip>`  
![ssh connection](./screens/connection_CLI_scaleway.jpg)

### Pull p8 repo

`mkdir App`  
`cd App`  
`git remote add origin <git_url>`  
`git pull origin <branch>`  
![pull repo](./screens/pull_p8_repo.jpg)  

### Creating environnement files

`cat > .env.prod`

### Building app

`docker-compose -f docker-compose.prod.yml up -d --build`  
![build up](./screens/docker-compose_up.jpg)  
`docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput`  
`docker-compose -f docker-compose.prod.yml exec web python manage.py collectstatic --noinput`  
![build up](./screens/docker-compose_up2.jpg)

### Logs
![logs](./screens/logs.jpg)
![logs](./screens/logs1_insert_data.jpg)

## Problèmes

- Le site n'est pas accessible.
- La commande `manage.py insert_data` ne fonctionne pas.

## Démarche

- Ajouter le module 'requests' au fichier 'requierements.txt.
- Modifier 'settings.py' pour autoriser toutes les connections.
- Modifier 'nging/nging.conf' pour configurer le reverse proxy

### Rebuild app
#### Down app
`docker-compose -f docker-compose.prod.yml down -v`
#### Pull modified files:
`pull git pull origin <branch>`  
![requirements.txt modified](./screens/requirements_modified.jpg)  
![nging.conf modified](./screens/nging_modified.jpg)
#### Modify ALLOWED_HOSTS in env file : 
`vi .env.prod`  
![env.prod modified](./screens/env_modified.jpg)
#### Build app :
`docker-compose -f docker-compose.prod.yml up -d --build`  
`docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput`  
`docker-compose -f docker-compose.prod.yml exec web python manage.py collectstatic --noinput`

### Logs
![logs](./screens/logs2.jpg)
