# Creating Reverse Proxy (Nginx) Docker Container
Source: https://enable-cors.org/server_nginx.html

Use case: In some circumstances, CORS prevents web apps from accessing API's. Scenarios such as running an Azure Function App in one docker container, and a web app in another, will trigger an error. This is a security feature and should not be implimented in prod without serious consideration, however is useful for local development environments.

---------------------------------------------

## Quick Deploy
1. Create the following nginx.conf file:

   *host.docker.internal is the internal network name for Docker containers.*
```
server 
{
    listen       80;
    server_name  localhost;

    location /api/ 
    {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-NginX-Proxy true;
        proxy_pass http://host.docker.internal:8080/api/;

        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            #
            # Custom headers and headers various browsers *should* be OK with but aren't
            #
            add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
            #
            # Tell client that this pre-flight info is valid for 20 days
            #
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Content-Type' 'text/plain; charset=utf-8';
            add_header 'Content-Length' 0;
            return 204;
        }
        if ($request_method = 'POST') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
            add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
        }
        if ($request_method = 'GET') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
            add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
        }
    }        
}
```
2. Create the following DOCKERFILE:
```
FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
ENTRYPOINT ["nginx","-g","daemon off;"]
```
3. Build the Docker image: ```docker build -f Dockerfile -t nginx-reverseproxy:latest .```
4. Deploy Docker container: ```docker run -p 8080:80 nginx-reverseproxy:latest```

---------------------------------------------

## Basic Configuration Format:
```
server {
  server_setting
  server_setting
  
  location \
  {
    proxy_pass <URI>
  }
  
  location \api\
  {
    proxy_pass <URI>
  }
}
```
