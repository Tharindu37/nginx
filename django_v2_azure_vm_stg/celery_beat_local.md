Perfect 👌 Let’s test everything **locally on your computer** before deploying to Azure VM.
This is the safest way to confirm Celery + Beat + Redis are working correctly.

I’ll give you a clean step-by-step guide.

---

# ✅ WHAT YOU NEED LOCALLY

You need:

* Python virtual environment
* Django project
* Redis installed locally
* Celery configured

---

# ✅ STEP 1 — Install Redis Locally

## 🔹 On Ubuntu / Linux

```bash
sudo apt update
sudo apt install redis-server -y
sudo systemctl start redis
```

Test:

```bash
redis-cli ping
```

Expected output:

```
PONG
```

---

## 🔹 On Windows

Download Redis (use WSL recommended)
OR install using Docker:

```bash
docker run -d -p 6379:6379 redis
```

---

## 🔹 On Mac

```bash
brew install redis
brew services start redis
```

---

# ✅ STEP 2 — Activate Your Virtual Environment

```bash
cd your_project_folder
source venv/bin/activate   # Linux/Mac
venv\Scripts\activate     # Windows
```

---

# ✅ STEP 3 — Install Celery + Beat

```bash
pip install celery==5.3.1
pip install django-celery-beat
pip install redis
```

---

# ✅ STEP 4 — Confirm settings.py

Make sure you have:

```python
CELERY_BROKER_URL = 'redis://127.0.0.1:6379/0'
CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/0'

CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
```

And:

```python
INSTALLED_APPS += [
    "django_celery_beat",
]
```

---

# ✅ STEP 5 — Run Django Migrations

```bash
python manage.py migrate
```

---

# ✅ STEP 6 — Test Celery Worker

Open Terminal 1:

```bash
celery -A MultiPass worker --loglevel=info
```
or
```bash
celery -A MultiPass worker --pool=solo --loglevel=info
```
You should see:

```
Connected to redis://127.0.0.1:6379/0
```

If you see errors → tell me.

---

# ✅ STEP 7 — Test Celery Beat

Open Terminal 2:

```bash
celery -A MultiPass beat --loglevel=info --scheduler django_celery_beat.schedulers:DatabaseScheduler
```

You should see:

```
Scheduler: Sending due task ...
```

---

# ✅ STEP 8 — Test Manual Task Call (VERY IMPORTANT)

Open Django shell:

```bash
python manage.py shell
```

Run:

```python
from web.tasks import update_vessels_task
update_vessels_task.delay()
```

If working:

* Worker terminal will show task received
* Your `start_update()` will execute

If nothing happens → configuration problem.

---

# ✅ STEP 9 — Test 2-Hour Schedule (Quick Test Mode)

Instead of waiting 2 hours, change your schedule to:

Crontab:

```
Minute: */1
Hour: *
```

This runs every 1 minute.

Watch:

Worker terminal logs → task runs automatically.

Once confirmed → change back to:

```
Minute: 0
Hour: */2
```

---

# ✅ HOW TO CONFIRM EVERYTHING WORKS

You should see in worker terminal:

```
Task web.tasks.update_vessels_task received
Task succeeded
```

If you want to see logs from inside `start_update()`, add:

```python
import logging
logger = logging.getLogger(__name__)

logger.info("Starting vessel update...")
```

---

# 🚀 Final Local Test Checklist

| Component               | Status |
| ----------------------- | ------ |
| Redis running           | ✅      |
| Django running          | ✅      |
| Celery worker connected | ✅      |
| Celery beat running     | ✅      |
| Manual delay() works    | ✅      |
| Auto schedule works     | ✅      |

---

If you want, next I can:

* Show how to debug if task is not running
* Add retry if API fails
* Add logging to file
* Show how to monitor Celery properly

Tell me what you want next 👌
