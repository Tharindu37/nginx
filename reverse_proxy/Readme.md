###  Reverse Proxy
A reverse proxy forwards client requests to backend services (like Node.js, Django) and returns the response to the client — acting as a middleman.
1. Start a basic Node.js server (for test)
```
apt install curl -y
curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt install -y nodejs
```
Create app
```
mkdir /app && cd /app
nano server.js
```
Paste
```
const http = require('http');
http.createServer((req, res) => {
  res.end('Hello from Node.js Backend!');
}).listen(3000, () => console.log('Node running on port 3000'));
```
Run
```
node server.js
```
2. Nginx reverse proxy config

Create
```
nano /etc/nginx/sites-available/node_proxy
```
Paste:
```
server {
    listen 80;
    server_name node.localhost;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
Link, reload:
```
ln -s /etc/nginx/sites-available/node_proxy /etc/nginx/sites-enabled/
nginx -t && service nginx reload
```
Update /etc/hosts:
```
127.0.0.1 node.localhost
```
Visit: http://node.localhost:8080 → You'll see: Hello from Node.js Backend!