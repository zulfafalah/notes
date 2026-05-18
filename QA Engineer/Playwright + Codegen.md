## Install

```bash
npm init playwright@latest
```

Ikuti prompt: pilih TypeScript/JavaScript, folder tes, dan apakah mau tambah GitHub Actions workflow.

---

## Rekam tes dengan Codegen

```bash
npx playwright codegen --output=tests/login.spec.ts "http://localhost:3000/login?next=/"
```

- Gunakan tanda kutip di URL agar karakter `?` dan `=` tidak diinterpretasi shell
- Folder `tests/` harus sudah ada sebelumnya
- Pilih browser tertentu dengan flag `--browser=firefox`

Setelah selesai merekam, tutup browser window. File `.spec.ts` otomatis tersimpan.

---

## Edit kode hasil rekaman

Tambahkan assertion secara manual karena Codegen hanya merekam aksi:

```ts
await expect(page).toHaveTitle('Dashboard');
await expect(page.locator('h1')).toContainText('Selamat datang');
```

---

## Jalankan tes

```bash
# Jalankan satu file
npx playwright test tests/login.spec.ts

# Tampilkan browser saat tes berjalan
npx playwright test tests/login.spec.ts --headed

# Mode UI interaktif untuk debug
npx playwright test tests/login.spec.ts --ui

# Jalankan semua file tes
npx playwright test
```

---

## Lihat laporan hasil tes

```bash
npx playwright show-report
```

Laporan HTML lengkap dengan screenshot, trace, dan detail kegagalan per langkah.