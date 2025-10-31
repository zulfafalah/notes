# 🎫 Teknik Mencegah Double Booking

> **Bayangkan:** Dua orang mencoba memesan kursi pesawat yang sama dalam waktu bersamaan. Siapa yang seharusnya mendapatkannya? Bagaimana sistem mencegah kedua reservasi berhasil?

## 🔒 1. Pessimistic Locking (Kunci Pesimis)

**Prinsip:** *"Kunci dulu, baru proses!"*

Saat membaca record untuk reservasi, transaksi menggunci record tersebut sampai satu transaksi tersebut selesai.

### ✅ Kelebihan
- Simple dan sederhana untuk diimplementasikan
- Konsistensi kuat (guaranteed consistency)
- Tidak ada kemungkinan konflik karena one-at-a-time

### ❌ Kekurangan
- **Bottleneck**: Lock dapat menyebabkan throughput turun bila banyak request paralel
- **Risiko deadlock:** transaksi lain yang mengunci beberapa baris berbeda bisa deadlock
- User harus menunggu jika ada yang sedang mengakses

### 💻 Contoh Implementation
```mysql
BEGIN;
-- Lock row dengan FOR UPDATE
SELECT id, status FROM seats WHERE id = 123 FOR UPDATE;

-- Jika status = 'available' maka lanjutkan:
UPDATE seats SET status='reserved', reserved_by = 'alice' WHERE id = 123;

COMMIT; -- Lock dilepas setelah commit
```

**Analogi:** Seperti mengunci pintu toilet - orang lain harus menunggu sampai kamu selesai!


---

## 🏃 2. Optimistic Locking (Kunci Optimis)

**Prinsip:** *"Baca bebas, update dengan cek versi!"*

Baca record + versi, saat update sertakan kondisi bahwa versi masih sama. Jika tidak sama, update gagal dan aplikasi retry/beri feedback.

### ✅ Kelebihan
- Tidak ada lock DB eksplisit (database tetap responsif)
- Throughput lebih tinggi (no blocking, multiple reads)
- Read performance bagus (tidak mengunci)
- Ideal untuk situasi low contention

### ❌ Kekurangan
- Pada kontensi tinggi banyak `UPDATE` akan gagal → perlu retrying (membuang compute & pengalaman pengguna buruk)
- Lebih kompleks dalam handling retry logic
- User mungkin perlu mencoba beberapa kali

### 💻 Contoh Implementation

#### Skenario: Alice dan Bob mencoba booking kursi yang sama

**Step 1:** User A (Alice) membaca data kursi
```mysql
SELECT seat_id, status, version 
FROM seats 
WHERE seat_id = 1;
```

**Hasil:**
```
seat_id | status     | version
--------|------------|--------
1       | available  | 0
```

**Step 2:** User B (Bob) juga membaca data yang sama (bisa berbarengan!)
```mysql
SELECT seat_id, status, version 
FROM seats 
WHERE seat_id = 1;
```

**Hasil yang didapat Bob:**
```
seat_id | status     | version
--------|------------|--------
1       | available  | 0
```

**Step 3:** Alice mencoba update terlebih dahulu
```mysql
UPDATE seats 
SET status = 'reserved', 
    reserved_by = 'Alice', 
    version = version + 1 
WHERE seat_id = 1 
  AND status = 'available'
  AND version = 0;  -- ✅ Berhasil! (1 row affected)
```

**Step 4:** Bob mencoba update (tapi terlambat!)
```mysql
UPDATE seats 
SET status = 'reserved', 
    reserved_by = 'Bob', 
    version = version + 1 
WHERE seat_id = 1 
  AND status = 'available'
  AND version = 0;  -- ❌ Gagal! (0 rows affected)
                     -- Karena version sudah berubah jadi 1
```

### 🎯 Key Point

Ketika mengupdate data **HARUS** melakukan komparasi version. Jika versinya masih sama dengan yang dibaca sebelumnya, berarti tidak ada orang lain yang mengubah data, dan user dapat melakukan update.

**Analogi:** Seperti sistem edit dokumen Google Docs - semua orang bisa membaca, tapi kalau ada konflik saat save, sistem akan memberi tahu!

---

## 🤔 Kapan Pakai Yang Mana?

| Kondisi | Rekomendasi |
|---------|-------------|
| 🎪 **High Contention** (banyak user berebut resource yang sama) | **Pessimistic Locking** - Lebih reliable |
| 🏖️ **Low Contention** (jarang konflik) | **Optimistic Locking** - Lebih performant |
| 💰 **Critical Transaction** (uang, pembayaran) | **Pessimistic Locking** - Konsistensi mutlak |
| 📝 **Read-Heavy System** (lebih banyak baca daripada tulis) | **Optimistic Locking** - No blocking |
| ⚡ **Need Speed** | **Optimistic Locking** - Parallel processing |

---

## 🚀 Bonus: Hybrid Approach

Kombinasikan keduanya untuk mendapatkan yang terbaik:
- Gunakan **optimistic** untuk operasi normal
- Fallback ke **pessimistic** setelah N kali retry gagal

```python
def book_seat(seat_id, user):
    max_retries = 3
    
    # Try optimistic first
    for attempt in range(max_retries):
        if try_optimistic_update(seat_id, user):
            return "Success!"
    
    # Fallback to pessimistic if contention too high
    return try_pessimistic_update(seat_id, user)
```

**Hasil:** Best of both worlds! 🎉
