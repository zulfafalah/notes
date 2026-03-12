## Overview

Dalam Linux, teknologi container seperti Docker dan Kubernetes dibangun dari dua fitur utama kernel:
- **Namespace** → mengisolasi resource
- **Cgroup (Control Group)** → membatasi penggunaan resource

Keduanya memungkinkan proses berjalan seolah-olah berada di sistem yang terpisah, walaupun sebenarnya masih menggunakan **kernel Linux yang sama**.

---

# 1. Namespace

## Definisi

Namespace adalah fitur pada kernel Linux yang digunakan untuk **mengisolasi resource sistem** sehingga sekelompok proses hanya dapat melihat resource miliknya sendiri.
Tanpa namespace:
- semua proses dapat melihat semua proses lain
- semua proses berbagi network yang sama
- semua proses berbagi filesystem yang sama

Dengan namespace:
- setiap group proses memiliki **environment sendiri**
### Analogi
Bayangkan sebuah **gedung apartemen**:
- Gedung → Kernel Linux
- Unit apartemen → Namespace
- Penghuni → Process

Setiap penghuni merasa memiliki ruang sendiri walaupun masih berada di gedung yang sama.

---

# 2. Jenis-Jenis Namespace

## 1. PID Namespace

Mengisolasi **Process ID (PID)**.
Contoh:

Host:
```
PID 1   -> systemd
PID 100 -> nginx
```

Container:
```
PID 1 -> nginx
PID 2 -> worker
```

Proses dalam container merasa menjadi **PID 1**, walaupun sebenarnya memiliki PID berbeda di host.

---

## 2. Network Namespace

Mengisolasi **network stack**.
Setiap namespace memiliki:
- network interface sendiri
- routing table sendiri
- IP address sendiri
- firewall rules sendiri

Contoh:
Host
```
eth0 -> 192.168.1.10
```

Container
```
eth0 -> 172.17.0.2
```

---

## 3. Mount Namespace

Mengisolasi **filesystem mount**.

Host:
```
/home
/etc
/var
```

Container dapat melihat:
```
/
/app
/data
```

Filesystem container biasanya berasal dari **container image**.

---

## 4. UTS Namespace

Mengisolasi:
- hostname
- domain name

Contoh:
Host
```
hostname = production-server
```

Container
```
hostname = nginx-container
```

---

## 5. IPC Namespace

Mengisolasi mekanisme komunikasi antar proses:
- shared memory
- message queue
- semaphore

Ini mencegah proses container berkomunikasi dengan proses di luar container menggunakan IPC.

---

## 6. User Namespace

Mengisolasi **user ID dan group ID**.

Contoh:
Dalam container:
```
root = UID 0
```

Di host:
```
UID 100000
```

Artinya root di container **tidak memiliki akses root sebenarnya di host**.

---

# 3. Cgroup (Control Groups)

## Definisi
Cgroup adalah fitur kernel Linux untuk **mengontrol dan membatasi penggunaan resource oleh proses**.
Resource yang dapat dikontrol:
- CPU
- Memory
- Disk I/O
- Network
- jumlah process

---

## Fungsi Utama

Cgroup mencegah satu proses menghabiskan seluruh resource server.
Contoh tanpa cgroup:

```
while true:
    allocate_memory()
```

Program tersebut bisa menghabiskan seluruh RAM dan membuat server crash.
Dengan cgroup, kita bisa membatasi:

```
memory = 512MB
cpu = 1 core
```

---

# 4. Resource yang Diatur Cgroup
## CPU
Mengatur penggunaan CPU:
- cpu shares
- cpu quota
- cpu period

Contoh limit:

```
CPU = 1.5 core
```

---

## Memory
Mengatur:
- maksimum RAM
- swap usage

Contoh:

```
memory limit = 512MB
```

---

## Disk I/O
Mengatur:
- kecepatan baca/tulis disk
- prioritas akses disk

---
## Process Limit
Membatasi jumlah proses.
Contoh:

```
max process = 100
```

---

# 5. Container = Namespace + Cgroup

Container sebenarnya hanyalah **process biasa yang diisolasi dan dibatasi resource-nya**.

Formula sederhananya:

```
Container = Process + Namespace (Isolation) + Cgroup (Resource Limit)
```

---

# 6. Perbedaan Container vs Virtual Machine

## Virtual Machine

Virtual machine menjalankan sistem operasi lengkap.

```
Host OS
 └ Hypervisor
     └ Guest OS
         └ Applications
```

VM memiliki:
- kernel sendiri
- OS sendiri

---

## Container

Container hanya menjalankan proses terisolasi.

```
Host Kernel
 ├ Container A
 │   └ Process
 ├ Container B
 │   └ Process
 └ Container C
     └ Process
```

Container **berbagi kernel yang sama**.

---

# 7. Cara Kerja Container (Simplified)

Ketika container dijalankan:
1. Kernel membuat namespace baru
2. Kernel menetapkan cgroup
3. Kernel menjalankan proses utama container

Contoh proses utama:
```
/usr/sbin/nginx
```

Dalam container proses ini menjadi:
```
PID 1
```

---

# 8. Ringkasan

| Konsep    | Fungsi                  |
| --------- | ----------------------- |
| Namespace | Mengisolasi environment |
| Cgroup    | Mengontrol resource     |
| Container | Process yang terisolasi |

Container modern seperti Docker dan Kubernetes memanfaatkan kedua fitur ini untuk menjalankan aplikasi secara ringan dan scalable.