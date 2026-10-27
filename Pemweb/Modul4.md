## 1. Tujuan pembelajaran & Prasyarat

Intinya, modul ini ngajarin kita bikin backend [bagian aplikasi yang memproses permintaan dan mengelola data] menggunakan Node.js [alat untuk menjalankan JavaScript di server], Express 5 [framework atau kerangka kerja untuk membuat API], dan TypeScript [JavaScript dengan pemeriksaan tipe data].

Kita bakal belajar alur komunikasi aplikasi. Misalnya, 
1. client mengirim request ke server
2. **Express** mengatur permintaan melalui router dan middleware [fungsi perantara untuk memeriksa atau mengolah permintaan].
3. Setelah data diproses, server mengirim response ke client

Komunikasi ini menggunakan **REST API** [Representational State Transfer Application Programming Interface, cara aplikasi saling bertukar data melalui HTTP]  

Untuk menyimpan data, kita pakai **PostgreSQL** [sistem database berbasis tabel] di **Neon Cloud** [layanan database PostgreSQL melalui internet]  
Bedanya, database lokal disimpan di komputer sendiri, sedangkan database cloud berada di server penyedia layanan.

Kita juga belajar **Prisma ORM 7** [Object-Relational Mapping, alat untuk mengakses database lewat kode tanpa harus selalu menulis SQL]  
Di dalamnya ada Prisma schema [rancangan struktur database], migrasi [proses menerapkan perubahan struktur database], dan CRUD  

Terakhir, kita belajar fitur pengolahan data:  
- Filtering [memilih data sesuai kondisi tertentu]  
- Pagination [membagi hasil data menjadi beberapa halaman]  
- Aggregate [menghitung ringkasan data, seperti jumlah atau rata-rata]  
- Transaction [menggabungkan beberapa operasi database supaya semuanya berhasil atau semuanya dibatalkan]  
- Raw query [menulis perintah SQL secara langsung]  
Jadi, target akhirnya kita bisa bikin REST API yang menerima permintaan, mengolah data, menyimpannya di database cloud, dan mengirim hasilnya kembali ke pengguna  

Sebelum mulai praktik, kita perlu menyiapkan beberapa tools.

* **Node.js** adalah alat untuk menjalankan JavaScript di luar browser, misalnya di server. Kita disarankan menggunakan versi 24 **LTS** [Long-Term Support, versi yang mendapat dukungan dan pembaruan jangka panjang]  
Versi 26 juga bisa digunakan, tetapi baru dijadwalkan menjadi LTS pada 28 Oktober 2026  
* **npm** [Node Package Manager] adalah alat untuk menginstal dan mengelola package [kumpulan kode siap pakai] yang dibutuhkan proyek. npm biasanya otomatis terpasang bersama Node.js. Alternatifnya ada **yarn** dan **pnpm** yang punya fungsi serupa  
* **Neon** adalah layanan database PostgreSQL berbasis cloud. Kita cukup membuat akun gratis tanpa kartu kredit  
* **Postman**, **Apidog**, atau **Hoppscotch** digunakan untuk menguji API. Misalnya, kita mengirim request untuk mengambil data pengguna, lalu memeriksa response dari server  
* Terakhir, kita perlu memahami dasar TypeScript, terutama tipe data, function, dan interface dari Modul 3  
Jadi, Node.js untuk menjalankan backend, npm untuk memasang kebutuhan proyek, Neon untuk menyimpan data, dan Postman untuk menguji API yang kita buat

## 2. Gambaran Besar
<img width="1587" height="289" alt="image" src="https://github.com/user-attachments/assets/c9d1159b-9bce-4f07-a5af-75d70f25dbba" />  

Gambar ini menjelaskan **alur komunikasi antara pengguna, backend, dan database**  
Bayangin kita membuka website dan ingin melihat daftar pengguna yang tersimpan  
<br>
Alurnya ada 6 tahap:  
1. **Client** [browser, frontend, atau Postman] mengirim **HTTP request** [permintaan melalui protokol komunikasi web] ke Express  
2. **Express 5** [framework untuk membuat backend] menerima permintaan, lalu memanggil method dari Prisma Client untuk mengambil data  
3. **Prisma Client** [alat untuk berkomunikasi dengan database melalui kode] menggunakan **driver adapter** [penghubung Prisma dengan database] untuk menjalankan SQL ke Neon melalui internet  
4. **Neon** [layanan database PostgreSQL berbasis cloud] mencari data yang diminta, lalu mengembalikan baris data ke Prisma  
5. **Prisma Client** mengubah hasil database menjadi objek JavaScript yang bentuk datanya dikenali TypeScript, lalu mengirimkannya ke Express  
6. **Express** mengirim **JSON response** [balasan berupa data berformat JavaScript Object Notation] ke client supaya hasilnya bisa ditampilkan  
**Intinya:** Express bertugas menerima dan membalas permintaan, Prisma menjadi penghubung antara kode backend dan database, sedangkan Neon menjadi tempat penyimpanan datanya  
Jadi, kita nggak perlu membuat frontend berkomunikasi langsung dengan database. Semua akses data diatur melalui backend

### Versi detail

<img width="1402" height="565" alt="image" src="https://github.com/user-attachments/assets/fa7236d5-a87e-496e-8019-975d50c1d349" />  

Diagram ini menjelaskan **alur lengkap sebuah request**, mulai dari pengguna meminta data sampai mendapatkan balasan  
Misalnya, kita ingin mencari pengguna dengan ID 1 melalui Postman, alurnya ada 8 tahap:  
1. **Client** [Postman] mengirim GET ke /api/users/1. Artinya, kita meminta data pengguna dengan ID 1.
2. **Middleware** menjalankan **logger** [pencatat aktivitas request] dan express.json() [pembaca data JSON dari request body], kemudian next() meneruskan request ke proses berikutnya  
3. **Router** [pengatur jalur request] mencocokkan URL /api/users/1 dengan pola /api/users/:id, lalu meneruskannya ke controller  
4. **Controller** [fungsi yang mengatur logika penanganan request] memanggil prisma.user.findUnique() untuk mencari pengguna berdasarkan ID  
5. **Prisma Client** [alat penghubung kode dengan database] mengirim perintah SQL SELECT ... WHERE id = 1 ke Neon untuk mencari data tersebut  
6. **Neon PostgreSQL** [database cloud] mencari pengguna dengan ID 1, lalu mengembalikan satu baris data jika ditemukan  
7. **Prisma Client** menerima hasil database dan mengembalikannya ke controller dalam bentuk objek JavaScript [data yang disusun menggunakan pasangan key dan value]  
8. **Controller** mengirim response berupa JSON dengan status 200 OK, beserta status, message, dan data pengguna  
Kalau pengguna tidak ditemukan, controller bisa mengirim 404 Not Found, bukan 200 OK.

Bedanya Router dan Controller?  
Ini yang penting supaya nggak ketukar  
**Router** : menentukan request harus ditangani oleh fungsi yang mana  
**Controller** : menentukan apa yang harus dilakukan setelah request diterima  
**Middleware** : menjalankan pemeriksaan atau proses tambahan sebelum request dilanjutkan, misalnya mencatat aktivitas atau memeriksa login  
Jadi, gampangnya, **middleware memeriksa, router mengarahkan, controller mengatur pekerjaan, Prisma mengakses database, dan Neon menyimpan datanya.**

**ExpressJS** adalah framework [kerangka kerja yang menyediakan fitur siap pakai] untuk Node.js [alat menjalankan JavaScript di server]. Jadi, kita nggak perlu membuat backend dari nol atau 'from scratch', dan bisa langsung fokus ke **business logic** [aturan utama aplikasi, misalnya proses login atau pengelolaan data pengguna].

Perbedaannya, kalau menggunakan modul HTTP [Hypertext Transfer Protocol, protokol komunikasi web] bawaan Node.js, kita harus mengatur banyak hal secara manual. Sedangkan Express menyediakan cara yang lebih praktis:
- app.get('/users', ...) : menangani request GET [permintaan mengambil data] ke alamat /users, tanpa banyak pengecekan if.
- app.use(express.json()) : membaca JSON [JavaScript Object Notation, format pertukaran data] dari request body [isi data yang dikirim client] secara otomatis.
- res.status(200).json(...) : mengirim response [balasan server] berbentuk JSON dengan kode 200 [permintaan berhasil], tanpa mengatur header [informasi tambahan HTTP] secara manual.
**Intinya:** Express membuat pengembangan backend lebih simpel, kodenya lebih singkat, dan kita bisa lebih fokus pada fungsi utama aplikasi.
