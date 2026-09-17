
# Tabel Enumerasi Lengkap — Bab 3 Active Reconnaissance

## Info Target

- **IP Target**: 192.168.56.103
- **Metode Deploy**: VirtualBox (Opsi A — Metasploitable2 menggunakan file `.vmdk`)
- **Tanggal Scan**: 17 September 2026

# Hasil Full Port Scan (TCP)

Hasil scanning menggunakan Nmap:

```bash
nmap -p- -T4 192.168.56.103
````

| Port  | Status | Service    | Versi (Jika Terdeteksi)         | Catatan                      |
| ----- | ------ | ---------- | ------------------------------- | ---------------------------- |
| 21    | Open   | FTP        | vsftpd 2.3.4                    | Service FTP aktif            |
| 22    | Open   | SSH        | OpenSSH 4.7p1 Debian 8ubuntu1   | Remote access service        |
| 23    | Open   | Telnet     | Linux telnetd                   | Remote access tanpa enkripsi |
| 25    | Open   | SMTP       | Postfix smtpd                   | Mendukung enumerasi user     |
| 53    | Open   | DNS        | ISC BIND 9.4.2                  | DNS service aktif            |
| 80    | Open   | HTTP       | Apache httpd 2.2.8 (Ubuntu)     | Web server aktif             |
| 111   | Open   | RPCbind    | RPC #100000                     | RPC service                  |
| 139   | Open   | NetBIOS    | Samba smbd 3.X - 4.X            | SMB service aktif            |
| 445   | Open   | SMB        | Samba smbd 3.0.20-Debian        | File sharing service         |
| 512   | Open   | rexec      | netkit-rsh rexecd               | Remote execution service     |
| 513   | Open   | rlogin     | login service                   | Remote login service         |
| 514   | Open   | rsh        | Netkit rshd                     | Remote shell service         |
| 1099  | Open   | Java RMI   | GNU Classpath grmiregistry      | Java remote service          |
| 1524  | Open   | Bindshell  | Metasploitable root shell       | Service dengan risiko tinggi |
| 2049  | Open   | NFS        | RPC #100003                     | Network file sharing         |
| 2121  | Open   | FTP        | ProFTPD 1.3.1                   | FTP alternatif               |
| 3306  | Open   | MySQL      | MySQL 5.0.51a-3ubuntu5          | Database service             |
| 3632  | Open   | distccd    | distccd v1                      | Distributed compiler service |
| 5432  | Open   | PostgreSQL | PostgreSQL 8.3.x                | Database service             |
| 5900  | Open   | VNC        | VNC Protocol 3.3                | Remote desktop service       |
| 6000  | Open   | X11        | Access denied                   | X Window service             |
| 6667  | Open   | IRC        | UnrealIRCd                      | IRC server                   |
| 6697  | Open   | IRC SSL    | UnrealIRCd                      | IRC encrypted service        |
| 8009  | Open   | AJP13      | Apache Jserv Protocol v1.3      | Tomcat connector             |
| 8180  | Open   | HTTP       | Apache Tomcat/Coyote JSP Engine | Web application server       |
| 8787  | Open   | DRb        | Ruby DRb RMI                    | Ruby remote service          |
| 46030 | Open   | mountd     | RPC #100005                     | NFS related service          |
| 46844 | Open   | Java RMI   | GNU Classpath grmiregistry      | Java remote service          |
| 51906 | Open   | nlockmgr   | RPC #100021                     | RPC locking service          |
| 53563 | Open   | status     | RPC #100024                     | RPC status service           |

---

# DNS Enumeration

Metode pengujian:

```bash
dig axfr @192.168.56.103 metasploitable.local
```

* **Zone transfer berhasil?**: Tidak

Hasil:

```
Transfer failed.
```

* **Record yang ditemukan**:

Tidak terdapat record DNS yang berhasil diperoleh karena zone transfer tidak diizinkan oleh server.

---

# SMB Enumeration

Metode pengujian:

```bash
enum4linux -a 192.168.56.103
```

dan:

```bash
smbclient -L //192.168.56.103/ -N
```

## Share yang ditemukan:

| Share  | Keterangan       |
| ------ | ---------------- |
| print$ | Printer Drivers  |
| tmp    | Shared Directory |
| opt    | Shared Directory |
| IPC$   | IPC Service      |
| ADMIN$ | IPC Service      |

---

## Bisa diakses tanpa autentikasi?

**Ya**

Bukti:

```
Anonymous login successful
```

Share yang berhasil diakses:

```
tmp
```

Status:

```
Mapping: OK
Listing: OK
```

---

## User/Group yang Ter-enumerate

User yang berhasil ditemukan:

* root
* msfadmin
* user
* nobody
* www-data
* postgres
* mysql
* distccd
* proftpd
* tomcat55
* ftp
* irc
* sshd

Informasi workgroup:

```
WORKGROUP
```

Nama server:

```
METASPLOITABLE
```

---

# SMTP Enumeration

Metode pengujian:

```bash
nc 192.168.56.103 25
```

Pengujian VRFY:

```
VRFY root
252 2.0.0 root

VRFY msfadmin
252 2.0.0 msfadmin

VRFY fakeuser
550 User unknown
```

---

## VRFY diaktifkan?

**Ya**

---

## User valid yang berhasil dikonfirmasi:

| Username | Status |
| -------- | ------ |
| root     | Valid  |
| msfadmin | Valid  |

User tidak valid:

| Username | Status               |
| -------- | -------------------- |
| fakeuser | User tidak ditemukan |

---

# SNMP Enumeration

Metode pengujian:

```bash
snmpwalk -v2c -c public 192.168.56.103
```

Hasil:

```
Timeout: No Response from 192.168.56.103
```

---

## Community string `public` berfungsi?

**Tidak**

---

## Informasi yang ter-expose:

Tidak ditemukan informasi SNMP karena target tidak memberikan respons terhadap request SNMP.

Kemungkinan penyebab:

* Service SNMP tidak aktif
* UDP port 161 tidak terbuka
* Community string berbeda

---

# Ringkasan dan Prioritas Investigasi Lanjutan

Berdasarkan hasil Active Reconnaissance, terdapat beberapa service yang menarik untuk dilakukan investigasi lebih lanjut pada tahap Vulnerability Scanning.

## 1. FTP — vsftpd 2.3.4 (Port 21)

Service FTP menggunakan versi lama yang memiliki riwayat kerentanan keamanan. Service ini menjadi salah satu kandidat utama untuk pemeriksaan vulnerability.

## 2. SMB — Samba 3.0.20 (Port 139/445)

Service SMB dapat diakses menggunakan anonymous login dan memungkinkan enumerasi user serta share. Konfigurasi ini dapat menjadi indikasi adanya kelemahan keamanan.

## 3. distccd (Port 3632)

Service distccd merupakan distributed compiler service yang memiliki riwayat kerentanan remote command execution pada versi lama.

## 4. Web Server — Apache 2.2.8 dan Tomcat (Port 80 dan 8180)

Web server menggunakan versi lama sehingga perlu dilakukan pemeriksaan terhadap kemungkinan vulnerability pada aplikasi maupun konfigurasi server.

---

# Kesimpulan

Hasil Active Reconnaissance terhadap Metasploitable2 menunjukkan banyak service aktif dengan beberapa service menggunakan versi lama. Service yang menjadi prioritas untuk dilakukan vulnerability scanning adalah:

1. FTP vsftpd 2.3.4
2. SMB Samba 3.0.20
3. distccd
4. Apache/Tomcat Web Server

Hasil enumerasi ini akan digunakan sebagai dasar untuk tahap berikutnya yaitu **Bab 4 — Vulnerability Scanning**.
