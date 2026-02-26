You’re still getting:

```
SSL connection error:
tls_process_server_certificate: certificate verify failed
```

This means **the SSL handshake is failing before authentication**.
Since you're using **Microsoft Azure** Database for MySQL, let’s fix this the correct production way.

---

# 🚨 Most Common Real Cause (On Ubuntu Azure VM)

On many Azure VMs:

* OpenSSL CA store is outdated
* System does NOT trust DigiCert root
* Even if you pass `ssl: { ca: ... }`, verification fails

So instead of manually passing CA file, we fix the **system CA trust store**.

---

# ✅ PERMANENT FIX (Recommended)

## 🔹 Step 1 — Install DigiCert Root to System

On your VM:

```bash
sudo apt update
sudo apt install ca-certificates -y
```

Download certificate:

```bash
wget https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem
```

Move it to trusted store:

```bash
sudo cp DigiCertGlobalRootCA.crt.pem /usr/local/share/ca-certificates/DigiCertGlobalRootCA.crt
```

Update system certificates:

```bash
sudo update-ca-certificates
```

You should see:

```
1 added, 0 removed; done.
```

---

## 🔹 Step 2 — REMOVE SSL OPTIONS From Django

After adding to system trust, change your Django config to:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'bluepass_db',
        'USER': 'bluepass_mysql_admin@bluepassmysqldb',
        'PASSWORD': 'NEW_PASSWORD',
        'HOST': 'bluepassmysqldb.mysql.database.azure.com',
        'PORT': '3306',
        'OPTIONS': {
            'ssl': {'ssl_mode': 'REQUIRED'}
        }
    }
}
```

⚠ Do NOT pass custom `ca` path anymore.

---

## 🔹 Step 3 — Restart Everything

```bash
sudo systemctl restart gunicorn
sudo systemctl restart nginx
```

---

# 🔎 If It STILL Fails

Run this:

```bash
openssl s_client -connect bluepassmysqldb.mysql.database.azure.com:3306 -starttls mysql
```

If you see:

```
Verify return code: 0 (ok)
```

→ SSL trust is fixed.

If not → your VM OpenSSL is outdated.

---

# 🎯 Why This Works

Instead of Django trusting one certificate file,
the **entire OS trusts DigiCert**.

This is the correct enterprise solution for Azure production.

---

# 🔥 Quick Emergency Restore (If Site Is Down)

Temporarily disable verification:

```python
'OPTIONS': {
    'ssl': {'check_hostname': False}
}
```

Restart → site will work.

Then fix properly.

---

If you want, tell me:

* Ubuntu version (`lsb_release -a`)
* Is this Azure MySQL Flexible Server?

I’ll give you the exact final configuration for your server.
