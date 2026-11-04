# Hardening Tambahan (Langkah 54 dst.)

Semua ini baru dan tidak mengulang langkah 0-53. Aturan tetap: tanpa port 22, tanpa password sudo, jangan sentuh Teleport, tanpa install runtime. Mulai sesi baru dengan `umask 022` supaya file konfigurasi tidak terbuat 640 (efek langkah 5).

## 54. Celah PAM dan shell yang belum ditutup

`nullok` membolehkan akun berpassword kosong login. `nologin` di `/etc/shells` membuat shell itu dianggap valid (temuan CIS).

```bash
umask 022
sudo cp -a /etc/pam.d /root/backup-hardening/pam.d.step54
sudo cp /etc/shells /root/backup-hardening/shells.bak

# hapus nullok di live file DAN template, supaya tidak balik saat pam-auth-update
sudo sed -i 's/[[:space:]]\+nullok\b//' /usr/share/pam-configs/unix /etc/pam.d/common-auth /etc/pam.d/common-password
grep -n nullok /etc/pam.d/common-* /usr/share/pam-configs/unix || echo "nullok hilang"
sudo -n true && echo "sudo masih normal"

# nologin dan false tidak boleh terdaftar sebagai shell valid
sudo sed -i '\|/nologin$|d;\|/bin/false$|d' /etc/shells
cat /etc/shells

# umask root
grep -q '^umask' /root/.bashrc || echo 'umask 027' | sudo tee -a /root/.bashrc >/dev/null
```

Jangan menjalankan `pam-auth-update --force` karena akan menghapus baris `pam_umask` dan `pam_limits` yang ditambahkan manual di langkah 27 dan 31.

## 55. Integritas akun dan umur password akun yang sudah ada

`login.defs` hanya berlaku untuk akun baru. Akun lama harus diatur dengan `chage`.

```bash
echo "== entri legacy '+' ==";            grep '^+:' /etc/passwd /etc/shadow /etc/group
echo "== password belum di-shadow ==";     awk -F: '($2!="x"){print $1}' /etc/passwd
echo "== GID di passwd tak ada di group =="; for g in $(cut -d: -f4 /etc/passwd | sort -u); do getent group "$g" >/dev/null || echo "GID hilang: $g"; done
echo "== GID/nama group duplikat ==";      cut -d: -f3 /etc/group | sort | uniq -d; cut -d: -f1 /etc/group | sort | uniq -d
echo "== group shadow harus kosong ==";    getent group shadow
echo "== home hilang / bukan milik user =="
awk -F: '($3>=1000 && $3<65534){print $1,$3,$6}' /etc/passwd | while read u uid h; do
  [ -d "$h" ] || echo "HOME HILANG: $u $h"
  [ -d "$h" ] && [ "$(stat -c %u "$h")" != "$uid" ] && echo "BUKAN MILIK $u: $h"
done
echo "== izin home lebih longgar dari 750 =="
awk -F: '($3>=1000 && $3<65534){print $6}' /etc/passwd | while read h; do
  [ -d "$h" ] && [ $(( 8#$(stat -c %a "$h") & 8#027 )) -ne 0 ] && { echo "$h $(stat -c %a "$h")"; sudo chmod g-w,o-rwx "$h"; }
done
echo "== tanggal ganti password di masa depan =="
sudo awk -F: -v t=$(( $(date +%s)/86400 )) '$3+0>t{print "FUTURE: "$1}' /etc/shadow
```

Terapkan umur password ke akun yang punya password (gaya CIS). Lewati ini kalau policy bergaya NIST (tanpa paksa ganti):

```bash
for u in $(sudo awk -F: '$2 ~ /^\$/ && $1!="root" && $1!="ubuntu"{print $1}' /etc/shadow); do
  sudo chage --mindays 1 --maxdays 365 --warndays 7 --inactive 30 "$u"
done
```

## 56. Rahasia berdasarkan isi file (bukan hanya nama)

Langkah 5 mencari rahasia berdasarkan nama file. Ini mencari berdasarkan isinya.

```bash
# 1. daftar dulu
sudo grep -rIlEi --exclude-dir={teleport,.git,node_modules} \
  -e '(password|passwd|secret|api[_-]?key|token|access[_-]?key)[A-Za-z_]*[[:space:]]*[=:][[:space:]]*[^[:space:]]+' \
  -e 'BEGIN [A-Z ]*PRIVATE KEY' -e 'AKIA[0-9A-Z]{16}' \
  /home /root /opt /srv /var/www /var/backups 2>/dev/null

# 2. kunci hasilnya ke owner saja (cek daftar di atas dulu)
sudo grep -rIlEi --exclude-dir={teleport,.git,node_modules} \
  -e '(password|passwd|secret|api[_-]?key|token|access[_-]?key)[A-Za-z_]*[[:space:]]*[=:][[:space:]]*[^[:space:]]+' \
  -e 'BEGIN [A-Z ]*PRIVATE KEY' -e 'AKIA[0-9A-Z]{16}' \
  /home /root /opt /srv /var/www /var/backups 2>/dev/null | while read -r f; do sudo chmod go-rwx "$f" && echo "dikunci: $f"; done

# 3. file kredensial standar
for f in .pgpass .my.cnf .netrc .git-credentials .npmrc .pypirc .s3cfg .boto .vault_pass .htpasswd; do
  sudo find /home /root /opt /srv /var/www -name "$f" -type f -exec chmod 600 {} \; -print 2>/dev/null
done
sudo find /home /root \( -path '*/.aws/credentials' -o -path '*/.docker/config.json' -o -path '*/.kube/config' \) -type f -exec chmod 600 {} \; 2>/dev/null

# 4. file berisi rahasia di /etc yang bisa dibaca "others" (periksa manual)
sudo grep -rIlE --exclude-dir={teleport,ssl,ssh,ca-certificates,apt,pam.d,security,ufw,alternatives} \
  '(PASSWORD|PASSWD|SECRET|TOKEN)[A-Za-z_]*=[^[:space:]]+' /etc 2>/dev/null | xargs -r sudo stat -c '%a %U:%G %n'
```

File world-readable di area data dan sisa backup:

```bash
sudo find /home /opt /srv /var/www /var/backups /root -xdev -type f -perm /007 -not -path '*/teleport/*' -not -path '/srv/lab/public/*' -not -path '/srv/course/*' 2>/dev/null | head -50
# setelah ditinjau:
# sudo find /home /opt /srv /var/www /var/backups -xdev -type f -perm /007 -not -path '/srv/lab/public/*' -not -path '/srv/course/*' -exec chmod o-rwx {} +

sudo find / -xdev -type f \( -name '*.bak' -o -name '*.old' -o -name '*.orig' -o -name '*.save' -o -name '*~' -o -name '*.swp' -o -name '*.sql' -o -name '*.dump' -o -name '*.tar' -o -name '*.tar.gz' -o -name '*.zip' \) \
  -not -path '/usr/*' -not -path '/var/lib/*' -not -path '/snap/*' -not -path '/proc/*' -not -path '/var/cache/*' -not -path '/var/log/*' -not -path '/root/backup-hardening/*' -ls 2>/dev/null
```

## 57. Kalau service tidak boleh dimatikan, amankan konfigurasinya

Policy tertulis "no anonymous FTP", artinya FTP bisa saja tetap ada asal tidak anonymous. Blok ini hanya jalan kalau service-nya ada.

```bash
# vsftpd
if [ -f /etc/vsftpd.conf ]; then
  sudo cp /etc/vsftpd.conf /root/backup-hardening/vsftpd.conf.bak
  sudo tee /etc/vsftpd.conf >/dev/null <<'EOF'
listen=YES
listen_ipv6=NO
anonymous_enable=NO
local_enable=YES
write_enable=NO
chroot_local_user=YES
allow_writeable_chroot=NO
hide_ids=YES
ls_recurse_enable=NO
xferlog_enable=YES
log_ftp_protocol=YES
ssl_enable=YES
force_local_logins_ssl=YES
force_local_data_ssl=YES
ssl_sslv2=NO
ssl_sslv3=NO
ssl_tlsv1=NO
ssl_ciphers=HIGH
require_ssl_reuse=NO
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
pam_service_name=vsftpd
userlist_enable=YES
userlist_deny=NO
userlist_file=/etc/vsftpd.userlist
EOF
  : | sudo tee /etc/vsftpd.userlist >/dev/null     # daftar kosong = default deny
  sudo chmod 600 /etc/vsftpd.conf && sudo systemctl restart vsftpd
fi

# snmpd: hapus community "public", hanya localhost, hanya subtree system
if [ -f /etc/snmp/snmpd.conf ]; then
  sudo cp /etc/snmp/snmpd.conf /root/backup-hardening/snmpd.conf.bak
  C=$(openssl rand -hex 12)
  sudo tee /etc/snmp/snmpd.conf >/dev/null <<EOF
agentaddress udp:127.0.0.1:161
view systemonly included .1.3.6.1.2.1.1
rocommunity $C 127.0.0.1 -V systemonly
sysLocation Lab
sysContact admin@localhost
EOF
  sudo chmod 600 /etc/snmp/snmpd.conf && sudo systemctl restart snmpd
fi
```

## 58. Lapisan Data: enkripsi dan backup (slide "encrypt, back up")

Sertifikat TLS dengan izin yang benar:

```bash
sudo install -d -m 700 /etc/ssl/lab
sudo openssl req -x509 -newkey rsa:3072 -sha256 -nodes -days 365 \
  -keyout /etc/ssl/lab/server.key -out /etc/ssl/lab/server.crt \
  -subj "/CN=$(hostname -f)/O=Lab" -addext "subjectAltName=DNS:$(hostname -f)"
sudo chmod 600 /etc/ssl/lab/server.key; sudo chmod 644 /etc/ssl/lab/server.crt
sudo chmod 710 /etc/ssl/private 2>/dev/null
sudo openssl x509 -in /etc/ssl/lab/server.crt -noout -subject -dates
grep -nE 'MinProtocol|CipherString' /etc/ssl/openssl.cnf
```

Backup terenkripsi AES-256 beserta timer harian:

```bash
sudo install -d -m 700 /var/backups/secure /root/.keys
[ -f /root/.keys/backup.key ] || { openssl rand -base64 48 | sudo tee /root/.keys/backup.key >/dev/null; sudo chmod 400 /root/.keys/backup.key; }

sudo tee /usr/local/sbin/secure-backup.sh >/dev/null <<'EOF'
#!/bin/bash
set -eu
umask 077
OUT=/var/backups/secure/data-$(date +%F-%H%M).tar.gz.enc
tar -czf - /etc /srv /home /opt 2>/dev/null | openssl enc -aes-256-cbc -pbkdf2 -iter 200000 -salt -pass file:/root/.keys/backup.key -out "$OUT"
sha256sum "$OUT" > "$OUT.sha256"
find /var/backups/secure -type f -mtime +14 -delete
EOF
sudo chmod 700 /usr/local/sbin/secure-backup.sh

sudo tee /etc/systemd/system/secure-backup.service >/dev/null <<'EOF'
[Unit]
Description=Encrypted backup
[Service]
Type=oneshot
ExecStart=/usr/local/sbin/secure-backup.sh
EOF
sudo tee /etc/systemd/system/secure-backup.timer >/dev/null <<'EOF'
[Unit]
Description=Daily encrypted backup
[Timer]
OnCalendar=daily
Persistent=true
[Install]
WantedBy=timers.target
EOF
sudo systemctl daemon-reload && sudo systemctl enable --now secure-backup.timer
sudo /usr/local/sbin/secure-backup.sh && ls -l /var/backups/secure
sudo bash -c 'openssl enc -d -aes-256-cbc -pbkdf2 -iter 200000 -pass file:/root/.keys/backup.key -in "$(ls -t /var/backups/secure/*.enc | head -1)" | tar -tzf - | head -3'
```

Volume terenkripsi LUKS (data at rest). Hanya jalan kalau lab mendukung loop device. Tidak dimasukkan ke `crypttab` supaya boot tidak tersangkut.

```bash
if [ -e /dev/loop-control ] && sudo apt-get install -y cryptsetup >/dev/null; then
  sudo fallocate -l 256M /var/lib/secure.img && sudo chmod 600 /var/lib/secure.img
  openssl rand -base64 48 | sudo tee /root/.keys/luks.key >/dev/null; sudo chmod 400 /root/.keys/luks.key
  sudo cryptsetup luksFormat --batch-mode --type luks2 --cipher aes-xts-plain64 --key-size 512 --hash sha256 --key-file /root/.keys/luks.key /var/lib/secure.img
  sudo cryptsetup open --key-file /root/.keys/luks.key /var/lib/secure.img securevol
  sudo mkfs.ext4 -q /dev/mapper/securevol
  sudo mkdir -p /srv/secure && sudo mount -o nosuid,nodev,noexec /dev/mapper/securevol /srv/secure
  sudo chmod 700 /srv/secure
  sudo cryptsetup status securevol
else
  echo "loop device tidak tersedia, lewati"
fi
```

## 59. Parameter boot, sysctl tambahan, systemd, dan logind

```bash
sudo tee /etc/default/grub.d/99-zz-more.cfg >/dev/null <<'EOF'
GRUB_CMDLINE_LINUX="$GRUB_CMDLINE_LINUX randomize_kstack_offset=on page_alloc.shuffle=1 vsyscall=none debugfs=off"
EOF
sudo update-grub

sudo tee /etc/sysctl.d/93-more.conf >/dev/null <<'EOF'
net.ipv6.conf.all.router_solicitations = 0
net.ipv6.conf.default.router_solicitations = 0
net.ipv6.conf.all.accept_ra_rtr_pref = 0
net.ipv6.conf.default.accept_ra_rtr_pref = 0
net.ipv6.conf.all.accept_ra_pinfo = 0
net.ipv6.conf.default.accept_ra_pinfo = 0
net.ipv4.conf.all.route_localnet = 0
net.ipv4.conf.all.accept_local = 0
net.ipv4.conf.default.accept_local = 0
kernel.ctrl-alt-del = 0
EOF
sudo sysctl --system >/dev/null

sudo mkdir -p /etc/systemd/system.conf.d /etc/systemd/logind.conf.d
sudo tee /etc/systemd/system.conf.d/99-hardening.conf >/dev/null <<'EOF'
[Manager]
CtrlAltDelBurstAction=none
DumpCore=no
DefaultLimitCORE=0
EOF
sudo tee /etc/systemd/logind.conf.d/99-hardening.conf >/dev/null <<'EOF'
[Login]
KillUserProcesses=yes
StopIdleSessionSec=900
RemoveIPC=yes
NAutoVTs=0
HandlePowerKey=ignore
HandleSuspendKey=ignore
HandleHibernateKey=ignore
HandleLidSwitch=ignore
EOF
sudo systemctl daemon-reload
sudo systemctl kill -s HUP systemd-logind

sudo systemctl set-default multi-user.target
```

Cek mitigasi CPU dan Secure Boot (hanya membaca):

```bash
grep . /sys/devices/system/cpu/vulnerabilities/* | sed 's|/sys/devices/system/cpu/vulnerabilities/||'
mokutil --sb-state 2>/dev/null
cat /proc/cmdline
```

## 60. Polkit: default deny untuk user non-admin

Slide "default deny" diterapkan ke polkit. Root dan group `sudo` tidak terpengaruh. Rollback: hapus file lalu `sudo systemctl restart polkit`.

```bash
if [ -d /etc/polkit-1 ]; then
  sudo tee /etc/polkit-1/rules.d/00-hardening.rules >/dev/null <<'EOF'
polkit.addRule(function(action, subject) {
    if (subject.user == "root" || subject.isInGroup("sudo")) {
        return polkit.Result.NOT_HANDLED;
    }
    return polkit.Result.NO;
});
EOF
  sudo chmod 644 /etc/polkit-1/rules.d/00-hardening.rules
  sudo systemctl restart polkit
  sudo -u nobody bash -c 'pkcheck --action-id org.freedesktop.login1.reboot --process $$; echo "exit=$? (1 = ditolak)"'
fi
```

## 61. Patching lanjutan

```bash
sudo apt-get install -y needrestart
sudo mkdir -p /etc/needrestart/conf.d
sudo tee /etc/needrestart/conf.d/99-hardening.conf >/dev/null <<'EOF'
$nrconf{restart} = 'a';
$nrconf{override_rc}->{qr(^teleport)} = 0;
$nrconf{override_rc}->{qr(^systemd-logind)} = 0;
EOF
sudo needrestart -b | head       # mode batch, tidak merestart apa pun

sudo tee /etc/apt/apt.conf.d/53unattended-extra >/dev/null <<'EOF'
Unattended-Upgrade::SyslogEnable "true";
Unattended-Upgrade::Automatic-Reboot-WithUsers "false";
Unattended-Upgrade::InstallOnShutdown "false";
EOF

grep -nE '^\s*"\$\{distro_id\}.*security' /etc/apt/apt.conf.d/50unattended-upgrades   # repo security tidak boleh dikomentari
ubuntu-security-status 2>/dev/null || sudo pro security-status
ls /etc/apt/trusted.gpg 2>/dev/null; ls /etc/apt/trusted.gpg.d/                      # keyring legacy sebaiknya kosong
sudo dpkg --audit; sudo apt-get check
dpkg -l | awk '/^rc/{print $2}' | xargs -r sudo apt-get purge -y                     # sisa konfigurasi paket yang sudah dihapus

# timer yang menghubungi internet. JANGAN sentuh apt-daily*.timer
for t in ua-timer.timer update-notifier-download.timer update-notifier-motd.timer fwupd-refresh.timer; do
  sudo systemctl disable --now "$t" 2>/dev/null; sudo systemctl mask "$t" 2>/dev/null
done
sudo pro config set apt_news=false 2>/dev/null
systemctl list-timers --all --no-pager
```

## 62. Rekonsiliasi firewall dan listener, plus jebakan port scan

```bash
echo "== Listener non-loopback vs aturan UFW =="
sudo ss -tulnH | awk '{print $1, $5}' | grep -vE '127\.|\[::1\]|%lo' | while read -r proto addr; do
  port=${addr##*:}
  if sudo ufw status | grep -qE "^$port(/| )"; then s="ADA ATURAN UFW"; else s="TIDAK ADA ATURAN (hanya default deny): perlu atau tidak?"; fi
  echo "$proto $addr -> $s"
done
ip -br link | grep -i promisc || echo "tidak ada interface promiscuous"
```

Jebakan port scan (opsional): sumber yang menyentuh port umpan diblokir 10 menit. Hapus port dari daftar kalau ada yang benar-benar dipakai. Kalau auditor memindai dari laptopnya ke port umpan, IP-nya ikut terblokir 10 menit.

```bash
sudo cp /etc/ufw/before.rules /root/backup-hardening/before.rules.step62
cat > /tmp/ufw-trap.rules <<'EOF'
# --- hardening: perangkap port scan ---
-A ufw-before-input -m recent --name portscan --rcheck --seconds 600 -j DROP
-A ufw-before-input -p tcp -m multiport --dports 23,2323,3389,445,1433,3306,5900,6379,9200 --syn -m recent --name portscan --set -j DROP
EOF
grep -q 'perangkap port scan' /etc/ufw/before.rules || sudo sed -i '/^# End required lines/r /tmp/ufw-trap.rules' /etc/ufw/before.rules
sudo ufw reload || sudo cp /root/backup-hardening/before.rules.step62 /etc/ufw/before.rules
sudo ufw status | head -3
sudo iptables -S ufw-before-input | grep -i recent
```

## 63. Integritas library dan binary

```bash
# persistensi lewat preload library
[ -e /etc/ld.so.preload ] && { echo "ADA ld.so.preload:"; cat /etc/ld.so.preload; } || echo "ld.so.preload tidak ada"
env | grep -E '^LD_(PRELOAD|LIBRARY_PATH)=' || echo "LD_* bersih"
sudo grep -rE 'LD_PRELOAD|LD_LIBRARY_PATH' /etc/environment /etc/profile /etc/profile.d /etc/bash.bashrc /root/.bashrc /home/*/.bashrc /home/*/.profile 2>/dev/null
sudo chmod 644 /etc/ld.so.conf; sudo chmod -R go-w /etc/ld.so.conf.d

# direktori dan file sistem tidak boleh bisa ditulis group/other
sudo find /bin /sbin /usr/bin /usr/sbin /lib /usr/lib /usr/local/bin /usr/local/sbin /etc -xdev ! -type l -perm /022 ! -perm -1000 2>/dev/null | head -30
# setelah ditinjau:
# sudo find /bin /sbin /usr/bin /usr/sbin /lib /usr/lib /usr/local/bin /usr/local/sbin -xdev ! -type l -perm /022 ! -perm -1000 -exec chmod go-w {} +

# downloader hanya untuk root dan group sudo (curl opsional, lewati kalau dipakai script lab)
for b in wget curl; do
  p=$(command -v $b) && p=$(readlink -f "$p") && sudo dpkg-statoverride --update --add root sudo 0750 "$p" 2>/dev/null && echo "dibatasi: $p"
done

# /boot dan tmp
sudo chmod go-rwx /boot/vmlinuz-* /boot/initrd.img-* /boot/System.map-* 2>/dev/null
sudo chmod 1777 /tmp /var/tmp
```

## 64. Password GRUB (RISIKO rendah, aktif setelah reboot)

Mencegah orang di konsol mengedit parameter boot untuk masuk sebagai root. Ini bukan password sudo. Password tampil di terminal (ikut tercatat di rekaman sesi), jadi catat lalu ganti kalau perlu.

```bash
sudo cp /etc/grub.d/10_linux /root/backup-hardening/10_linux.bak
sudo cp /etc/grub.d/40_custom /root/backup-hardening/40_custom.bak
GP=$(openssl rand -base64 15 | tr -d '/+=')
HASH=$(printf '%s\n%s\n' "$GP" "$GP" | grub-mkpasswd-pbkdf2 | awk '/PBKDF2 hash/{print $NF}')
echo "Password GRUB untuk grubadmin: $GP"

sudo tee -a /etc/grub.d/40_custom >/dev/null <<EOF
set superusers="grubadmin"
password_pbkdf2 grubadmin $HASH
EOF
sudo sed -i 's/^CLASS="--class gnu-linux --class gnu --class os"/CLASS="--class gnu-linux --class gnu --class os --unrestricted"/' /etc/grub.d/10_linux
sudo update-grub
sudo grub-script-check /boot/grub/grub.cfg && echo "grub.cfg valid"
sudo grep -c unrestricted /boot/grub/grub.cfg      # harus > 0 supaya boot normal tidak minta password
```

Rollback: salin balik dua file `.bak` di `/root/backup-hardening/` ke `/etc/grub.d/`, lalu `sudo update-grub`.

## 65. Zona waktu dan retensi log

```bash
sudo timedatectl set-timezone Asia/Jakarta
timedatectl | grep -E 'Time zone|synchronized'

sudo sed -i 's/^rotate .*/rotate 12/' /etc/logrotate.conf
grep -qE '^compress' /etc/logrotate.conf || echo 'compress' | sudo tee -a /etc/logrotate.conf >/dev/null
sudo sed -i 's/^\(\s*\)rotate 4$/\1rotate 12/' /etc/logrotate.d/rsyslog /etc/logrotate.d/ufw 2>/dev/null
sudo logrotate -d /etc/logrotate.conf 2>&1 | grep -i error || echo "logrotate OK"
```

## 66. Verifikasi zero trust dari laptop

Ketik sendiri di laptop, bukan di server. Ini hanya membaca.

```bash
tsh status        # lihat user, role, logins, dan masa berlaku sertifikat
tsh ls            # hanya lab yang diizinkan role yang tampil
```

## Urutan pengerjaan

1. Aman: 54, 55, 56, 57, 61, 63, 65
2. Konfigurasi biasa: 58, 59, 60
3. Menyentuh firewall: 62 (siapkan tab Teleport kedua)
4. Paling akhir: 64

Sebelum menutup tab lama, pastikan akses lab masih hidup:

```bash
systemctl is-active teleport && sudo -n true && sudo ufw status | head -3
```