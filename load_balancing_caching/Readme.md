### Load Balancing + Caching
Example: Round-robin load balancing

Let’s simulate 2 Node.js apps on different ports:
1. Run 2 apps
```
# App1
cd /app && cp server.js server1.js
sed -i 's/3000/3001/' server1.js
node server1.js &

# App2
cp server.js server2.js
sed -i 's/3000/3002/; s/Node.js/Node.js #2/' server2.js
node server2.js &
```
2. Nginx config
```
nano /etc/nginx/sites-available/load_balancer
```
Paste
```
upstream backend {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}

server {
    listen 80;
    server_name lb.localhost;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
Enable and reload:
```
ln -s /etc/nginx/sites-available/load_balancer /etc/nginx/sites-enabled/
nginx -t && service nginx reload
```
Add to /etc/hosts:
```
127.0.0.1 lb.localhost
```
 Visit http://lb.localhost:8080 multiple times — see responses from both backends.

 #### Enable Caching
 Inside that same block:
```
location / {
    proxy_cache my_cache;
    proxy_cache_valid 200 10s;
    proxy_pass http://backend;
}
```
Add this to http block in nginx.conf:
```
proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=my_cache:10m inactive=60m use_temp_path=off;
```