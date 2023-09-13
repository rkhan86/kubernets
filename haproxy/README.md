```bash
docker build -t rzkhan/haproxy-alpine:2.7 .
 docker network create --driver=bridge haproxy
 docker network ls
 sudo docker run -d \
   --name haproxy \
   --net devnet \
   --sysctl net.ipv4.ip_unprivileged_port_start=0 \
   -v $(pwd):/usr/local/etc/haproxy:ro \
   -p 80:80 \
   -p 8404:8404 \
   haproxytech/haproxy-alpine:2.4
```
