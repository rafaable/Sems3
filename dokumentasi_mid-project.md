# Pembuktian dan dokumentasi

## Nomor 3
**Seluruh LAN dapat saling berkomunikasi hanya dengan static route (dan default route untuk arah keluar)**  
<br> 
Misal kantor pelabuhan ke barat, tes `ping` dan lihat route yang dilewati dengan `tracert`  
<br><img width="826" height="376" alt="image" src="https://github.com/user-attachments/assets/1da590c8-2b0c-49a2-a72a-2e812eb1b2f1" /><br>   
Sudah sesuai, urutanya dari PC kantor pelabuhan ke R-Pelabuhan, ke R-UTARA, lalu R-BARAT  
<br>
   
**Default route hanya boleh mengarah ke router di atasnya / keluar**. Misalnya, default route R-DC ke R-PUSAT, R-PELABUHAN ke R-UTARA, dan R-PUSAT ke R-ISP  
<br>
R-DC ke R-PUSAT  
<br><img width="710" height="287" alt="image" src="https://github.com/user-attachments/assets/da65f98e-714f-40b1-840c-a435b97c26ab" /><br>  
  
R-Pelabuhan ke R-UTARA  
<br><img width="837" height="293" alt="image" src="https://github.com/user-attachments/assets/d3c6ff98-783b-48e2-91a2-6ae9c42a1c57" /><br>  
  
R-PUSAT ke R-ISP  
<br><img width="727" height="358" alt="image" src="https://github.com/user-attachments/assets/cfe03daf-904e-4ecd-ae18-265eb0bf0c7b" /><br>  
<br>
    
**R-PUSAT punya static route ke semua LAN milik R-DC, R-UTARA, R-PELABUHAN, R-SELATAN, R-TIMUR, dan R-BARAT**  
<br>
<img width="728" height="550" alt="image" src="https://github.com/user-attachments/assets/1b0674c3-41c6-41e7-be0d-c042e8a7ebd3" />  
Ada 8 interface yang ditampilkan, sesuai dengan jumlah 8 port yang terhubung
<br>
   
**R-UTARA punya static route ke semua LAN milik R-PELABUHAN**  
<br>
<img width="846" height="338" alt="image" src="https://github.com/user-attachments/assets/72c73e77-c56e-4fe0-929c-d4865fe16be9" />
<br>
  
## Nomor 4
**Interface router dan seluruh server menggunakan IP statis**  
* Dengan `show ip interface brief`  
  <br><img width="727" height="170" alt="image" src="https://github.com/user-attachments/assets/e631859c-2d24-49b2-82a7-e21fdf317c63" /><br>
  IP terisi pada 3 port dan method manual yang menandakan IP statis diisi manual
* Config server DC  
  <br><img width="635" height="185" alt="image" src="https://github.com/user-attachments/assets/43ece564-6d8f-4ad4-8295-6fe6dd3f11c5" /><br>
  <br><img width="662" height="129" alt="image" src="https://github.com/user-attachments/assets/6b1553a3-fa29-44c3-8fa5-acdcda921945" /><br>  
* Config server balaikota  
  <br><img width="634" height="185" alt="image" src="https://github.com/user-attachments/assets/e9f35ac0-9f7f-4d46-8848-0ceb7689c1d5" /><br>
  <br><img width="661" height="126" alt="image" src="https://github.com/user-attachments/assets/f0fa6c8f-39a8-4f0e-9806-e0f487195061" /><br>  
* Config server internet  
  <br><img width="630" height="191" alt="image" src="https://github.com/user-attachments/assets/3603b037-b201-4e95-b882-140934b62404" /><br>
  <br><img width="664" height="125" alt="image" src="https://github.com/user-attachments/assets/c0775786-32af-47ba-b3ab-d0334646110f" /><br>
  
**Perangkat pengguna mendapat IP dinamis secara otomatis, lengkap dengan gateway dan DNS server (soal 8), tanpa bentrok dengan alamat IP statis**  
Dapat IP dinamis otomatis  
<br><img width="854" height="322" alt="image" src="https://github.com/user-attachments/assets/f4e81ca4-2dac-44c6-b1e3-ea525fd04a3f" /><br>
   
Gateway dikecualikan  
<br><img width="771" height="156" alt="image" src="https://github.com/user-attachments/assets/66a303f1-e057-423d-8f73-591de2ba173a" /><br>
  
**Khusus untuk subnet CCTV Kota (subnet R-DC) mendapatkan alamat dari server DHCP di LAN 'Server dan Storage DC', bukan dari router**  
CCTV DC dapat alamat dari server DC  
<br><img width="757" height="342" alt="image" src="https://github.com/user-attachments/assets/98f7dfe8-7a09-4130-b7f0-50c3c2ea1943" /><br>
Terdapat pool CCTV di server DC  
<br><img width="841" height="531" alt="image" src="https://github.com/user-attachments/assets/21e4a5ed-9c8d-4f1f-b2d7-b211bbda5e93" /><br>
  
## Nomor 5
Ping dari smartphone utara dengan IP 10.113.0.x lalu `show IP translations` dari R-PUSAT menampilkan IP dari smartphone utara
<br><img width="1175" height="627" alt="image" src="https://github.com/user-attachments/assets/e986f7dc-47c5-4117-8f2b-762a79e1da06" /><br>
  
Router tujuan R-ISP tidak ada yang menuju router dengan base 10.112.x.x dan hanya menuju pusat, artinya komunikasi ke R-ISP hanya bisa melalui R-PUSAT
<br><img width="675" height="266" alt="image" src="https://github.com/user-attachments/assets/75f0b127-789b-4de1-98e7-68c493c92004" /><br>

## Nomor 6
**Bangun tunnel GRE antara R-PUSAT dan R-PELABUHAN**  
* Dari R-PUSAT  
  <br><img width="842" height="180" alt="image" src="https://github.com/user-attachments/assets/70f599ba-14d9-487f-a103-07139470073e" /><br>
  <br><img width="843" height="620" alt="image" src="https://github.com/user-attachments/assets/5559a7aa-a0a7-4841-90fc-bfc7c4729b25" /><br>
* Dari R-Pelabuhan
  <br><img width="839" height="186" alt="image" src="https://github.com/user-attachments/assets/f4632e28-eefe-4057-94f5-731872bec591" /><br>
  <br><img width="834" height="358" alt="image" src="https://github.com/user-attachments/assets/1be11efe-1a72-48cd-b533-9ba9ce893cf6" /><br>
  
**Lalu lintas antara subnet Balai Kota dan subnet Pelabuhan harus melewati tunnel**  
<br><img width="499" height="291" alt="image" src="https://github.com/user-attachments/assets/73587b6e-cb87-4bbf-a94f-a48e75cd4823" /><br>
  
**IPSEC**
* Perintah `show crypto isakmp sa` menghasilkan state QM_IDLE, artinya  
  IPsec bernegosiasi dalam dua fase:  
  - Fase 1 (ISAKMP/IKE): kedua router saling autentikasi dengan pre-shared key dan membuat jalur kontrol yang aman  
  - Fase 2 (Quick Mode): lewat jalur itu, kedua router sepakat soal enkripsi untuk data yang sebenarnya (ESP, AES, SHA)  
  QM_IDLE artinya fase 1 sudah selesai, SA-nya aktif, dan jalur kontrol standby (“idle”) menunggu atau melayani negosiasi fase 2  
  <br><img width="542" height="72" alt="image" src="https://github.com/user-attachments/assets/9fd99c09-c888-4c0b-acde-1e2cfa865c4b" /><br>
* `show crypto ipsec sa` harus punya #pkts encaps dan #pkts decaps yang angkanya naik setiap melakukan ping  
  <br><img width="657" height="601" alt="image" src="https://github.com/user-attachments/assets/deb2fd7e-1143-41e0-8c05-8803396f5dd3" /><br>
  Ping kedua  
  <br><img width="657" height="454" alt="image" src="https://github.com/user-attachments/assets/2e6eb8e7-cb2a-484b-b9d8-a0e5f15300f7" /><br>
* Di mode Simulation, kirim ping Pegawai ke Kantor Pelabuhan `10.113.160.2` (atau ip yang didapat dari dhcp) lalu buka PDU di R-UTARA. Header paketnya harus menunjukkan ESP  
  <br><img width="550" height="575" alt="image" src="https://github.com/user-attachments/assets/7e344da0-ff72-4391-9ee2-fd1a9bfb9a9c" /><br>

## Nomor 8
Layanan aplikasi smart city, domain internal : k14.smartcity.lab

**DNS di LAN Server dan Storage DC. Nama www, ftp, dan mail dapat diakses dari seluruh wilayah**  

* DNS di server DC  
  <br><img width="851" height="436" alt="image" src="https://github.com/user-attachments/assets/c3586d2f-76e5-47f3-bd18-cda943f7fb30" /><br>
* NS Lookup pegawai balai
  <br><img width="862" height="553" alt="image" src="https://github.com/user-attachments/assets/a16a70e5-b874-4bbf-ad9e-7177a825cb9f" /><br>
* NS Lookup sensor pelabuhan
  <br><img width="822" height="431" alt="image" src="https://github.com/user-attachments/assets/430b321c-b992-457d-8802-aa1058290c92" /><br>
* NS Lookup PC Selatan  
  <br><img width="821" height="619" alt="image" src="https://github.com/user-attachments/assets/6e486d09-c67b-4cdb-bf4e-64385c601251" /><br>

**Web di LAN Server Balai Kota, berupa halaman "Portal Smart City Surabaya" yang mencantumkan nomor kelompok dan dapat dibuka dengan nama domain dari semua wilayah**

* Service HTTP di server balai  
  <br><img width="860" height="460" alt="image" src="https://github.com/user-attachments/assets/a3cc92f3-fa38-4c4d-8888-4f67343b088a" /><br>
* Pencarian web dengan nama domain, dari PC Timur
  <br><img width="861" height="430" alt="image" src="https://github.com/user-attachments/assets/566d239c-835b-4b84-a45a-3081ae0a4b21" /><br>
* Pencarian web dengan nama domain, dari PC Sensor Pelabuhan
  <br><img width="862" height="441" alt="image" src="https://github.com/user-attachments/assets/f17826db-9734-4c01-aba9-ac11d200d3df" /><br>
* Pencarian web dengan nama domain, dari PC Balai
  <br><img width="862" height="430" alt="image" src="https://github.com/user-attachments/assets/8d77116b-232b-4afb-9a69-b20ff89e3d5e" /><br>

**FTP di LAN Server dan Storage DC sebagai penyimpanan rekaman CCTV, dengan akun login. Perangkat di LAN CCTV Kota dapat mengunggah file ke sana**
  
* FTP di server & storage DC  
  <br><img width="858" height="445" alt="image" src="https://github.com/user-attachments/assets/f40df883-e458-4234-a3d7-dd6f6717cfb9" /><br>
* Login dan pindahkan file yang telah dibuat di text editor  
  <br><img width="851" height="304" alt="image" src="https://github.com/user-attachments/assets/11ca6f9c-acc5-4795-9174-f794e15f43d3" /><br>
* Perintah dir menampilkan file yang telah dipindahkan
  <br><img width="835" height="825" alt="image" src="https://github.com/user-attachments/assets/cb6d0b13-f867-4df5-84bc-6ca9ec983840" /><br>


**Email antarpegawai. Pengguna di Balai Kota dan Pelabuhan dapat saling mengirim dan menerima email**

* Tes email dari pegawai balai ke pelabuhan
  <br><img width="870" height="222" alt="image" src="https://github.com/user-attachments/assets/59612227-110e-4362-a070-5f027a085041" /><br>
* Pelabuhan receive
  <br><img width="852" height="274" alt="image" src="https://github.com/user-attachments/assets/4e2fb951-4450-4d7f-adca-f2b0d70552f3" /><br>
* Buat reply dari pelabuhan
  <br><img width="853" height="253" alt="image" src="https://github.com/user-attachments/assets/e399d72c-c31c-4ca1-9774-4da82be111fc" /><br>
* Reply diterima balai  
  <br><img width="861" height="269" alt="image" src="https://github.com/user-attachments/assets/b1ee705e-2f73-4cc5-b09d-720ed7632598" /><br>

# OSPF
Tes link putus (jangan L6):

Dari PC WiFi Utara jalankan ping 8.8.8.8 -t.
- Di R-PUSAT: `configure terminal`, `interface FastEthernet0/1`, `shutdown`, `end`
- Tunggu sekitar 40 detik. Ping harus pulih lewat L7 (UTARA, DC, lalu L1) - Cek dengan tracert 8.8.8.8
- Jalankan no shutdown, lalu pastikan jalur kembali ke L2
- Ulangi dengan L1 (Ethernet0/0/0 di R-PUSAT) dan tes dari PC CCTV  
  
<br><img width="772" height="288" alt="image" src="https://github.com/user-attachments/assets/421e0e5c-5b00-47c4-83c2-6be3b81f6808" /><br>
  
Dari smartphone-U  
<br><img width="431" height="168" alt="image" src="https://github.com/user-attachments/assets/cabcf135-e11a-4634-9682-fef5c510f52b" /><br>
  
| Hop | Alamat | Router |
|---|---|---|
| 1 | `10.113.0.1` | R-UTARA (gateway WiFi Utara) |
| 2 | `10.112.97.5` | R-DC, sisi **L7** |
| 3 | `10.112.97.1` | R-PUSAT, sisi **L1** |
| 4 | `203.0.113.113` | R-ISP |
| 5 | `8.8.8.8` | Server internet |
  
Saat L2 mati, trafik dari Utara tidak lagi langsung ke PUSAT. Jalurnya memutar lewat L7 ke R-DC, lalu L1 ke R-PUSAT, dan keluar ke internet lewat NAT seperti biasa. Itu yang diminta soal 7: konektivitas pulih otomatis lewat jalur lain tanpa kamu mengubah apa pun  
  
<br><img width="672" height="126" alt="image" src="https://github.com/user-attachments/assets/7abd08f0-36d3-4f91-a2f2-bf6aa4cde2a1" /><br>
Saat shutdown, neighbor 6.6.6.6 dan 7.7.7.7 hilang dan sisanya tetap FULL

Mengembalikan L2
```
configure terminal
interface FastEthernet0/1
 no shutdown
end
```
<br><img width="669" height="156" alt="image" src="https://github.com/user-attachments/assets/ba4dc8af-6b8d-47eb-b4ed-ee41a2644f33" /><br>
Tracert dari utara harus kembali  
<br><img width="425" height="151" alt="image" src="https://github.com/user-attachments/assets/03282998-7fed-4a40-b065-e7e2638c38dc" /><br>

## Memeriksa passive interface
```
! Di R-PUSAT
show ip protocols
```
<br><img width="580" height="484" alt="image" src="https://github.com/user-attachments/assets/f7cc1947-1669-470c-8910-9b410b83e814" /><br>
- Interface LAN menggunakan passive-interface
- Interface antar-router tidak passive
- Jaringan yang diumumkan sudah sesuai
- OSPF berjalan pada Area 0

## Menguji jalur tunnel
Dari PC Balai, `tracert` ke pelabuhan `10.113.160.2`  
<br><img width="525" height="291" alt="image" src="https://github.com/user-attachments/assets/f4cd2320-ddbc-4600-9c50-d647336c67f0" /><br>  
Bukti paket berangkat lewat tunnel : Jalur fisik pasti punya hop R-UTARA (10.113.66.2) di tengah dengan total empat hop. Hasil tracert hanya menunjukkan tiga hop (R-PUSAT, R-PELABUHAN, lalu PC) yang menunjukkan tunnel sudah dipakai  

Jika kita periksa di R-PUSAT, maka akan terlihat bahwa jalur menuju kantor pelabuhan akan melewati IP tunnel dengan keterangan tunnel0  
<br><img width="510" height="114" alt="image" src="https://github.com/user-attachments/assets/eda44e50-30ac-407d-9f5e-d7fe11712664" /><br>  

## Tes internet
```
ping 8.8.8.8
```
<br><img width="495" height="332" alt="image" src="https://github.com/user-attachments/assets/dd65aa38-70d3-4f01-b3ff-8a12901e25ec" /><br>  
<br><img width="492" height="358" alt="image" src="https://github.com/user-attachments/assets/5e000183-6393-44f0-b514-af498d6c6f0e" /><br>  
<br><img width="483" height="355" alt="image" src="https://github.com/user-attachments/assets/4f374b87-8b1a-4a5a-ae3d-bc2dc48edfac" /><br>  
<br><img width="489" height="352" alt="image" src="https://github.com/user-attachments/assets/f9d57f34-5109-47f6-b1e0-2f70265cc3cc" /><br>  
<br><img width="508" height="354" alt="image" src="https://github.com/user-attachments/assets/d46f171f-f2aa-4aa2-b55e-becd0d81de5f" /><br>  
<br><img width="501" height="353" alt="image" src="https://github.com/user-attachments/assets/48f0509f-e67c-4caf-a99a-b1a56529a919" /><br>  
<br><img width="496" height="324" alt="image" src="https://github.com/user-attachments/assets/d5e9500d-6c9c-4c23-8103-36c7b853f18b" /><br>   
  
## Tes DNS
Pilih host acak, misal dari smartphone B
```
nslookup www.k14.smartcity.lab
```
<br><img width="296" height="131" alt="image" src="https://github.com/user-attachments/assets/af0a1c16-e9ed-47d7-a812-786489386461" /><br>

## Tes web
Buka Web Browser lalu akses:
`http://www.k14.smartcity.lab`
<br><img width="861" height="595" alt="image" src="https://github.com/user-attachments/assets/4d09cdb9-5433-4148-a9e5-12c63ad50378" /><br>  

## Tes FTP
Dari PC CCTV:
```bash
ftp ftp.k14.smartcity.lab
```
username: cctv, password: cctv14

Pada PC-CCTV, put file yang sudah dibuat, lalu dir  
<br><img width="604" height="857" alt="image" src="https://github.com/user-attachments/assets/e141b34c-4e64-4e20-a249-a2e8750ff399" /><br>

## Tes email
user midnight (misal dari puskesmas) & user daylight (misal dari smartphone U)  

1. Daylight (smartphone utara) ke midnight, puskesmas  
   <br><img width="856" height="272" alt="image" src="https://github.com/user-attachments/assets/37cffb84-e81a-4ea4-88b9-4cb3b682cda3" /><br>
2. Balasan kiriman dari midnight ke daylight
   <br><img width="844" height="252" alt="image" src="https://github.com/user-attachments/assets/7209ba95-6c73-4627-8f71-44efcfe11b88" /><br>

## Tes konektivitas OSPF

Memeriksa OSPF Neighbor  

Pada router, jalankan:
```bash
show ip ospf neighbor
```
  
R-PUSAT diharapkan memiliki neighbor ke:
- R-UTARA
- R-DC
- R-BARAT
- R-TIMUR
- R-SELATAN
- R-PELABUHAN melalui Tunnel0
  
<br><img width="669" height="172" alt="image" src="https://github.com/user-attachments/assets/3aa41fb1-d993-40cf-8408-6df8b4b319f5" /><br>

R-BARAT, R-TIMUR, R-SELATAN hanya punya 1 neighbor yaitu R-PUSAT  
<br><img width="690" height="74" alt="image" src="https://github.com/user-attachments/assets/feb22c1a-a99d-4437-8dd4-ee62c835ebc2" /><br>  
<br><img width="661" height="76" alt="image" src="https://github.com/user-attachments/assets/bdf83cc3-1940-45b7-8a13-97c084dfdc97" /><br>  
<br><img width="673" height="77" alt="image" src="https://github.com/user-attachments/assets/3dfb8eaf-d35e-4804-ae4a-c4a771b80aa6" /><br>  

R-DC punya dua neighbor yaitu R-PUSAT & R-UTARA  
<br><img width="652" height="95" alt="image" src="https://github.com/user-attachments/assets/bc97d403-4d20-4881-b82b-7934b67c76c1" /><br>  

R-UTARA punya tiga neighbor : R-PUSAT, R-PELABUHAN, dan R-DC  
<br><img width="677" height="106" alt="image" src="https://github.com/user-attachments/assets/74fd20ca-a0fe-49df-a578-4e7d33b6b4de" /><br>

R-PELABUHAN punya dua neighbor : R-PUSAT & R-SELATAN  
<br><img width="668" height="89" alt="image" src="https://github.com/user-attachments/assets/5adbc866-21fb-496d-ae29-001f208af10e" /><br>

## Pengujian pemulihan jaringan saat link terputus

Pengujian ini membuktikan bahwa OSPF memulihkan konektivitas secara otomatis lewat jalur lain ketika salah satu link backbone terputus. Link cadangan L7 (R-DC ke R-UTARA, `10.112.97.4/30`) menjadi jalur pengganti  

### Skenario 1 : Link L2 terputus  

Kondisi normal  
Misal `tracert 8.8.8.8` dari utara ke NAT  
<br><img width="428" height="166" alt="image" src="https://github.com/user-attachments/assets/7f037185-49ed-485e-807f-a00d4de0d521" /><br>

Jalurnya : Host Utara > L2 > Link to link R-PUSAT & R-ISP > 8.8.8.8  
  
Matikan L2 di R-PUSAT dengan
```
configure terminal
interface FastEthernet0/1
 shutdown
end
```

Hasil perintah `tracert` setelah L2 diputus  
<br><img width="469" height="295" alt="image" src="https://github.com/user-attachments/assets/6c2b7783-6248-4278-97b6-2372bf3a21f4" /><br>

Jalurnya menjadi : Host utara > L7 > L1 > Link to link R-PUSAT & R-ISP > 8.8.8.8

## Static route sudah diganti OSPF

Di semua router:
```
show run | include ip route
```
  
* R-PUSAT hanya menampilkan ip route 0.0.0.0 0.0.0.0 203.0.113.113 (default ke ISP, memang diperbolehkan)
  <br><img width="326" height="53" alt="image" src="https://github.com/user-attachments/assets/652a57bf-21da-4faa-a9f7-888701a46006" /><br>
* Router lain (R-DC, R-BARAT, R-TIMUR, R-SELATAN, R-UTARA, R-PELABUHAN, R-ISP) tidak menampilkan apa-apa
  <br><img width="289" height="50" alt="image" src="https://github.com/user-attachments/assets/cdcd3ca9-a5d1-4ea7-956a-36aafeae149e" /><br>
  <br><img width="298" height="47" alt="image" src="https://github.com/user-attachments/assets/4212b84c-3725-4463-ab67-62832d9f9415" /><br>
  <br><img width="308" height="49" alt="image" src="https://github.com/user-attachments/assets/0660961c-0bdf-4367-96b7-db46c997392f" /><br>

Pembuktian melalui `show IP route`, kode O artinya rute yang sebelumnya static kini sudah jadi OSPF  
<br><img width="640" height="383" alt="image" src="https://github.com/user-attachments/assets/112f2adb-a225-469c-a98e-bc54d3179897" /><br>

## Default route hanya di R-PUSAT, router lain dapat dari OSPF
Perintah:
```
show run | include ip route
show run | section router ospf
```
<br><img width="321" height="258" alt="image" src="https://github.com/user-attachments/assets/a2c1bcf6-fadb-4092-bde8-3fe8e025549d" /><br>  
  
Satu-satunya ip route adalah default ke ISP, dan di blok OSPF ada baris default-information originate. Itu bukti default dikonfigurasi dan disebarkan dari R-PUSAT  
  
Sementara dari router lain tidak ada `default-information originate`  
<br><img width="326" height="212" alt="image" src="https://github.com/user-attachments/assets/007fb635-ad13-4121-b641-6fae70dc22cf" /><br>  

Di router lain (minimal R-DC, R-UTARA, R-PELABUHAN, plus satu router wilayah):
```
show ip route
show ip route 0.0.0.0
```
* Di show ip route harus ada Gateway of last resort is ... to network 0.0.0.0 dan baris O*E2 0.0.0.0/0   
* Di show ip route 0.0.0.0 harus tertulis Known via "ospf 1" dan type extern 2. Tidak boleh Known via "static"  

## Router tidak mengirim OSPF ke arah LAN

Bukti utama, di tiap router:
```
show ip protocols
```
Bagian Passive Interface(s): harus berisi interface LAN router itu

| Router | Interface yang harus passive |
|---|---|
| R-PUSAT | Fa1/0, Fa1/1 |
| R-DC | Fa0/1, Fa1/0 |
| R-BARAT | Fa0/1, Fa1/0 |
| R-TIMUR | Fa0/1, Fa1/0 |
| R-SELATAN | Fa0/1, Fa1/0 |
| R-UTARA | Fa1/0, Fa1/1 |
| R-PELABUHAN | Fa0/1, Fa1/0 |

Misal di DC  
<br><img width="536" height="469" alt="image" src="https://github.com/user-attachments/assets/5bf00393-176f-4608-92a5-1d525fb7bd44" /><br>  
Nb: Sisanya merupakan interface yang belum terpakai














 



    

