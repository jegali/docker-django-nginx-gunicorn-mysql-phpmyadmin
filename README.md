# docker-django-nginx-gunicorn-mysql-phpmyadmin

## Abstract
In this repository you will find a description how to dockerize django, gunicorn, nginx and mysql. phpMyAdmin is only on board to provide a convenient administration interface.

## Intended use
This Docker image is to be used for a vulnerable web store that is built in Python. Until that happens, the repository will serve as a think tank so I don't forget over time how to set up this project.

## Structure
Prebuilt Docker images exist in Docker Hub for both phpMyAdmin, mySQL, and nginx. These images are assembled via docker-compose.

For Django there is no current image, this must be created individually. For this purpose, there is a Dockerfile in the repository, the structure of which I will explain in a moment. 
