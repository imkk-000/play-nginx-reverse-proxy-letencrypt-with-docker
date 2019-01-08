```sh
#!/bin/sh
docker stop nginx-proxy-service nginx-letsencrypt-service
docker rm -f nginx-proxy-service nginx-letsencrypt-service
rm -rf nginx
mkdir -p nginx/certs nginx/vhost nginx/html

docker run -d -p 80:80 -p 443:443 \
  --name nginx-proxy-service \
  -v $PWD/nginx/certs:/etc/nginx/certs:ro \
  -v $PWD/nginx/vhost:/etc/nginx/vhost.d \
  -v $PWD/nginx/html:/usr/share/nginx/html \
  -v /var/run/docker.sock:/tmp/docker.sock:ro \
  --label com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy \
  jwilder/nginx-proxy

docker run -d \
  --name nginx-letsencrypt-service \
  --volumes-from nginx-proxy-service \
  -v $PWD/nginx/certs:/etc/nginx/certs:rw \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -e NGINX_PROXY_CONTAINER=nginx-proxy-service \
  jrcs/letsencrypt-nginx-proxy-companion
    
docker run -d -e VIRTUAL_HOST="<hosts>" -e LETSENCRYPT_HOST="<hosts>" <web-service>
```
