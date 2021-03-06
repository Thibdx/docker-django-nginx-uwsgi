Docker Django
=============

This an an adaptation from erroneousboat/docker-django for my projects. Erroneousboat is more likely to keep it up to date than me. However feel free to take some inspiration if you need the additional features of this repository.

## Sources
* https://github.com/erroneousboat/docker-django
* https://testdriven.io/blog/dockerizing-django-with-postgres-gunicorn-and-nginx/
* https://github.com/testdrivenio/django-on-docker

## Modifications from erroneousboat/docker-django
* Added rotated backup crons that save database exports in Docker volumes.
* Backing up your docker volumes can then be done via a local Cron or https://github.com/blacklabelops/volumerize
* Added media directory support
* Adapted for remote development
* More flat structure
* Only one Django project in the webapp container
* Minor updates on versions (2019/03)


## Setting up

### Docker
See installation instructions at: [docker documentation](https://docs.docker.com/install/)
### Docker Compose
Install [docker compose](https://github.com/docker/compose), see installation
instructions at [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)


### Environment variables
The file `environment/.env.sample` contains the environment
variables needed in the containers. You can edit this as you see fit, and at
the moment these are the defaults that this project uses.
It has to be renamed .env.
.env is in the .gitignore file of the project.

## Fire it up
Start the container by issuing one of the following commands:
```bash
$ docker-compose up             # run in foreground
$ docker-compose up -d          # run in background
```

## Other commands
Build images:
```bash
$ docker-compose build
$ docker-compose build --no-cache       # build without cache
```

See processes:
```bash
$ docker-compose ps                 # docker-compose processes
$ docker ps -a                      # docker processes (sometimes needed)
$ docker stats [container name]     # see live docker container metrics
```

See logs:
```bash
# See logs of all services
$ docker-compose logs

# See logs of a specific service
$ docker-compose logs -f [service_name]
```

Run commands in container:
```bash
# Name of service is the name you gave it in the docker-compose.yml
$ docker-compose run [service_name] /bin/bash
$ docker-compose run [service_name] python /srv/starter/manage.py shell
$ docker-compose run [service_name] env
```

Remove all docker containers:
```bash
docker rm $(docker ps -a -q)
```

Remove all docker images:
```bash
docker rmi $(docker images -q)
```

### Some commands for managing the webapp
To initiate a command in an existing running container use the `docker exec`
command.

```bash
# Find container_name by using docker-compose ps
$ sudo docker-compose ps

# We will consider [container_name] = docker-django-nginx-uwsgi_webapp_1

# Create an alias for shotening commands
$ sudo alias  sudo docker exec -it docker-django-nginx-uwsgi_webapp_1 python3 /srv/myproject/manage.py = djman

# Prefix for running manage.py commands 
$ sudo docker exec -it docker-django-nginx-uwsgi_webapp_1 \
    python3 /srv/myproject/manage.py

# Run dev server for remote development on port 8001
$ sudo docker exec -it docker-django-nginx-uwsgi_webapp_1 \
    python3 /srv/myproject/manage.py runserver 0.0.0.0:8001

# restart uwsgi in a running container.
$ docker exec docker-django-nginx-uwsgi_webapp_1 \
    touch /etc/uwsgi/reload-uwsgi.ini

# create migration file for an app
$ docker exec -it docker-django-nginx-uwsgi_webapp_1 \
    python /srv/myproject/manage.py makemigrations scheduler

# migrate
$ docker exec -it docker-django-nginx-uwsgi_webapp_1 \
    python3 /srv/myproject/manage.py migrate

# get sql contents of a migration
$ docker exec -it docker-django-nginx-uwsgi_webapp_1 \
    python3 /srv/myproject/manage.py sqlmigrate [appname] 0001

# get to interactive console
$ docker exec -it docker-django-nginx-uwsgi_webapp_1 \
    python3 /srv/myproject/manage.py shell

# testing
docker exec docker-django-nginx-uwsgi_webapp_1 \
    python3 /srv/myproject/manage.py test
```

## Troubleshooting
Q: I get the following error message when using the docker command:

```
FATA[0000] Get http:///var/run/docker.sock/v1.16/containers/json: dial unix /var/run/docker.sock: permission denied. Are you trying to connect to a TLS-enabled daemon without TLS? 

```

A: Add yourself (user) to the docker group, remember to re-log after!

```bash
$ usermod -a -G docker <your_username>
$ service docker restart
```

Q: Changes in my code are not being updated despite using volumes.

A: Remember to restart uWSGI for the changes to take effect.

```bash
# Find container_name by using docker-compose ps
$ docker exec [container_name] touch /etc/uwsgi/reload-uwsgi.ini
```
