FROM nginx:latest

RUN echo "deb http://deb.debian.org/debian stretch-backports main" >/etc/apt/sources.list.d/stretchbp.list

RUN apt-get update && \
    apt-get install -y certbot python-certbot-nginx -t stretch-backports

COPY ./notebook.conf /etc/nginx/conf.d/.
