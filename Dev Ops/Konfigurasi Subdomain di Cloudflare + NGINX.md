

## DNS Cloudflare

Login ke Cloudflare Dashboard, pilih domain abc.my.id, masuk ke DNS Records.

Tambahkan record berikut:

### App1
- Type: A
- Name: app1
- IPv4: 188.166.244.175
- Proxy: Proxied
- TTL: Auto

### App2
- Type: A
- Name: app2
- IPv4: 188.166.244.175
- Proxy: Proxied
- TTL: Auto

### Photoboost
- Type: A
- Name: photoboost
- IPv4: 188.166.244.175
- Proxy: Proxied
- TTL: Auto

## Konfigurasi NGINX

### App1 (Port 8000)

Buat file /etc/nginx/sites-available/app1.abc.my.id
```
server {
    listen 80;
    server_name app1.abc.my.id;
    
    location / {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### App2 (Port 8001)

Buat file /etc/nginx/sites-available/app2.abc.my.id
```
server {
    listen 80;
    server_name app2.abc.my.id;
    
    location / {
        proxy_pass http://localhost:8001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### SuperApp (Static atau Port Spesifik)

Buat file /etc/nginx/sites-available/superapp.abc.my.id

Static files:
```
server {
    listen 80;
    server_name superapp.abc.my.id;
    root /var/www/superapp;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

Atau jika pakai port (contoh 3000):
```
server {
    listen 80;
    server_name superapp.abc.my.id;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Enable Site
```bash
sudo ln -s /etc/nginx/sites-available/app1.abc.my.id /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/app2.abc.my.id /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/superapp.abc.my.id /etc/nginx/sites-enabled/
```

## Test dan Restart NGINX
```bash
sudo nginx -t
sudo systemctl restart nginx
```

## Install SSL

Install Certbot:
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

Setup SSL:
```bash
sudo certbot --nginx -d app1.abc.my.id -d app2.abc.my.id -d superapp.abc.my.id
```

## Verifikasi

Cek aplikasi running:
```bash
sudo netstat -tulpn | grep 8000
sudo netstat -tulpn | grep 8001
```

Test akses:
```bash
curl http://localhost:8000
curl http://localhost:8001
```

Akses via browser:
- https://app1.abc.my.id
- https://app2.abc.my.id
- https://superapp.abc.my.id

## Troubleshooting

Cek log error:
```bash
sudo tail -f /var/log/nginx/error.log
```

Cek status NGINX:
```bash
sudo systemctl status nginx
```

Reload konfigurasi tanpa downtime:
```bash
sudo nginx -s reload
```