
## 📋 Pengenalan
Dokumen ini berisi panduan umum untuk menggunakan filter dan parameter dalam SQL Query di Metabase, dengan fokus pada database MySQL.

---

## 🔧 Tipe-Tipe Parameter Metabase

### 1. Text Parameter
**Kegunaan:** Filter berdasarkan teks (nama, kode, deskripsi, dll)

**Setup di Metabase:**
- Variable type: `Text`
- Required: Unchecked (untuk optional filter)

**Sintaks SQL:**
```sql
-- Pencarian exact match
[[AND column_name = {{parameter_name}}]]

-- Pencarian partial match (mengandung kata)
[[AND column_name LIKE CONCAT('%', {{parameter_name}}, '%')]]

-- Pencarian dimulai dengan
[[AND column_name LIKE CONCAT({{parameter_name}}, '%')]]

-- Pencarian diakhiri dengan
[[AND column_name LIKE CONCAT('%', {{parameter_name}})]]
```

**Contoh Use Case:**
- Filter nama customer, supplier, karyawan
- Cari nomor dokumen, invoice, PO
- Filter kode produk, SKU, barcode

---

### 2. Date Parameter
**Kegunaan:** Filter berdasarkan tanggal

**Setup di Metabase:**
- Variable type: `Date`
- Field to map to: Pilih kolom tanggal terkait
- Required: Unchecked

**Sintaks SQL:**
```sql
-- Tanggal mulai (dari)
[[AND date_column >= {{date_from}}]]

-- Tanggal akhir (sampai)
[[AND date_column <= {{date_to}}]]

-- Tanggal spesifik
[[AND date_column = {{specific_date}}]]

-- Bulan dan tahun
[[AND YEAR(date_column) = {{year}}]]
[[AND MONTH(date_column) = {{month}}]]
```

**Contoh Use Case:**
- Filter tanggal transaksi, invoice, pembayaran
- Range periode laporan
- Filter berdasarkan bulan/tahun

---

### 3. Number Parameter
**Kegunaan:** Filter berdasarkan angka (ID, jumlah, harga, dll)

**Setup di Metabase:**
- Variable type: `Number`
- Required: Unchecked

**Sintaks SQL:**
```sql
-- Nilai spesifik
[[AND numeric_column = {{number_param}}]]

-- Lebih besar dari
[[AND numeric_column > {{min_value}}]]

-- Lebih kecil dari
[[AND numeric_column < {{max_value}}]]

-- Range
[[AND numeric_column BETWEEN {{min_value}} AND {{max_value}}]]
```

**Contoh Use Case:**
- Filter berdasarkan ID
- Minimum/maksimum harga
- Filter quantity, amount
- Range nilai tertentu

---

### 4. Dropdown/List Parameter
**Kegunaan:** Pilihan dari daftar nilai tertentu

**Setup di Metabase:**
- Variable type: `Field Filter` atau `Text`
- Input type: Dropdown list
- Values: Manual atau dari query

**Sintaks SQL:**
```sql
-- Single selection
[[AND status_column = {{status}}]]

-- Multiple selection (field filter)
[[AND {{status_filter}}]]
```

**Contoh Use Case:**
- Filter status (Active, Inactive, Pending)
- Filter kategori produk
- Pilih departemen, divisi
- Filter tipe dokumen

---

## 🎯 Sintaks Conditional Filter `[[...]]`

### Cara Kerja
- `[[...]]` membuat filter menjadi **optional**
- Jika parameter kosong/null, clause tidak dijalankan
- Jika parameter diisi, clause akan dieksekusi

### Contoh:
```sql
WHERE 1=1
[[AND name LIKE CONCAT('%', {{name}}, '%')]]
[[AND created_date >= {{date_from}}]]
[[AND status = {{status}}]]
```

**Behavior:**
- Jika semua parameter kosong → menampilkan semua data
- Jika hanya `name` diisi → filter hanya berdasarkan name
- Jika `name` dan `status` diisi → filter kombinasi keduanya

---

## ⚠️ Perbedaan Sintaks Database

### MySQL
```sql
-- Konkatenasi string
CONCAT('text', {{param}}, 'text')

-- LIKE dengan parameter
LIKE CONCAT('%', {{param}}, '%')
```

### PostgreSQL / Oracle
```sql
-- Konkatenasi string
'text' || {{param}} || 'text'

-- LIKE dengan parameter
LIKE '%' || {{param}} || '%'
```

**💡 Tip:** Pastikan gunakan sintaks yang sesuai dengan database Anda!

---

## 📊 Best Practices

### 1. Naming Convention
```sql
-- ✅ Good: Deskriptif dan konsisten
{{customer_name}}
{{date_from}}
{{status_filter}}

-- ❌ Avoid: Nama ambigu
{{param1}}
{{x}}
{{temp}}
```

### 2. Default Values
Pertimbangkan memberikan default value untuk parameter penting:
```sql
-- Contoh: Default 30 hari terakhir
WHERE date_column >= COALESCE({{date_from}}, DATE_SUB(CURDATE(), INTERVAL 30 DAY))
```

### 3. Performance Optimization
```sql
-- ⚠️ Slow: LIKE dengan % di depan (tidak pakai index)
WHERE name LIKE CONCAT('%', {{param}}, '%')

-- ✅ Better: LIKE tanpa % di depan (bisa pakai index)
WHERE name LIKE CONCAT({{param}}, '%')

-- ✅ Best: Exact match dengan index
WHERE id = {{param}}
```

### 4. Kombinasi Multiple Filters
```sql
WHERE 1=1
  [[AND customer_name LIKE CONCAT('%', {{customer}}, '%')]]
  [[AND order_date >= {{date_from}}]]
  [[AND order_date <= {{date_to}}]]
  [[AND status = {{status}}]]
  [[AND amount >= {{min_amount}}]]
  [[AND category IN ({{categories}})]]
```

---

## 🔍 Pattern Umum Filter

### Filter Text dengan Case Insensitive
```sql
[[AND LOWER(column_name) LIKE LOWER(CONCAT('%', {{param}}, '%'))]]
```

### Filter Multiple Values (IN clause)
```sql
-- Setup: Variable type = Text, user input comma separated
[[AND status IN ({{status_list}})]]
-- Input example: 'Active','Pending','Done'
```

### Filter dengan Null Check
```sql
[[AND ({{param}} IS NULL OR column_name = {{param}})]]
```

### Filter Kombinasi OR
```sql
[[AND (
  column1 LIKE CONCAT('%', {{search}}, '%') OR
  column2 LIKE CONCAT('%', {{search}}, '%') OR
  column3 LIKE CONCAT('%', {{search}}, '%')
)]]
```

---

## 🐛 Troubleshooting

### Problem: Filter tidak bekerja
**Solusi:**
- ✅ Cek nama parameter sama persis dengan di query `{{nama_parameter}}`
- ✅ Pastikan menggunakan `[[...]]` untuk optional filter
- ✅ Cek sintaks CONCAT sesuai database (MySQL vs PostgreSQL)
- ✅ Pastikan parameter tidak di-set sebagai Required jika optional

### Problem: Error "Parameter not found"
**Solusi:**
- ✅ Pastikan parameter sudah dibuat di Metabase
- ✅ Cek typo pada nama parameter
- ✅ Refresh query atau reload page

### Problem: Query terlalu lambat
**Solusi:**
- ✅ Tambahkan index pada kolom yang sering difilter
- ✅ Hindari LIKE dengan `%` di awal jika memungkinkan
- ✅ Batasi data dengan filter tanggal
- ✅ Gunakan LIMIT jika tidak perlu semua data

### Problem: Hasil tidak sesuai ekspektasi
**Solusi:**
- ✅ Test parameter satu per satu
- ✅ Cek data type parameter (Text, Number, Date)
- ✅ Cek case sensitivity untuk text filter
- ✅ Cek format tanggal (YYYY-MM-DD)

---

## 📝 Template Query Standar

### Template Dasar dengan Multiple Filters
```sql
SELECT 
  *
FROM your_table
WHERE 1=1
  [[AND text_column LIKE CONCAT('%', {{text_param}}, '%')]]
  [[AND date_column >= {{date_from}}]]
  [[AND date_column <= {{date_to}}]]
  [[AND status_column = {{status}}]]
  [[AND numeric_column >= {{min_value}}]]
ORDER BY date_column DESC
```

### Template dengan Join
```sql
SELECT 
  t1.*,
  t2.related_column
FROM main_table t1
LEFT JOIN related_table t2 ON t1.id = t2.main_id
WHERE 1=1
  [[AND t1.column1 LIKE CONCAT('%', {{param1}}, '%')]]
  [[AND t2.column2 = {{param2}}]]
  [[AND t1.date_column BETWEEN {{date_from}} AND {{date_to}}]]
```

### Template dengan Aggregation
```sql
SELECT 
  category,
  COUNT(*) as total,
  SUM(amount) as total_amount
FROM transactions
WHERE 1=1
  [[AND transaction_date >= {{date_from}}]]
  [[AND transaction_date <= {{date_to}}]]
  [[AND status = {{status}}]]
GROUP BY category
HAVING 1=1
  [[AND SUM(amount) >= {{min_total}}]]
ORDER BY total_amount DESC
```

---

## 🎓 Tips & Tricks

### 1. Multi-purpose Search Filter
Buat satu parameter untuk mencari di multiple kolom:
```sql
[[AND (
  column1 LIKE CONCAT('%', {{search}}, '%') OR
  column2 LIKE CONCAT('%', {{search}}, '%') OR
  column3 LIKE CONCAT('%', {{search}}, '%')
)]]
```

### 2. Dynamic Date Range Presets
```sql
-- Last 7 days
WHERE date_column >= DATE_SUB(CURDATE(), INTERVAL 7 DAY)

-- This month
WHERE YEAR(date_column) = YEAR(CURDATE()) 
  AND MONTH(date_column) = MONTH(CURDATE())

-- This year
WHERE YEAR(date_column) = YEAR(CURDATE())
```

### 3. Smart Default Filters
```sql
-- Show only active records by default
WHERE status = COALESCE({{status}}, 'Active')

-- Last 30 days by default
WHERE date_column >= COALESCE({{date_from}}, DATE_SUB(CURDATE(), INTERVAL 30 DAY))
```

---

## 📚 Resources

### Dokumentasi Resmi
- [Metabase SQL Parameters](https://www.metabase.com/docs/latest/questions/native-editor/sql-parameters)
- [MySQL String Functions](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html)
- [MySQL Date Functions](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html)


---

**Terakhir diupdate:** 31 Oktober 2024  
**Database:** MySQL (dengan catatan untuk PostgreSQL/Oracle)  
**Platform:** Metabase
t