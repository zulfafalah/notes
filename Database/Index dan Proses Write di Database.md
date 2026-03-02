## Struktur Tabel

```
users
- id (PK)
- name
- email
- phone
```

## Daftar Index
- PRIMARY KEY (id)
- INDEX (email)
- INDEX (phone)
- INDEX (name)
- INDEX (email, phone) → composite index
Total: 5 index.
---
## Proses INSERT

Contoh perintah:
```sql
INSERT INTO users (id, name, email, phone)
VALUES (1, 'John', 'j@mail.com', '0812');
```

### Langkah yang terjadi:
1. Database menulis row baru ke tabel utama.
2. Database memperbarui semua struktur index:
    - Menambah entry ke index PRIMARY
    - Menambah entry ke index email
    - Menambah entry ke index phone
    - Menambah entry ke index name
    - Menambah entry ke index email_phone

Semakin banyak index, semakin banyak proses tulis pada setiap operasi write.

---

## Lokasi Fisik Index

- **MySQL InnoDB**: index disimpan dalam berkas `.ibd` (tergabung atau per tabel tergantung konfigurasi).
- **PostgreSQL**: setiap index adalah berkas fisik terpisah.
Secara konsep: insert data berarti menulis ke tabel utama dan juga ke setiap struktur index.
---
## Ringkasan

| Hal                             | Catatan              |
| ------------------------------- | -------------------- |
| Insert hanya ke tabel utama     | Tidak                |
| Insert juga ke index            | Ya                   |
| Banyak index memperlambat write | Ya                   |
| Banyak index mempercepat read   | Ya, jika index tepat |
