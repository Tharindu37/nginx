Here’s a **step-by-step guide to deploy a Django MVT project on an Azure VM** (Ubuntu-based), using **Gunicorn + Nginx + PostgreSQL (or your DB)** setup.

---

### ✅ Prerequisites

* A Django project (MVT structure)
* Azure VM running Ubuntu (you have SSH access)
* Domain name (optional but good for production)
* PostgreSQL or MySQL setup (optional, or use SQLite for small apps)

---

### 🔧 Step-by-Step Deployment Process

---

### 1. **SSH into Azure VM**

```bash
ssh your_username@your_vm_ip
```

---

### 2. **Update and Install Dependencies**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip python3-dev libpq-dev nginx curl git -y
```

---

### 3. **Clone or Upload Your Django Project**

```bash
cd /var/www/
sudo git clone https://github.com/your-repo/your-django-app.git
cd your-django-app
```

Or use `scp` to upload project from local:

```bash
scp -r /path/to/your/project your_username@your_vm_ip:/var/www/
```

---

### 4. **Set Up a Virtual Environment**

```bash
sudo apt install python3-venv -y
python3 -m venv venv
source venv/bin/activate
```

Install project dependencies:

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

---

### 5. **Set Environment Variables & Django Settings**

* In `settings.py`, ensure:

```python
DEBUG = False
ALLOWED_HOSTS = ['your_vm_ip', 'your_domain.com']
```

* Configure static and media file paths properly.

* Optionally create a `.env` file and load variables using `python-decouple` or similar.

---

### 6. **Apply Migrations & Collect Static Files**

```bash
python manage.py migrate
python manage.py collectstatic --noinput
```

---

### 7. **Install & Configure Gunicorn**

Install Gunicorn:

```bash
pip install gunicorn
```

Test Gunicorn:

```bash
gunicorn --bind 0.0.0.0:8000 your_project_name.wsgi:application
```

Stop it and create a systemd service:

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

Paste:

```ini
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=your_username
Group=www-data
WorkingDirectory=/var/www/your-django-app
ExecStart=/var/www/your-django-app/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/var/www/your-django-app/gunicorn.sock your_project_name.wsgi:application

[Install]
WantedBy=multi-user.target
```

Start and enable the service:

```bash
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

---

### 8. **Configure Nginx**

```bash
sudo nano /etc/nginx/sites-available/your_project
```

Paste:

```nginx
server {
    listen 80;
    server_name your_domain.com your_vm_ip;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /var/www/your-django-app;
    }

    location /media/ {
        root /var/www/your-django-app;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/your-django-app/gunicorn.sock;
    }
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/your_project /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

### 9. **Allow Firewall Rules (if UFW is enabled)**

```bash
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
sudo ufw enable
```

---

### 10. **Optional: Configure HTTPS with Let's Encrypt**

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d your_domain.com
```

---

### ✅ Final Test

* Visit: `http://your_vm_ip` or `http://your_domain.com`
* Your Django site should be live.

---

Let me know if you're using **SQLite/PostgreSQL**, or want **Docker-based deployment**, or **Azure App Service** instead.


To **enable HTTPS (SSL/TLS) with auto-renewal** and use a **custom domain** on your Django project deployed in an **Azure Ubuntu VM**, follow this guide:

---

## 🔐 Part 1: Point Your Custom Domain to Azure VM

### 1. Go to Your Domain DNS Settings (e.g., GoDaddy, Namecheap, Cloudflare)

* Add an **A record**:

  * **Host**: `@` or `www`
  * **Value**: Your Azure VM **public IP address**
  * **TTL**: Default or 1 hour

✅ Wait up to 1–2 hours for DNS propagation.

---

## ⚙️ Part 2: Configure Nginx for Your Custom Domain

1. **Edit Nginx site config:**

```bash
sudo nano /etc/nginx/sites-available/your_project
```

Update `server_name` to include your domain:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /var/www/your-django-app;
    }

    location /media/ {
        root /var/www/your-django-app;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/your-django-app/gunicorn.sock;
    }
}
```

2. **Test and reload:**

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔒 Part 3: Install HTTPS with Let’s Encrypt SSL (Free)

### 1. Install Certbot

```bash
sudo apt install certbot python3-certbot-nginx -y
```

### 2. Run Certbot for HTTPS

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

* Follow prompts.
* Certbot will automatically:

  * Issue an SSL certificate.
  * Configure Nginx to redirect HTTP → HTTPS.
  * Set up auto-renewal.

---

## 🔁 Part 4: Auto-Renewal Cron Job (Optional — Certbot does this by default)

Verify it’s already scheduled:

```bash
sudo systemctl status certbot.timer
```

Or manually test renewal:

```bash
sudo certbot renew --dry-run
```

---

## 🔁 Optional: Redirect Non-WWW to WWW or Vice Versa

Example: redirect `yourdomain.com` to `www.yourdomain.com`:

Add this server block **above** the existing one:

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://www.yourdomain.com$request_uri;
}
```

---

## ✅ Final Checklist

* ✅ `https://yourdomain.com` works
* ✅ Auto-renewal scheduled (`certbot.timer`)
* ✅ `ALLOWED_HOSTS` in Django `settings.py`:

```python
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']
```

---

Would you like to include **Cloudflare**, **HTTP security headers**, or **force HSTS** for better security?



To serve **static files via a CDN** in your Django project (deployed on Azure VM), you need to:

---

## ✅ Overview Steps

1. **Upload static files to a CDN-compatible storage** (e.g., Azure Blob Storage, AWS S3, or Cloudflare R2)
2. **Set the CDN URL in Django settings**
3. **Collect static files to that storage**
4. **Ensure Nginx doesn’t serve them anymore**

---

## 🔹 Option 1: Use **Azure Blob Storage + Azure CDN**

This is ideal if you're already on Azure.

---

### 🔧 Step-by-Step

### 1. **Create Azure Storage Account and Blob Container**

* Go to [Azure Portal](https://portal.azure.com)
* Create:

  * A **Storage Account**
  * A **Blob Container** (e.g., `staticfiles`)
* Set container access level: **"Blob (anonymous read access)"**

---

### 2. **Create Azure CDN Endpoint**

* Go to **Azure CDN Profiles**
* Create one and link it to the **Blob Storage**
* You’ll get a **CDN endpoint URL** like:

```
https://yourproject.azureedge.net/
```

---

### 3. **Install Azure Storage Django Backend**

```bash
pip install django-storages azure-storage-blob
```

---

### 4. **Update `settings.py`**

```python
INSTALLED_APPS += ['storages']

AZURE_ACCOUNT_NAME = "your_storage_account"
AZURE_ACCOUNT_KEY = "your_storage_key"
AZURE_CONTAINER = "staticfiles"
AZURE_CUSTOM_DOMAIN = "yourcdnendpoint.azureedge.net"

STATICFILES_STORAGE = "storages.backends.azure_storage.AzureStorage"
STATIC_URL = f"https://{AZURE_CUSTOM_DOMAIN}/"
```

You can alternatively use:

```python
STATICFILES_STORAGE = 'storages.backends.azure_storage.AzureStorage'
DEFAULT_FILE_STORAGE = 'storages.backends.azure_storage.AzureStorage'
```

---

### 5. **Collect Static Files to Azure Blob**

```bash
python manage.py collectstatic
```

---

### 6. **Verify**

* Open your browser and load:

  ```
  https://yourcdnendpoint.azureedge.net/css/style.css
  ```
* Or load your site and inspect the static file URLs → should use the CDN.

---

## ✅ Optional: Cache Busting

Django provides cache-busting with hashed filenames via:

```python
STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.ManifestStaticFilesStorage'
```

Use it together with Azure if needed for aggressive caching.

---

## 🔄 Remove Nginx Static Serving

If static files are now on CDN, you can remove or comment the Nginx `location /static/` block.

---

## 💡 Alternative: Use **Cloudflare CDN** on top of your domain

If your static files are served from your domain (e.g., `https://yourdomain.com/static/`), just:

1. Point domain to **Cloudflare DNS**
2. Enable **caching rules** for `/static/*` and `/media/*`

It's simpler, no change to your storage or Django settings required.

---

Let me know which CDN provider you're using (Azure, Cloudflare, AWS S3, or others), and I’ll tailor the config for that.
