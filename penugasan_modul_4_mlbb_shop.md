# Penugasan Modul 4
## MLBB Shop: Diamond & BP

*SUDAH WAKTUNYA LOGIN ML*  
*MINGGIR IMO MAU LEWAT*

---

## Ketentuan Umum
1. Soal dirilis hari Kamis, 29 Oktober 2026 pukul 12.45. Pengerjaan sampai dengan hari Minggu, 1 November 2026 pukul 23.59.
2. Pengerjaan sesuai dengan materi Modul 4 dan requirement pada dokumen ini.
3. Apabila ada revisi requirement, asisten akan mengumumkannya di Discord.
4. Setiap anggota kelompok harus menerima beban tugas yang sama rata.
5. DIWAJIBKAN membuat repository GitHub public dengan format: `pweb-express-[nomor kelompok]-2026`.
6. Link repository dikumpulkan pada Google Form yang disediakan.
7. Praktikum bersifat berkelompok. Kerja sama dengan kelompok lain tidak diperkenankan.
8. Seluruh anggota kelompok wajib mengikuti demo dan menjelaskan bagian yang dikerjakannya.
9. File `.env` tidak boleh di-commit ke GitHub.

---

## Penugasan: MLBB Shop
Jamal baru saja mulai main Mobile Legends. Saat membuka menu Shop, ia langsung bingung: ada Diamond, ada Battle Point, ada hero, ada skin. Sebenarnya apa bedanya, dan hero bisa dibeli pakai apa? Kebetulan HMIT ITS sedang menyiapkan turnamen Mobile Legends antar-angkatan, dan banyak peserta baru yang bingung seperti Jamal. Karena itu, HMIT ITS ingin membuat versi sederhana dari toko di dalam game Mobile Legends, supaya pemain baru bisa memahami cara kerjanya sebelum turnamen dimulai.

Di toko ini, setiap Player punya dua mata uang:
- **Diamond**: didapat dari top up.
- **Battle Point (BP)**: didapat dari bermain match.

Keduanya bisa dipakai untuk membeli hero, jadi di sinilah Jamal bisa membeli hero pertamanya. Data hero dan skin yang dipakai adalah data asli Mobile Legends: 133 hero (Miya sampai Hirara) dan 892 skin. Skin cukup ditampilkan saja (biar Jamal bisa lihat-lihat), tidak perlu dibuat fitur beli skin.

**Tugas:** Bantu HMIT ITS dan Jamal dengan membuat Backend Express MLBB Shop.

---

## Tech Stack
Wajib menggunakan stack berikut:
- **Express 5 + TypeScript**
- **Prisma ORM 7** (tulis versinya saat install: `prisma@7`)
- **PostgreSQL di Neon**, dihubungkan dengan `@prisma/adapter-neon`

Library tambahan boleh dipakai. Tugas ini tidak memakai login/JWT.

---

## Aturan Diamond dan BP
Baca bagian ini dulu sebelum mulai coding.

### Alur Saldo
- **Cara mendapat:**
  - **Top up:** amount bebas (angka > 0) $\rightarrow$ `+ Diamond` $\rightarrow$ **Diamond** (saldo awal 0)
  - **Main match:** WIN +300 BP, LOSE +100 BP $\rightarrow$ `+ BP` $\rightarrow$ **Battle Point (BP)** (saldo awal 10.000)
- **Cara memakai:**
  - **Beli hero:** pilih BP atau Diamond (harga null = tidak bisa). Mengurangi Diamond (`- Diamond`) atau BP (`- BP`).
- *Skin hanya ditampilkan (katalog), tidak perlu dibuat fitur beli skin.*

| Aturan | Nilai |
| :--- | :--- |
| **Saldo awal player baru** | 10.000 BP dan 0 Diamond |
| **Top up Diamond** | Diamond bertambah sebesar amount. amount harus bilangan bulat lebih dari 0 |
| **Hadiah match** | Menang (WIN) +300 BP, kalah (LOSE) +100 BP |
| **Beli hero** | Boleh pakai BP (harga `priceBP`) atau Diamond (harga `priceDiamond`). Jika harganya kosong (`null`), hero tidak bisa dibeli dengan mata uang tersebut |
| **Satu hero satu kali** | Player tidak boleh membeli hero yang sudah dimilikinya |

*Contoh:* player punya 10.000 BP. Dia membeli Layla seharga 2.000 BP, maka saldonya menjadi 8.000 BP dan Layla masuk ke daftar hero miliknya.

---

## Urutan Pengerjaan
Kerjakan sesuai urutan berikut. Setiap langkah memakai hasil langkah sebelumnya.
1. **Siapkan project.** Buat project Express + TypeScript. Pastikan `npm run dev` berjalan.
2. **Buat database di Neon.** Salin dua connection string (pooled dan direct) ke file `.env`.
3. **Tulis schema Prisma.** Ikuti gambar di bagian Skema Database, lalu jalankan `npx prisma migrate dev --name init` dan `npx prisma generate`.
4. **Salin dataset.** Taruh `heroes.json` dan `skins.json` ke folder `data/` di root project.
5. **Buat seed.** Ikuti bagian Dataset dan Seed, lalu jalankan `npx tsx prisma/seed.ts`. Cek di `npx prisma studio`: harus ada 133 hero, 892 skin, dan 3 player.
6. **Buat endpoint `/health`.** Buka di browser dan pastikan muncul pesan sukses.
7. **Kerjakan Fitur 1 sampai 7** di bagian Spesifikasi Fitur, berurutan.
8. **Uji di Postman atau Platform Sejenis** mengikuti tabel Urutan Testcase di bagian akhir. Bandingkan hasilnya dengan gambar contoh.

---

## Skema Database
Ada 4 tabel dan 4 enum. Terjemahkan gambar berikut ke `prisma/schema.prisma`.  
Nama tabel, kolom, dan nilai enum harus sama persis.

### Struktur Entitas

#### **Hero**
- `id` (PK, Int, autoincrement)
- `name` (String, unique)
- `role` (Role)
- `lane` (Lane)
- `releaseYear` (Int)
- `priceBP` (Int, optional/nullable)
- `priceDiamond` (Int, optional/nullable)
- Relasi: $1 \rightarrow n$ ke `Skin`
- Relasi: $1 \rightarrow n$ ke `PlayerHero`

#### **Skin**
- `id` (PK, Int, autoincrement)
- `name` (String)
- `rarity` (Rarity)
- `priceDiamond` (Int, optional/nullable)
- `heroId` (FK, Int)
- Relasi: $n \rightarrow 1$ ke `Hero` (onDelete: Cascade)

#### **Player**
- `id` (PK, Int, autoincrement)
- `nickname` (String, unique)
- `bp` (Int, default 10000)
- `diamond` (Int, default 0)
- `createdAt` (DateTime, default now)
- Relasi: $1 \rightarrow n$ ke `PlayerHero`

#### **PlayerHero**
- `id` (PK, Int, autoincrement)
- `playerId` (FK, Int)
- `heroId` (FK, Int)
- `currency` (Currency)
- `pricePaid` (Int)
- Constraints: `@@unique([playerId, heroId])`
- Relasi: $n \rightarrow 1$ ke `Player` (onDelete: Cascade)
- Relasi: $n \rightarrow 1$ ke `Hero`

### Enum
- **Role**: `TANK`, `FIGHTER`, `ASSASSIN`, `MAGE`, `MARKSMAN`, `SUPPORT`
- **Lane**: `EXP`, `GOLD`, `MID`, `JUNGLE`, `ROAM`
- **Rarity**: `COMMON`, `EXCEPTIONAL`, `DELUXE`, `EXQUISITE`, `GRAND`, `SUPREME`
- **Currency**: `BP`, `DIAMOND`

### Penjelasan singkat:
- Satu Hero punya banyak Skin.
- PlayerHero mencatat hero yang dimiliki player, beserta mata uang (`currency`) dan harga (`pricePaid`) saat membelinya.
- `Player.bp` default 10000, `Player.diamond` default 0.
- Kolom unik: `Hero.name`, `Player.nickname`, dan pasangan `playerId` + `heroId` di `PlayerHero` (agar satu player tidak punya hero yang sama dua kali).
- Pasang `onDelete: Cascade` pada relasi Skin $\rightarrow$ Hero dan PlayerHero $\rightarrow$ Player. Jadi saat player dihapus, data hero miliknya ikut terhapus.

---

## Dataset dan Seed
Dataset Bisa Diakses Pada Link Berikut:  
[https://drive.google.com/file/d/1pwBUD6HnjybfGc4sobwSDYF7myHA7rW3/view?usp=sharing](https://drive.google.com/file/d/1pwBUD6HnjybfGc4sobwSDYF7myHA7rW3/view?usp=sharing)

| File | Isi | Kolom yang dipakai |
| :--- | :--- | :--- |
| `data/heroes.json` | 133 hero, urut dari Miya sampai Hirara | `name`, `primaryRole` $\rightarrow$ simpan ke `role`, `primaryLane` $\rightarrow$ simpan ke `lane`, `releaseYear`, `priceBP`, `priceDiamond` |
| `data/skins.json` | 892 skin | `name`, `heroId`, `rarity`, `priceDiamond` |

Kolom lain di file JSON tidak perlu disimpan. Buat `prisma/seed.ts` dengan urutan berikut:
1. Kosongkan semua tabel dengan raw query:  
   `TRUNCATE "PlayerHero", "Player", "Skin", "Hero" RESTART IDENTITY CASCADE;`  
   Ini juga mengembalikan id ke angka 1.
2. Masukkan semua hero dengan `createMany`. Id tidak perlu diisi, karena urutan di file sudah benar (Miya mendapat id 1, Hirara id 133).
3. Masukkan semua skin dengan `createMany` (id juga tidak perlu diisi).
4. Masukkan 3 player dan data PlayerHero dari tabel di bawah.

### Data Awal Player
| id | nickname | bp | diamond |
| :---: | :--- | :---: | :---: |
| 1 | ArekITS | 1500 | 75 |
| 2 | SangPushRank | 13000 | 2892 |
| 3 | MageSurabaya | 10000 | 0 |

### Data Awal PlayerHero
| playerId | heroId | currency | pricePaid |
| :---: | :---: | :---: | :---: |
| 1 | 18 (Layla) | BP | 2000 |
| 1 | 1 (Miya) | BP | 10800 |
| 2 | 84 (Ling) | DIAMOND | 599 |
| 2 | 56 (Gusion) | BP | 32000 |

---

## Format Response dan Daftar Endpoint
Semua response wajib memakai format berikut.

**Response sukses:**
```json
{
  "status": true,
  "message": "Berhasil mengambil data hero",
  "data": [...],
  "metadata": { ... }
}
```

**Response gagal:**
```json
{
  "status": false,
  "message": "Hero tidak ditemukan"
}
```

`metadata` hanya ada di endpoint yang memakai pagination, berisi `currentPage`, `limit`, `totalItems`, dan `totalPages`. Isi message boleh berbeda dari contoh, asalkan jelas.

### Seluruh Endpoint
| Fitur | Metode | Endpoint |
| :--- | :--- | :--- |
| **1. Health** | GET | `/health` |
| **2. Hero** | GET | `/api/heroes` |
| **2. Hero** | GET | `/api/heroes/:hero_id` |
| **2. Hero** | POST | `/api/heroes` |
| **3. Skin** | GET | `/api/skins` |
| **3. Skin** | GET | `/api/skins/:skin_id` |
| **4. Player** | POST | `/api/players` |
| **4. Player** | GET | `/api/players/:player_id` |
| **4. Player** | PATCH | `/api/players/:player_id` |
| **4. Player** | DELETE | `/api/players/:player_id` |
| **5. Top Up & Match** | POST | `/api/players/:player_id/topup` |
| **5. Top Up & Match** | POST | `/api/players/:player_id/matches` |
| **6. Beli Hero** | POST | `/api/players/:player_id/heroes` |
| **7. Statistik** | GET | `/api/statistics` |

---

## Spesifikasi Fitur
Setiap fitur dilengkapi gambar contoh hasil yang benar. Angka di gambar mengikuti tabel Urutan Testcase di bagian akhir, jadi uji endpoint sesuai urutan tersebut.

### 1. Middleware
- **Logger**: Setiap request dicatat di terminal, misalnya `GET /api/heroes 200`.
- **Error Handler**: Gunakan `HttpError` dan `errorHandler` agar semua error dibalas dengan format response gagal.
- **Endpoint tidak ada**: Request ke URL yang tidak terdaftar dibalas 404 (middleware `notFound`).

---

### 2. Hero
#### **Get All Heroes (`GET /api/heroes`)**
Tampilkan daftar hero. Dukung query berikut:
| Query | Contoh | Fungsi |
| :--- | :--- | :--- |
| `page`, `limit` | `page=2&limit=5` | Pagination. Default `page=1` dan `limit=10` |
| `search` | `search=la` | Nama hero mengandung kata ini (huruf besar/kecil tidak berpengaruh) |
| `role` | `role=MARKSMAN` | Hanya hero dengan role tersebut |

*Contoh response (`GET /api/heroes?role=MARKSMAN&limit=5`):*
```json
{
  "status": true,
  "message": "Berhasil mengambil data hero",
  "data": [
    {
      "id": 1,
      "name": "Miya",
      "role": "MARKSMAN",
      "lane": "GOLD",
      "releaseYear": 2016,
      "priceBP": 10800,
      "priceDiamond": 399
    },
    {
      "id": 12,
      "name": "Bruno",
      "role": "MARKSMAN",
      "lane": "GOLD",
      "releaseYear": 2016,
      "priceBP": 15000,
      "priceDiamond": 399
    },
    {
      "id": 13,
      "name": "Clint",
      "role": "MARKSMAN",
      "lane": "GOLD",
      "releaseYear": 2016,
      "priceBP": 15000,
      "priceDiamond": 399
    },
    {
      "id": 18,
      "name": "Layla",
      "role": "MARKSMAN",
      "lane": "GOLD",
      "releaseYear": 2016,
      "priceBP": 2000,
      "priceDiamond": 299
    },
    {
      "id": 31,
      "name": "Moskov",
      "role": "MARKSMAN",
      "lane": "GOLD",
      "releaseYear": 2017,
      "priceBP": 32000,
      "priceDiamond": 599
    }
  ],
  "metadata": {
    "currentPage": 1,
    "limit": 5,
    "totalItems": 20,
    "totalPages": 4
  }
}
```

#### **Get Hero Detail (`GET /api/heroes/:hero_id`)**
Tampilkan hero beserta daftar skin-nya (gunakan `include`). Jika tidak ada, balas 404.

#### **Create Hero (`POST /api/heroes`)**
Body berisi: `name`, `role`, `lane`, `releaseYear`, `priceBP` (boleh kosong), `priceDiamond` (boleh kosong).  
Periksa dengan urutan berikut:
1. `name`, `role`, `lane`, atau `releaseYear` tidak diisi $\rightarrow$ 400
2. Nama hero sudah ada $\rightarrow$ 409
3. Berhasil $\rightarrow$ 201 berisi hero baru

---

### 3. Skin
#### **Get All Skins (`GET /api/skins`)**
Tampilkan skin beserta nama hero-nya (gunakan `include`). Dukung query berikut:
| Query | Contoh | Fungsi |
| :--- | :--- | :--- |
| `page`, `limit` | `page=2&limit=5` | Pagination. Default `page=1` dan `limit=10` |
| `heroId` | `heroId=18` | Hanya skin milik hero tersebut |
| `rarity` | `rarity=GRAND` | Hanya skin dengan rarity tersebut |

*Contoh response (`GET /api/skins?heroId=18`):*
```json
{
  "status": true,
  "message": "Berhasil mengambil data skin",
  "data": [
    {
      "id": 25,
      "name": "Green Flash",
      "rarity": "COMMON",
      "priceDiamond": 269,
      "heroId": 18,
      "hero": {
        "name": "Layla"
      }
    },
    {
      "id": 33,
      "name": "Bunny Babe",
      "rarity": "EXCEPTIONAL",
      "priceDiamond": null,
      "heroId": 18,
      "hero": {
        "name": "Layla"
      }
    }
    // ... data lainnya
  ],
  "metadata": {
    "currentPage": 1,
    "limit": 10,
    "totalItems": 11,
    "totalPages": 2
  }
}
```

#### **Get Skin Detail (`GET /api/skins/:skin_id`)**
Tampilkan skin beserta hero-nya. Jika tidak ada, balas 404.

---

### 4. Player
#### **Create Player (`POST /api/players`)**
- Body hanya berisi: `nickname`.
- Saldo otomatis 10000 BP dan 0 Diamond (dari default schema).
- Validasi:
  - `nickname` kosong $\rightarrow$ 400
  - sudah dipakai $\rightarrow$ 409
  - berhasil $\rightarrow$ 201

*Contoh request body:*
```json
{
  "nickname": "Bankai"
}
```

*Contoh response (201):*
```json
{
  "status": true,
  "message": "Player berhasil dibuat",
  "data": {
    "id": 5,
    "nickname": "Bankai",
    "bp": 10000,
    "diamond": 0,
    "createdAt": "2026-10-29T04:28:06.547Z"
  }
}
```

#### **Get Player Detail (`GET /api/players/:player_id`)**
Tampilkan data player beserta daftar hero yang dimilikinya (lihat gambar di Fitur 6). Jika tidak ada, balas 404.

#### **Update Player (`PATCH /api/players/:player_id`)**
Mengganti `nickname`.
- Player tidak ada $\rightarrow$ 404
- `nickname` kosong $\rightarrow$ 400
- sudah dipakai player lain $\rightarrow$ 409

#### **Delete Player (`DELETE /api/players/:player_id`)**
Menghapus player beserta data hero miliknya. Jika tidak ada, balas 404.

---

### 5. Top Up dan Match
#### **Top Up (`POST /api/players/:player_id/topup`)**
Body berisi `amount`.
1. `amount` bukan bilangan bulat lebih dari 0 $\rightarrow$ 400
2. Player tidak ditemukan $\rightarrow$ 404
3. Berhasil: tambah diamond player dengan `increment`, lalu balas 200 berisi saldo Diamond terbaru.

*Contoh request body (`POST /api/players/3/topup`):*
```json
{
  "amount": 706
}
```

*Contoh response (200):*
```json
{
  "status": true,
  "message": "Top up berhasil",
  "data": {
    "playerId": 3,
    "amount": 706,
    "diamond": 1412
  }
}
```

#### **Match (`POST /api/players/:player_id/matches`)**
Body berisi `result` (`"WIN"` atau `"LOSE"`).
1. `result` bukan WIN/LOSE $\rightarrow$ 400
2. Player tidak ditemukan $\rightarrow$ 404
3. Berhasil: tambah bp player (WIN 300, LOSE 100) dengan `increment`, lalu balas 200 berisi BP yang didapat dan saldo BP terbaru.

*Contoh request body (`POST /api/players/3/matches`):*
```json
{
  "result": "WIN"
}
```

*Contoh response (200):*
```json
{
  "status": true,
  "message": "Hadiah match berhasil ditambahkan",
  "data": {
    "playerId": 3,
    "result": "WIN",
    "bpEarned": 300,
    "bp": 10600
  }
}
```

---

### 6. Beli Hero
Ini fitur terpenting. Player memilih mau bayar pakai BP atau Diamond.

**Endpoint:** `POST /api/players/:player_id/heroes`  
**Body:** `heroId` dan `currency` (`BP` atau `DIAMOND`)

#### Urutan pemeriksaan:
Periksa satu per satu. Berhenti di pemeriksaan pertama yang gagal.
1. `heroId` kosong, atau `currency` bukan BP/DIAMOND $\rightarrow$ 400
2. Player tidak ditemukan $\rightarrow$ 404
3. Hero tidak ditemukan $\rightarrow$ 404
4. Harga hero untuk currency tersebut kosong $\rightarrow$ 400 *(contoh: Freya tidak punya harga BP)*
5. Player sudah memiliki hero tersebut $\rightarrow$ 409
6. Saldo player kurang dari harga hero $\rightarrow$ 400

#### Simpan dengan transaction:
Gunakan `prisma.$transaction([...])` berisi dua query berikut, agar keduanya berhasil bersama atau gagal bersama:
1. Kurangi saldo bp atau diamond player sesuai currency (gunakan `decrement`).
2. Buat data `PlayerHero` dengan `currency` dan `pricePaid`.

**Response:** 201 berisi hero yang dibeli, mata uang, harga, dan saldo BP serta Diamond terbaru.

*Contoh request body (`POST /api/players/3/heroes`):*
```json
{
  "heroId": 22,
  "currency": "DIAMOND"
}
```

*Contoh response (201):*
```json
{
  "status": true,
  "message": "Hero berhasil dibeli",
  "data": {
    "hero": {
      "id": 22,
      "name": "Freya"
    },
    "currency": "DIAMOND",
    "pricePaid": 599,
    "balance": {
      "bp": 10600,
      "diamond": 813
    }
  }
}
```

*Contoh jika hero tidak punya harga untuk mata uang yang dipilih (`POST /api/players/3/heroes`):*
```json
// Request: { "heroId": 22, "currency": "BP" }
// Response (400):
{
  "status": false,
  "message": "Hero Freya tidak bisa dibeli dengan BP"
}
```

*Contoh detail player setelah membeli hero (`GET /api/players/3`):*
```json
{
  "status": true,
  "message": "Berhasil mengambil data player",
  "data": {
    "id": 3,
    "nickname": "MageSurabaya",
    "bp": 10600,
    "diamond": 813,
    "createdAt": "2026-10-27T04:24:24.200Z",
    "heroes": [
      {
        "heroId": 22,
        "name": "Freya",
        "currency": "DIAMOND",
        "pricePaid": 599
      }
    ]
  }
}
```

---

### 7. Statistik
**Endpoint:** `GET /api/statistics`

| Field | Isi | Petunjuk |
| :--- | :--- | :--- |
| `totalPlayers` | Jumlah player | `count` |
| `totalHeroesOwned` | Jumlah data di tabel PlayerHero | `count` |
| `totalDiamond` | Total saldo Diamond semua player | `aggregate` dengan `_sum` |
| `mostOwnedHero` | Hero yang paling banyak dimiliki player, beserta jumlah pemiliknya | `groupBy` pada PlayerHero, urutkan `_count` terbesar |
| `heroesPerRole` | Jumlah hero untuk setiap role | `groupBy` pada Hero berdasarkan `role` |

*Contoh response (200):*
```json
{
  "status": true,
  "message": "Berhasil mengambil statistik",
  "data": {
    "totalPlayers": 5,
    "totalHeroesOwned": 5,
    "totalDiamond": 3780,
    "mostOwnedHero": {
      "heroId": 1,
      "name": "Miya",
      "owners": 1
    },
    "heroesPerRole": [
      { "role": "FIGHTER", "total": 37 },
      { "role": "MAGE", "total": 26 },
      { "role": "ASSASSIN", "total": 20 },
      { "role": "MARKSMAN", "total": 20 },
      { "role": "TANK", "total": 18 },
      { "role": "SUPPORT", "total": 13 }
    ]
  }
}
```

---

## Urutan Testcase
Jalankan seed ulang sebelum demo, lalu uji di Postman atau Platform Sejenis dengan urutan berikut.  
**Tinggal IMPORT AJA YA GESS:**  
[https://drive.google.com/file/d/1bi5FbMwt2Ai9oBqhLwg2fAm7rSr5xh6c/view?usp=drive_link](https://drive.google.com/file/d/1bi5FbMwt2Ai9oBqhLwg2fAm7rSr5xh6c/view?usp=drive_link)