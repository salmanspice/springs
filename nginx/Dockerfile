FROM nginx

RUN apt-get update; apt-get install -y \
    openssl

COPY ./nexus3.key /etc/nginx/external/key.pem
COPY ./nexus3.crt /etc/nginx/external/cert.pem

COPY nginx.conf /etc/nginx/nginx.conf