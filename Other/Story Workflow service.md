# [BE] Story 1: Create Workflow Service
Sebagai developer,  
saya ingin membuat service baru bernama service-workflow menggunakan Go,  
supaya service lain (dimulai dari service-noo) bisa menggunakan service ini untuk mengelola approval workflow dokumen.

### Latar belakang:  
Service ini berdiri sendiri dan tidak berbagi DB dengan service lain. Service-noo akan memanggil service-workflow via HTTP, dan setelah proses selesai, service-workflow akan mengirim webhook balik ke service-noo.

Alur yang harus diimplementasi:
1. service-noo kirim POST /workflow/transition dengan payload { doc_type, doc_id, action, actor_id, notes }
2. service-workflow validasi role dan cek apakah transisi valid
3. Jika valid, jalankan atomic transaction: update WorkflowState + insert WorkflowHistory dalam satu operasi
4. Setelah transaction sukses, kirim webhook ke callback_url yang terdaftar

### Table yang dibutuhkan:

**workflow_definition**

|id|name|doc_type|is_active|created_at|
|---|---|---|---|---|
|wfd-001|PO Approval Flow|PurchaseOrder|true|2025-01-10 08:00|
|wfd-002|SO Approval Flow|SalesOrder|true|2025-01-10 08:30|

---

**workflow_transition**

|id|definition_id|from_state|action|to_state|allowed_roles|
|---|---|---|---|---|---|
|wft-001|wfd-001|draft|submit|pending_approval|["staff","supervisor"]|
|wft-002|wfd-001|pending_approval|approve|approved|["manager","director"]|
|wft-003|wfd-001|pending_approval|reject|rejected|["manager","director"]|
|wft-004|wfd-001|rejected|revise|draft|["staff","supervisor"]|

---

**webhook_registry**

| id      | doc_type      | callback_url                                    |
| ------- | ------------- | ----------------------------------------------- |
| whr-001 | PurchaseOrder | http://service-noo/api/noo/webhook/workflow     |
| whr-002 | SalesOrder    | http://service-sales/api/sales/webhook/workflow |

---

**workflow_state** _(satu baris per dokumen, diupdate setiap transition)_

|id|definition_id|doc_type|doc_id|current_state|updated_at|
|---|---|---|---|---|---|
|wfs-001|wfd-001|PurchaseOrder|po-2025-001|approved|2025-01-15 14:32|
|wfs-002|wfd-001|PurchaseOrder|po-2025-002|pending_approval|2025-01-15 16:10|
|wfs-003|wfd-001|PurchaseOrder|po-2025-003|rejected|2025-01-15 17:05|
|wfs-004|wfd-001|PurchaseOrder|po-2025-004|draft|2025-01-15 17:30|

---

**workflow_history** _(hanya insert, tidak pernah diupdate — audit trail permanen)_

|id|doc_id|from_state|action|to_state|actor_name|notes|created_at|
|---|---|---|---|---|---|---|---|
|wfh-001|po-2025-001|draft|submit|pending_approval|Budi Santoso|—|2025-01-15 09:00|
|wfh-002|po-2025-001|pending_approval|approve|approved|Siti Manajer|Budget sesuai Q1|2025-01-15 14:32|
|wfh-003|po-2025-003|draft|submit|pending_approval|Andi Staff|—|2025-01-15 15:00|
|wfh-004|po-2025-003|pending_approval|reject|rejected|Siti Manajer|Harga terlalu tinggi|2025-01-15 17:05|

## Flow lengkap

service-noo
    │
    │ 1. POST /workflow/transition
    │    { doc_type, doc_id, action, actor_id, notes }
    ▼
service-workflow
    │ 2. validate role & state
    │
    │ 3. atomic block:
    │    ├── update WorkflowState (current_state)
    │    └── insert WorkflowHistory 
    │
    │ 4. POST webhook ke service-noo
    │    { doc_id, new_state, doc_type }
    ▼
service-noo
    │ 5. update PO status di DB-nya sendiri



# [BE] Story 2: Integrasi service-workflow di service-noo
Sebagai developer,  
saya ingin service-noo bisa berkomunikasi dengan service-workflow,  
supaya dokumen NOO di service-noo memiliki approval flow yang terkontrol.


# **[FE] Story 3: Tampilan Workflow di Halaman Purchase Order**

Sebagai pengguna,  
saya ingin bisa melihat status approval dan melakukan aksi approve/reject/submit dll langsung dari halaman NOO,  
supaya proses approval bisa dilakukan tanpa berpindah halaman.
