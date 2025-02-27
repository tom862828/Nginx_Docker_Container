# Nginx_Docker_Container
 Nginx docker container mounting certificate to implement HTTPS connection.

* Step 1:
Use openssl tool to generate certificate and private.key in your local side.
```cmd
openssl req -x509 -newkey rsa:2048 -keyout private.key -out certificate.crt -days 3650 -nodes
```
* Step 2:
Pull Nginx Image.
```cmd
docker pull nginx
```
* Step 3:
Write the nginx config file.
```nginx  
server {
listen 443 ssl;
ssl_certificate /etc/nginx/ssl/certificate.crt;
ssl_certificate_key /etc/nginx/ssl/private.key;
server_name localhost;
ssl_session_timeout 5m;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
ssl_prefer_server_ciphers on;
client_max_body_size 1024m;

    location / { 
        proxy_set_header X-Real-IP $remote_addr; 
        proxy_set_header Host $http_host; 
        proxy_set_header X-NginX-Proxy true;
        # proxy_pass <URL>:<port>; In this example is passing the request to the other local container.
        proxy_pass http://host.docker.internal:8004;
    }
}
server {
    listen 80;
    server_name localhost;
    return 301 https://$host$request_uri;
}
``` 
* Step 4:
Create the nginx container and mount the default.conf, certificate and private.key 
```cmd
docker run --name nginx -e TZ=Asia/Taipei --restart=always -p 80:80 -p 443:443 -v <local_path>/default.conf:/etc/nginx/conf.d/default.conf  -v <local_certificate_private.key>/cert:/etc/nginx/ssl -d nginx
```
* Nginx has https protocol as below picture.
![Image](https://github.com/tom862828/Nginx_Docker_Container/blob/main/nginx_https_result.png)