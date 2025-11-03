Digunakan untuk melakukan filter hasil pengolahan data, seperti agreagasi (SUM, COUNT, MAX, MIN)
Contoh query
```mysql
SELECT 
    kategori,
    MAX(harga) AS harga_tertinggi
FROM produk
GROUP BY kategori
HAVING MAX(harga) > 2000000;

## Penulisan Having setelah clausa group by
```
Perbedaan HAVING dan WHERE
`HAVING` = digunakan untuk filter data hasil olahan (agregasi)
`WHERE`  = digunakan untuk filter data mentah