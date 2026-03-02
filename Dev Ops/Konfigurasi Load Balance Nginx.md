
## 1. Round Robin
Nginx akan mengirim request ke server 3000 → 3001 → 3002 secara bergiliran.
```conf
# Load balancer upstream
upstream app2_backend {
    # Round robin (default) + failover
    server 127.0.0.1:8001 max_fails=3 fail_timeout=10s;
    server 127.0.0.1:8002 max_fails=3 fail_timeout=10s;
    server 127.0.0.1:8003 max_fails=3 fail_timeout=10s;
}

server {
    listen 80;
    server_name app2.abc.my.id;
    
    location / {
        proxy_pass http://app2_backend;   # diarahkan ke load balancer

        # Websocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Forward original headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Optional bypass
        proxy_cache_bypass $http_upgrade;

        # Failover cepat ke server selanjutnya
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_next_upstream_tries 3;
    }
}
```

## 2. Least Connections
Akan mengirim request ke server yang **sedang memiliki koneksi paling sedikit**.
```
upstream backend_app {
    least_conn;
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}
```

## 3. IP Hash (Sticky by IP)
User yang punya IP yang sama akan selalu diarahkan ke server yang sama.
```
upstream backend_app {
    ip_hash;
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
}
```


## Konfigurasi Weighted Load Balancing (30% : 70%)
- **30% → server 3001**
- **70% → server 3002**
Ini berarti bobotnya:
- 3001 = **3**
- 3002 = **7**
```
upstream traffic_split {
    server 127.0.0.1:3001 weight=3 max_fails=3 fail_timeout=10s;
    server 127.0.0.1:3002 weight=7 max_fails=3 fail_timeout=10s;
}

server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://traffic_split;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_next_upstream_tries 3;
    }
}
```

### `max_fails`

`max_fails` = **berapa kali Nginx membiarkan backend gagal sebelum dianggap "down".**
**Gagal** itu biasanya:
- backend tidak merespons
- timeout
- connection refused
- error seperti 500/502/503/504 (tergantung konfigurasi)
#### Contoh:
```
server 127.0.0.1:3001 max_fails=3;
```
Artinya:
> Kalau server 3001 gagal **3 kali**, Nginx akan menganggap server itu **rusak/down** dan tidak akan dipakai sementara.


### `fail_timeout`
`fail_timeout` = **waktu server dianggap down** setelah melewati `max_fails`.
Selama waktu ini:
- Nginx **tidak akan mengirim request** ke server itu
- Nginx **skip** server itu dan lanjut ke server lainnya
#### Contoh:
```
server 127.0.0.1:3001 max_fails=3 fail_timeout=10s;
```

Artinya:

> Jika server 3001 gagal 3 kali, maka selama **10 detik** dianggap “down”, lalu setelah 10 detik Nginx akan mencoba pakai server itu lagi.


### `weight`
`weight` adalah parameter di Nginx yang digunakan untuk **mengatur porsi distribusi traffic** ke setiap server dalam load balancer.

Dengan kata lain:
> **weight = seberapa besar kemungkinan sebuah server menerima request.**

Semakin besar nilai weight, semakin banyak request yang dikirim ke server tersebut.