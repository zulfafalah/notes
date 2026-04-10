 ---

## 1. Amazon EC2 = VPS

Merupakan sebuah VPS / server tradisional, cocok dipakai untuk backend monolith dan legacy system.
-  full control atas OS, dependencies, dan konfigurasi server
- Harus manage sendiri: update, security patch, monitoring
- Billing per jam atau per detik (tergantung OS)

**Use case:**
- Backend monolith (Laravel, Django, Spring Boot)
- Legacy system yang belum bisa di-containerize
- Aplikasi yang butuh konfigurasi server khusus

---

## 2. AWS Lambda = Serverless

Layanan serverless milik AWS, user tanpa manage server. Cocok dipakai untuk deploy API kecil atau microservice. AWS Lambda itu sesimple platform yang digunakan untuk deploy single function.
- Auto-scale otomatis
- Billing per invocation (bayar kalau dipakai)
- Batas waktu eksekusi maksimal 15 menit

**Use case:**
- REST API ringan
- Event-driven (trigger dari S3, DynamoDB, SQS, dll)
- Scheduled task / cron job

---

## 3. Amazon ECS / Amazon EKS

Untuk Docker / container.
- **ECS** → lebih simple, managed container orchestration milik AWS
- **EKS** → Kubernetes, enterprise banget, cocok untuk tim yang sudah familiar dengan K8s

**Use case:**
- Aplikasi yang sudah di-containerize dengan Docker
- Microservice architecture
- EKS untuk workload skala besar dengan tim DevOps dedicated

---

## 4. Amazon S3

Penyimpanan file (gambar, video, dokumen).
- Object storage, bukan file system
- Highly durable: 99.999999999% (11 nines)
- Bisa dikonfigurasi sebagai static website hosting

**Use case:**
- Upload user (foto profil, dokumen)
- Backup & archiving
- Static website (HTML, CSS, JS)
- CDN origin untuk CloudFront

---

## 5. Amazon RDS

Managed relational database: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server.
- AWS yang handle backup, patching, failover
- Support Multi-AZ untuk high availability
- Bisa auto-scale storage

**Use case:**
- Aplikasi bisnis
- E-commerce
- ERP / CRM

---

## 6. Amazon DynamoDB

NoSQL (key-value & document).
- Super scalable, single-digit millisecond latency
- Serverless, tidak perlu manage infrastruktur
- Support TTL untuk auto-expire data

**Use case:**
- High traffic apps
- Realtime system
- Session management
- Gaming leaderboard

---

## 7. Application Load Balancer (ALB)

Mendistribusikan traffic ke beberapa instance EC2 / container.
- Bekerja di Layer 7 (HTTP/HTTPS)
- Support routing berdasarkan path, host, header
- Terintegrasi dengan Target Groups dan Health Check

### Komponen Penting

|Komponen|Fungsi|
|---|---|
|**Listener**|Mendengarkan traffic di port tertentu (misal port 80, 8080)|
|**Target Group**|Kumpulan instance / container yang menerima traffic|
|**Health Check**|Cek apakah target masih sehat sebelum dikirim traffic|
|**Security Group**|Firewall yang mengontrol traffic masuk/keluar ke LB dan instance|

### Status Target Group

|Status|Artinya|
|---|---|
|**Healthy**|Instance normal, menerima traffic|
|**Unhealthy**|Health check gagal, tidak menerima traffic|
|**Unused**|Instance berada di AZ yang tidak terdaftar di LB|
|**Initial**|Baru didaftarkan, sedang proses health check|
|**Draining**|Sedang di-deregister, menunggu koneksi aktif selesai|

### Step-by-Step: Setup EC2 + Application Load Balancer

#### Step 1 — Launch EC2 Instance

1. Pergi ke **EC2 → Instances → Launch instances**
2. Pilih AMI (misal Ubuntu 22.04)
3. Pilih instance type (misal `t2.micro` untuk testing)
4. Di bagian **Network settings**:
    - Pilih VPC yang akan dipakai
    - Pilih **Subnet** — catat AZ-nya (misal `ap-southeast-1a`), ini penting nanti
    - Enable **Auto-assign public IP** jika butuh akses langsung
5. Di bagian **Firewall (Security Groups)**, buat SG baru dengan inbound rules:
    - SSH → port 22 → My IP (untuk remote)
    - Custom TCP → port aplikasi kamu (misal 8080) → 0.0.0.0/0
6. Launch instance

> ⚠️ **Catat Availability Zone (AZ) instance kamu** — Load Balancer harus mencakup AZ yang sama agar traffic bisa sampai ke instance

---

#### Step 2 — Buat Target Group

Target Group adalah kumpulan instance yang akan menerima traffic dari ALB.

1. Pergi ke **EC2 → Target Groups → Create target group**
2. Pilih target type: **Instances**
3. Isi konfigurasi:
    - **Target group name**: nama bebas (misal `my-app-tg`)
    - **Protocol**: HTTP
    - **Port**: port aplikasi kamu (misal `8080`)
    - **VPC**: pilih VPC yang sama dengan EC2
4. Di bagian **Health checks**:
    - **Protocol**: HTTP
    - **Path**: endpoint yang return HTTP 200 (misal `/health` atau `/ping`)
    - **Success codes**: `200`
5. Klik **Next** → centang instance yang ingin didaftarkan → klik **Include as pending below**
6. Klik **Create target group**

> 💡 Pastikan health check path benar-benar return **HTTP 200**, cek dulu di instance:
> 
> ```bash
> curl -v http://localhost:8080/health
> ```

---

#### Step 3 — Buat Application Load Balancer

1. Pergi ke **EC2 → Load Balancers → Create load balancer**
2. Pilih **Application Load Balancer**
3. Isi konfigurasi:
    - **Name**: nama bebas (misal `my-alb`)
    - **Scheme**: Internet-facing (untuk akses dari internet)
    - **IP address type**: IPv4
4. Di bagian **Network mapping**:
    - Pilih VPC yang sama
    - **Centang minimal 2 AZ** — dan pastikan AZ tempat instance kamu berada ikut dicentang
    - Pilih subnet untuk masing-masing AZ
5. Di bagian **Security groups**:
    - Buat atau pilih SG untuk ALB
    - Pastikan SG ini punya inbound rule untuk semua port yang akan di-listen (misal port 80, 8080)
6. Di bagian **Listeners and routing**:
    - **Protocol**: HTTP, **Port**: 8080
    - **Default action**: Forward to → pilih target group yang dibuat di Step 2
7. Klik **Create load balancer**

> ⚠️ **Jangan lupa**: AZ di Load Balancer harus mencakup AZ tempat instance kamu berada. Kalau beda AZ, status target akan **Unused** dan traffic tidak akan diteruskan.

---

#### Step 4 — Verifikasi

Setelah ALB aktif, cek status target group:

1. Pergi ke **EC2 → Target Groups → pilih target group kamu**
2. Tab **Targets** → lihat kolom **Health status**

|Status yang diharapkan|Artinya|
|---|---|
|✅ Healthy|Siap menerima traffic|
|❌ Unhealthy|Health check gagal — cek path & aplikasi|
|⚪ Unused|AZ tidak terdaftar di LB — tambahkan AZ di Network Mapping|
|🔄 Initial|Sedang proses health check, tunggu sebentar|

Kalau sudah **Healthy**, akses via DNS ALB:

```
http://<dns-alb>:8080
```

DNS bisa dilihat di halaman detail Load Balancer, kolom **DNS name**.

### Arsitektur Security Group yang Benar

```
Internet → [SG Load Balancer] → ALB → [SG Instance] → EC2
```

- **SG Load Balancer**: Allow inbound dari internet (0.0.0.0/0) di port listener (80, 443, 8080, dll)
- **SG Instance**: Allow inbound dari SG Load Balancer (bukan 0.0.0.0/0) di port aplikasi

> 💡 **Tips**: Kalau akses langsung ke IP EC2 bisa tapi lewat ALB tidak bisa, kemungkinan besar masalah di Security Group Load Balancer — port listener belum di-allow.

---

## Referensi Cepat: Pilih Layanan yang Tepat

|Kebutuhan|Layanan|
|---|---|
|Server penuh (full control)|EC2|
|Deploy function / API ringan|Lambda|
|Docker / container|ECS atau EKS|
|Simpan file / media|S3|
|Database relasional|RDS|
|Database NoSQL scalable|DynamoDB|
|Distribusi traffic|ALB / NLB|