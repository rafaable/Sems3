# Kumpulan Soal Android: Pilihan Ganda dan Isian

Dokumen ini berisi 30 soal pilihan ganda beserta jawaban dan pembahasan, serta 10 soal isian beserta jawabannya.

## A. Soal Pilihan Ganda

## Soal 1

Komponen Android yang menjadi wadah dan mengatur susunan komponen UI di dalamnya disebut ....

a. Intent

**b. ViewGroup**

c. Cursor

d. SharedPreferences

Pembahasan:

ViewGroup adalah komponen Android yang berfungsi sebagai wadah untuk menampung dan mengatur tata letak komponen UI (User Interface). Contohnya adalah LinearLayout, RelativeLayout, dan ConstraintLayout.

---

## Soal 2

Layout yang menyusun komponen secara berurutan dalam satu baris atau satu kolom adalah ....

a. RelativeLayout

b. ConstraintLayout

**c. LinearLayout**

d. ImageView

Pembahasan:

LinearLayout digunakan untuk menyusun komponen UI secara horizontal (satu baris) atau vertikal (satu kolom). Arah penyusunannya ditentukan oleh atribut `android:orientation`.

---

## Soal 3

Agar teks berukuran mengikuti pengaturan ukuran huruf pengguna, satuan yang digunakan pada `android:textSize` adalah ....

a. px

b. dp

c. cm

**d. sp**

Pembahasan:

sp (Scale-independent Pixels) adalah satuan ukuran teks yang mengikuti pengaturan ukuran font pengguna pada perangkat Android. Berbeda dengan dp yang digunakan untuk ukuran komponen UI, sp memperhitungkan preferensi skala font pengguna.

---

## Soal 4

Dalam `onCreate()`, kapan `findViewById()` untuk widget layout Activity sebaiknya dipanggil?

a. Sebelum `super.onCreate()`

b. Sebelum layout dipasang

**c. Setelah `setContentView()` memasang layout yang memuat widget**

d. Hanya setelah `finish()`

Pembahasan:

Method `findViewById()` digunakan untuk mencari komponen UI berdasarkan ID. Method ini sebaiknya dipanggil setelah `setContentView()` karena layout harus dipasang terlebih dahulu agar widget di dalamnya dapat ditemukan.

---

## Soal 5

Aplikasi kalkulator menjalankan kode berikut:

```java
Double.parseDouble(etAngka.getText().toString());
```

Apa langkah yang tepat agar input kosong tidak menyebabkan kegagalan konversi?

a. Mengganti EditText menjadi ImageView

**b. Memeriksa bahwa teks tidak kosong sebelum melakukan konversi**

c. Memanggil `finish()` sebelum membaca input

d. Menghapus ID EditText

Pembahasan:

`Double.parseDouble()` digunakan untuk mengubah String menjadi bilangan desimal bertipe double. Jika teks kosong, proses konversi akan menghasilkan `NumberFormatException`. Oleh karena itu, input harus diperiksa sebelum dikonversi. Input yang tidak kosong tetapi bukan angka juga perlu ditangani.

---

## Soal 6

Komponen yang dibahas dalam materi sebagai representasi satu layar interaktif aplikasi adalah ....

a. Cursor

b. ContentValues

**c. Activity**

d. SQLiteDatabase

Pembahasan:

Activity merupakan komponen utama Android yang merepresentasikan satu layar interaktif dalam aplikasi. Activity mengatur tampilan serta interaksi pengguna dengan komponen UI.

---

## Soal 7

Urutan callback saat Activity baru pertama kali dibuka hingga siap menerima interaksi adalah ....

a. `onResume()` → `onStart()` → `onCreate()`

b. `onCreate()` → `onStop()` → `onDestroy()`

c. `onStart()` → `onDestroy()` → `onResume()`

**d. `onCreate()` → `onStart()` → `onResume()`**

Pembahasan:

Siklus hidup Activity saat pertama kali dibuka dimulai dari `onCreate()` untuk inisialisasi, kemudian `onStart()` ketika Activity mulai terlihat, dan `onResume()` ketika Activity siap berinteraksi dengan pengguna.

---

## Soal 8

Perhatikan kode berikut:

```java
new Intent(MainActivity.this, DetailActivity.class);
```

Kode tersebut merupakan contoh ....

a. Broadcast receiver

**b. Explicit Intent**

c. Implicit Intent

d. Query database

Pembahasan:

Explicit Intent digunakan ketika aplikasi menentukan secara langsung komponen atau kelas tujuan. Pada contoh tersebut, `DetailActivity.class` merupakan kelas Activity yang akan dituju.

---

## Soal 9

Pasangan method untuk mengirim dan membaca data String melalui Intent adalah ....

**a. `putExtra()` dan `getStringExtra()`**

b. `insert()` dan `rawQuery()`

c. `setText()` dan `getCount()`

d. `apply()` dan `commit()`

Pembahasan:

`putExtra()` digunakan untuk mengirim data melalui Intent dengan pasangan key dan value. Sementara itu, `getStringExtra()` digunakan untuk mengambil data String berdasarkan key yang telah diberikan.

---

## Soal 10

Aplikasi ingin membuka dialer dengan nomor sudah terisi, tanpa langsung melakukan panggilan. Action yang paling tepat adalah ....

a. `Intent.ACTION_EDIT`

**b. `Intent.ACTION_DIAL`**

c. `Intent.ACTION_CALL`

d. `Intent.ACTION_SEND`

Pembahasan:

`Intent.ACTION_DIAL` digunakan untuk membuka aplikasi telepon dengan nomor yang sudah terisi tanpa melakukan panggilan secara otomatis. Berbeda dengan `ACTION_CALL` yang digunakan untuk memulai panggilan secara langsung dan memerlukan izin panggilan.

---

## Soal 11

Pilihan penyimpanan yang paling sesuai untuk preferensi sederhana seperti nama pengguna terakhir adalah ....

**a. SharedPreferences**

b. Intent-filter

c. ViewHolder

d. OnClickListener

Pembahasan:

SharedPreferences digunakan untuk menyimpan data sederhana dalam bentuk pasangan key-value, seperti nama pengguna, pengaturan aplikasi, atau preferensi tampilan. Data tetap tersimpan meskipun aplikasi ditutup.

---

## Soal 12

Perbedaan utama `apply()` dan `commit()` pada `SharedPreferences.Editor` adalah ....

a. `apply()` membaca data, sedangkan `commit()` menghapus data

**b. `apply()` menulis ke disk secara asinkron, sedangkan `commit()` menulis secara sinkron dan mengembalikan boolean**

c. Keduanya hanya dapat menyimpan angka

d. `commit()` hanya digunakan untuk SQLite

Pembahasan:

`apply()` memperbarui data di memori segera dan menjadwalkan penulisan ke disk secara asinkron tanpa mengembalikan nilai keberhasilan. Sementara itu, `commit()` melakukan penyimpanan secara sinkron dan mengembalikan boolean untuk menunjukkan keberhasilan operasi.

---

## Soal 13

Fungsi utama ContentValues dalam operasi database adalah ....

**a. Menampung pasangan nama kolom dan nilai untuk insert atau update**

b. Mengatur posisi tombol

c. Memindahkan Activity ke background

d. Menampilkan daftar aplikasi browser

Pembahasan:

ContentValues merupakan kelas Android yang digunakan untuk menyimpan pasangan nama kolom dan nilai sebelum melakukan operasi `insert()` atau `update()` pada database SQLite.

---

## Soal 14

Tabel mhs berisi beberapa mahasiswa. Perhatikan kode berikut:

```java
dbku.delete("mhs", null, null);
```

Apa akibat kode tersebut jika dijalankan dengan sukses?

a. Hanya mahasiswa pertama dihapus

**b. Seluruh baris pada tabel mhs dihapus**

c. Nama tabel berubah menjadi null

d. Database berubah menjadi SharedPreferences

Pembahasan:

Method `delete()` digunakan untuk menghapus data pada tabel SQLite. Ketika parameter kondisi penghapusan bernilai `null`, tidak ada pembatasan baris sehingga seluruh data dalam tabel `mhs` akan dihapus, tetapi struktur tabel tetap ada.

---

## Soal 15

Perhatikan kode SharedPreferences yang menggunakan argumen `"Tamu"`.

Apa fungsi argumen `"Tamu"`?

a. Nama berkas database

b. Key kedua yang harus selalu disimpan

**c. Nilai default jika key username belum tersedia**

d. Perintah untuk menghapus username

Pembahasan:

Pada SharedPreferences, method `getString()` dapat menerima dua parameter, yaitu key dan nilai default. String `"Tamu"` digunakan sebagai nilai pengganti apabila key `username` belum memiliki data tersimpan.

---

## Soal 16

Peran Adapter pada ListView adalah ....

a. Memberikan izin kamera

b. Menutup koneksi database

c. Mengganti nama package aplikasi

**d. Menghubungkan sumber data dengan tampilan setiap baris**

Pembahasan:

Adapter berfungsi sebagai penghubung antara sumber data dan komponen ListView. Adapter mengambil data, kemudian menyediakan tampilan View untuk setiap item yang ditampilkan dalam daftar.

---

## Soal 17

Apa tujuan utama View Recycling pada ListView?

a. Menyimpan semua data ke SharedPreferences

b. Membuat satu Activity untuk setiap item

**c. Memakai ulang View baris yang keluar dari layar untuk menghemat pembuatan View**

d. Menghapus data ketika daftar digulir

Pembahasan:

View Recycling merupakan teknik penggunaan kembali View yang sudah tidak terlihat pada layar. Teknik ini mengurangi pembuatan objek View baru sehingga penggunaan memori lebih efisien dan proses scrolling lebih lancar.

---

## Soal 18

Method custom ArrayAdapter yang bertugas menghasilkan tampilan sebuah baris adalah ....

a. `onUpgrade()`

**b. `getView()`**

c. `startActivity()`

d. `getWritableDatabase()`

Pembahasan:

Method `getView()` digunakan untuk membuat atau menggunakan kembali tampilan setiap baris pada ListView. Method ini dapat mengatur isi widget berdasarkan data pada posisi tertentu.

---

## Soal 19

Manfaat utama ViewHolder Pattern dalam custom adapter adalah ....

a. Mengubah ListView menjadi SQLite

b. Menghilangkan kebutuhan sumber data

c. Membuka dialog secara otomatis untuk semua baris

**d. Menyimpan referensi widget agar pencarian findViewById() tidak perlu diulang saat View dipakai kembali**

Pembahasan:

ViewHolder Pattern digunakan untuk menyimpan referensi komponen UI dalam sebuah objek. Dengan demikian, pemanggilan `findViewById()` tidak perlu dilakukan berulang kali setiap kali View digunakan kembali, sehingga kinerja ListView lebih efisien.

---

## Soal 20

Agar kontak yang disimpan dapat tampil lagi setelah aplikasi ditutup total dan dibuka kembali, langkah yang diperlukan adalah ....

a. Menambah kontak hanya ke variabel lokal

**b. Menyimpan kontak ke SQLite, lalu memuat data tersimpan ke adapter saat aplikasi dibuka**

c. Menyimpan teks tombol Tambah

d. Memanggil `dialog.dismiss()` tanpa menyimpan data

Pembahasan:

SQLite digunakan untuk menyimpan data secara permanen pada perangkat. Ketika aplikasi dibuka kembali, data kontak harus dibaca dari database dan dimasukkan ke adapter agar dapat ditampilkan kembali pada ListView.

---

## Soal 21

Selain memulai Activity, dua penggunaan dasar Intent yang disebutkan pada materi adalah ....

**a. Memulai Service dan mengirim broadcast**

b. Membuat tabel dan menghapus database

c. Mengatur ukuran font dan margin

d. Mengompres gambar dan memutar layar

Pembahasan:

Intent merupakan objek pesan pada Android yang digunakan untuk meminta suatu aksi dari komponen aplikasi. Selain memulai Activity, Intent juga dapat digunakan untuk memulai Service dan mengirim broadcast.

---

## Soal 22

Callback yang dijalankan sebelum Activity yang sebelumnya berhenti (stopped) memasuki `onStart()` kembali adalah ....

a. `onCreate()`

**b. `onRestart()`**

c. `onDestroy()`

d. `onPause()`

Pembahasan:

`onRestart()` dipanggil ketika Activity yang sebelumnya berada dalam keadaan stopped akan dijalankan kembali. Setelah `onRestart()`, sistem melanjutkan ke `onStart()` dan kemudian `onResume()`.

---

## Soal 23

Sebuah Activity ingin membuka alamat web tanpa menyebut nama kelas browser tertentu. Jenis Intent yang sesuai adalah ....

a. Explicit Intent

b. SQLite Intent

**c. Implicit Intent**

d. Layout Intent

Pembahasan:

Implicit Intent digunakan ketika aplikasi hanya menentukan tindakan yang ingin dilakukan tanpa menyebutkan kelas tujuan tertentu. Sistem Android akan mencari aplikasi yang mampu menangani tindakan tersebut, misalnya browser untuk membuka halaman web.

---

## Soal 24

Perhatikan kode berikut:

```java
new Intent(Intent.ACTION_CALL,
    Uri.parse("tel:08123456789"));
```

Awalan `tel:` menunjukkan bahwa data Intent berupa ....

a. Alamat situs web

b. Nama tabel database

c. Nama layout XML

**d. Nomor telepon**

Pembahasan:

Awalan `tel:` merupakan skema URI yang menunjukkan bahwa data mengacu pada nomor telepon. Android menggunakan informasi tersebut untuk menangani tindakan terkait panggilan telepon.

---

## Soal 25

Jika sistem menemukan beberapa aplikasi yang cocok untuk sebuah implicit Intent dan belum ada pilihan default yang berlaku, apa yang dapat ditampilkan kepada pengguna?

a. Form pembuatan tabel SQLite

b. Semua kode sumber aplikasi

**c. Pilihan aplikasi untuk menangani Intent tersebut**

d. Daftar variabel lokal Activity

Pembahasan:

Ketika beberapa aplikasi mampu menangani Implicit Intent, Android dapat menampilkan dialog pemilihan aplikasi atau resolver. Pengguna kemudian dapat menentukan aplikasi yang ingin digunakan untuk menjalankan tindakan tersebut.

---

## Soal 26

Action yang digunakan dalam materi untuk meminta aplikasi kamera mengambil foto adalah ....

a. `Intent.ACTION_DIAL`

**b. `MediaStore.ACTION_IMAGE_CAPTURE`**

c. `Intent.ACTION_EDIT`

d. `Intent.ACTION_SEND`

Pembahasan:

`MediaStore.ACTION_IMAGE_CAPTURE` merupakan konstanta Intent yang digunakan untuk meminta aplikasi kamera mengambil gambar. Hasil pengambilan foto dapat dikembalikan kepada aplikasi pemanggil.

---

## Soal 27

Dalam pola berikut:

```java
startActivityForResult(intent, kodekamera);
```

Apa fungsi `kodekamera`?

a. Menentukan kualitas kompresi foto

b. Menentukan nomor kamera fisik yang wajib dipakai

c. Mengatur ukuran ImageView

**d. Mengidentifikasi permintaan saat hasil dikembalikan**

Pembahasan:

`kodekamera` berfungsi sebagai request code, yaitu kode pengenal untuk membedakan permintaan hasil dari Activity lain. Pada pola API lama, kode ini digunakan untuk mengenali hasil yang diterima melalui `onActivityResult()`.

---

## Soal 28

Dalam contoh pengambilan foto tanpa EXTRA_OUTPUT, objek gambar dari kode berikut:

```java
data.getExtras().get("data")
```

Di-cast menjadi ....

a. Cursor

b. SQLiteDatabase

**c. Bitmap**

d. ContentValues

Pembahasan:

Bitmap merupakan representasi gambar dalam memori Android. Pada pola pengambilan foto tanpa `EXTRA_OUTPUT`, aplikasi kamera dapat mengembalikan gambar thumbnail dalam bentuk Bitmap melalui key `"data"`.

---

## Soal 29

Fungsi MediaStore.EXTRA_OUTPUT pada Intent pengambilan foto adalah ....

a. Mengganti judul Activity

**b. Menentukan URI tujuan penyimpanan hasil foto**

c. Menambahkan nomor telepon ke kontak

d. Memilih jenis layout layar

Pembahasan:

`MediaStore.EXTRA_OUTPUT` digunakan untuk menentukan URI tempat hasil pengambilan foto akan disimpan. Dengan parameter ini, aplikasi dapat meminta hasil foto disimpan pada lokasi yang telah disiapkan.

---

## Soal 30

Pada kode BitmapFactory.Options, apa tujuan pengaturan berikut?

```java
options.inSampleSize = 2;
```

**a. Memuat versi gambar dengan ukuran lebih kecil untuk mengurangi kebutuhan memori**

b. Mengambil dua foto sekaligus

c. Menggandakan jumlah file foto

d. Mengubah nama foto menjadi 2.jpg

Pembahasan:

`inSampleSize` digunakan untuk mengurangi ukuran gambar ketika proses decoding Bitmap. Nilai 2 umumnya membuat lebar dan tinggi gambar hasil decoding menjadi sekitar setengah ukuran aslinya, sehingga kebutuhan memori berkurang.

---

# B. Soal Isian

## Soal 1

Lengkapi nama widget XML untuk menerima input teks dari pengguna.

```xml
<____
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="Masukkan nama" />
```

**Jawaban: EditText**

---

## Soal 2

Lengkapi method yang dipakai untuk memasang listener klik pada tombol.

```java
btnTambah.____(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        tvHasil.setText("Tombol ditekan");
    }
});
```

**Jawaban: setOnClickListener**

---

## Soal 3

Lengkapi nama kelas objek pesan untuk membuka DetailActivity.

```java
Intent intent = new ____(MainActivity.this, DetailActivity.class);
```

**Jawaban: Intent**

---

## Soal 4

Lengkapi method untuk membaca extra bertipe String dengan key yang sama.

```java
String nama = getIntent().____("NAMA");
```

**Jawaban: getStringExtra**

---

## Soal 5

Lengkapi nama kelas wadah pasangan kolom dan nilai untuk operasi insert.

```java
ContentValues nilai = new ____();
nilai.put("nrp", "S025001001");
nilai.put("nama", "Budi");
dbku.insert("mhs", null, nilai);
```

**Jawaban: ContentValues**

---

## Soal 6

Lengkapi method SQLiteDatabase untuk mengubah nama mahasiswa yang NRP-nya cocok.

```java
ContentValues nilai = new ContentValues();
nilai.put("nama", "Siti");
dbku.____("mhs", nilai, "nrp = ?",
    new String[]{"S025001001"});
```

**Jawaban: update**

---

## Soal 7

Lengkapi method ArrayAdapter yang di-override untuk membentuk tampilan satu baris.

```java
@Override
public View ____(int position, View convertView, ViewGroup parent) {
    return LayoutInflater.from(getContext())
        .inflate(R.layout.item_kontak, parent, false);
}
```

**Jawaban: getView**

---

## Soal 8

Lengkapi method View untuk menyimpan objek ViewHolder agar bisa diambil kembali dengan getTag().

```java
ViewHolder holder = new ViewHolder();
holder.tvNama = convertView.findViewById(R.id.tvNama);
convertView.____(holder);
```

**Jawaban: setTag**

---

## Soal 9

Lengkapi action untuk meminta aplikasi lain menampilkan alamat web.

```java
Intent web = new Intent(Intent.____,
    Uri.parse("https://www.its.ac.id"));
```

**Jawaban: ACTION_VIEW**

---

## Soal 10

Lengkapi konstanta MediaStore untuk meminta pengambilan foto.

```java
Intent kamera = new Intent(MediaStore.____);
```

**Jawaban: ACTION_IMAGE_CAPTURE**
