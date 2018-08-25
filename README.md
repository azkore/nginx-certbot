# nginx-certbot

docker-ized nginx with let's encrypt certbot for ssl certz!

### demo

__docker-compose.yml__

```sh
version: "2"
services:
  nginx-certbot:
    image: 3dwardsharp/nginx-certbot
    environment:
      - DOMAINS=demo.youoke.party,youoke.party
      - EMAIL=edward@edwardsharp.net
    volumes:
      - /home/core/letsencrypt:/etc/letsencrypt
      - ./nginx.template:/etc/nginx/conf.d/nginx.template
    ports:
      - "80:80"
      - "443:443"
    environment:
      - BASE_SERVER=youoke.party
      - BASE_SERVER_PROXY="helloworld:80"
      - ADMIN_SERVER=demo.youoke.party
      - ADMIN_SERVER_PROXY="demo:80"
    command: /bin/bash -c "envsubst < /etc/nginx/conf.d/nginx.template > /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'"
  helloworld: 
    image: 3dwardsharp/helloworld
  demo: 
    image: 3dwardsharp/helloworld
```

__nginx.template__

```
server {
  listen 80;
  server_name ${BASE_SERVER};
  
  include /etc/nginx/snippets/letsencrypt.conf;

  location / {
    return 301 https://${BASE_SERVER};
  }
}
server {
  listen 80;
  server_name ${ADMIN_SERVER};

  include /etc/nginx/snippets/letsencrypt.conf;

  location / {
    return 301 https://${ADMIN_SERVER};
  }
}

server {
  server_name ${BASE_SERVER};
  listen 443 ssl http2;

  ssl_certificate /etc/letsencrypt/live/${BASE_SERVER}/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/${BASE_SERVER}/privkey.pem;
  ssl_trusted_certificate /etc/letsencrypt/live/${BASE_SERVER}/fullchain.pem;
  include /etc/nginx/snippets/ssl.conf;

  location / {
    proxy_pass http://${BASE_SERVER_PROXY};
    client_max_body_size 100m;
    proxy_buffering off;
  }
}
server {
  server_name ${ADMIN_SERVER};
  listen 443 ssl http2;

  ssl_certificate /etc/letsencrypt/live/${ADMIN_SERVER}/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/${ADMIN_SERVER}/privkey.pem;
  ssl_trusted_certificate /etc/letsencrypt/live/${ADMIN_SERVER}/fullchain.pem;
  include /etc/nginx/snippets/ssl.conf;

  location / {
    proxy_pass http://${ADMIN_SERVER_PROXY};
    client_max_body_size 100m;
    proxy_buffering off;
  }
}
```