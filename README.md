# Python Jupyter Notebook Web server
Setting up a Python Jupyter Notebook Web Server with Docker containers.

## Objectives
This project provides a set of Dockerfiles and some must configration files for creating a Python Jupyter Notebook Web Server with minimium affects

## Overall Design
A docker container is used to run a nginx server as a proxy server to the jupyter notebook server. The nginx supports https secure connections to the users.
Another container is created to run jupyter notebook server.
This two containers configuration gives the maximum flexibility for server migrations and multiple jupyter notebook servers suppoting.

## Preconditions
It assumes that a Ubuntu Linux server is used to set up the web server.
On the Ubuntu server, it requires:
### 1. Docker has been installed.
Use command:
```
$ docker -v
```
The response on my system is as below:
```
Docker version 18.09.6, build 481bc77
```
If Docker hasn't been installed, refer to https://docs.docker.com/install/linux/docker-ce/ubuntu/
### 2. Make sure you are in 'docker' group
If you use below command and get the similar error response:
```
$ docker ps -a
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.39/containers/json?all=1: dial unix /var/run/docker.sock: connect: permission denied
```
Then use the below command to add your self to the 'docker' group
```
$ sudo usermod -aG docker $USER
```

## Create and configure Nginx Container
If you have a Nginx server has been set up, you can simply copy the file <webpyjnb_home>/nginx/notebook.conf into /etc/nginx/conf.d/., and use command '$ certbot --nginx' to configure the ssl certificate. Otherwise, use below commands to create a new nginx server in a container:
```
$ cd <webpyjnb_home>/nginx
$ docker build -t webpyjnb/nginx .
```



