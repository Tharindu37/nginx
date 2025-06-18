1. /etc/nginx/nginx.conf
```
user www-data;
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout 65;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```
- Sets global directives
- __conf.d/*.conf:__ All files here are included.
- __sites-enabled/*:__ Enables virtual hosts.

2. /etc/nginx/sites-available/ & /etc/nginx/sites-enabled/
sites-available
- Contains all your site/server blocks (like Apache's vhosts).
- Not active until linked into __sites-enabled__.

 Enable a site (Symlink)
 ```
ln -s /etc/nginx/sites-available/my_site /etc/nginx/sites-enabled/
 ```
 Disable a site
 ```
rm /etc/nginx/sites-enabled/my_site
 ```

 3. Example: Create a new site
- Create a web root:
```
mkdir -p /www/myapp
echo "<h1>Welcome to My App</h1>" > /www/myapp/index.html
```
- Create config:
```
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name myapp.localhost;

    root /www/myapp;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
- Link it:
```
ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
nginx -t && service nginx reload
```
- Add to __/etc/hosts__:
```
127.0.0.1 myapp.localhost
```
- Access it via browser: http://myapp.localhost:8080

4. /etc/nginx/conf.d/
- All .conf files here are automatically included.
- Often used for default or small setups inside containers.
Example: Static site in conf.d
```
# /etc/nginx/conf.d/static.conf
server {
    listen 80;
    server_name static.localhost;

    root /www/static;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Works even without sites-available, useful in Docker or microservice configs.

5. /etc/nginx/snippets/
Optional directory used to include reusable config blocks.

Example: Create SSL snippet
```
# /etc/nginx/snippets/ssl-common.conf
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
```
Use it in a site config:
```
include snippets/ssl-common.conf;
```

6. /var/www/html/ (Default Web Root)
- Default root for Nginx on Ubuntu.
- You can replace this with your own path (e.g. /www/myapp).

7. /var/log/nginx/
- access.log: All HTTP requests.
- error.log: Any issues Nginx encounters.
Example: Check logs
```
tail -f /var/log/nginx/access.log
```