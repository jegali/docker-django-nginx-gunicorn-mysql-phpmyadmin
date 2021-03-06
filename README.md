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
A nginx server is used as the web server. I also thought about using an Apache web server, but after some research I decided that nginx is the better option. The nginx:aplpine image from Docker Hub is used. I give it the name "webserver_mallowz" so that I can find the corresponding container in Docker later. The Django/Gunicorn installation runs on port 8000, this port is forwarded to the outside via port 7072 - so the webserver and the django installation can be reached under localhost:7072. The next section in the configuration deals with the sharing or mounting of some directories of the Django installation, which should be able to be used and reached by the web server. I went the way of including directories on the local machine into the image. This allows me - at least during development - to store files locally and still be able to access them via the web server without having to copy files in a complicated way. The depends section indicates that the web server should wait for the Django installation. Thus, it specifies the order in which the individual containers must be started. This makes sense, because if Django is not yet started, but the web server already wants to access Django directories, this can lead to unsightly errors. Here again the section from the compose file for the web server installation. 

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

### pypMyAdmin
As mentioned above, the pypMyAdmin instance is only for more convenient maintenance of the database. Instead of having to log in to the container and enter console commands, I allowed myself the luxury of accessing the database via a web interface. The official Docker Hub image of phpmyadmin is used, which usually makes the service available via port 80. I changed this port to port 7070, so that the installation can now be reached under localhost:7070. PMA_HOST is used to specify the name of the database instance that phpMyAdmin should access, which in my case is mysql_mallowz. As you can see very nicely, in the Docker Compose file you have to pass a password with which you can log in to phpMyAdmin. In my case the password is in plain text in the file - clearly a security hole. Similar to the web server, there is also a Depends statement. Here it waits for the successful start of the database instance before phpMyAdmin itself can be started and called.


```
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
```

### The mySQL / mariaDB database
The database is also available for download as a ready-made image in Docker Hub. Similar to the web server, a volume was also released here - all mySQL configurations and database files are stored on the local computer and included in the image. The access data for the database must be stored in the environment variables. In this case, too, these are available in plain text and are thus to be classified as a security vulnerability. A better way is to either set sensitive information as environment variables in advance or to use secrects. 

```
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

```

### The Django installation
The Django installation is not based on any image. This is created via a Docker file itself and then mounted and started as part of the docker compose. This is what the build section in docker compose is for. The files are collected from the current directory "." and built to an image via the Dockerfile - also located in the current directory. The "command" section instructs Docker to run the command "gunicorn -b 0.0.0.0:8000 -w 4 site_project.wsgi" when the image is started, thus starting Django via gunicorn. Of course, django also needs some directories to work correctly, which are again mounted via volumes as usual. The service is passed out of the container on port 8000. Finally and finally, django needs access to the database. Therefore, of course, the "depends on" section can be found in the description. The functionality has already been described.

```
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

## The Dockerfile for Django
Of course, we'll take a look at the Dockerfile as well. The content is docker typical. First, a base image is loaded from Dockerhub based on the python:3.9 image. Within this image, some updates and basic installation are done before the working directory of the Django project is mounted. Note: At this point, the working directory should be set via an environment variable and queried in the appropriate places. This increases the error tolerance when deploying the application. The next step is to load the dependencies needed for the image from the requirements.txt file and install them in the image. To do this, the requirements file is copied to the image and the python interpreter or the pip tool is started locally. Finally, the link between the local Django directory and the directory in the image is created.

```
FROM python:3.9
ENV PYTHONUNBUFFERED 1

# Prerequisites for python-ldap
RUN apt update && apt-get install -y libsasl2-dev python-dev libldap2-dev libssl-dev php

# Create working directory
RUN mkdir /home/site_project

# Specify working directory
WORKDIR /home/site_project

# Copy the requirements file to the working directory
ADD ./requirements.txt /home/site_project

# Install all the python packages inside the requirements file
RUN pip install --no-cache-dir -r /home/site_project/requirements.txt

# Copy the Django site to the working directory
ADD ./site_project /home/site_project
```

### Obstacles
Obstacles in the development of the Django image was that even when calling "docker-compose down --volumes" and then rebuilding the image via "docker-compose up -d", changed requirements regarding the versioning of the components were not implemented. I had to change the Django version several times because the connection to the mySQL database and the necessary "migrate" command did not work without errors. But even after changing the Django version the error remained and could only be solved by removing the image permanently from my Docker repository via "docker rmi django_mallowz".
<br/><br/>

### Requirements.txt
Eventually I found that this software constellation worked to deploy the image without errors. 

```
django>=3.0.7
pytz==2021.1
sqlparse==0.4.1
gunicorn>=19.9.0
django-mysql==3.9
mysqlclient>=2.0
```

## The nginx configuration
I also faced some challenges with nginx. For example, I had to figure out that a config file needs to be created in the first place. However, after I had implemented the basic configuration of the web server correctly, the Django installation would not start. After some experimentation, I came up with this solution:

```
# Nginx configuration

upstream django {
	server django:8000;
}

server {

    listen 8000;
    server_name django;

    location /static/ {
        expires 30d;
        autoindex on;
        add_header Cache-Control private;
        root /home/site_project;
    }
    location / {
        proxy_pass http://django/;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }
    
}
```

The file is - as specified in the dockercompose file - to be saved in the directory "etc/nginx/default.conf"

# Project installation and configuration

1. Please make sure you have python and django installed
2. Open up command prompt
3. Create a new project folder and cd into it
4. Create etc/nginx folder and place default.conf inside it
5. Create a new django project with "django-admin startproject site_project ."
6. Edit site_project/site_project/settings.py and exchange

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': str(BASE_DIR / 'db.sqlite3'),
    }
}
```
with

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', 
        'NAME': 'mallowz',			# mysql database from docker-compose
        'USER': 'root',				# user from docker-compose
        'PASSWORD': 'root',			# password from docker-compose
        'HOST': 'database', 			# name of service from docker-compose
        'PORT': '3306',				# internal port from service in docker-compose
    }
}
```

7. From inside the "site_project" directory do a "python manage.py migrate" to set up the database.
8. From inside the "site_project" directory do a "django-admin collectstatic" to collect the css and other static files.
9. Execute a "docker-compose up -d"
10. Open up the browser and navigate to "localhost:7072". You should see the django rocket.
