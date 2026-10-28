
## 📚 Daftar Isi

1. [README](#readme)
2. [Bab 8 (Praktikum) — Simulasi Phishing (Lab Tertutup)](#bab-8-praktikum--simulasi-phishing-lab-tertutup)
3. [Step by Step & Tutorial Phishing & Rekayasa Sosial](#step-by-step--tutorial-phishing--rekayasa-sosial)
4. [Resource Praktikum — Bab 8: Simulasi Phishing (Lab Tertutup)](#resource-praktikum--bab-8-simulasi-phishing-lab-tertutup)
5. [Laporan Simulasi Phishing — Bab 8](#laporan-simulasi-phishing--bab-8)

# README

Isi paketnya:

- **`Bab_08_Ringkasan_Textbook_Phishing_dan_Rekayasa_Sosial.md`** — psikologi social engineering (urgensi/otoritas/kelangkaan), ragam vektor (email/smishing/vishing/quishing), SPF/DKIM/DMARC sebagai pertahanan domain, credential phishing klasik vs Adversary-in-the-Middle (pembajakan sesi meski MFA aktif), dampak AI generatif/deepfake, dan cara mengukur+melaporkan kampanye secara etis.
- **`Bab_08_Modul_Praktikum_Simulasi_Phishing.md`** — praktikum **GoPhish + MailHog** sepenuhnya di jaringan tertutup.
- **`resources/`**:
    - `docker-compose.yml` — jaringan Docker saya set **`internal: true`** (isolasi total, tanpa akses internet sama sekali) sebagai pengaman teknis, bukan cuma instruksi tertulis.
    - `landing_page_contoh.html` & `redirect_page_contoh.html` — halaman generik fiktif (bukan kloningan layanan nyata manapun), dan halaman redirect-nya secara eksplisit **mengedukasi "korban"** bahwa itu simulasi, sekaligus mengajarkan tanda-tanda phishing.
    - `email_template_contoh.html` — pretext netral bergaya notifikasi IT internal, ditulis untuk keperluan edukasi, bukan dioptimalkan untuk menipu semaksimal mungkin.
    - `get_gophish_credentials.sh` — bantu ambil kredensial admin default.
    - `template_laporan_simulasi_phishing.md` — dengan penekanan pada rekomendasi konstruktif, bukan "daftar korban yang gagal".
    - `README.md` — dibuka dengan **peringatan keras** (aturan mutlak) sebelum langkah teknis apapun

# Step by Step & Tutorial Phishing & Rekayasa Sosial

Ada fakta yang agak bikin rendah hati kalau kamu sudah lama berkecimpung di dunia teknis: setelah semua yang kita pelajari soal SQL Injection, buffer overflow, dan berbagai teknik eksploitasi canggih di bab-bab lain, cara paling efektif untuk masuk ke sebuah organisasi seringkali justru... cuma dengan mengirim satu email yang meyakinkan. Nggak butuh nol-day, nggak butuh exploit rumit. Cukup satu klik dari satu orang yang sedang buru-buru atau lengah.

Ini bukan berarti skill teknis nggak penting — justru phishing yang efektif sering dikombinasikan dengan skill teknis untuk membuat pretext lebih meyakinkan, atau untuk mengeksploitasi apa yang didapat setelah korban "termakan". Tapi bab ini mengajak kamu melihat sisi manusia dari keamanan siber, yang seringkali jadi titik terlemah dibanding sistem teknis manapun.

## 8.1 Psikologi di Balik Social Engineering

Kenapa phishing terus berhasil meski sudah puluhan tahun ada dan sudah banyak kampanye awareness? Karena teknik ini nggak menyerang sistem, tapi menyerang bias kognitif yang memang secara natural ada di setiap manusia.

**Urgensi** adalah prinsip paling sering dipakai — "Akun Anda akan ditangguhkan dalam 24 jam", "Transaksi mencurigakan terdeteksi, verifikasi segera". Ketika seseorang merasa terburu-buru, kemampuan berpikir kritis secara alami menurun — kita cenderung bereaksi dulu, mikir belakangan.

**Otoritas** memanfaatkan kecenderungan manusia untuk patuh pada figur yang dianggap berwenang — email yang seolah dari CEO, dari tim IT, atau dari lembaga pemerintah cenderung mendapat kepatuhan lebih tinggi dibanding permintaan dari sumber "biasa".

**Kelangkaan** ("hanya tersisa untuk 10 orang pertama", "penawaran berakhir hari ini") memicu rasa takut kehilangan kesempatan (FOMO), yang lagi-lagi mempercepat pengambilan keputusan tanpa verifikasi menyeluruh.

Memahami prinsip-prinsip ini penting bukan cuma untuk menyusun simulasi yang realistis (dalam konteks red teaming yang sah), tapi juga untuk mengenali pola ini saat kamu sendiri jadi target — dan yang lebih penting lagi, untuk mengajarkan orang lain mengenali pola ini lewat security awareness training.

## 8.2 Ragam Vektor: Bukan Cuma Email

Ketika orang dengar kata "phishing", biasanya yang terbayang cuma email. Padahal spektrumnya jauh lebih luas.

**Email phishing** tetap yang paling umum, tapi ada variasi **spear phishing** yang jauh lebih ditargetkan — disusun khusus berdasarkan riset OSINT terhadap individu atau organisasi spesifik (ingat Bab 2? Di sinilah OSINT yang kamu kumpulkan benar-benar berguna).

**Smishing** adalah phishing lewat SMS, memanfaatkan kepercayaan orang terhadap notifikasi singkat yang biasanya dari bank atau layanan pengiriman.

**Vishing** dilakukan lewat panggilan suara — sering menyamar sebagai staf bank, dukungan teknis, atau bahkan kolega internal.

**Quishing** adalah istilah yang lebih baru, memanfaatkan QR code berbahaya — menarik karena QR code secara visual "menyembunyikan" URL asli sampai benar-benar di-scan, membuat verifikasi visual jadi lebih sulit dibanding link teks biasa.

## 8.3 Anatomi Email Phishing dan Pertahanan Domain

Untuk memahami cara menyerang (dan bertahan), kamu perlu paham dulu bagaimana email seharusnya diverifikasi keasliannya.

**Domain spoofing** dan **typosquatting/homoglyph** adalah teknik untuk membuat domain pengirim terlihat meyakinkan — mengganti huruf `l` dengan `1`, `o` dengan `0`, atau mendaftarkan domain yang mirip banget dengan domain asli (`bank-negara-indonesia.com` alih-alih domain resmi, misalnya).

Untuk melawan ini, ada tiga mekanisme pertahanan domain yang saling melengkapi:

**SPF (Sender Policy Framework)** adalah record DNS yang mendaftarkan server mana saja yang diizinkan mengirim email atas nama domain tersebut. Kalau email datang dari server yang tidak terdaftar di SPF record domain itu, penerima (atau mail server penerima) punya alasan kuat untuk curiga.

**DKIM (DomainKeys Identified Mail)** menambahkan tanda tangan kriptografis ke setiap email yang dikirim, yang bisa diverifikasi penerima menggunakan public key yang dipublikasikan lewat DNS — ini membuktikan email memang benar-benar dikirim dari domain tersebut dan tidak dimodifikasi di tengah jalan.

**DMARC (Domain-based Message Authentication, Reporting & Conformance)** adalah "kebijakan" yang memberi tahu penerima apa yang harus dilakukan kalau email gagal verifikasi SPF/DKIM — apakah harus ditolak, dikarantina, atau cuma dilaporkan.

Ketiga mekanisme ini kalau dikonfigurasi dengan benar oleh organisasi, membuat domain spoofing jauh lebih sulit dilakukan — meski tetap bukan solusi sempurna (karena penyerang masih bisa mendaftarkan domain lookalike yang punya SPF/DKIM/DMARC valid milik mereka sendiri).

## 8.4 Payload dan "Speed Bump" yang Perlu Dilewati

Sebuah kampanye phishing yang berhasil biasanya perlu melewati beberapa lapisan pertahanan sebelum benar-benar sampai dan "berhasil" di sisi korban.

**Inbound email filter** organisasi biasanya sudah cukup canggih mendeteksi pola-pola mencurigakan — dari situ, penyerang terus beradaptasi mencari cara menghindarinya (misalnya lewat teknik yang akan kita bahas soal AI di bagian akhir bab ini).

**Malicious Office macro** tetap jadi vektor populer meski sudah dibahas juga di Bab 9 nanti dari sisi teknisnya — di sini kita fokus ke sisi "pembungkusnya": bagaimana dokumen tersebut dikemas dan disampaikan supaya korban mau membuka dan mengaktifkan macro.

**File berbahaya** biasa disamarkan sebagai dokumen yang terlihat legitimate — invoice, CV, atau dokumen "penting" lainnya yang relevan dengan konteks pekerjaan korban.

**Link berbahaya** sering disamarkan lewat URL shortener, atau teknik yang membuat teks yang ditampilkan berbeda dari URL asli tujuan (perhatikan selalu: apa yang ditampilkan bukan berarti itu URL sungguhan).

## 8.5 Credential Phishing vs Membajak Sesi MFA

Ini pembeda yang penting untuk kamu pahami karena berimplikasi besar ke tingkat kesulitan pertahanan.

**Credential phishing klasik** cuma mengincar username dan password — begitu MFA (Multi-Factor Authentication) diterapkan dengan benar, kredensial curian saja biasanya sudah tidak cukup untuk login.

**Adversary-in-the-Middle (AiTM)** adalah evolusi yang jauh lebih berbahaya — alih-alih cuma menampilkan halaman kredensial palsu, penyerang menempatkan diri secara real-time di antara korban dan layanan asli (proxy), menangkap bukan cuma username/password, tapi juga **session token** yang dihasilkan setelah korban berhasil melewati MFA sungguhan. Dengan session token ini, penyerang bisa membajak sesi yang sudah terautentikasi penuh — MFA sudah dilewati korban sendiri, jadi penyerang nggak perlu mengalahkannya sama sekali, cukup mencuri hasil akhirnya.

Ini salah satu alasan kenapa tren keamanan modern mulai bergeser ke **passwordless authentication** dan **hardware security key** (seperti FIDO2/WebAuthn) yang secara desain lebih resisten terhadap teknik AiTM dibanding kombinasi password + OTP biasa.

## 8.6 AI Generatif dan Deepfake: Mengubah Lanskap Permainan

Beberapa tahun terakhir, satu hal yang paling mengubah dunia phishing adalah kemajuan AI generatif. Dulu, salah satu tanda paling gampang dikenali dari email phishing adalah kesalahan tata bahasa atau terjemahan yang kaku — sekarang, dengan bantuan AI generatif, pretext bisa disusun dengan bahasa yang sempurna, disesuaikan dengan gaya komunikasi organisasi tertentu, bahkan dipersonalisasi massal dalam skala yang dulu butuh riset manual berjam-jam per target.

**Voice cloning** dan **deepfake video** membawa vishing ke level yang benar-benar baru — sudah ada kasus nyata di berbagai belahan dunia di mana suara eksekutif perusahaan di-clone untuk meminta transfer dana mendesak ke staf keuangan, dan berhasil karena terdengar (dan terlihat, dalam beberapa kasus video call) sangat meyakinkan.

Implikasinya untuk security awareness training: verifikasi "dengar suaranya, kelihatan meyakinkan" sudah tidak lagi cukup sebagai patokan. Organisasi perlu membangun prosedur verifikasi independen (misalnya, konfirmasi lewat kanal komunikasi terpisah untuk permintaan finansial yang tidak biasa) yang tidak bergantung semata pada "apakah ini terdengar/terlihat seperti orang yang saya kenal".

## 8.7 Mengukur dan Melaporkan Kampanye Secara Etis

Kalau kamu ditugaskan menjalankan simulasi phishing resmi untuk klien, ada beberapa metrik standar yang biasa dilaporkan: **open rate** (berapa persen yang membuka email), **click rate** (berapa persen yang klik link), **submit rate** (berapa persen yang benar-benar memasukkan kredensial), dan **report rate** (berapa persen yang justru melaporkan email tersebut sebagai mencurigakan — metrik yang sebenarnya paling menggembirakan buat tim keamanan).

Penting digarisbawahi: tujuan simulasi phishing yang etis bukan untuk "menjebak" atau mempermalukan karyawan yang termakan, tapi untuk mengidentifikasi kebutuhan training tambahan dan mengukur efektivitas program awareness dari waktu ke waktu. Laporan hasil simulasi sebaiknya disampaikan secara agregat dan konstruktif, bukan sebagai daftar nama yang "gagal" untuk dipermalukan.

## Rangkuman Bab

- Phishing menyerang bias kognitif manusia (urgensi, otoritas, kelangkaan), bukan sistem teknis.
- Vektornya beragam: email, smishing, vishing, quishing — masing-masing punya karakteristik berbeda.
- SPF, DKIM, dan DMARC saling melengkapi sebagai pertahanan domain terhadap spoofing.
- Adversary-in-the-Middle jauh lebih berbahaya dari credential phishing klasik karena bisa membajak sesi yang sudah melewati MFA.
- AI generatif dan deepfake mengubah lanskap phishing secara fundamental — verifikasi berbasis "kedengarannya/kelihatannya meyakinkan" sudah tidak cukup.
- Simulasi phishing yang etis fokus pada perbaikan sistemik, bukan mempermalukan individu.

## Checkpoint Mandiri

1. Bisakah kamu menjelaskan bedanya SPF, DKIM, dan DMARC dalam satu paragraf masing-masing, dan bagaimana ketiganya saling melengkapi?
2. Kenapa Adversary-in-the-Middle dianggap jauh lebih berbahaya dibanding phishing kredensial klasik, meski organisasi sudah menerapkan MFA?
3. Bagaimana kemajuan AI generatif mengubah cara organisasi seharusnya melatih karyawan mengenali phishing?

## Tantangan Mastery

Rancang **di atas kertas/dokumen saja** (tanpa dikirim ke siapapun di luar lab) sebuah skenario kampanye phishing lengkap untuk keperluan latihan perencanaan red team: tentukan pretext yang akan dipakai, kalender pengiriman bertahap, dan metrik keberhasilan yang akan diukur. Fokus latihan ini adalah kemampuan perencanaan strategis, bukan eksekusi terhadap target sungguhan.

## Sumber Bacaan Lanjutan

- SANS Security Awareness — sumber riset dan tren phishing terkini
- NIST SP 800-177, panduan keamanan email termasuk SPF/DKIM/DMARC
- Laporan tahunan tren phishing dari penyedia security awareness training (mis. laporan publik tahunan yang sering dirilis vendor-vendor besar di bidang ini)

> **Catatan penting**: materi bab ini murni untuk pemahaman defensif maupun ofensif dalam ruang lingkup lab tertutup atau engagement resmi dengan izin eksplisit. Simulasi kredensial harvesting yang akan kita praktikkan di modul praktikum HANYA dilakukan pada infrastruktur lab isolated milik sendiri.

# Bab 8 (Praktikum) — Simulasi Phishing (Lab Tertutup)

Bab praktikum ini beda dari bab-bab sebelumnya dalam satu hal penting: kita nggak akan pernah, dalam kondisi apapun, mengarahkan simulasi ini ke penerima atau domain sungguhan di luar laptop kamu sendiri. Seluruh latihan berjalan di jaringan tertutup, mengirim "email" hanya ke mail server palsu yang juga jalan lokal di lab kamu.

Kenapa segitu ketatnya? Karena tools yang kita pakai di bab ini — **GoPhish** — adalah tool simulasi phishing sungguhan yang dipakai tim keamanan profesional untuk engagement resmi. Powerful, tapi karena itu juga harus diperlakukan dengan sangat hati-hati bahkan dalam konteks belajar.

## 8.1 Menjalankan Infrastruktur Lab

```bash
docker compose up -d
```

Ini menjalankan dua komponen:
- **GoPhish** — platform simulasi phishing, admin panel di `https://localhost:3333` (HTTPS dengan sertifikat self-signed, browser akan komplain, itu normal untuk lab lokal).
- **MailHog** — mail server palsu yang menangkap SEMUA email yang "dikirim" GoPhish, tanpa pernah benar-benar keluar ke internet. Web interface-nya di `http://localhost:8025`.

Login default GoPhish: cek `README.md` paket resource untuk cara mengambil password admin yang di-generate otomatis saat container pertama kali jalan.

## 8.2 Konfigurasi Sending Profile (Mengarahkan Email ke MailHog)

1. Di GoPhish admin panel, masuk ke menu **Sending Profiles**, buat profile baru.
2. Isi **SMTP Host** dengan alamat container MailHog dan port SMTP-nya (lihat `docker-compose.yml` untuk detail port — MailHog defaultnya listen di port 1025 untuk SMTP).
3. **JANGAN PERNAH** mengisi kredensial SMTP asli (Gmail, Office365, dsb.) di sini — seluruh tujuan lab ini adalah supaya email tidak pernah keluar ke internet sungguhan.

## 8.3 Membuat Landing Page (Halaman Kredensial Palsu, Lab-Only)

Kita pakai halaman contoh yang sudah disediakan di paket resource — sebuah "halaman login webmail korporat generik" fiktif, BUKAN kloningan dari layanan nyata manapun.

1. Di GoPhish, masuk ke menu **Landing Pages**, buat baru.
2. Import HTML dari file `landing_page_contoh.html` yang sudah disediakan.
3. Aktifkan opsi **"Capture Submitted Data"** dan **"Capture Passwords"** — ingat, ini cuma untuk demonstrasi mekanisme capture di lab tertutup, bukan untuk kredensial sungguhan siapapun.
4. Set **redirect URL** ke halaman netral (misalnya halaman "Anda telah berhasil menyelesaikan simulasi keamanan" — ada contohnya juga di paket resource, `redirect_page_contoh.html`).

## 8.4 Membuat Email Template

Gunakan `email_template_contoh.html` yang sudah disediakan sebagai starting point — ini adalah contoh pretext generik bertema "notifikasi keamanan IT internal", ditulis dengan gaya netral untuk keperluan edukasi, bukan pretext yang dioptimalkan untuk menipu secara maksimal.

Import ke menu **Email Templates** di GoPhish.

## 8.5 Membuat Kampanye

1. Masuk ke menu **Users & Groups**, buat grup baru berisi HANYA alamat email dummy (misalnya `korban1@lab.local`, `korban2@lab.local` — tidak perlu benar-benar ada, karena tujuannya cuma masuk ke MailHog).
2. Masuk ke menu **Campaigns**, buat kampanye baru: pilih email template, landing page, sending profile, dan grup yang sudah dibuat.
3. Launch kampanye.

## 8.6 Mengamati Hasil

1. Buka MailHog (`http://localhost:8025`) — kamu akan lihat email yang "terkirim" muncul di sana, lengkap dengan link ke landing page palsu.
2. Klik link tersebut (simulasikan sebagai "korban"), coba masukkan kredensial dummy di halaman landing page.
3. Kembali ke GoPhish dashboard, lihat menu **Campaign Results** — kamu akan melihat statistik open rate, click rate, dan submit rate ter-capture secara real-time.

## 8.7 Menyusun Laporan Simulasi

Pindahkan hasil observasi ke `template_laporan_simulasi_phishing.md`. Ingat prinsip dari textbook: laporan simulasi phishing yang baik itu konstruktif, fokus ke rekomendasi awareness training, bukan daftar "yang gagal" untuk dipermalukan (meski dalam lab ini semua datanya toh cuma dummy buatanmu sendiri).

## Checkpoint Praktikum

- [ ] GoPhish dan MailHog berhasil dijalankan, admin panel bisa diakses.
- [ ] Sending profile berhasil dikonfigurasi mengarah ke MailHog (bukan SMTP asli manapun).
- [ ] Landing page contoh berhasil di-import dan capture data aktif.
- [ ] Kampanye berhasil dijalankan dan email muncul di MailHog.
- [ ] Statistik open/click/submit rate berhasil diamati di GoPhish dashboard.
- [ ] Laporan simulasi (`template_laporan_simulasi_phishing.md`) sudah terisi dengan rekomendasi yang konstruktif.

## Sumber Daya Pendukung

Semua file teknis — `docker-compose.yml` (GoPhish + MailHog), contoh landing page & email template generik (bukan kloningan layanan nyata), dan template laporan — ada di paket resource yang menyertai modul ini.

> **PENGINGAT KERAS**: seluruh infrastruktur bab ini HANYA boleh dijalankan di jaringan lab tertutup di laptop kamu sendiri. Jangan pernah mengisi kredensial SMTP asli, jangan pernah mengarahkan kampanye ke alamat email sungguhan di luar kendali kamu, dan jangan pernah expose port GoPhish/MailHog ke internet.


## YouTube Tutorial Step-by-step 

https://youtu.be/xMAOe5rahl4

# Resource Praktikum — Bab 8: Simulasi Phishing (Lab Tertutup)

## ⚠️ Baca Ini Dulu Sebelum Mulai

Paket ini berisi infrastruktur simulasi phishing sungguhan (GoPhish). Tools ini
dipakai profesional keamanan untuk engagement resmi berizin. Aturan mutlak untuk
penggunaan di buku ini:

1. **Jaringan Docker diset `internal: true`** — GoPhish dan MailHog TIDAK punya akses
   internet sama sekali. Jangan ubah pengaturan ini.
2. **Jangan pernah isi kredensial SMTP asli** (Gmail, Office365, dll.) di sending profile GoPhish.
3. **Jangan pernah gunakan alamat email sungguhan** sebagai target kampanye — pakai domain dummy seperti `@lab.local`.
4. **Jangan pernah expose port 3333/8080/8025/1025 ke internet.**

## Isi Paket

| File | Fungsi |
|---|---|
| `docker-compose.yml` | Menjalankan GoPhish + MailHog di jaringan terisolasi total |
| `Vagrantfile` | Provisioning VM Kali (hanya untuk menjalankan Docker) |
| `provision.sh` | Instalasi Docker |
| `landing_page_contoh.html` | Halaman kredensial palsu generik (bukan kloningan layanan nyata) |
| `redirect_page_contoh.html` | Halaman edukasi yang muncul setelah "korban" submit data |
| `email_template_contoh.html` | Contoh pretext email generik untuk simulasi |
| `get_gophish_credentials.sh` | Bantu temukan kredensial admin default GoPhish |
| `template_laporan_simulasi_phishing.md` | Template laporan hasil simulasi |

## Langkah 1 — Setup

```bash
vagrant up
vagrant ssh
```

atau jika sudah punya VM Docker dari bab sebelumnya, cukup jalankan `sudo ./provision.sh`.

## Langkah 2 — Jalankan Infrastruktur

```bash
docker compose up -d
chmod +x get_gophish_credentials.sh
./get_gophish_credentials.sh
```

## Langkah 3 — Akses Admin Panel

- GoPhish: `https://localhost:3333` (abaikan warning sertifikat self-signed)
- MailHog: `http://localhost:8025`

## Langkah 4 — Ikuti Modul Praktikum

Buka `Bab_08_Modul_Praktikum_Simulasi_Phishing.md` untuk langkah lengkap konfigurasi sending profile, landing page, email template, dan menjalankan kampanye.

## Langkah 5 — Dokumentasikan

```bash
cp template_laporan_simulasi_phishing.md ~/Lab-Notes/Bab-08-Phishing-Sim/laporan-simulasi.md
```

## Membersihkan Lab Setelah Selesai

```bash
docker compose down -v
```

Flag `-v` memastikan volume data (termasuk email yang tertangkap di MailHog dan kredensial dummy yang ter-capture) benar-benar dihapus.

# Laporan Simulasi Phishing — Bab 8

## Info Umum
- **Tanggal simulasi**:
- **Jumlah target dummy**:
- **Pretext yang digunakan**: (mis. "notifikasi verifikasi akun IT internal")

## Metrik Hasil

| Metrik | Jumlah | Persentase |
|---|---|---|
| Email terkirim (tercatat di MailHog) | | 100% |
| Email dibuka (open rate) | | |
| Link diklik (click rate) | | |
| Kredensial dummy disubmit (submit rate) | | |

## Observasi Teknis
- Apakah landing page berhasil menangkap data submission dengan benar?
- Apakah redirect ke halaman edukasi berfungsi sebagaimana mestinya?
- Kendala teknis yang ditemui saat setup (jika ada):

## Analisis Pretext
> Menurutmu, prinsip psikologi apa (urgensi/otoritas/kelangkaan) yang dipakai di pretext contoh ini? Seberapa efektif menurutmu dalam konteks lab ini?

## Rekomendasi Awareness Training (Gaya Laporan Profesional)
> Tulis rekomendasi seolah-olah ini laporan sungguhan ke klien — konstruktif, agregat, fokus pada perbaikan sistemik (bukan menyalahkan individu).
