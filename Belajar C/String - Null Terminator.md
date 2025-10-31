Null terminator dalam C digunakan sebagai penanda akhir sebuah string, karena dalam C tidak menyimpan panjang dari string.

```C
char nama[50] = "John";
// C tidak tahu berapa panjang "John" tanpa null terminator
// Array berisi: ['J']['o']['h']['n']['\0'][sampah][sampah]...

### **Bagaimana C Mengetahui Akhir String?**
// Tanpa null terminator:
char nama[4] = {'J', 'o', 'h', 'n'};  // BERBAHAYA!
printf("%s", nama); // Akan terus membaca sampai menemukan '\0' secara kebetulan

// Dengan null terminator:
char nama[5] = {'J', 'o', 'h', 'n', '\0'};  // AMAN!
printf("%s", nama); // Berhenti di '\0'

### **Fungsi String Bergantung pada `\0`**
strlen("Hello");    // Menghitung sampai bertemu '\0'
strcpy(dest, src);  // Menyalin sampai bertemu '\0'
printf("%s", str);  // Mencetak sampai bertemu '\0'

### **Contoh Masalah Tanpa Null Terminator:**
char nama[4] = {'J', 'o', 'h', 'n'};  // Tidak ada '\0'
printf("Nama: %s\n", nama);
// Output bisa jadi: "Nama: John�������garbage�������"
// Karena printf terus membaca memory sampai menemukan '\0'
```

