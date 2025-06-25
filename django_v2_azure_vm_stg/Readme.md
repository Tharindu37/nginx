1. Remove Nginx Completely

Stop the Nginx service
```
sudo systemctl stop nginx
```
Disable Nginx from auto-starting
```
sudo systemctl disable nginx
```
Uninstall Nginx and related packages
```
sudo apt-get purge nginx nginx-common nginx-full -y
```
Auto-remove unused dependencies
```
sudo apt-get autoremove -y
```
Clean up configuration files and logs
```
sudo rm -rf /etc/nginx
sudo rm -rf /var/log/nginx
sudo rm -rf /var/www/html
```
Verify Removal
```
which nginx
# Should return nothing

sudo systemctl status nginx
# Should say: Unit nginx.service could not be found.
```

2. Reinstall Nginx

SSH into Your VM
```
ssh <username>@<your-vm-public-ip>
```
Install Nginx
```
sudo apt update
sudo apt install nginx -y
```
Start and enable Nginx:
```
sudo systemctl start nginx
sudo systemctl enable nginx
```
Allow Web Traffic in Azure
```
Go to your VM → Networking
Click Add inbound port rule
Allow:
- Port: 80
- Protocol: TCP
- Name: HTTP
```
Upload Your Static Website Files

SCP (from your local terminal)
```
scp -r ./my-website-folder <username>@<vm-ip>:/tmp
```
Use Git (on the VM)
```
cd /var/www/
sudo git clone https://github.com/<your-repo>.git mysite
```
Move Website Files to Nginx Web Directory

Default Nginx root: /var/www/html
```
sudo rm -rf /var/www/html/*
sudo cp -r /tmp/my-website-folder/* /var/www/html/
```
Change ownership (optional):
```
sudo chown -R www-data:www-data /var/www/html
```
Configure Nginx for Your Static Site (optional)

If your site needs custom config (e.g., index.html, custom error pages):
```
sudo nano /etc/nginx/sites-available/default
```
Ensure it looks like this:
```
server {
    listen 80;
    server_name _;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Save (Ctrl + O, then Enter, then Ctrl + X) and restart:
```
sudo systemctl reload nginx
```
Access Your Website
```
http://<your-vm-public-ip>
```

IMPORTANT: You must have a domain name (like example.com) pointing to your VM's public IP to use Let's Encrypt.

1. Point Your Domain to Your Azure VM

Go to your domain registrar (GoDaddy, Namecheap, etc.) and Set an A record:
| Type | Host         | Value (IP)      |
| ---- | ------------ | --------------- |
| A    | `@` or `www` | `40.123.213.46` |

Wait a few minutes for DNS to update

You can test with:
```
nslookup yourdomain.com
```

2. Install Certbot and Get Free SSL Certificate

Add Certbot repo & install
```
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```
Request SSL certificate
```
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```
Certbot will:
- Auto-detect your Nginx config
- Ask for email & agreement
- Get and install cert
- Reload Nginx with HTTPS enabled 

Test HTTPS
```
https://yourdomain.com
```

3. Auto-Renew SSL (Let’s Encrypt lasts 90 days)
Set up cron job:
```
sudo systemctl status certbot.timer
```
If inactive, enable it:
```
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```
Or add this to crontab:
```
sudo crontab -e
```
Add line:
```
0 0 * * * /usr/bin/certbot renew --quiet
```
```
0 0 * * * /usr/bin/certbot renew --quiet --deploy-hook "/bin/systemctl reload nginx" >> /var/log/certbot-renew.log 2>&1
```
Test Auto-Renew Now
```
sudo certbot renew --dry-run
```
Delete the certificate
```
sudo certbot delete --cert-name neonautica.ae
```
Confirm Certificate Is Gone
```
sudo certbot certificates
```
SSL certificate expiry date
```
sudo openssl x509 -enddate -noout -in /etc/letsencrypt/live/nginx-ssl-test.duckdns.org/cert.pem
```


## Deploy Django MVT Project on Azure Ubuntu Server

We'll use:
- Gunicorn as the WSGI app server
- Nginx as the reverse proxy
- Certbot for SSL

1. Upload Your Django Project to Server

Use scp or Git:
```
scp -r /path/to/your/project user@your_server_ip:/home/youruser/
```
Or clone from GitHub:
```
git clone https://github.com/yourusername/yourproject.git
```

2. Set Up Python Environment
```
sudo apt update
sudo apt install python3-pip python3-venv
cd yourproject/
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

3. Configure Django Settings

Set ALLOWED_HOSTS in settings.py:
```
ALLOWED_HOSTS = ['your_domain_or_ip', 'www.your_domain']
```

Collect Static Files:
```
python manage.py collectstatic
```

Run Migrations:
```
python manage.py migrate
```

(Optional) Create superuser:
```
python manage.py createsuperuser
```

4. python manage.py createsuperuser
```
gunicorn --bind 127.0.0.1:8000 yourproject.wsgi:application
```
You should see Gunicorn worker messages.

If it works, stop it (Ctrl+C) and create a systemd service.

5. Create systemd Service for Gunicorn
```
sudo nano /etc/systemd/system/gunicorn.service
```
Paste:
```
[Unit]
Description=gunicorn daemon for Django project
After=network.target

[Service]
User=yourusername
Group=www-data
WorkingDirectory=/home/yourusername/yourproject
ExecStart=/home/yourusername/yourproject/venv/bin/gunicorn --access-logfile - --workers 3 --bind 127.0.0.1:8000 yourproject.wsgi:application

[Install]
WantedBy=multi-user.target
```
Then enable and start:
```
sudo systemctl daemon-reexec
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
sudo systemctl status gunicorn

```
```
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
sudo systemctl status gunicorn
```

6. Configure Nginx
```
sudo nano /etc/nginx/sites-available/yourproject
```
Paste:
```
server {
    listen 80;
    server_name your_domain.com www.your_domain.com;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/yourusername/yourproject;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Enable it:
```
sudo ln -s /etc/nginx/sites-available/yourproject /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

7. Re-Enable SSL
```
sudo certbot --nginx -d your_domain.com -d www.your_domain.com
```
Your Django App Is Live!
```
https://your_domain.com
```

## Upload And Downlaod File

### Step-by-Step Fix: Set Proper .pem File Permissions on Windows
Open PowerShell as Administrator
```
# Replace with your actual file path
$KeyPath = "E:\Silverline IT\Silverline_Project_2024_12\bluepass_key\stgmultipass_key.pem"

# Remove all inherited permissions
icacls "$KeyPath" /inheritance:r

# Remove all users
icacls "$KeyPath" /remove "Authenticated Users" "Users" "Everyone"

# Grant read-only access to the current user only
icacls "$KeyPath" /grant:r "$($env:USERNAME):R"
```
Upload
```
scp -i ../../bluepass_key/stgmultipass_key.pem db.sqlite3 NeoBluePass@40.123.213.46:/opt/Webapp-Lemonade
```
Download
```
scp -i ../bluepass_key/prodmultipass_key-20240127.pem -r NeoBluePass@20.203.103.1:/opt/MultiPass ./MultiPass
```
Zip File
```
sudo apt install zip
zip -r compressed_folder.zip folder_name/
```