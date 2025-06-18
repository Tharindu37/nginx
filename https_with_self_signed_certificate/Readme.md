### HTTPS with Self-Signed Certificate
HTTPS encrypts traffic using SSL. We'll use a self-signed cert to test locally.

Secure your static website
1. Generate cert
```
mkdir -p /etc/nginx/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/self.key \
  -out /etc/nginx/ssl/self.crt \
  -subj "/CN=secure.localhost"
```
2. Create HTTPS config
```
nano /etc/nginx/sites-available/secure_site
```
Paste
```
server {
    listen 443 ssl;
    server_name secure.localhost;

    ssl_certificate /etc/nginx/ssl/self.crt;
    ssl_certificate_key /etc/nginx/ssl/self.key;

    root /www/secure;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Enable site
```
mkdir -p /www/secure && echo "<h1>Secure Site</h1>" > /www/secure/index.html
ln -s /etc/nginx/sites-available/secure_site /etc/nginx/sites-enabled/
nginx -t && service nginx reload
```
Add to /etc/hosts
```
127.0.0.1 secure.localhost
```
Visit: https://secure.localhost:8080 → Accept the self-signed cert warning.