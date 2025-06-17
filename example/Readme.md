index.html
```
/www/data/index.html
```
nginx.conf
```
/etc/nginx/nginx.conf
```
```
nginx -s reload
```

 Step 1: Uninstall NGINX
 ```
apt remove nginx nginx-common -y
apt purge nginx nginx-common -y
apt autoremove -y
 ```
1. "remove" removes the package
2. "purge" removes config files
3. "autoremove" cleans up unused dependencies

Step 2: Reinstall NGINX
```
apt update
apt install nginx -y
```
Step 3: Start NGINX and Enable It
```
nginx   # Start NGINX manually
```
Or if you're in a full system with systemd (like a VM or full Ubuntu):
```
systemctl start nginx
systemctl enable nginx
```