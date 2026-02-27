Perfect 👍 I’ll give you the **complete clean call steps again** for running your `start_update()` every 2 hours on an **Azure VM (Linux)** using:

* Django
* Celery
* Redis
* systemd (auto start)

No confusion — just clear steps.

---

# ✅ STEP 1 — SSH Into Azure VM

```bash
ssh username@your_vm_ip
```

---

# ✅ STEP 2 — Install Redis (Broker)

```bash
sudo apt update
sudo apt install redis-server -y
```

Enable + start Redis:

```bash
sudo systemctl enable redis
sudo systemctl start redis
```
or
```
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

Test:

```bash
redis-cli ping
```
or
```
sudo systemctl status redis-server
```

If working → `PONG`

---

# ✅ STEP 3 — Activate Your Virtual Environment

Go to your project folder:

```bash
cd /home/ubuntu/your_project
source venv/bin/activate
```

---

# ✅ STEP 4 — Install Required Packages

```bash
pip install celery==5.3.1
pip install django-celery-beat
pip install redis
```

---

# ✅ STEP 5 — Configure Celery

### Create: `MultiPass/celery.py`

```python
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'MultiPass.settings')

app = Celery('MultiPass')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()
```

---

### Update: `MultiPass/__init__.py`

```python
from .celery import app as celery_app
__all__ = ('celery_app',)
```

---

# ✅ STEP 6 — Update `settings.py`

Add:

```python
CELERY_BROKER_URL = 'redis://127.0.0.1:6379/0'
CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/0'

CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
```

Add to `INSTALLED_APPS`:

```python
"django_celery_beat",
```

---

# ✅ STEP 7 — Create Task

Create: `web/tasks.py`

```python
from celery import shared_task
from .kpler_update import Kpler

@shared_task
def update_vessels_task():
    kpler = Kpler()
    kpler.start_update()
```

---

# ✅ STEP 8 — Run Migrations

```bash
python manage.py migrate
```

Create admin user:

```bash
python manage.py createsuperuser
```

---

# ✅ STEP 9 — Create 2-Hour Schedule

Open browser:

```
http://your_vm_ip/admin
```

Go to:

### → Crontab Schedule

Set:

* Minute: `0`
* Hour: `*/2`
* Day of week: `*`
* Day of month: `*`
* Month: `*`

Save.

Then:

### → Periodic Tasks → Add

* Name: `Update vessels`
* Task: `web.tasks.update_vessels_task`
* Schedule: Select above
* Enabled: ✅

Save.

---

# ✅ STEP 10 — Test Manually (IMPORTANT)

Open Terminal 1:

```bash
celery -A MultiPass worker --loglevel=info
```

Open Terminal 2:

```bash
celery -A MultiPass beat --loglevel=info --scheduler django_celery_beat.schedulers:DatabaseScheduler
```

If no errors → Working.

Stop with `CTRL + C`.

---

# ✅ STEP 11 — Make It Auto Start (Production)

Create Celery Worker Service:

```bash
sudo nano /etc/systemd/system/celery.service
```

Paste:

```
[Unit]
Description=Celery Worker
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/your_project
ExecStart=/home/ubuntu/your_project/venv/bin/celery -A MultiPass worker --loglevel=info
Restart=always

[Install]
WantedBy=multi-user.target
```

---

Create Celery Beat Service:

```bash
sudo nano /etc/systemd/system/celery-beat.service
```

Paste:

```
[Unit]
Description=Celery Beat
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/your_project
ExecStart=/home/ubuntu/your_project/venv/bin/celery -A MultiPass beat --loglevel=info --scheduler django_celery_beat.schedulers:DatabaseScheduler
Restart=always

[Install]
WantedBy=multi-user.target
```

---

# ✅ STEP 12 — Enable + Start Services

```bash
sudo systemctl daemon-reload
sudo systemctl enable celery
sudo systemctl enable celery-beat
sudo systemctl start celery
sudo systemctl start celery-beat
```

Check status:

```bash
sudo systemctl status celery
sudo systemctl status celery-beat
```

Stop

```bash
sudo systemctl stop celery
sudo systemctl stop celery-beat
```

---

# 🎉 DONE

Now:

* Redis running
* Celery worker running
* Celery beat running
* `start_update()` runs every 2 hours
* Auto restart on reboot
* Production ready

---

If you want next, I can:

* Show how to log update results
* Prevent duplicate vessel inserts
* Optimize performance for large AIS data
* Add error retry logic

Tell me what you want next 👌
