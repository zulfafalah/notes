Model Transformer (transformator) adalah jenis arsitektur [jaringan neural](https://www.ibm.com/id-id/think/topics/neural-networks) yang  memproses data sekuensial, 
Lebih jelasnya lihat contoh berikut: 
## 1. Pada Transformer, semua kata dibaca sekaligus secara paralel

Transformer tidak punya “langkah ke-1, ke-2, ke-3…” seperti Recurrent Neural Network (RNN).
Semua kata langsung di-_encode_ bersama:

1. "Saya"
2. "makan"
3. "nasi"
4. "goreng"

Lalu model menghitung **hubungan antar semua kata**, contohnya:
- seberapa penting "makan" terhadap "Saya"
- seberapa penting "goreng" terhadap "nasi"
- seberapa penting "makan" terhadap "goreng" 

Semua ini dihitung **dalam satu kali operasi matriks besar**, bukan berurutan langkah demi langkah.

Pada model lama seperti **_Recurrent Neural Network (RNN)_**, pemrosesan wajib dilakukan begini:

1 → "Saya"  
2 → "makan"  
3 → "nasi"  
4 → "goreng"

Karena model lama hanya bisa memproses **satu token per langkah**, dan langkah berikutnya bergantung pada hasil sebelumnya.

Artinya:  
Kalau mau tahu arti “goreng”, dia harus menunggu proses “Saya”, “makan”, dan “nasi” selesai dulu.

Sehingga arsitektur Model transformer lebih unggul dari pada arsitektur lama seperti _Recurrent Neural Network_ (RNN) 
Karena: 
- jauh lebih cepat
- bisa melihat konteks global (hubungan kata jarak jauh)
- tidak terjebak pada "ingatan pendek" seperti RNN
- **Self-Attention**
 
 Model Transformer juga menjadi pondasi model model LLM modern seperti GPT, Gemini, dan Claude.
 
## 2. Self-Attention
### **2.1 Tujuan Self-Attention**
Self-attention bertugas mencari **hubungan antar token dalam satu sequence**.

Contoh kalimat:
“Kucing itu memakan ikan karena **lapar**.”

Model harus tahu:
- Kata “lapar” berhubungan dengan “kucing”
- Bukan dengan “ikan”

Self-attention menghitung hubungan ini **secara matematis**, bukan dengan aturan manual.

### 2.1 Dari mana self-attention berasal? 
Self-attention lahir dari ide berikut:

 **“Untuk memahami setiap token, kita harus melihat token lain dan menentukan mana yang penting.”**

Cara mencapainya:  
**mengalikan representasi token (embedding) dengan bobot yang belajar sendiri untuk menghasilkan Query, Key, dan Value.**

Jadi self-attention _tidak muncul dari luar_ — ia **dihasilkan oleh parameter (weight matrix) yang dipelajari saat training**, yaitu:
- W_Q (matriks Query)
- W_K (matriks Key)
- W_V (matriks Value)
#### Embeding
adalah mengubah data mentah (teks, posisi, kategori) menjadi representasi vektor numerik.
#### **Training**
- Melatih model agar bisa memahami pola dan menghasilkan output yang benar.
- Model **belajar** dan mengupdate bobot (weights).
#### **Inference**
- Menggunakan model yang sudah dilatih untuk menghasilkan output.
- Model **tidak belajar**, hanya _memakai_ bobot yang sudah ada.



