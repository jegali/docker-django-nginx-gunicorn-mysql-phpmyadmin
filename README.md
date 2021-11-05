# docker-django-nginx-gunicorn-mysql-phpmyadmin

## Abstract
In this repository you will find a description how to dockerize django, gunicorn, nginx and mysql. phpMyAdmin is only on board to provide a convenient administration interface.
<br/><br/>

## Intended use
This Docker image is to be used for m< vulnerable web store called "mallow" zthat is built in Python. Until that happens, the repository will serve as a think tank so I don't forget over time how to set up this project.
<br/><br/>

## Structure
Prebuilt Docker images exist in Docker Hub for both phpMyAdmin, mySQL, and nginx. These images are assembled via docker-compose.

For Django there is no current image, this must be created individually. For this purpose, there is a Dockerfile in the repository, the structure of which I will explain in a moment. 
<br/><br/>

## Docker Compose File
The docker-compose file assembles the whole application stack. Before we go into detail, let's have a look at the whole file. Be warned - this app builds a vulnerable application, so the dockercompose file has some vulnerabilities itself.

```
version: '3'

services:
    web:
        image: nginx:alpine
        container_name: webserver_mallowz
        restart: always
        ports:
            - 7072:8000
        volumes:
            - "./etc/nginx/default.conf:/etc/nginx/conf.d/default.conf"
            - "./site_project/media:/home/site_project/media"
            - "./site_project/static:/home/site_project/static"
        depends_on:
            - django
    
    phpmyadmin:
        image: phpmyadmin/phpmyadmin
        container_name: phpmyadmin_mallowz
        restart: always
        ports:
            - 7070:80
        environment: 
            PMA_HOST: mysql_mallowz
            PMA_ARBITRARY: 1
            MYSQL_ROOT_PASSWORD: "root"
        depends_on:
            - database
    
    database:
        image: mysql:5.7.22
        container_name: mysql_mallowz
        restart: always
        ports:
            - 7071:3306
        volumes:
            - ./data/db/mysql:/var/lib/mysql
        environment:
            MYSQL_ROOT_PASSWORD: root
            MYSQL_DATABASE: mallowz
            MYSQL_USER: mallowz
            MYSQL_PASSWORD: mallowz

    django:
        build: 
            context: .
            dockerfile: Dockerfile
        container_name: django_mallowz
        command: gunicorn -b 0.0.0.0:8000 -w 4 site_project.wsgi
        volumes:
            - "./site_project:/home/site_project"
            - "./site_project/media:/home/site_project/media"
            - "./site_project/static:/home/site_project/static"
        expose:
            - 8000
        depends_on:
            - database
```

<br/><br/>
### The Web Server
A nginx server is used as the web server. I also thought about using an Apache web server, but after some research I decided that nginx is the better option. The nginx:aplpine image from Docker Hub is used. I give it the name "webserver_mallowz" so that I can find the corresponding container in Docker later. The Django/Gunicorn installation runs on port 8000, this port is forwarded to the outside via port 7072 - so the webserver can be reached under localhost:7072. The next section in the configuration deals with the sharing or mounting of some directories of the Django installation, which should be able to be used and reached by the web server. I went the way of including directories on the local machine into the image. This allows me - at least during development - to store files locally and still be able to access them via the web server without having to copy files in a complicated way. The depends section indicates that the web server should wait for the Django installation. Thus, it specifies the order in which the individual containers must be started. This makes sense, because if Django is not yet started, but the web server already wants to access Django directories, this can lead to unsightly errors. Here again the section from the compose file for the web server installation. 

```
    web:
        image: nginx:alpine
        container_name: webserver_mallowz
        restart: always
        ports:
            - 7072:8000
        volumes:
            - "./etc/nginx/default.conf:/etc/nginx/conf.d/default.conf"
            - "./site_project/media:/home/site_project/media"
            - "./site_project/static:/home/site_project/static"
        depends_on:
            - django
```

