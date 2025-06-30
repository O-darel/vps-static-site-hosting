# 🚀 Deploy a Static HTML Page to VPS with Python HTTP Server or Nginx

This guide shows how to:

- Copy files from your **local machine** to a **VPS**
- Install and configure **Nginx**
- OR run a lightweight **Python HTTP server**
- Leave the Python server running using `nohup`
- Check logs after

---

## 🧱 1. Copy Local HTML File to VPS

From your local terminal:

```bash
scp /path/to/your/index.html root@<VPS_IP>:/var/www/html/
```

Replace:

- `/path/to/your/index.html` → with your actual local file path
- `<VPS_IP>` → with your VPS IP address

---

## 🔐 2. Log in to the VPS

```bash
ssh root@<VPS_IP>
```

---

## 🌐 3. Install Nginx on the VPS

```bash
sudo apt update
sudo apt install nginx -y
```

---

## ▶️ 4. Start and Enable Nginx

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

Visit your IP to test:

```text
http://<VPS_IP>
```

You should see the Nginx welcome page.

---

## 🔄 5. Replace the Default Web Page

```bash
sudo mv /var/www/html/index.html /var/www/html/index.old.html
sudo mv /path/to/uploaded/index.html /var/www/html/index.html
```

Or just re-upload from local again using `scp`.

---

## 🐍 6. Run Python HTTP Server (Alternative to Nginx)

Navigate to the directory where your HTML is (e.g., `/var/www/html`):

```bash
cd /var/www/html
```

Run the Python server **in background with logs**:

```bash
nohup python3 -m http.server 3000 > server.log 2>&1 &
```

- Runs on port `3000`
- Logs go to `server.log`
- Runs even after you log out

---

## 🔎 7. Check Server Logs

```bash
cat server.log
```

Or to watch live:

```bash
tail -f server.log
```

---

## 🛑 8. Stop the Python Server (If Needed)

Find the process:

```bash
ps aux | grep http.server
```

Then kill it:

```bash
kill <PID>
```

---

## ✅ You're Done!

- Access your Python server at: `http://<VPS_IP>:3000`
- Or your Nginx site at: `http://<VPS_IP>/`

**Next up ..** :

1. ✅ Domain setup
2. 🔒 HTTPS with Let’s Encrypt
3. 📁 Serving from a custom directory

You can combine it with the previous guide or keep it separate.

---

# 🌍 Advanced VPS Web Hosting Setup

---

## ✅ 1. Point a Domain to Your VPS

You’ll need to:

- Own a domain (e.g. `example.com`)
- Have access to your domain DNS settings

### 📌 Steps:

1. Go to your domain registrar’s dashboard (e.g. Namecheap, GoDaddy, Freenom).
2. Add an **A record**:

| Type | Name | Value         |
| ---- | ---- | ------------- |
| A    | @    | `your_vps_ip` |
| A    | www  | `your_vps_ip` |

> Propagation may take 5–30 minutes.

---

## 🔒 2. Enable HTTPS with Let’s Encrypt + Nginx

### 📦 Install Certbot

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

### ⚙️ Configure Nginx for Your Domain

Create a config file:

```bash
sudo nano /etc/nginx/sites-available/example.com
```

Paste:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/example;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable the site and reload:

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 🪄 Request an HTTPS Certificate

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Choose option to redirect HTTP to HTTPS.

---

## 🔁 3. Serve Files from a Custom Directory

Let’s say you want to serve your HTML files from `/home/ubuntu/my-site`.

### 📁 1. Create and Set Permissions

```bash
mkdir -p /home/ubuntu/my-site
chmod -R 755 /home/ubuntu/my-site
```

### 📤 2. Upload Files from Local

```bash
scp -r /local/path/to/site/* root@<VPS_IP>:/home/ubuntu/my-site/
```

### 🔄 3. Update Nginx Config

Edit Nginx config for your domain:

```nginx
root /home/ubuntu/my-site;
```

Then:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Now your site is served from that custom folder.

---

## 🎉 All Done!

- Visit: `https://example.com`
- Files are served securely from `/home/ubuntu/my-site`
- You can now:

  - Upload new files using `scp`
  - Monitor logs with `journalctl` or `tail -f /var/log/nginx/access.log`
  - Renew certs automatically with:

```bash
sudo systemctl enable certbot.timer
```

---
