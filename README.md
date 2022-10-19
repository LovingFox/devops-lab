# devops-lab at *.rtru.tk

### Make certificates

```
mkdir -p $(pwd)/data/certbot/{conf,www}

curl -s https://raw.githubusercontent.com/certbot/certbot/master/certbot-nginx/certbot_nginx/_internal/tls_configs/options-ssl-nginx.conf \
> $(pwd)/data/certbot/conf/options-ssl-nginx.conf

curl -s https://raw.githubusercontent.com/certbot/certbot/master/certbot/certbot/ssl-dhparams.pem \
> $(pwd)/data/certbot/conf/ssl-dhparams.pem

docker run -d -it --rm --name certbot \
-p 80:80 \
-v "$(pwd)/data/certbot/conf:/etc/letsencrypt" \
-v "$(pwd)/data/certbot/www:/var/lib/letsencrypt" \
--entrypoint sh \
certbot/certbot

docker exec -ti certbot certbot \
certonly --standalone -m lovingfox@gmail.com --agree-tos \
-d gitlab.rtru.tk -d jenkins.rtru.tk -d nexus.rtru.tk -n

docker stop certbot
```

### Start lab

```
mkdir -p $(pwd)/gitlab/{config,data,logs}

export DOCKER_COMMAND=$(which docker)
export DOCKER_GROUP=$(stat -c '%g' /var/run/docker.sock)
docker-compose up -d
```

### Renew certificates

```
docker run -it --rm --name certbot \
-v "$(pwd)/data/certbot/conf:/etc/letsencrypt" \
-v "$(pwd)/data/certbot/www:/var/lib/letsencrypt" \
certbot/certbot \
renew --webroot -w /var/lib/letsencrypt
```

### Revoke certificates

```
docker exec -ti certbot certbot revoke --cert-name gitlab.rtru.tk -n
```

