# 🚀 n8n Installation & Configuration Guide (Custom Domain + HTTPS)

This guide walks you through installing and running **n8n** on a small VPS using `npm`, with reverse proxy, HTTPS, and basic authentication.


---

## 🔧 Server Requirements

- **OS**: Ubuntu (or Debian-based)
- **Specs**: 1 vCore, 2 GB RAM, 20 GB Disk
- **Domain**: Pointed to your server's IP (e.g., `n8n.yourdomain.com`)

---

## 📦 Step 1: Install Dependencies

```bash
# Update packages
sudo apt update

# Install Node.js (if not already)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install n8n
sudo npm install -g n8n

# Install PM2 (to run n8n in background)
sudo npm install -g pm2

# Install NGINX (reverse proxy)
sudo apt install nginx

# Install Certbot (for SSL)
sudo apt install certbot python3-certbot-nginx
````

---

## 📁 Step 2: Set Up n8n with PM2

```bash
mkdir ~/n8n
cd ~/n8n
```

### Create `.env` file:

```bash
vi .env
```

Paste the following and update your values if you want to use username/password for login instead google oath2:

```env
N8N_PORT=5678
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=yourusername
N8N_BASIC_AUTH_PASSWORD=yourpassword
N8N_HOST=n8n.yourdomain.com
N8N_PROTOCOL=https
WEBHOOK_URL=https://n8n.yourdomain.com/
N8N_EDITOR_BASE_TITLE=My Awesome n8n
N8N_EDITOR_BASE_URL=https://n8n.yourdomain.com
TZ=Asia/Tehran
VUE_APP_URL_BASE_API=https://n8n.yourdomain.com/
```

### Create `start-n8n.sh` script:

```bash
vi start-n8n.sh
```

Paste:

```bash
#!/bin/bash
export $(grep -v '^#' .env | xargs)
n8n
```

Make executable:

```bash
chmod +x start-n8n.sh
```

### Run with PM2:

```bash
pm2 start ./start-n8n.sh --name n8n
pm2 save
pm2 startup   # Follow the output command to enable auto-start on reboot
```

### ✅ 2. **How to restart n8n using PM2?**

```bash
pm2 restart n8n
```

Other helpful PM2 commands:

| Command           | What it does                              |
| ----------------- | ----------------------------------------- |
| `pm2 list`        | Show running apps                         |
| `pm2 logs n8n`    | Live log output for n8n                   |
| `pm2 restart n8n` | Restart n8n                               |
| `pm2 stop n8n`    | Stop n8n                                  |
| `pm2 delete n8n`  | Remove n8n from PM2 (won’t start on boot) |
| `pm2 save`        | Save current process list                 |
| `pm2 reload n8n`  | Zero-downtime restart (like restart)      |

---

## 🌐 Step 3: Set Up NGINX Reverse Proxy

### Create config:

```bash
sudo vi /etc/nginx/sites-available/n8n
```

Paste and update domain:

```nginx
server {
    listen 80;
    server_name n8n.yourdomain.com;

    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Enable site:

```bash
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔒 Step 4: Enable HTTPS with Let's Encrypt

```bash
sudo certbot --nginx -d n8n.yourdomain.com
```

- Choose to redirect HTTP → HTTPS.
    
- Auto-renewal is enabled by default.
    

Test renewal:

```bash
sudo certbot renew --dry-run
```

### ✨ After doing SSL stuffs, add these lines to the `location /` part into the `/etc/nginx/sites-available/n8n` file:

```bash
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

---

## ✅ Done!

Now access your secure n8n instance:

🌐 [https://n8n.yourdomain.com](https://n8n.yourdomain.com/)  
🔐 Login with the basic auth credentials you set in `.env`.

---

## 🧠 Tips

- Monitor memory: `htop` or `pm2 monit`
    
- Restart n8n: `pm2 restart n8n`
    
- View logs: `pm2 logs n8n`
    
- Update n8n: `npm install -g n8n && pm2 restart n8n`
    

---

## 🛡 Optional Hardening

- Use firewall to block public IP access
    
- Limit file uploads in NGINX
    
- Use `.env` for secret management
    

---

## Backup Full N8N Files

For giving a full backup of all of these in N8N:
- Workflows 
- Credentials
- Execution Logs
- Encryption Key
You just need to zip the `.n8n` directory into your home directory.

```bash 
tar -czvf n8n_full_backup.tar.gz ~/.n8n
```

---

## Restoring Full N8N Files

Just return the backed up file of N8N into the home directory and have fun

```bash 
tar -xzvf n8n_full_backup.tar.gz -C ~/
```