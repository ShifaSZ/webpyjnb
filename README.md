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
### 3. Clone this repository to directory <webpyjnb> with below command
```
$ get clone https://github.com/ShifaSZ/webpyjnb.git
```

## Create and configure Nginx Container
If you have a Nginx server has been set up, you can simply copy the file <webpyjnb_home>/nginx/notebook.conf into /etc/nginx/conf.d/., and use command '$ certbot --nginx' to configure the ssl certificate. Otherwise, use below steps to create a new nginx server in a container:
### Create a docker image with nginx and certbot in it.
Modify the domain name for the server_name in <webpyjnb_home>/nginx/notebook.conf to your own domain name. If you don't have a domain name, use your server IP address should also work.
Run below commands:
```
$ cd <webpyjnb_home>/nginx
$ docker build -t webpyjnb/nginx .
```
### Create a container withe image just created
```
$ docker run -d --name=web -p 80:80 -p 443:443 webpyjnb/nginx
```
### Configure the certificate with cerbot
```
$ docker exec -it web bash
# certbot --nginx
```
Enter your email address, answer the questions, choose the notebook server domain, choose to redirect all the http requests to https
Exit the container and restart it with below commands:
```
# exit
$ docker container restart web
```

## Create and configure jupyter notebook container
### Modify login password
In file <webpyjnb_home>/notebook/jupyter_notebook_config.py, look for below item:
```
c.NotebookApp.password = 'sha1:b3abcdddc34b:ed5888b71e8eb85f67e79cda5e0a1c842b541d29'
```
You can replace the password hash with the one generated below python codes"
```
from notebook.auth import passwd; 
passwd();
```

### Create a jupyter notebook docker image
```
$ cd <webpyjnb_home>/notebook
$ docker build -t webpyjnb/notebook .
```
### Create a jupyter notebook docker container
```
$ docker run -d --name=notebook webpyjnb/notebook
```

### Update the IP address of jupyter notebook container to nginx
Check the IP address of jupyter notebook container with below command:
```
$  docker inspect notebook |grep IPAddress
```
Update the file web:/etc/nginx/conf.d/notebook.conf for the line:
```
proxy_pass http://172.17.0.5:8100/notebook;
```
Change the IP address to the one for the notebook container.

Tips: You can use "$ docker cp" to copy out the file notebook.conf from the web container and copy it back after modification.

Restart the nginx container:
```
$ docker container restart web
```

### User a different URL path for the jupyter notebook container
If you have multiple jupyter notebook containers, you have to use different URL path for each of them. You need to:
#### 1. duplicate a locaion directives
In web:/etc/nginx/conf.d/notebook.conf, create a location directive for each of the jupyter notebook. A location directive looks as below:
```
    location /*notebook* {
        proxy_pass http://172.17.0.5:8100/*notebook*;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $http_host;
        proxy_http_version 1.1;
        proxy_redirect off;
        proxy_buffering off;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400;
    }
```
You have to change the "notebook" after "location" and "proxy_pass" to the URL path you choose.
#### 2. Modify the base URL for jupyter notebook
In file <webpyjnb_home>/notebook/jupyter_notebook_config.py, look for below item:
```
c.NotebookApp.base_url = '/notebook'
```
Replace the "notebook" to the URL path you choose.

## Usage
You can open https://you.domain.name.com/notebook to use the jupyter notebook.
