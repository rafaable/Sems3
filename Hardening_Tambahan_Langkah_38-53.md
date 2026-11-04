# Hardening Tambahan (Langkah 38 dst.)

Semua ini baru, tidak mengulang langkah 0-37. Aturan tetap sama: tanpa port 22, tanpa password sudo, jangan sentuh Teleport, tanpa install runtime. Blok bertanda **RISIKO** punya rollback atau harus dites dulu.

## 38. Perbaiki efek `umask 027` dari langkah sebelumnya

Sejak langkah 5, shell baru memakai umask 027. Akibatnya file yang dibuat lewat `sudo tee` di `/etc` bisa jadi 640. File seperti `/etc/profile.d/*.sh` lalu tidak terbaca user biasa dan aturannya tidak berlaku.

```bash
umask 022    # jalankan di awal tiap sesi baru sebelum membuat file konfigurasi
sudo find /etc/profile.d /etc/apt/apt.conf.d /etc/modprobe.d /etc/sysctl.d /etc/default /etc/pam.d /usr/share/pam-configs /etc/apparmor.d -type f ! -perm -044 -exec chmod 644 {} + 2>/dev/null
ls -l /etc/profile.d/99-*.sh /etc/issue /etc/issue.net
```

## 39. Cari backdoor dan persistensi

Lab hardening bisa menyisipkan sisa konfigurasi lama atau persistensi. Semua perintah ini hanya membaca dulu.

```bash
# listener non-loopback beserta prosesnya
sudo ss -tulpnH | awk '$5 !~ /^(127\.|\[::1\]|::1)/'

# service berjalan yang tidak umum
systemctl list-units --type=service --state=running --no-pager --no-legend | grep -viE 'teleport|systemd|dbus|cron|rsyslog|auditd|unattended|ufw|apparmor|getty|polkit|networkd|resolved|timesyncd|multipath|udev|snapd|accounts|acct'

# unit dan file custom yang baru diubah (60 hari)
sudo find /etc/systemd/system /usr/local /opt -type f -mtime -60 2>/dev/null | grep -vi teleport

# proses yang berjalan dari lokasi mencurigakan
sudo ls -l /proc/[0-9]*/exe 2>/dev/null | grep -E '/tmp/|/dev/shm/|/var/tmp/|\(deleted\)'

# cron semua user dan cron.d
for u in $(cut -d: -f1 /etc/passwd); do sudo crontab -l -u "$u" 2>/dev/null | grep -v '^#' | sed "s/^/$u: /"; done
sudo grep -vE '^\s*#|^\s*$' /etc/crontab /etc/cron.d/* 2>/dev/null

# pola reverse shell
sudo grep -rIlE '/dev/tcp/|bash -i|nc -e|ncat -e|mkfifo /tmp|curl[^|]*\|\s*(ba)?sh|wget[^|]*\|\s*(ba)?sh' /etc /home /root /usr/local /opt /var/spool/cron --exclude-dir=teleport 2>/dev/null

# semua authorized_keys di sistem (hapus yang tidak dikenal)
sudo find / -xdev \( -name authorized_keys -o -name authorized_keys2 \) -type f -exec sh -c 'echo "== $1"; cat "$1"' _ {} \; 2>/dev/null

# rc.local dan profile.d
ls -la /etc/rc.local /etc/profile.d/ 2>/dev/null
```

Tindakan jika ketemu (ganti nama sesuai temuan):

```bash
sudo systemctl disable --now NAMA.service && sudo systemctl mask NAMA.service
sudo crontab -r -u USER
sudo kill -9 PID
sudo rm -f /path/authorized_keys_mencurigakan
```

## 40. Batasi tool serang ke root dan group sudo (least privilege)

`dpkg-statoverride` membuat izin bertahan walau paket diupdate.

```bash
for b in nc nc.openbsd nc.traditional ncat netcat socat tcpdump tshark nmap gdb strace ltrace traceroute mtr-tiny telnet ftp; do
  p=$(command -v "$b" 2>/dev/null) && p=$(readlink -f "$p") && \
  sudo dpkg-statoverride --update --add root sudo 0750 "$p" 2>/dev/null && echo "dibatasi: $p"
done
sudo dpkg-statoverride --list | head -20
sudo -u nobody nc -h 2>&1 | head -1     # harus permission denied
```

## 41. Modul kernel tambahan (belum ada di langkah 8 dan 16)

```bash
for m in firewire-core firewire-ohci thunderbolt bluetooth btusb uas ax25 netrom rose x25 appletalk atm can nfc; do
  printf 'install %s /bin/false\nblacklist %s\n' "$m" "$m" | sudo tee /etc/modprobe.d/$m.conf >/dev/null
  sudo modprobe -r "$m" 2>/dev/null
done
lsmod | grep -E 'bluetooth|firewire|thunderbolt|uas' || echo "modul bersih"
```

## 42. Sysctl jaringan tambahan dan cek bentrok

```bash
sudo tee /etc/sysctl.d/95-net-extra.conf >/dev/null <<'EOF'
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 3
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_challenge_ack_limit = 999999999
net.ipv4.conf.all.drop_gratuitous_arp = 1
net.ipv4.conf.all.arp_filter = 1
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
kernel.printk = 3 3 3 3
EOF
# OPSIONAL, hanya jika policy meminta server tidak membalas ping:
# echo 'net.ipv4.icmp_echo_ignore_all = 1' | sudo tee /etc/sysctl.d/94-noping.conf
sudo sysctl --system >/dev/null

# bandingkan nilai di file dengan nilai yang benar-benar aktif
for f in /etc/sysctl.d/9[4-9]-*.conf; do
  grep -vE '^\s*(#|$)' "$f" | while IFS='=' read -r k v; do
    k=$(echo $k); v=$(echo $v)
    live=$(sysctl -n "$k" 2>/dev/null | tr -s '[:space:]' ' ' | sed 's/ $//')
    [ "$live" = "$v" ] || echo "BEDA: $k  file=[$v] aktif=[$live]"
  done
done

# UFW punya sysctl sendiri yang bisa menimpa nilai kita
grep -vE '^\s*(#|$)' /etc/ufw/sysctl.conf || echo "ufw sysctl: tidak ada override"
```

Kalau ada `BEDA`, cari file yang menimpa dengan `grep -rn NAMA_PARAMETER /etc/sysctl.conf /etc/sysctl.d/ /etc/ufw/sysctl.conf`.

## 43. UFW: filter paket scan dan ICMP flood (`before.rules`)

Rule ini dievaluasi sebelum rule bawaan UFW. Siapkan tab Teleport kedua dulu.

```bash
sudo cp /etc/ufw/before.rules /root/backup-hardening/before.rules.bak
cat > /tmp/ufw-extra.rules <<'EOF'
# --- hardening: drop paket abnormal dan batasi ping ---
-A ufw-before-input -f -j DROP
-A ufw-before-input -p tcp --tcp-flags ALL NONE -j DROP
-A ufw-before-input -p tcp --tcp-flags ALL ALL -j DROP
-A ufw-before-input -p tcp --tcp-flags ALL FIN,PSH,URG -j DROP
-A ufw-before-input -p tcp --tcp-flags SYN,FIN SYN,FIN -j DROP
-A ufw-before-input -p tcp --tcp-flags SYN,RST SYN,RST -j DROP
-A ufw-before-input -p icmp --icmp-type echo-request -m limit --limit 5/s --limit-burst 10 -j ACCEPT
-A ufw-before-input -p icmp --icmp-type echo-request -j DROP
EOF
grep -q 'hardening: drop paket abnormal' /etc/ufw/before.rules || sudo sed -i '/^# End required lines/r /tmp/ufw-extra.rules' /etc/ufw/before.rules
sudo ufw reload || sudo cp /root/backup-hardening/before.rules.bak /etc/ufw/before.rules
sudo ufw status | head -3
sudo iptables -S ufw-before-input | head -12
```

### Egress default deny (RISIKO, dengan rollback otomatis)

Ini memperluas "default deny" ke traffic keluar. Teleport butuh koneksi keluar ke proxy-nya, jadi cek tujuannya dulu.

```bash
sudo ss -tnp state established | grep -i teleport      # catat IP:PORT proxy
sudo systemd-run --unit=ufw-out-rollback --on-active=300 /usr/sbin/ufw default allow outgoing

sudo ufw allow out 53 comment 'dns'
sudo ufw allow out 80/tcp comment 'http apt'
sudo ufw allow out 443/tcp comment 'https'
sudo ufw allow out 123/udp comment 'ntp'
# jika Teleport bukan di 443, wajib: sudo ufw allow out to IP_PROXY port PORT_PROXY proto tcp
sudo ufw default deny outgoing
sudo ufw status verbose
```

Buka tab Teleport baru. Kalau masih bisa login, batalkan rollback:

```bash
sudo systemctl stop ufw-out-rollback.timer
```

## 44. Mount option: `/var/tmp` dan `/proc`

`hidepid=2` menerapkan need to know: user biasa tidak bisa melihat proses user lain.

```bash
# /var/tmp: nosuid, nodev, noexec
grep -q ' /var/tmp ' /etc/fstab || echo '/var/tmp /var/tmp none bind,nosuid,nodev,noexec 0 0' | sudo tee -a /etc/fstab
sudo mount --bind /var/tmp /var/tmp
sudo mount -o remount,bind,nosuid,nodev,noexec /var/tmp
findmnt /var/tmp

# /proc: sembunyikan proses user lain, group adm tetap boleh lihat
sudo usermod -aG adm ubuntu
grep -q hidepid /etc/fstab || echo 'proc /proc proc defaults,hidepid=2,gid=adm 0 0' | sudo tee -a /etc/fstab
sudo mount -o remount,hidepid=2,gid=adm /proc
findmnt /proc -o TARGET,OPTIONS
sudo -u nobody ps -e | wc -l      # harus sangat sedikit
```

## 45. Sudo lanjutan dan hapus grup `admin` lama

```bash
sudo tee /etc/sudoers.d/02-hardening-more >/dev/null <<'EOF'
Defaults ignore_dot
Defaults !pwfeedback
Defaults lecture=once
Defaults syslog=authpriv
Defaults syslog_goodpri=info
Defaults syslog_badpri=alert
EOF
sudo chmod 440 /etc/sudoers.d/02-hardening-more
sudo visudo -cf /etc/sudoers.d/02-hardening-more || sudo rm -f /etc/sudoers.d/02-hardening-more

# lihat aturan grup di sudoers utama
sudo grep -nE '^%|^[a-z]+ +ALL' /etc/sudoers

# nonaktifkan grup 'admin' (warisan lama), lewat salinan yang divalidasi dulu
sudo cp /etc/sudoers /tmp/sudoers.new
sudo sed -i 's/^%admin/#%admin/' /tmp/sudoers.new
sudo visudo -cf /tmp/sudoers.new && sudo install -m 440 -o root -g root /tmp/sudoers.new /etc/sudoers
sudo rm -f /tmp/sudoers.new
sudo visudo -c
```

Jangan menambahkan `Defaults !authenticate` atau menghapus `NOPASSWD` milik `ubuntu`.

## 46. Siklus hidup akun

```bash
# akun baru default tanpa shell login (default deny)
sudo useradd -D -s /usr/sbin/nologin
useradd -D

# akun yang tidak pernah login atau tidak aktif > 90 hari
sudo lastlog -b 90 | awk 'NR==1 || $1 !~ /^(root|ubuntu|auditor)$/'

# kunci dan kedaluwarsakan akun tertentu (ganti USER)
sudo usermod -L -e 1 USER
sudo usermod -s /usr/sbin/nologin USER

# akun sementara kedaluwarsa otomatis 7 hari
# sudo chage -E $(date -d '+7 days' +%F) USER

# hapus akun lama beserta home (jangan hapus ubuntu, auditor, atau akun Teleport)
# sudo userdel -r USER

# akun tanpa password sama sekali (NP) atau tanpa kedaluwarsa
sudo passwd -Sa | awk '$2=="NP"{print "TANPA PASSWORD: "$1}'
sudo awk -F: '$2 !~ /^[!*]/ && $1!="ubuntu"{print $1}' /etc/shadow | while read u; do sudo chage -l "$u" | grep -H --label="$u" 'Password expires'; done
```

## 47. Cloud-init, netplan, dan paket telemetri

```bash
# cloud-init tidak boleh menyalakan lagi password login dan root
sudo tee /etc/cloud/cloud.cfg.d/99-hardening.cfg >/dev/null <<'EOF'
ssh_pwauth: false
disable_root: true
EOF

# netplan dengan izin longgar memunculkan warning dan bisa terbaca user lain
sudo chmod 600 /etc/netplan/*.yaml 2>/dev/null
ls -l /etc/netplan/

# paket telemetri dan crash report
for p in popularity-contest whoopsie ubuntu-report apport apport-symptoms landscape-common; do
  dpkg -s "$p" >/dev/null 2>&1 && sudo apt-get purge -y "$p"
done
sudo apt-get autoremove --purge -y
```

## 48. Audit rules lanjutan dan process accounting

Rule ini ditulis portable (x86 dan arm64).

```bash
sudo tee /etc/audit/rules.d/95-more.rules >/dev/null <<'EOF'
-a always,exit -F arch=b64 -S fchmod,fchmodat,fchown,fchownat,setxattr,lsetxattr,fsetxattr,removexattr,lremovexattr,fremovexattr -F auid>=1000 -F auid!=unset -k perm_mod
-a always,exit -F arch=b64 -S openat,truncate,ftruncate -F exit=-EACCES -F auid>=1000 -F auid!=unset -k access
-a always,exit -F arch=b64 -S openat,truncate,ftruncate -F exit=-EPERM -F auid>=1000 -F auid!=unset -k access
-a always,exit -F arch=b64 -S mount,umount2 -F auid>=1000 -F auid!=unset -k mounts
-a always,exit -F arch=b64 -S unlinkat,renameat -F auid>=1000 -F auid!=unset -k delete
-w /var/run/faillock/ -p wa -k logins
-w /etc/cron.d/ -p wa -k cron
-w /etc/crontab -p wa -k cron
-w /var/spool/cron/ -p wa -k cron
-w /etc/systemd/system/ -p wa -k systemd
-w /etc/modprobe.d/ -p wa -k modules
-w /etc/sysctl.d/ -p wa -k sysctl
-w /etc/netplan/ -p wa -k network
-w /etc/localtime -p wa -k time-change
-w /etc/fstab -p wa -k mounts
EOF

# semua binary SUID/SGID otomatis diawasi
sudo find / -xdev \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null | awk '{print "-a always,exit -F path="$1" -F perm=x -F auid>=1000 -F auid!=unset -k privileged"}' | sudo tee /etc/audit/rules.d/96-privileged.rules >/dev/null

sudo augenrules --load
sudo auditctl -l | wc -l

# process accounting: siapa menjalankan apa
sudo apt-get install -y acct
sudo systemctl enable --now acct 2>/dev/null
sudo accton on 2>/dev/null
lastcomm | head -5
```

Opsional, kerjakan paling akhir: kunci aturan audit sampai reboot. Setelah ini rule audit tidak bisa diubah lagi.

```bash
echo '-e 2' | sudo tee /etc/audit/rules.d/99-zz-immutable.rules
sudo augenrules --load
```

## 49. Log: tolak log remote dan central logging yang andal

```bash
# server tidak boleh menerima log dari luar
grep -RnE '^\s*(module|input)\(.*(imtcp|imudp|imrelp)|^\s*\$(ModLoad|UDPServerRun|InputTCPServerRun)' /etc/rsyslog.conf /etc/rsyslog.d/ || echo "tidak menerima log remote"

# auditor boleh baca journal tanpa sudo
id auditor >/dev/null 2>&1 && sudo usermod -aG systemd-journal auditor
```

Kalau IP log server diberikan, ini versi dengan antrean disk dan hanya mengirim log autentikasi. Isi variabelnya dulu:

```bash
LOGSERVER=ISI_IP_LOG_SERVER
sudo tee /etc/rsyslog.d/91-remote-auth.conf >/dev/null <<EOF
if (\$syslogfacility-text == 'auth' or \$syslogfacility-text == 'authpriv') then {
  action(type="omfwd" target="$LOGSERVER" port="514" protocol="tcp"
         queue.type="LinkedList" queue.filename="fwd_auth" queue.maxdiskspace="200m"
         queue.saveonshutdown="on" action.resumeRetryCount="-1")
}
EOF
sudo rsyslogd -N1 && sudo systemctl restart rsyslog
```

Kalau egress deny (langkah 43) aktif, izinkan dulu: `sudo ufw allow out to $LOGSERVER port 514 proto tcp`.

## 50. RBAC skenario kuliah (sesuai slide: Lecturer, Assistant, Student)

Ini memetakan slide RBAC dan MAC ke hak akses nyata. Hak diberikan ke role (group), user hanya dimasukkan ke role.

```bash
for g in lecturer assistant student; do sudo groupadd -f $g; done
sudo mkdir -p /srv/course/{grades,slides,submissions,forum}
sudo chown root:root /srv/course; sudo chmod 755 /srv/course

# grades: lecturer tulis, assistant baca, student tidak ada akses
sudo chown root:lecturer /srv/course/grades; sudo chmod 2770 /srv/course/grades
sudo setfacl -m g:assistant:rx /srv/course/grades
sudo setfacl -d -m g:lecturer:rwx,g:assistant:r-x,o::--- /srv/course/grades

# slides: lecturer tulis, assistant dan student baca
sudo chown root:lecturer /srv/course/slides; sudo chmod 2770 /srv/course/slides
sudo setfacl -m g:assistant:rx,g:student:rx /srv/course/slides
sudo setfacl -d -m g:lecturer:rwx,g:assistant:r-x,g:student:r-x,o::--- /srv/course/slides

# submissions: student hanya bisa mengumpulkan (tidak bisa melihat atau menghapus milik orang lain)
sudo chown root:lecturer /srv/course/submissions; sudo chmod 3770 /srv/course/submissions
sudo setfacl -m g:assistant:rx,g:student:wx /srv/course/submissions
sudo setfacl -d -m g:lecturer:rwx,g:assistant:r-x,g:student:---,o::--- /srv/course/submissions

# forum: semua role boleh tulis, sticky bit mencegah hapus milik orang lain
sudo chown root:lecturer /srv/course/forum; sudo chmod 3770 /srv/course/forum
sudo setfacl -m g:assistant:rwx,g:student:rwx /srv/course/forum
sudo setfacl -d -m g:lecturer:rwx,g:assistant:rwx,g:student:rw-,o::--- /srv/course/forum

echo nilai | sudo tee /srv/course/grades/grades.csv >/dev/null
echo slide | sudo tee /srv/course/slides/week10.txt >/dev/null
```

Tes dengan user sementara, lalu langsung dihapus supaya tidak ada akun asing:

```bash
sudo useradd -M -s /usr/sbin/nologin -G lecturer rina
sudo useradd -M -s /usr/sbin/nologin -G assistant andi
for u in budi citra eko; do sudo useradd -M -s /usr/sbin/nologin -G student $u; done

t(){ u=$1; shift; if sudo -u "$u" "$@" >/dev/null 2>&1; then echo "DITERIMA  $u: $*"; else echo "DITOLAK   $u: $*"; fi; }
t rina cat /srv/course/grades/grades.csv
t rina sh -c 'echo 90 >> /srv/course/grades/grades.csv'
t andi cat /srv/course/grades/grades.csv
t andi sh -c 'echo 99 >> /srv/course/grades/grades.csv'
t eko  cat /srv/course/grades/grades.csv
t eko  cat /srv/course/slides/week10.txt
t eko  sh -c 'echo tugas > /srv/course/submissions/eko.txt'
t budi cat /srv/course/submissions/eko.txt
t budi rm /srv/course/submissions/eko.txt
t citra sh -c 'echo halo > /srv/course/forum/citra.txt'
t eko  rm /srv/course/forum/citra.txt

# Eko pindah role: satu perubahan saja
# sudo usermod -aG student eko

# bersihkan akun uji
for u in rina andi budi citra eko; do sudo userdel $u; done
getfacl -p /srv/course/grades /srv/course/submissions
```

## 51. Separation of duties: yang meminta tidak boleh menyetujui

```bash
sudo groupadd -f requesters; sudo groupadd -f approvers
sudo mkdir -p /srv/change/{pending,approved}
sudo chown root:root /srv/change; sudo chmod 755 /srv/change

# pending: requester boleh membuat berkas, approver boleh membaca
sudo chown root:requesters /srv/change/pending; sudo chmod 1730 /srv/change/pending
sudo setfacl -m g:approvers:rx /srv/change/pending
sudo setfacl -d -m g:approvers:r--,o::--- /srv/change/pending

# approved: hanya approver yang boleh menulis, requester hanya membaca
sudo chown root:approvers /srv/change/approved; sudo chmod 2770 /srv/change/approved
sudo setfacl -m g:requesters:rx /srv/change/approved
sudo setfacl -d -m g:requesters:r-x,o::--- /srv/change/approved

# aturan inti: tidak boleh ada orang yang ada di kedua group (hasil harus kosong)
comm -12 <(getent group requesters | cut -d: -f4 | tr ',' '\n' | sort) <(getent group approvers | cut -d: -f4 | tr ',' '\n' | sort)

# akun auditor tidak boleh punya hak ubah
id auditor 2>/dev/null | grep -Eq 'sudo|requesters|approvers' && echo "PELANGGARAN SoD: auditor punya hak ubah" || echo "auditor aman"
```

## 52. AIDE: deteksi perubahan file konfigurasi penting

```bash
sudo apt-get install -y aide aide-common
sudo tee /etc/aide/aide.conf.d/99_local_rules >/dev/null <<'EOF'
/etc/ssh/sshd_config.d p+i+n+u+g+s+sha512
/etc/sudoers.d p+i+n+u+g+s+sha512
/etc/pam.d p+i+n+u+g+s+sha512
/etc/ufw p+i+n+u+g+s+sha512
/etc/audit p+i+n+u+g+s+sha512
/etc/apparmor.d p+i+n+u+g+s+sha512
/etc/sysctl.d p+i+n+u+g+s+sha512
!/var/lib/teleport
!/var/log
EOF
sudo update-aide.conf
sudo nohup aideinit -y -f > /tmp/aideinit.log 2>&1 &
sudo systemctl enable --now dailyaidecheck.timer 2>/dev/null
```

Inisialisasi butuh beberapa menit di background. Cek dengan `tail /tmp/aideinit.log`.

## 53. MFA untuk login konsol lokal (OPSIONAL)

Hanya menyentuh `/etc/pam.d/login`. Jangan diterapkan ke `sudo`, `sshd`, atau `teleport` karena risiko dianggap password sudo dan bisa memutus akses.

```bash
sudo apt-get install -y libpam-google-authenticator
grep -q pam_google_authenticator /etc/pam.d/login || sudo sed -i '0,/^@include common-auth/s//auth required pam_google_authenticator.so nullok\n@include common-auth/' /etc/pam.d/login
grep -n 'google_authenticator\|common-auth' /etc/pam.d/login

# tiap user mendaftar sendiri (bukan sebagai root)
google-authenticator -t -d -f -r 3 -R 30 -w 3
```

`nullok` membuat user yang belum mendaftar tetap bisa login, jadi tidak ada yang terkunci.

## Urutan pengerjaan yang aman

1. Aman dan hanya membaca: 38, 39, 40, 41, 46, 47
2. Konfigurasi biasa: 42, 45, 48, 49, 50, 51, 52
3. Menyentuh mount dan firewall: 44, lalu 43 (`before.rules`)
4. Berisiko: egress deny di 43, MFA di 53, dan `-e 2` di 48 paling akhir

Setelah langkah 43, 44, dan 53, pastikan akses lab masih hidup di tab Teleport kedua:

```bash
systemctl is-active teleport && sudo ufw status | head -3
```

Backup konfigurasi ada di `/root/backup-hardening/`.