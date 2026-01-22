
## Deploying MERN Stack Project on Hostinger VPS




- Preparing the VPS Environment
- Setting Up the MongoDB Database
- Deploying the Express and Node.js Backend
- Deploying the React Frontends
- Configuring Nginx as a Reverse Proxy
- Setting Up SSL Certificates
### 1. Preparing the VPS Environment

#### Get you VPS Hosting here : [Hostinger VPS](https://hostinger.pk?REFERRALCODE=AJGHASEEBLEL)

Log in to Your VPS in Terminal 

```bash
 ssh root@your_vps_ip
```

Update and Upgrade Your System

```bash
  sudo apt update
```
```bash
  sudo apt upgrade -y
```

Install Node.js and npm ( if not pre-installed)

```bash
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```
```bash
  \. "$HOME/.nvm/nvm.sh"
```
```bash
  nvm install 22
```
Install Git 
```bash
  sudo apt install -y git
```


###  2. Setting Up the MongoDB Database

If you want to setup MongoDB on VPS Follow this Guide: [click here](https://github.com/haseeb-012/notes/blob/main/MongoDB_Setup_on_VPS.md)

### 3. Deploying the Express and Node.js Backend

Clone Your Backend Repository

```bash
 mkdir /var/www
```

```bash
 cd /var/www
```
```bash
 git clone https://github.com/yourusername/your-repo.git
```
```bash
 cd your-repo/backend
```

Install Dependencies

```bash
 npm install
```
Create .env file & configure Environment Variables

```bash
 nano .env
```

add environment variables then save and exit (Ctrl + X, then Y and Enter).


Installing pm2 to Start Backend

```bash
 npm install -g pm2
```
```bash
 pm2 start server.js --name project-backend
```
Start Backend on startup
```bash
 pm2 startup
```
```bash
 pm2 save
```
Allowing backend port in firewall 

```bash
 sudo ufw status
```
If firewall is disable then enable it using 
```bash
 sudo ufw enable
```
```bash
 sudo ufw allow 'OpenSSH'
```
```bash
 sudo ufw allow 4000
```

### 4. Deploying the React Frontends

Creating Build of React Applications
```bash
 cd path-to-your-first-react-app
```
```bash
 npm install
```
If you have ".env" file in your project

Create .env file and paste the variables
```bash
 nano .env
```
Create build of project
```bash
 npm run build
```

Repeat for the second or mulitiple React app.

Install Nginx

```bash
 sudo apt install -y nginx
```

adding Nginx in firewall

```bash
 sudo ufw status
```
```bash
 sudo ufw allow 'Nginx Full'
```


Configure Nginx for React Frontends


```bash
 nano /etc/nginx/sites-available/yourdomain1.com.conf
```

```bash
 server {
    listen 80;
    server_name yourdomain1.com www.yourdomain1.com;

    location / {
        root /var/www/your-repo/frontend/dist;
        try_files $uri /index.html;
    }
}
```
Save and exit (Ctrl + X, then Y and Enter).

Create a similar file for the second or multiple React app.

```bash
 nano /etc/nginx/sites-available/yourdomain2.com.conf
```

```bash
server {
    listen 80;
    server_name yourdomain2.com www.yourdomain2.com;

    location / {
        root /var/www/react-app-2/dist;
        try_files $uri /index.html;
    }
}
```

Create symbolic links to enable the sites.

```bash
ln -s /etc/nginx/sites-available/yourdomain1.com.conf /etc/nginx/sites-enabled/
```

```bash
ln -s /etc/nginx/sites-available/yourdomain2.com.conf /etc/nginx/sites-enabled/
```

Test the Nginx configuration for syntax errors.

```bash
nginx -t
```

```bash
systemctl restart nginx
```

### 5. Configuring Nginx as a Reverse Proxy

Update Backend Nginx Configuration

```bash
nano /etc/nginx/sites-available/api.yourdomain.com.conf
```
```bash
server {
    listen 80;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://localhost:4000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Create symbolic links to enable the sites.

```bash
ln -s /etc/nginx/sites-available/api.yourdomain.com.conf /etc/nginx/sites-enabled/
```

Restart nginx

```bash
systemctl restart nginx
```

### Connect Domain Name with Website

Point all your domain & sub-domain on VPS IP address by adding DNS records in your domain manager 

Now your website will be live on domain name

### 6. Setting Up SSL Certificates 

Install Certbot

```bash
sudo apt install -y certbot python3-certbot-nginx
```

Obtain SSL Certificates

```bash
certbot --nginx -d yourdomain1.com -d www.yourdomain1.com -d yourdomain2.com -d api.yourdomain.com
```

Verify Auto-Renewal

```bash
certbot renew --dry-run
```


## 7. Production-ready Nginx Configuration (HTTPS)

### 7.1 Frontend (React) Config

```nginx
# Redirect HTTP → HTTPS
server {
    listen 80;
    server_name frontend.example.com www.frontend.example.com;
    return 301 https://$host$request_uri;
}

# HTTPS frontend
server {
    listen 443 ssl;
    server_name frontend.example.com www.frontend.example.com;

    ssl_certificate /etc/letsencrypt/live/frontend.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/frontend.example.com/privkey.pem;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        root /var/www/react-app/dist;
        access_log off;
        expires 30d;
    }

    # Serve SPA
    location / {
        root /var/www/react-app/dist;
        index index.html;
        try_files $uri /index.html;
    }
}
```

---

### 7.2 Backend (Express API) Config

```nginx
# Redirect HTTP → HTTPS
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$host$request_uri;
}

# HTTPS backend reverse proxy
server {
    listen 443 ssl;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

---

## 8. Final Steps

1. Test Nginx configuration:

```bash
sudo nginx -t
```

2. Reload Nginx:

```bash
sudo systemctl reload nginx
```

3. Ensure PM2 is running backend:

```bash
pm2 list
```

4. Firewall: close backend port from public if desired

```bash
sudo ufw deny 5000
```

---

✅ **Now the MERN stack is fully production-ready with HTTPS, redirects, caching, and security headers.**

---


If you still need help in deployment:

Contact us on email : haseebsajjad666@gmail.com
