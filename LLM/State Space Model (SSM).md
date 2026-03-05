## Overview

**State Space Model (SSM)** adalah arsitektur untuk memproses data berurutan (_sequence_) yang berasal dari teori sistem dinamis dan kontrol.

SSM memodelkan bagaimana **state internal** berkembang seiring waktu ketika menerima input baru.
Dalam beberapa tahun terakhir, SSM menjadi alternatif dari **Transformer** untuk menangani sequence yang sangat panjang.

## Intuisi Dasar

Alih-alih menghitung hubungan antar semua token seperti pada **self-attention**, SSM menyimpan **state internal** yang diperbarui setiap kali token baru masuk.

```
token → update state → output
```

State ini bertindak sebagai **memori ringkasan dari seluruh sequence sebelumnya**.

## Kompleksitas

Perbandingan dengan Transformer:

|Model|Kompleksitas|
|---|---|
|Transformer|O(n²)|
|SSM|O(n)|

Karena tidak menghitung attention matrix antar semua token, SSM lebih efisien untuk sequence panjang.

---

## Kelebihan SSM

1. Efisiensi Sequence Panjang
2. Memori Lebih Stabil, tidak membutuhkan matriks attention besar.
3. Cocok untuk Data Continuous

---

## Kekurangan SSM

1. Paralelisme Lebih Sulit, transformer sangat mudah diparalelkan di GPU.
2. Reasoning Kompleks, self-attention masih lebih kuat dalam beberapa tugas reasoning.
3. Ekosistem Lebih Baru, tooling dan implementasi masih berkembang.

---

## Model SSM Modern

Beberapa model yang menggunakan konsep SSM:

- **S4** (Structured State Space Sequence Model)
- **S5**
- **Mamba**
- **Gated SSM**
- **Hyena**

Model **Mamba** saat ini adalah implementasi SSM yang paling populer.

---

## Mamba

Mamba adalah SSM modern yang memperkenalkan konsep **Selective State Space**.

**Model ini dapat:**
- Memilih token penting
- Mempertahankan efisiensi linear
- Memproses sequence sangat panjang

**Beberapa keunggulan:**
- Kompleksitas O(n)
- Efisien di GPU
- Mampu menangani jutaan token

---

## Perbandingan Transformer vs SSM

|Aspek|Transformer|SSM|
|---|---|---|
|Mekanisme|Self-attention|State dynamics|
|Kompleksitas|O(n²)|O(n)|
|Sequence panjang|mahal|sangat efisien|
|Paralelisme|sangat baik|lebih terbatas|
|Multimodal|matang|masih berkembang|

---

## Hubungan Dengan Model Lama

SSM dapat dianggap sebagai evolusi dari model sequential lama:

RNN → LSTM → Transformer → SSM
Namun secara matematis SSM berasal dari **control theory**, bukan langsung dari arsitektur neural network klasik.

---

## Use Case SSM

SSM cocok untuk:

- Long document modeling
- Genomics
- Audio processing
- Sensor data
- Video sequence
- System logs

---

## Catatan Penting

Self-attention memiliki kemampuan **global context lookup**, sedangkan SSM menggunakan **compressed state memory**.



------------------------------------------------
# Global Context Lookup (Transformer)

## Ide Utama

Model dapat **mengakses semua token sebelumnya secara langsung** setiap kali memproses token baru.

Artinya:

- Tidak ada ringkasan memori    
- Seluruh konteks tetap tersedia

Model hanya perlu **melihat kembali semua token** dan menentukan mana yang penting.

---

## Cara Kerja

Misalkan sequence:
t1 t2 t3 t4 t5
Ketika memproses token **t5**, self-attention akan menghitung:

```
attention(t5, t1)  
attention(t5, t2)  
attention(t5, t3)  
attention(t5, t4)  
attention(t5, t5)

```

Semua token tersedia.

Model melakukan **lookup langsung ke seluruh konteks**.

---

## Analogi
Bayangkan kamu membaca buku dan setiap kali ingin memahami satu kalimat kamu bisa:
> membuka kembali seluruh halaman buku kapan saja.
Tidak perlu mengingat semuanya di kepala.

## Kelebihan
- Reasoning kuat
- Memahami relasi kompleks
- Efektif untuk NLP

---

## Kekurangan

- Mahal untuk sequence panjang
- Memori GPU besar
- Scaling terbatas


# Compressed State Memory (SSM / RNN)

## Ide Utama

Alih-alih menyimpan semua token, model menyimpan **ringkasan dari seluruh history** dalam sebuah **state vector**.

State ini diperbarui setiap token.

## Cara Kerja

Misalkan sequence:
t1 t2 t3 t4 t5

Prosesnya:

```
state1 = f(state0, t1)  
state2 = f(state1, t2)  
state3 = f(state2, t3)  
state4 = f(state3, t4)  
state5 = f(state4, t5)

```

State selalu menjadi **representasi terkompresi dari seluruh masa lalu**.

---

## Analogi

Bayangkan kamu membaca buku tanpa boleh membuka halaman sebelumnya.
Kamu hanya boleh:

> menyimpan ringkasan cerita di kepala.
Jika ringkasannya bagus → kamu masih paham cerita.

Jika ringkasannya buruk → detail hilang.


## Kelebihan

- Sangat efisien untuk sequence panjang    
- Memori stabil
- Cocok untuk data streaming

Contoh:

- Audio
- Video
- Genomics
- Sensor

---

## Kekurangan

- Informasi bisa hilang saat dikompresi
- Hubungan kompleks lebih sulit dipelajari
- Reasoning kadang lebih lemah

---

# Perbandingan Intuitif

|Model|Cara Mengingat|
|---|---|
|Transformer|membaca ulang seluruh buku|
|SSM|hanya menyimpan ringkasan|