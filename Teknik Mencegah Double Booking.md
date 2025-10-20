**1. Pessimistic locking** 
Saat membaca record untuk reservasi, transaksi menggunci rocord tersebut sampai satu transaksi tersebut selesai.

Kelebihan: 
- Simple dan sederhana
- Konsistensi kuat

Kekurangan: 
- **Bottleneck**: Lock dapat menyebabkan throughput turun bila banyak request paralel
- **Risiko deadlock:** transaksi lain yang kunci beberapa baris berbeda bisa deadlock.
Contoh query: 
```mysql
BEGIN;
SELECT id, status FROM seats WHERE id = 123 FOR UPDATE;
-- jika status = 'available' maka:
UPDATE seats SET status='reserved', reserved_by = 'alice' WHERE id = 123;
COMMIT;

```


**2. Optimistic locking (version)**
Baca record + versi, saat update sertakan kondisi bahwa versi masih sama, jika tidak sama, update gagal dan aplikasi retry/beri feedback. 

Kelebihan: 
- Tidak ada lock DB eksplisit.
- Throughput lebih tinggi (no blocking).
- Read performan bagus (tidak mengunci).
Kekurangan: 
- Pada kontensi tinggi banyak `UPDATE` akan gagal → perlu retrying (membuang compute & pengalaman pengguna buruk).

Contoh query:
User A membaca Data A
```mysql
SELECT seat_id, status, version 
FROM seats 
WHERE seat_id = 1;

```

Hasil:
```lua
seat_id | status     | version
-------- | -----------|---------
1        | available  | 0

```

User A Mengupdate data
```mysql
UPDATE seats 
SET status = 'reserved', reserved_by = 'Alice', version = version + 1 
WHERE seat_id = 1 
  AND status = 'available'
  AND version = 0;

```

Jadi pointnya, ketika mengupdate data harus melakukan comparasi version, jika versinya masih sama, berarti user dapat melakukan update.

