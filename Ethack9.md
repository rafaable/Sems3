
## Daftar Isi

1. [Isi Paket](#isi-paket)
2. [Bab 9 Client-Side Exploitation](#bab-9-client-side-exploitation)
3. [Step by Step dan Link Youtube](#step-by-step-dan-link-youtube)
4. [Resource Praktikum](#resource-praktikum)
5. [Template Analisis Blueteam](#template-analisis-blueteam)
6. [Template Observasi Peringatan Word](#template-observasi-peringatan-word)


# Isi Paket 

- **`Bab_09_Ringkasan_Textbook_Client_Side_Exploitation.md`** — server-side vs client-side attack surface, trust boundary & Mark-of-the-Web, eksploitasi makro Office (VBA, Protected View, Enable Content), Windows Library Files (dibahas di level konsep saja — sengaja tanpa PoC fungsional karena tekniknya cukup sensitif), Living-off-the-Land Binaries (LOLBins), dan perlombaan senjata mitigasi (MOTW, AMSI) vs teknik bypass.
- **`Bab_09_Modul_Praktikum_Client_Side_Exploitation.md`** — praktikum di **VM Windows Evaluation resmi Microsoft**, terisolasi total dari internet, dengan verifikasi wajib sebelum lanjut.
- **`resources/`**:
    - `macro_poc_harmless.vba` — makro PoC yang **sengaja saya buat 100% harmless** (cuma message box, atau `calc.exe` sebagai variasi opsional) — tidak ada download, tidak ada eksekusi command, tidak ada persistence.
    - `import_windows_eval.sh` — bantu import VHD evaluation ke VirtualBox dengan jaringan **host-only tanpa NAT** (dikunci secara teknis, bukan cuma instruksi).
    - `serve_poc_document.sh` — HTTP server sederhana untuk simulasi "delivery" agar dokumen dapat Mark-of-the-Web.
    - `template_observasi_motw.md` & `template_analisis_blueteam.md` — dokumentasi observasi teknis plus analisis dari sudut pandang defender.
    - `README.md` — dibuka dengan peringatan keras soal isolasi jaringan wajib.

Catatan jujur: untuk teknik Windows Library Files, saya sengaja hanya menjelaskannya di level konsep di textbook tanpa menyediakan PoC fungsional di lab, karena teknik itu lebih spesifik dan sudah pernah dipakai di kampanye intrusi nyata — fokus praktikum saya arahkan ke makro Office yang risikonya jauh lebih terkendali untuk pembelajaran

# Bab 9 Client-Side Exploitation

Coba pikirkan skenario ini: sebuah organisasi sudah investasi besar-besaran di firewall generasi terbaru, WAF yang di-tuning dengan baik, semua server sudah di-patch rutin, bahkan tim vulnerability management-nya rajin banget. Tapi satu karyawan di divisi finance buka lampiran "invoice_Q3.docx" dari email yang kelihatannya dari vendor mereka, klik "Enable Content" karena dokumennya "nggak kebaca" kalau nggak diaktifkan... dan dalam hitungan detik, attacker sudah punya akses ke laptop karyawan tersebut, di dalam jaringan internal yang tadinya begitu "dijaga ketat" dari luar.

Ini yang bikin client-side exploitation begitu menarik sekaligus berbahaya: dia melewati hampir semua pertahanan perimeter yang biasanya jadi fokus utama tim keamanan, karena serangannya nggak menyerang server — dia menyerang manusia yang duduk di depan komputer, lewat aplikasi yang mereka pakai sehari-hari dan mereka percayai.

## 9.1 Server-Side vs Client-Side: Dua Dunia dengan Aturan Berbeda

Sepanjang Bab 6-7 kemarin, fokus kita adalah **server-side attack surface** — kerentanan yang ada di server, dieksploitasi lewat request yang kita kirim, dan hasilnya (kalau berhasil) memberi kita akses ke server tersebut.

**Client-side attack surface** sepenuhnya berbeda arah: di sini, target sesungguhnya adalah aplikasi yang berjalan di komputer korban — browser, aplikasi Office, PDF reader, media player, dan semacamnya. Attacker menyiapkan "sesuatu" (dokumen, link, file), lalu menunggu korban membuka atau berinteraksi dengannya. Begitu korban berinteraksi, kerentanan di aplikasi klien tersebut yang dieksploitasi — bukan server manapun.

Perbedaan arah ini punya implikasi besar: kamu nggak bisa langsung "menyerang" client-side attack surface seperti kamu scan port server. Kamu butuh combo dengan teknik yang sudah kita bahas di Bab 8 — social engineering — untuk benar-benar membuat korban berinteraksi dengan payload yang sudah disiapkan.

## 9.2 Trust Boundary: Kenapa File dari Luar Dianggap Berbahaya Secara Default

Ada konsep penting dalam keamanan sistem bernama **trust boundary** — garis pemisah antara apa yang dianggap "dipercaya" dan apa yang "tidak dipercaya" oleh sebuah sistem.

File yang kamu buat sendiri di komputermu, ada dalam trust boundary yang tinggi — sistem operasi dan aplikasi cenderung memperlakukannya sebagai aman. Tapi file yang datang dari internet — diunduh dari web, atau lampiran email — secara default dianggap berada di luar trust boundary tersebut, meski secara teknis formatnya identik (sama-sama file `.docx` misalnya).

Inilah alasan kenapa sistem operasi modern punya mekanisme seperti **Mark-of-the-Web (MOTW)** — semacam "cap" tersembunyi yang ditambahkan Windows ke file yang diunduh dari internet, yang kemudian dipakai berbagai aplikasi (termasuk Microsoft Office) untuk menentukan apakah file tersebut perlu dibuka dalam mode terbatas atau tidak.

## 9.3 Reconnaissance dan Fingerprinting Sisi Klien

Sebelum menyiapkan payload, ada baiknya kamu tahu dulu "senjata" apa yang dipakai target — sama prinsipnya dengan fingerprinting di Bab 5, tapi sekarang fokusnya ke sisi klien, bukan server.

Informasi yang berguna dikumpulkan: versi sistem operasi target, versi aplikasi Office yang dipakai (versi lama sering punya kerentanan yang sudah dipatch di versi baru), browser dan plugin yang ter-install, bahkan kadang bisa disimpulkan dari metadata dokumen yang pernah dikirim korban sebelumnya (ingat OSINT dari kode sumber/dokumen di Bab 2?) atau dari signature email client yang mereka pakai.

## 9.4 Eksploitasi Lewat Makro Microsoft Office

Ini vektor client-side paling klasik dan masih sangat relevan sampai sekarang, meski Microsoft terus memperketat pertahanannya dari tahun ke tahun.

**VBA (Visual Basic for Applications)** adalah bahasa scripting bawaan aplikasi Office yang memungkinkan otomasi tugas — fitur ini legitimate dan sangat berguna untuk kebutuhan bisnis normal (misalnya otomasi laporan Excel). Masalahnya, kemampuan yang sama yang bikin VBA berguna untuk otomasi legitimate, juga bisa dipakai untuk menjalankan kode berbahaya begitu dokumen dibuka dan makro diaktifkan.

Secara historis, dokumen Office dengan makro (format `.docm`, `.xlsm` — perhatikan huruf "m" di akhir yang menandakan macro-enabled) langsung menjalankan makronya begitu dibuka. Sekarang, Microsoft menerapkan pertahanan berlapis: **Protected View** membuka dokumen dari sumber tidak dipercaya dalam mode sandbox terbatas (read-only, makro tidak aktif) sampai user secara eksplisit klik "Enable Editing", dan makro sendiri tetap butuh klik tambahan "Enable Content" sebelum benar-benar berjalan.

Di sinilah social engineering (Bab 8) dan client-side exploitation saling melengkapi: attacker perlu menyusun skenario (misalnya, dokumen "dilindungi" yang "perlu" diaktifkan kontennya supaya "terbaca dengan benar") yang cukup meyakinkan agar korban mau melewati kedua lapis peringatan tersebut secara sukarela.

## 9.5 Windows Library Files: Vektor yang Kurang Dikenal Tapi Tetap Relevan

Selain makro, ada teknik yang memanfaatkan file `.library-ms` — sebuah fitur Windows Library yang aslinya dirancang untuk mengelompokkan lokasi file dari berbagai folder jadi satu "pustaka" virtual (mirip folder "Documents" bawaan Windows yang sebenarnya adalah library, bukan folder tunggal).

Kerentanan konseptualnya: file library ini bisa dikonfigurasi menunjuk ke lokasi jaringan (network share) tertentu, dan dalam kondisi tertentu, interaksi user dengan file library yang dibuat khusus bisa memicu eksekusi kode tanpa banyak peringatan yang biasanya muncul untuk file executable biasa — karena secara superficial, file ini nggak "terlihat" seperti executable berbahaya.

Ini contoh bagus kenapa attacker terus mencari vektor "tidak biasa" — fitur-fitur Windows yang jarang jadi perhatian tim keamanan karena memang jarang dipakai untuk tujuan normal, tapi tetap ada dan tetap bisa disalahgunakan. Kita tidak akan membangun PoC fungsional untuk teknik ini di modul praktikum (levelnya cukup spesifik dan lebih rawan disalahgunakan di luar konteks lab) — cukup pahami dulu prinsipnya di level konsep.

## 9.6 Living-off-the-Land: Memanfaatkan yang Sudah Ada

Salah satu tren besar dalam client-side exploitation modern adalah teknik **Living-off-the-Land (LOLBins)** — alih-alih membawa tool berbahaya dari luar (yang gampang dikenali antivirus sebagai "asing"), attacker memanfaatkan binary yang **sudah legitimate ada** di sistem operasi itu sendiri untuk menjalankan aksi berbahaya.

Kenapa ini efektif? Karena binary seperti `certutil.exe`, `mshta.exe`, atau `regsvr32.exe` (untuk menyebut beberapa contoh) adalah bagian resmi dari Windows, ditandatangani secara digital oleh Microsoft, dan dipakai untuk banyak keperluan administratif legitimate — sulit bagi solusi keamanan untuk memblokir binary ini secara membabi buta tanpa mengganggu fungsi normal sistem, sementara attacker memanfaatkan fungsi-fungsi tersembunyi/tidak umum dari binary yang sama untuk tujuan jahat.

Proyek **LOLBAS** (yang akan kita lihat lagi di Bab 12) mendokumentasikan secara terbuka binary-binary mana saja yang punya potensi disalahgunakan seperti ini — sumber daya yang sama berharganya untuk attacker maupun defender yang ingin tahu apa yang perlu diwaspadai.

## 9.7 Mitigasi Modern dan Perlombaan Senjata yang Tak Pernah Berhenti

Penting untuk kamu sadari: pertahanan terhadap client-side exploitation terus berkembang, dan attacker terus beradaptasi mengikutinya — ini perlombaan senjata yang nggak akan pernah benar-benar "selesai".

**Mark-of-the-Web** yang sudah kita bahas tadi terus diperkuat — bahkan file di dalam arsip ZIP yang diekstrak dari file MOTW akan mewarisi tanda tersebut di versi Windows terbaru (dulu ini celah populer: kompres file berbahaya ke ZIP untuk "menghilangkan" MOTW-nya).

**AMSI (Antimalware Scan Interface)** adalah lapisan yang memungkinkan software keamanan memeriksa script (PowerShell, VBA, dan sejenisnya) sebelum benar-benar dieksekusi, bahkan setelah proses deobfuscation di runtime — sebelumnya, teknik obfuscation script sederhana sering cukup untuk lolos dari deteksi signature statis.

Poin pentingnya: setiap kali satu mitigasi baru diperkenalkan, komunitas riset keamanan (baik pihak baik maupun jahat) akan mencari cara menghindarinya. Sebagai calon profesional keamanan, penting untuk selalu mengikuti perkembangan ini — apa yang kamu pelajari hari ini soal state-of-the-art mitigasi bisa saja sudah ada teknik bypass barunya dalam hitungan bulan.

## Rangkuman Bab

- Client-side attack surface menyerang aplikasi yang dipakai korban, bukan server — butuh interaksi korban untuk berhasil.
- Trust boundary menjelaskan kenapa file dari internet diperlakukan berbeda dari file lokal (Mark-of-the-Web).
- Makro Office tetap jadi vektor populer, meski dilindungi berlapis (Protected View + Enable Content).
- Windows Library Files adalah contoh vektor "tidak biasa" yang memanfaatkan fitur jarang diperhatikan.
- LOLBins memanfaatkan binary legitimate bawaan OS untuk menghindari deteksi berbasis "software asing".
- Mitigasi (MOTW, AMSI) terus berkembang, dan attacker terus mencari cara menghindarinya — perlombaan senjata yang berkelanjutan.

## Checkpoint Mandiri

1. Bisakah kamu menjelaskan kenapa client-side exploitation butuh kombinasi dengan social engineering, berbeda dengan kebanyakan server-side exploitation?
2. Bagaimana Mark-of-the-Web bekerja, dan sebutkan satu contoh historis teknik yang pernah dipakai untuk mencoba menghindarinya?
3. Kenapa LOLBins sulit diblokir sepenuhnya oleh solusi antivirus/EDR tanpa mengganggu fungsi normal sistem?

## Tantangan Mastery

Di VM Windows terisolasi milikmu (lihat modul praktikum), buat dokumen Office dengan makro proof-of-concept yang **benar-benar harmless** (cukup menampilkan message box atau membuka `calc.exe` — jangan lebih dari itu), amati sendiri seluruh lapisan peringatan yang muncul dari sisi Windows/Office (Protected View, Enable Content, dsb.), lalu jelaskan dari sudut pandang blue team: mitigasi apa yang bisa diterapkan organisasi untuk mengurangi risiko vektor ini (baik teknis maupun non-teknis/awareness).

## Sumber Bacaan Lanjutan

- MITRE ATT&CK — Technique T1204 (User Execution): https://attack.mitre.org/techniques/T1204/
- MITRE ATT&CK — Technique T1566 (Phishing): https://attack.mitre.org/techniques/T1566/
- Microsoft Docs — Mark of the Web dan Protected View
- LOLBAS Project: https://lolbas-project.github.io/

# Step by Step dan Link Youtube

Bab ini butuh satu komponen yang agak beda dari bab-bab sebelumnya: **VM Windows sungguhan**, bukan container Docker (macro Office dan Windows-specific behavior nggak bisa disimulasikan dengan baik di container Linux). Kita akan pakai VM evaluation resmi dan gratis dari Microsoft, dijalankan sepenuhnya terisolasi tanpa akses internet.

## 9.1 Menyiapkan VM Windows Evaluation

Microsoft menyediakan VM evaluation Windows 10/11 Enterprise gratis untuk keperluan testing selama 90 hari.

1. Unduh **Windows 11 Enterprise Evaluation VM** (format VirtualBox/VHD) dari halaman resmi Microsoft Evaluation Center (link ada di `README.md` paket resource).
2. Ikuti `import_windows_eval.sh` untuk membantu proses import ke VirtualBox, dikonfigurasi otomatis dengan jaringan **host-only tanpa NAT** (benar-benar tanpa akses internet).
3. Nyalakan VM, selesaikan setup awal Windows (buat akun lokal, skip semua opsi yang butuh koneksi internet/akun Microsoft).

## 9.2 Verifikasi Isolasi Jaringan

Sebelum lanjut, ini langkah **WAJIB**: pastikan VM Windows ini benar-benar tidak punya akses internet.

Dari dalam VM Windows, buka Command Prompt, coba:
```
ping google.com
```

Harus **gagal total**. Kalau berhasil connect, STOP — cek kembali konfigurasi network adapter VM di VirtualBox (harus host-only, bukan NAT atau Bridged).

## 9.3 Instalasi Microsoft Office (Jika Diperlukan)

Karena lisensi Office tidak bisa didistribusikan ulang, kamu perlu instalasi manual menggunakan installer Office yang kamu punya lisensinya sendiri (trial, Microsoft 365 personal, dsb.) — panduan lengkap ada di `README.md`. Kalau tidak punya akses Office sama sekali, LibreOffice (yang mendukung sebagian macro VBA lewat kompatibilitas terbatas) bisa jadi alternatif untuk memahami konsep dasarnya, meski perilaku persisnya tidak akan 100% identik dengan Office asli.

## 9.4 Membuat Dokumen dengan Makro PoC Harmless

1. Buka Microsoft Word, buat dokumen baru, simpan sebagai **Word Macro-Enabled Document (`.docm`)**.
2. Buka Developer Tab (aktifkan dulu dari File → Options → Customize Ribbon jika belum terlihat) → Visual Basic.
3. Di jendela VBA Editor, cari event `Document_Open` di `ThisDocument`, masukkan kode dari file `macro_poc_harmless.vba` yang sudah disediakan (isinya cuma menampilkan message box, TIDAK ADA fungsi berbahaya apapun).
4. Simpan, tutup dokumen.

## 9.5 Simulasi "Delivery" dan Observasi Peringatan

Untuk mensimulasikan skenario "dokumen dari internet" (bukan file lokal biasa), kita perlu memberi dokumen ini Mark-of-the-Web secara sengaja.

1. Dari mesin Kali/host (bukan VM Windows), jalankan web server sederhana untuk "meng-hosting" dokumen:
   ```bash
   python3 -m http.server 8000
   ```
   (Jalankan ini di folder yang berisi `poc_document.docm` hasil langkah 9.4 yang sudah kamu salin.)
2. Dari dalam VM Windows (yang jaringannya host-only, pastikan bisa menjangkau IP host di jaringan yang sama), buka browser, akses `http://<IP-host>:8000/poc_document.docm`, unduh filenya.
3. Buka file yang baru diunduh tersebut. Amati dengan seksama:
   - Apakah muncul **Protected View** (banner kuning "Enable Editing")?
   - Setelah klik "Enable Editing", apakah muncul lagi peringatan **"Enable Content"** untuk makro?
   - Baru setelah klik "Enable Content", apakah message box PoC muncul?

Dokumentasikan urutan lapisan peringatan ini persis di `01-observasi-motw.md` — screenshot tiap tahap.

## 9.6 Analisis dari Sudut Pandang Blue Team

Setelah berhasil melihat seluruh rantai (delivery → download → MOTW → Protected View → Enable Content → eksekusi), sekarang balik perspektif. Buka `template_analisis_bluteam.md` dan jawab:

- Di titik mana dalam rantai ini organisasi punya kesempatan untuk mencegah/mendeteksi serangan?
- Kontrol teknis apa yang bisa diterapkan (misalnya, kebijakan Group Policy untuk memblokir makro dari internet sepenuhnya)?
- Kontrol non-teknis (awareness training) apa yang relevan, mengingat pembahasan kita di Bab 8?

## Checkpoint Praktikum

- [ ] VM Windows evaluation berhasil di-setup dan **terverifikasi tidak punya akses internet**.
- [ ] Dokumen dengan makro PoC harmless berhasil dibuat.
- [ ] Simulasi delivery via HTTP server berhasil, dokumen berhasil diunduh dengan Mark-of-the-Web.
- [ ] Seluruh lapisan peringatan (Protected View, Enable Content) teramati dan terdokumentasi dengan screenshot.
- [ ] Analisis blue team (`template_analisis_bluteam.md`) sudah terisi lengkap.

## Sumber Daya Pendukung

Semua file teknis — script bantu import VM Windows evaluation ke VirtualBox, kode makro PoC harmless, dan template dokumentasi/analisis — ada di paket resource yang menyertai modul ini.

> **PENGINGAT KERAS**: seluruh praktikum bab ini HANYA dilakukan di VM Windows yang benar-benar terisolasi dari internet. Makro PoC yang disediakan sengaja dibuat 100% harmless (cuma message box) — JANGAN modifikasi jadi payload berbahaya sungguhan, bahkan di lab sendiri, karena di luar konteks pembelajaran murni tentang mekanisme peringatan Windows/Office, itu bukan lagi tujuan modul ini.


## Link YouTube Tutorial

https://youtu.be/w5j8dVtinKc

# Resource Praktikum

Tujuan lab ini: melihat sendiri lapisan peringatan yang muncul saat membuka
dokumen Office bermakro yang "datang dari internet" (Protected View → Enable
Content), lalu menganalisisnya dari sisi defender.

Makro PoC-nya cuma menampilkan message box. Tidak ada download, tidak ada
eksekusi command, tidak ada payload. Jangan diubah jadi payload sungguhan.

## Yang Kamu Butuhkan

Microsoft Word, di **Windows atau macOS** — pilih yang kamu punya. Tidak perlu VM.
Satu-satunya alat tambahan adalah `python3` untuk menjalankan web server kecil
(sudah ada default di macOS; di Windows bisa lewat installer Python biasa).

## Isi Paket

| File | Fungsi |
|---|---|
| `macro_poc_harmless.vba` | Kode makro PoC (message box) |
| `serve_poc_document.sh` | Web server kecil untuk mensimulasikan dokumen "dari internet" |
| `template_observasi_motw.md` | Tempat mencatat peringatan yang kamu lihat |
| `template_analisis_blueteam.md` | Tempat menganalisis dari sisi defender |

## Langkah

**1. Buat dokumennya.**
Buka Word, dokumen baru, simpan sebagai `.docm` (Word Macro-Enabled Document).
Masuk ke VBA Editor (Windows: Developer tab → Visual Basic; Mac: Tools → Macro →
Visual Basic Editor), tempel kode dari `macro_poc_harmless.vba` ke `ThisDocument`.
Simpan, tutup.

**2. Kasih dokumen itu "tanda dari internet".**
Dokumen yang kamu buat sendiri dianggap tepercaya, jadi tidak akan memicu
peringatan. Untuk mensimulasikan file yang diunduh, salin `.docm` tadi ke sebuah
folder lalu jalankan:
```bash
chmod +x serve_poc_document.sh
./serve_poc_document.sh /path/ke/folder
```
Buka browser, akses `http://localhost:8000/`, unduh dokumennya dari situ.

**3. Buka dan amati.**
Buka file hasil unduhan. Perhatikan urutannya: Protected View (banner "Enable
Editing") → peringatan makro ("Enable Content") → baru message box PoC muncul.

**4. Catat dan analisis.**
Isi `template_observasi_motw.md` dan `template_analisis_blueteam.md`.

## Tanda "dari internet" itu disimpan di mana

Kalau mau cek langsung bahwa file-nya memang ditandai:

- **Windows** — klik kanan file → Properties, cari tombol/checkbox "Unblock".
- **macOS** — di Terminal: `xattr -l namafile.docm`, lihat `com.apple.quarantine`.

## Catatan

Makro di sini sengaja dibuat harmless (cuma message box) karena fokusnya memahami
mekanisme peringatan Word, bukan bikin serangan. Variasi `calc.exe` di file makro
hanya jalan di Windows. Jangan ganti dengan executable atau payload apa pun

# Template Analisis Blueteam

## Titik Deteksi/Pencegahan di Rantai Serangan

| Tahap | Peluang deteksi/pencegahan | Kontrol relevan |
|---|---|---|
| Delivery (email/link) | | email filtering, SPF/DKIM/DMARC (Bab 8) |
| Download dokumen | | proxy/web filtering, sandboxing |
| Dokumen dibuka (Protected View) | | Group Policy blokir makro dari internet |
| User klik Enable Content | | awareness training |
| Makro jalan | | AMSI, EDR |

## Rekomendasi Kontrol
- Teknis (kebijakan makro, email/proxy, EDR):
- Non-teknis (awareness, hubungkan dengan Bab 8):

## Kesimpulan
> 2-3 kalimat: kontrol mana yang paling cost-effective untuk organisasi
> kecil-menengah dengan budget keamanan terbatas?

# Template Observasi Peringatan Word

## Info Umum
- OS (Windows / macOS):
- Versi Word:
- Tanggal:

## Peringatan yang Muncul

| Tahap | Muncul? (Ya/Tidak) | Screenshot | Catatan |
|---|---|---|---|
| File ditandai "dari internet" oleh OS | | | |
| Protected View (banner "Enable Editing") | | | |
| Peringatan makro ("Enable Content") | | | |
| Message box PoC muncul (makro jalan) | | | |

## Cek Tanda "dari Internet"
- Windows: klik kanan file → Properties, ada tombol "Unblock"? Isinya apa?
- macOS: `xattr -l namafile.docm`, ada `com.apple.quarantine`? Salin isinya.

## Pertanyaan
- Ada beda perilaku kalau file diekstrak dari ZIP dibanding diunduh langsung?
- Dari semua peringatan tadi, di titik mana user awam paling mungkin tetap klik
  lanjut meski ada peringatan? Kenapa?
