Absolutely! Here's a complete **step-by-step guide to deploy a Django MVT project without Docker**, using:

✅ Nginx (as reverse proxy)
✅ Gunicorn (WSGI server)
✅ Ubuntu VM (e.g., Azure VM)
✅ Let’s Encrypt SSL (HTTPS)

---

# 🧱 Prerequisites

* An **Azure Ubuntu VM (22.04 recommended)**
* A **domain name** pointing to your VM's public IP (e.g., `yourdomain.com`)
* Allow **ports 22, 80, 443** in Azure Network Security Group

---

# 🚀 Step-by-Step: Django Deployment Without Docker

---

## ✅ 1. Update & Install Packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip python3-venv nginx certbot python3-certbot-nginx git -y
```

---

## ✅ 2. Create and Configure Your Django Project

### Option A: Clone Existing Project

```bash
cd ~
git clone https://github.com/yourname/your-django-project.git
cd your-django-project
```

### Option B: Create a New Project

```bash
cd ~
django-admin startproject mysite
cd mysite
```

---

## ✅ 3. Set Up Python Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install django gunicorn
```

If using a cloned project:

```bash
pip install -r requirements.txt
```

---

## ✅ 4. Update `settings.py`

Edit:

```python
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']
STATIC_ROOT = BASE_DIR / "static"
```

Collect static files:

```bash
python manage.py collectstatic
```

Run migrations:

```bash
python manage.py migrate
```

(Optional) Create a superuser:

```bash
python manage.py createsuperuser
```

---

## ✅ 5. Run Django with Gunicorn

```bash
gunicorn --bind 127.0.0.1:8000 mysite.wsgi:application
```

🔁 Keep it running with `systemd` in next step.

---

## ✅ 6. Set Up Gunicorn as a Systemd Service

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

Paste:

```ini
[Unit]
Description=gunicorn daemon for Django
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/mysite
ExecStart=/home/ubuntu/mysite/venv/bin/gunicorn --access-logfile - --workers 3 --bind 127.0.0.1:8000 mysite.wsgi:application

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reexec
sudo systemctl enable gunicorn
sudo systemctl start gunicorn
```

Check status:

```bash
sudo systemctl status gunicorn
```

---

## ✅ 7. Configure Nginx as a Reverse Proxy

```bash
sudo nano /etc/nginx/sites-available/mysite
```

Paste:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /static/ {
        alias /home/ubuntu/mysite/static/;
    }
}
```

Enable config:

```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

🔗 Visit: `http://yourdomain.com` — Django site is live!

---

# 🔐 Step 8: Enable HTTPS with Let’s Encrypt

Install Certbot and get SSL certificate:

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Follow prompts.

Certbot will:

* Configure SSL in your Nginx config
* Reload Nginx

Test:
🔗 Visit `https://yourdomain.com`

---

## 🔁 Auto-Renew SSL

```bash
sudo crontab -e
```

Add:

```bash
0 3 * * * /usr/bin/certbot renew --quiet
```

---

# ✅ Recap: What You Have Now

| Component          | Tool Used                   |
| ------------------ | --------------------------- |
| Web server         | **Nginx**                   |
| Application server | **Gunicorn**                |
| Framework          | **Django**                  |
| SSL (HTTPS)        | **Let’s Encrypt + Certbot** |
| Hosting            | **Azure Ubuntu VM**         |

---

# 🚀 Next Steps

Want to:

* Add **PostgreSQL/MySQL**?
* Set up **CI/CD (GitHub Actions / Azure Pipelines)?**
* Use **Azure Blob for media storage**?
* Add **security headers, gzip, caching** in Nginx?

Let me know — I can guide you next.



Here’s the **complete guide to configure HTTPS (SSL)** in your Nginx server for your Django app — assuming you're **not using Docker** and deployed Django with **Gunicorn + Nginx** on an **Azure Ubuntu VM** with a **real domain**.

---

# 🔐 Enable HTTPS with Let’s Encrypt (Full Setup)

---

## ✅ Step 0: What You Already Have

You should already have:

| ✅ Component | 🛠️ Tool                  |
| ----------- | ------------------------- |
| Django App  | Gunicorn (localhost:8000) |
| Nginx       | Reverse proxy (port 80)   |
| Ubuntu VM   | Azure or other VPS        |
| Domain      | Pointed to your public IP |

---

## ✅ Step 1: Install Certbot

If not already installed:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

---

## ✅ Step 2: Confirm Your Nginx Config Works

Check existing Nginx config:

```bash
sudo nano /etc/nginx/sites-available/mysite
```

It should look like this:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /static/ {
        alias /home/ubuntu/mysite/static/;
    }
}
```

Then:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

## ✅ Step 3: Run Certbot to Get SSL Certificate

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Certbot will:

* Automatically find your Nginx config
* Edit it to include SSL
* Reload Nginx

You’ll see:

```
Congratulations! You have successfully enabled https://yourdomain.com
```

---

## ✅ Step 4: What Certbot Adds to Nginx

Certbot updates your config to:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /static/ {
        alias /home/ubuntu/mysite/static/;
    }
}
```

---

## ✅ Step 5: Verify SSL Is Working

Visit:

```
https://yourdomain.com
```

Your browser should show a secure padlock ✅

You can also test using:

```bash
curl -I https://yourdomain.com
```

Should return: `HTTP/1.1 200 OK`

---

## ✅ Step 6: Auto-Renew SSL (Every 60 Days)

Check that auto-renewal works:

```bash
sudo certbot renew --dry-run
```

If successful, Certbot already installed a systemd timer:

```bash
sudo systemctl list-timers | grep certbot
```

---

## ✅ Optional: Check SSL Security

Use SSL test:

* 🔗 [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/)

---

# ✅ Final Directory Summary

| Path                                | Purpose             |
| ----------------------------------- | ------------------- |
| `/home/ubuntu/mysite/`              | Django project      |
| `/etc/nginx/sites-available/mysite` | Nginx config        |
| `/etc/letsencrypt/`                 | SSL certs & configs |

---

## ✅ Want to Go Further?

Would you like to:

* 🔐 Add security headers?
* ⚡️ Enable gzip/Brotli compression?
* 📦 Add PostgreSQL support?
* ⚙️ Set up CI/CD?

Let me know what you want next, and I’ll guide you.
