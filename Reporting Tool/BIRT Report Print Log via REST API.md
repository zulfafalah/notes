
Untuk menambahkan script di BIRT, buka report yang ingin diberi script, lalu klik **tab Script** pada panel bagian bawah. Berikut contohnya.
![[Pasted image 20260312092511.png]]

Di **BIRT**, terdapat beberapa **trigger** yang menentukan kapan sebuah script akan dieksekusi.

### BIRT Report Script Triggers

#### 1. Report Level
*(Klik area kosong report → tab Script)*

| Trigger             | Kapan Jalan                      | Cocok Untuk                                          |
| ------------------- | -------------------------------- | ---------------------------------------------------- |
| `beforeFactory`     | Sebelum report mulai di-render   | Log print, validasi parameter, setup variabel global |
| `afterFactory`      | Setelah report selesai di-render | Cleanup, kirim notifikasi selesai                    |
| `beforeOpenDataSet` | Sebelum dataset dibuka           | Modifikasi query / parameter dinamis                 |
| `afterCloseDataSet` | Setelah dataset ditutup          | Post-processing data                                 |
| `beforeRender`      | Sebelum output di-generate       | Modifikasi layout dinamis                            |
| `afterRender`       | Setelah output selesai           | Logging output, kirim email                          |

---
#### 2. Data Set Level
*(Klik kanan dataset → Edit → Script)*

| Trigger | Kapan Jalan | Cocok Untuk |
|---|---|---|
| `beforeOpen` | Sebelum dataset dibuka | Set parameter query dinamis |
| `afterOpen` | Setelah dataset terbuka | Validasi atau manipulasi data awal |
| `onFetch` | Setiap baris data di-fetch | Transform / filter data per baris |
| `beforeClose` | Sebelum dataset ditutup | Cleanup sebelum dataset selesai |
| `afterClose` | Setelah dataset ditutup | Logging atau post-processing |

---

#### 3. Element Level
*(Klik element → tab Script)*

Berlaku untuk: **Table, List, Text, Label, Image, Chart**

| Trigger | Kapan Jalan | Cocok Untuk |
|---|---|---|
| `onCreate` | Saat element dibuat | Set nilai dinamis |
| `onRender` | Saat element di-render ke output | Format kondisional |
| `onPageBreak` | Saat ada page break | Hitung halaman custom |

---

#### 4. Table / List Row Level

| Trigger    | Kapan Jalan          | Cocok Untuk                             |
| ---------- | -------------------- | --------------------------------------- |
| `onStart`  | Sebelum row pertama  | Inisialisasi counter / variabel         |
| `onCreate` | Setiap row dibuat    | Logika per baris                        |
| `onRender` | Setiap row di-render | Styling kondisional (zebra stripe, dll) |
| `onFinish` | Setelah row terakhir | Total atau summary                      |

---
## Example Script
### 1. BIRT Hit API GET

```java
// beforeFactory
importPackage(Packages.java.net);
importPackage(Packages.java.io);

try {
    var apiUrl = "http://192.168.200.208:8018/v1/test";
    var uri = new Packages.java.net.URI(apiUrl);
    var url = uri.toURL();
    
    var conn = url.openConnection();
    conn.setConnectTimeout(5000);
    conn.setReadTimeout(5000);
    conn.setRequestProperty("Accept", "application/xml");
    conn.connect();
    
    var responseCode = conn.getResponseCode();
    
    var reader = new Packages.java.io.BufferedReader(
        new Packages.java.io.InputStreamReader(
            conn.getInputStream(), "UTF-8"
        )
    );
    
    var sb = new Packages.java.lang.StringBuilder();
    var line;
    while ((line = reader.readLine()) != null) {
        sb.append(line);
    }
    reader.close();
    conn.disconnect();

    Packages.java.lang.System.out.println("Status: " + responseCode);
    Packages.java.lang.System.out.println("Response: " + sb.toString());
    
} catch (e) {
    Packages.java.lang.System.out.println("Error: " + e.toString());
}
```

## 2. BIRT Hit API POST

```java
// beforeFactory
importPackage(Packages.java.net);
importPackage(Packages.java.io);

try {
    var apiUrl = "http://192.168.200.208:8018/v1/test";
    
    // JSON payload
    var payload = '{"report_name":"nama_report","printed_by":"user123","printed_at":"' + new Date().toISOString() + '"}';
    
    var uri = new Packages.java.net.URI(apiUrl);
    var url = uri.toURL();
    
    var conn = url.openConnection();
    conn.setRequestMethod("POST");
    conn.setDoOutput(true);          // wajib untuk POST
    conn.setDoInput(true);
    conn.setConnectTimeout(5000);
    conn.setReadTimeout(5000);
    conn.setRequestProperty("Content-Type", "application/json");
    conn.setRequestProperty("Accept", "application/json");
    
    var os = conn.getOutputStream();
    var writer = new Packages.java.io.OutputStreamWriter(os, "UTF-8");
    writer.write(payload);
    writer.flush();
    writer.close();
    os.close();
    
    
    var responseCode = conn.getResponseCode();
    
    var reader = new Packages.java.io.BufferedReader(
        new Packages.java.io.InputStreamReader(
            conn.getInputStream(), "UTF-8"
        )
    );
    
    var sb = new Packages.java.lang.StringBuilder();
    var line;
    while ((line = reader.readLine()) != null) {
        sb.append(line);
    }
    reader.close();
    conn.disconnect();
    
    Packages.java.lang.System.out.println("=== POST LOG ===");
    Packages.java.lang.System.out.println("Status: " + responseCode);
    Packages.java.lang.System.out.println("Response: " + sb.toString());
    
} catch (e) {
    Packages.java.lang.System.out.println("Error: " + e.toString());
}
```

