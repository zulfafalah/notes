Menjamin **semua node dalam sistem terdistribusi sepakat pada satu keadaan data yang sama**,  walaupun ada node yang gagal, delay, atau terputus jaringan.
Tujuan dari Rarft ada mencapai **Single Source of Truth”**  yaitu **satu kebenaran tunggal dari data** di seluruh cluster.

### **Tiga Komponen Utama**

1. **Leader Election** 
    
    - Hanya **1 node jadi leader** dalam 1 term (periode).
    - Follower yang tidak menerima heartbeat dalam waktu tertentu → jadi **candidate** → memulai pemilihan.
    - Mayoritas suara = jadi **leader**.
        
2. **Log Replication** 
    
    - Leader menerima perintah (misal: tulis data baru).
    - Leader menyebarkan log ke semua follower.
    - Setelah mayoritas menyetujui (ACK), log dianggap **committed**.
    - Semua node akan punya log yang sama urutannya.
        
3. **Safety & Fault Tolerance** 
    
    - Keputusan yang sudah “commit” **tidak akan berubah** meski leader mati.
    - Node baru bergabung bisa **catch up** dari log leader.
        

---

### **Mekanisme Dasar**

| Peran         | Tugas                                                             |
| ------------- | ----------------------------------------------------------------- |
| **Leader**    | Menerima request dari client, mengirim heartbeat, mereplikasi log |
| **Follower**  | Menunggu perintah dari leader, memberikan suara saat election     |
| **Candidate** | Mengajukan diri jadi leader jika tidak menerima heartbeat         |

---

### **Siklus Sederhana RAFT**

`Follower → (timeout) → Candidate → (menang voting) → Leader Leader → (mati / disconnect) → Follower lain timeout → Election baru`

---

### **Istilah Penting**

| Istilah           | Arti                                                             |
| ----------------- | ---------------------------------------------------------------- |
| **Term**          | Periode waktu dalam Raft, setiap election = term baru            |
| **Heartbeat**     | Pesan rutin dari leader ke follower agar tahu leader masih aktif |
| **Majority Vote** | Setengah + 1 dari total node dibutuhkan untuk konsensus          |
| **Commit Log**    | Data yang sudah disetujui mayoritas dan dijamin konsisten        |

---

### **Contoh Sistem yang Menggunakan Raft**

- **TiDB / TiKV** (PingCAP)
- **Etcd** (dipakai Kubernetes)
- **Consul** (HashiCorp)
- **CockroachDB**, **RQLite**
    

---

### **Analogi Gampang**

Bayangkan **5 orang dalam rapat (node)**.

- Satu orang (leader) memimpin keputusan.
- Kalau pemimpin keluar ruangan → anggota lain memilih pemimpin baru.
- Setiap keputusan baru harus **disetujui minimal 3 orang (mayoritas)** supaya sah.
- Semua mencatat hasil rapat yang sama di notulen (log replication).
    

---

> “**Raft menjaga satu pemimpin, mayoritas sepakat, dan log semua sama.**”

