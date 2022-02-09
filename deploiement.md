# Deploiement 
## Démarche

- Déployer l'application sur scaleway.

### Connection ssh to scaleway CLI

`ssh -i<public_key> root@<ip>`
![ssh connection](./screens/connection_CLI_scaleway.jpg)

### Pull p8 repo

`mkdir App`
`cd App`
`git init`
`git remote add origin <git_url>`
![pull repo](./screens/pull_p8_repo.jpg)
`git pull origin <branch>`
![pull repo](./screens/pull_p8_repo1.jpg)

### Creating environnement files

`cat > .env.prod`

### Building app

`docker-compose.prod.yml up -d --build`
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
