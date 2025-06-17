# Introduction to Nginx
Nginx is a robust and efficient web server that is widely used in modern web infrastructure. Unlike traditional web servers that use a multi-threaded architecture, Nginx employs a non-threaded, event-driven architecture. This design allows Nginx to handle many more connections simultaneously, making it highly suitable for high-traffic websites.

Beyond serving static web pages, Nginx is versatile and can be configured to perform a variety of roles, including load balancing, HTTP caching, and acting as a reverse proxy. These capabilities make it a cornerstone in many complex web architectures.

## Forward Proxy vs. Reverse Proxy
To understand Nginx’s role as a reverse proxy, it’s helpful to first distinguish between a forward proxy and a reverse proxy.

- Forward Proxy: In a traditional HTTP connection, a client sends a request directly to a server. However, with a forward proxy (such as a VPN), the client sends the request to the proxy, which then forwards it to the server. The server is unaware of the original client; it only interacts with the proxy. This setup is useful for privacy and bypassing geo-restrictions.

- Reverse Proxy: In contrast, a reverse proxy sits between the client and multiple servers. The client sends a request to the reverse proxy, which then decides which server should handle the request. The client remains unaware of which specific server processes the request. Nginx is a popular choice for a reverse proxy because of its efficiency and flexibility.


For example:

- /admin requests could be routed to Server 1.
- /settings requests could be routed to Server 2.

Nginx can handle these types of routing efficiently, ensuring that the correct server processes each request based on predefined rules.


## Advantages of Nginx
Nginx provides several key advantages that make it a preferred choice for many web projects:

1. High Concurrency: Capable of handling 10,000+ concurrent requests.
2. HTTP Caching: Can cache HTTP requests, reducing server load and improving response times.
3. Reverse Proxy: Acts as a reverse proxy, routing client requests to the appropriate server.
4. Load Balancing: Distributes incoming requests across multiple servers to balance the load.
5. API Gateway: Can act as an API gateway, managing and routing API requests.
6. Static File Serving: Efficiently serves and caches static files like images and videos.
7. SSL Termination: Handles SSL certificates, providing secure connections to users.

### Installation and Setting Up Nginx

1. Using Docker

In this section, we’ll walk through the steps to install and set up Nginx using Docker, configure it, and set up a sample project.

Step 1: Installing Nginx with Docker

To install Nginx in a Docker container, follow these steps:
```
docker run -it -p 8080:80 ubuntu
```
This command starts an Ubuntu container and maps port 8080 on your host machine to port 80 in the container.

Next, update the package list and install Nginx:
```
apt-get update
apt-get install nginx
```
You can verify the installation with:
```
nginx -v
```
2. On Host Machine
```
sudo apt update
sudo apt install nginx
```
After installation, you can start Nginx with:
```
sudo systemctl start nginx
```
To check the status:
```
sudo systemctl status nginx
```

### Nginx Configuration Basics
The main configuration file for Nginx is nginx.conf. It's typically located in /etc/nginx/nginx.conf. Let's break down a simple Nginx configuration file:
```
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
    }
}
```
Let’s understand each part of this file:

```
worker_processes auto;
```
This directive tells Nginx how many worker processes to start. Worker processes are responsible for handling requests. Setting it to auto allows Nginx to automatically determine the optimal number based on your CPU cores.
```
events { worker_connections 1024; }
```
The events block sets the connection handling settings.
- worker_connections 1024; means that each worker process can handle 1024 simultaneous connections. If you have 4 worker processes, you can handle 4096 connections concurrently.
```
http { ... }
```
- The http block is where you define settings for handling HTTP requests. This is where most of your web server's configuration will reside.
```
include mime.types;
```
- This directive tells Nginx to include a file called mime.types, which maps file extensions to MIME types. MIME types tell the browser how to handle different types of files (e.g., .html as text/html, .jpg as image/jpeg).
```
default_type application/octet-stream;
```
- If Nginx can’t determine the MIME type of a file, it will use application/octet-stream as a fallback, which generally means "download this file."
```
sendfile on;
```
- This directive enables the sendfile system call, which allows Nginx to serve files more efficiently by bypassing the need to copy data between buffers.
```
keepalive_timeout 65;
```
- This setting determines how long a connection should stay open (in seconds) when it’s idle, before Nginx closes it. The default is 65 seconds.

Now let’s talk about the server block which is most important part of the file:
```
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```
```
server { ... }
```
- The server block defines the configuration for a virtual server, i.e., a server that will handle requests for a particular domain or IP address.
```
listen 80;
```
- This tells Nginx to listen for HTTP requests on port 80. Port 80 is the default port for HTTP.
```
server_name localhost;
```
- The server_name directive specifies the domain name that this server block will respond to. localhost is typically used for local testing, but in a real-world scenario, you'd replace it with your domain name (e.g., example.com).
```
location / { ... }
```
- The location block is used to define how to respond to different request URIs. The / means that this block will handle requests to the root URL (e.g., http://localhost/).
```
root /usr/share/nginx/html;
```
- The root directive specifies the directory where Nginx should look for files to serve. Here, it's set to /usr/share/nginx/html, which is the default directory where Nginx stores HTML files.
```
index index.html index.htm;
```
- The index directive lists the files Nginx should look for if a directory is requested. For example, if a user navigates to http://localhost/, Nginx will try to serve index.html or index.htm from the root directory.

### Configuring Nginx
Once Nginx is installed, a directory is created at /etc/nginx/. The most important file in this directory is nginx.conf, which contains the main configuration for Nginx.

First, let’s back up the original nginx.conf:
```
mv /etc/nginx/nginx.conf /etc/nginx/nginx-backup.conf
```
Now, you can create a new nginx.conf file using a text editor like vim:
```
vim /etc/nginx/nginx.conf
```

In this file, you’ll define your Nginx configuration. Here’s a simple example:
```
events {
}

http {
    server {
        listen 80;
        server_name _;  # This means it will respond to any server name
        location / {
            return 200 "Hello from Nginx Conf File";
        }
    }
}
```
After editing the configuration, you need to reload Nginx to apply the changes:
```
nginx -s reload
```

### Setting Up a Basic Website with Nginx
Now that we have a basic understanding of Nginx, let’s create a simple project to demonstrate its capabilities.

1. Create a Project Directory:
```
mkdir -p /www/data
echo "<h1>Welcome to Nginx</h1>" > /www/data/index.html
```
2. Configure Nginx:

Update the nginx.conf to point to the /www/data directory:
```
server {
    listen 80;
    server_name localhost;
    root /www/data;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
3. Reload Nginx:
```
nginx -s reload
```
Access the Website: Open your browser and go to http://localhost:8080. You should see the message "Welcome to Nginx."

### Let’s understand some of the most used blocks of NGINX:
Nginx uses various blocks to structure its configuration. The three most common are server, location, and upstream blocks.

1. Server Block
- Purpose: Defines the configuration for a particular domain or IP address.
- Example: Already covered in the basic configuration above.
2. Location Block
- Purpose: Defines how to respond to specific request URIs.
- Example:
```
location /images/ {
    root /var/www/images;
}
```
3. Upstream Block
- Purpose: Defines a group of backend servers for load balancing.
- Example:
```
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
}
```

### Reverse Proxy with Nginx
A reverse proxy sits between a client and a backend server, forwarding client requests to the backend and sending the backend’s response back to the client. Here’s how you configure Nginx as a reverse proxy:
```
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend_server;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
- __proxy_pass http://backend_server;__: This tells Nginx to forward requests to the backend server.
- __proxy_set_header Host $host;__: Ensures the correct Host header is passed to the backend.
- __3proxy_set_header X-Real-IP $remote_addr;__: Passes the client’s real IP address to the backend.
- __proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;__: Adds the client’s IP to the __X-Forwarded-For__  header, which is used by the backend to identify the original client IP.


### Load Balancing with Nginx
Nginx can distribute traffic among multiple backend servers to balance the load.
```
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
    }
}
```
- __upstream backend { ... }__: Defines a group of backend servers.
- __server backend1.example.com;__: Adds a server to the backend group.
- __proxy_pass http://backend;__: Forwards requests to one of the servers in the backend group.

### Securing Nginx with SSL/TLS
SSL/TLS encrypts the data between your server and clients. Here’s how to configure SSL in Nginx:
```
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;

    location / {
        root /var/www/html;
        index index.html;
    }
}

server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```
- __listen 443 ssl;__: Tells Nginx to listen on port 443 (the default port for HTTPS) with SSL enabled.
- __ssl_certificate and ssl_certificate_key__: Specifies the paths to your SSL certificate and private key.
- __return 301 https://$host$request_uri;__: Redirects all HTTP requests to HTTPS.

### Advanced Nginx Configuration
1. Caching

Nginx can cache responses from your backend to speed up response times for subsequent requests.
```
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;
```
- __proxy_cache_path__: Specifies the directory and settings for caching.
- __levels=1:2__: Defines the directory structure for the cache.
- __keys_zone=my_cache:10m__: Creates a shared memory zone named my_cache with 10MB of memory.
- __max_size=1g__: Limits the cache size to 1GB.
- __inactive=60m__: Removes items from the cache if they haven't been accessed in 60 minutes.

2. Monitoring and Logging

Nginx logs all requests and errors, which is essential for troubleshooting and monitoring.
```
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

access_log /var/log/nginx/access.log main;
```
- __log_format main ...__: Defines a custom log format called main.
- __access_log /var/log/nginx/access.log main;__: Stores access logs in the specified file using the main log format.

### Common Errors
403 Forbidden: Typically means Nginx can’t access the requested file.
500 Internal Server Error: Often indicates a problem with the backend server or a misconfiguration in Nginx.

### Conclusion
Nginx is a powerful and versatile web server that has become a cornerstone in the modern web infrastructure. From its origins as a solution to the C10K problem — handling thousands of concurrent connections — it has grown into a multi-purpose tool that can act as a reverse proxy, load balancer, and even a caching server. Its lightweight architecture, coupled with high performance and reliability, makes Nginx the go-to choice for many of the world’s largest and most trafficked websites.

As you continue to explore Nginx, you’ll find it to be not just a tool, but a fundamental part of building and managing robust, high-performance web services.