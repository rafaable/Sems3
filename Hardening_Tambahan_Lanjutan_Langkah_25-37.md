# Hardening Tambahan (Lanjutan Langkah 25 dst.)

Semua perintah di bawah belum ada di dua panduan sebelumnya. Aturan tetap: tanpa port 22, tanpa password sudo, tanpa menyentuh Teleport, tanpa install runtime. Urutannya dari yang paling aman ke yang berisiko.

## 25. Service dan paket tambahan

```bash
for u in bluetooth ModemManager wpa_supplicant rsync lldpd postfix exim4 multipathd packagekit udisks2 pollinate lxd snmptrapd nfs-idmapd rpc-statd smbd nmbd named dnsmasq autofs ypserv ntp openvpn; do
  if systemctl list-unit-files "$u*" --no-legend 2>/dev/null | grep -q .; then
    sudo systemctl disable --now "$u" 2>/dev/null
    sudo systemctl mask "$u" 2>/dev/null
    echo "mask: $u"
  fi
done

for p in avahi-daemon cups cups-daemon bluez modemmanager rpcbind nfs-kernel-server nfs-common autofs samba isc-dhcp-server slapd bind9 dovecot-core squid xinetd; do
  dpkg -s "$p" >/dev/null 2>&1 && sudo apt-get purge -y "$p"
done
sudo apt-get autoremove --purge -y

# GUI tidak boleh ada di server
dpkg -l | grep -Ei 'xserver|gdm3|lightdm|ubuntu-desktop' || echo "tidak ada GUI"

# wireless mati
sudo rfkill block all 2>/dev/null
nmcli radio all off 2>/dev/null
```

Compiler dicabut supaya penyerang tidak bisa build exploit di server (opsional, cek dulu):

```bash
dpkg -l gcc g++ make build-essential 2>/dev/null | grep '^ii'
sudo apt-get purge -y gcc g++ make build-essential 2>/dev/null; sudo apt-get autoremove --purge -y
```

## 26. SSH lanjutan (hanya konfigurasi, SSH tetap tidak dinyalakan)

Pada sshd nilai pertama yang menang. File `00-` sudah memakai nama itu, jadi file ini memakai `01-` dan hanya berisi opsi baru.

```bash
sudo groupadd -f sshusers
sudo usermod -aG sshusers ubuntu
id auditor >/dev/null 2>&1 && sudo usermod -aG sshusers auditor

sudo tee /etc/ssh/sshd_config.d/01-hardening-extra.conf >/dev/null <<'EOF'
AllowGroups sshusers
PermitUserEnvironment no
GatewayPorts no
PermitTunnel no
DisableForwarding yes
StrictModes yes
PrintLastLog yes
PrintMotd no
Compression no
UseDNS no
UsePAM yes
MaxStartups 10:30:60
KexAlgorithms sntrup761x25519-sha512@openssh.com,curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512
HostKeyAlgorithms ssh-ed25519,rsa-sha2-512,rsa-sha2-256
PubkeyAcceptedAlgorithms ssh-ed25519,rsa-sha2-512,rsa-sha2-256
EOF
sudo chmod 600 /etc/ssh/sshd_config.d/*.conf

# buang parameter Diffie-Hellman pendek (< 3072 bit)
sudo cp /etc/ssh/moduli /etc/ssh/moduli.bak
sudo awk '$5 >= 3071' /etc/ssh/moduli.bak | sudo tee /etc/ssh/moduli >/dev/null

# konfigurasi SSH client (ssh keluar dari server)
sudo tee /etc/ssh/ssh_config.d/99-hardening.conf >/dev/null <<'EOF'
Host *
    HashKnownHosts yes
    ForwardAgent no
    ForwardX11 no
    PasswordAuthentication no
    StrictHostKeyChecking ask
EOF

# root tidak boleh punya authorized_keys
sudo rm -f /root/.ssh/authorized_keys

# audit kunci publik yang lemah (ssh-rsa < 2048 atau DSA = hapus barisnya)
for f in /home/*/.ssh/authorized_keys; do [ -f "$f" ] && echo "== $f" && sudo ssh-keygen -l -f "$f"; done

sudo sshd -t && echo "SSHD OK"
sudo sshd -T | grep -Ei '^(allowgroups|disableforwarding|maxstartups|kexalgorithms|permituserenvironment|usedns)'
```

## 27. Password, PAM, dan login.defs

```bash
# pwquality tambahan
sudo mkdir -p /etc/security/pwquality.conf.d
sudo tee /etc/security/pwquality.conf.d/99-extra.conf >/dev/null <<'EOF'
difok = 8
maxrepeat = 3
maxsequence = 3
gecoscheck = 1
enforce_for_root
retry = 3
EOF

# riwayat password: tidak boleh memakai 5 password terakhir
sudo tee /usr/share/pam-configs/pwhistory >/dev/null <<'EOF'
Name: pwhistory password history checking
Default: yes
Priority: 512
Password-Type: Primary
Password:
    requisite pam_pwhistory.so remember=5 use_authtok
EOF
sudo DEBIAN_FRONTEND=noninteractive pam-auth-update --enable pwhistory
grep -E 'pwhistory|pwquality' /etc/pam.d/common-password

# login.defs
setdef(){ if grep -qE "^#?\s*$1\b" /etc/login.defs; then sudo sed -i -E "s|^#?\s*$1\b.*|$1 $2|" /etc/login.defs; else echo "$1 $2" | sudo tee -a /etc/login.defs >/dev/null; fi; }
setdef LOGIN_RETRIES 3
setdef LOGIN_TIMEOUT 60
setdef SYSLOG_SU_ENAB yes
setdef SYSLOG_SG_ENAB yes
setdef HOME_MODE 0750
setdef PASS_WARN_AGE 7
setdef USERGROUPS_ENAB yes
setdef ENCRYPT_METHOD YESCRYPT
setdef YESCRYPT_COST_FACTOR 11
sudo sed -i 's/^DIR_MODE=.*/DIR_MODE=0750/' /etc/adduser.conf

# umask lewat PAM dan bashrc
sudo apt-get install -y libpam-umask libpam-tmpdir
for f in /etc/pam.d/common-session /etc/pam.d/common-session-noninteractive; do
  grep -q pam_umask "$f" || echo 'session optional pam_umask.so' | sudo tee -a "$f" >/dev/null
done
grep -q 'umask 027' /etc/bash.bashrc || echo 'umask 027' | sudo tee -a /etc/bash.bashrc >/dev/null

# akun user biasa: kunci otomatis 30 hari setelah password kedaluwarsa (ubuntu dilewati)
for u in $(awk -F: '$3>=1000 && $3<65534 && $1!="ubuntu"{print $1}' /etc/passwd); do sudo chage -I 30 "$u"; done

# akun sistem: password dikunci dan shell nologin
for u in $(awk -F: '$3<1000 && $1!="root"{print $1}' /etc/passwd); do
  sudo passwd -l "$u" >/dev/null 2>&1
  sudo usermod -s /usr/sbin/nologin "$u" 2>/dev/null
done
sudo usermod -s /bin/sync sync 2>/dev/null
```

Kalau policy menyebut password aging, pilih salah satu sesuai teksnya:

```bash
# gaya CIS (ganti paksa maksimal 365 hari)
sudo sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS   365/;s/^PASS_MIN_DAYS.*/PASS_MIN_DAYS   1/' /etc/login.defs
# gaya NIST 800-63B-4 (tidak ada paksa ganti): biarkan PASS_MAX_DAYS 99999
```

## 28. Kernel dan boot

```bash
sudo tee /etc/sysctl.d/96-hardening-more.conf >/dev/null <<'EOF'
kernel.kexec_load_disabled = 1
kernel.perf_event_paranoid = 3
kernel.core_uses_pid = 1
kernel.apparmor_restrict_unprivileged_userns = 1
vm.mmap_min_addr = 65536
vm.unprivileged_userfaultfd = 0
dev.tty.ldisc_autoload = 0
net.ipv4.tcp_rfc1337 = 1
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.all.proxy_arp = 0
net.ipv4.conf.all.bootp_relay = 0
net.ipv4.icmp_ratelimit = 100
net.ipv6.conf.all.forwarding = 0
net.ipv6.conf.default.accept_ra_defrtr = 0
EOF
sudo sysctl --system | tail -3

# parameter boot: AppArmor, audit sebelum auditd jalan, hardening memori
sudo tee /etc/default/grub.d/99-hardening.cfg >/dev/null <<'EOF'
GRUB_CMDLINE_LINUX="$GRUB_CMDLINE_LINUX apparmor=1 security=apparmor audit=1 audit_backlog_limit=8192 init_on_alloc=1 slab_nomerge"
EOF
sudo update-grub
sudo chmod 600 /boot/grub/grub.cfg
sudo chown root:root /boot/grub/grub.cfg
```

Parameter boot baru aktif setelah reboot. Jangan reboot lab kecuali memang diizinkan.

## 29. Firewall lanjutan

```bash
sudo ufw allow in on lo
sudo ufw allow out on lo
sudo ufw deny in from 127.0.0.0/8
sudo ufw deny in from ::1
sudo ufw deny out 21/tcp
sudo ufw deny out 69/udp
sudo ufw deny out 161/udp
sudo ufw deny out 514/tcp comment 'no rsh'
sudo ufw logging high
sudo ufw reload
sudo ufw status numbered
```

Cek `ss -tulpn` untuk service yang listen di `0.0.0.0` atau `[::]`. Service yang tidak perlu dibuat listen ke localhost saja.

## 30. DNS dan waktu

```bash
sudo mkdir -p /etc/systemd/resolved.conf.d
sudo tee /etc/systemd/resolved.conf.d/99-hardening.conf >/dev/null <<'EOF'
[Resolve]
LLMNR=no
MulticastDNS=no
DNSSEC=allow-downgrade
EOF
sudo systemctl restart systemd-resolved

sudo mkdir -p /etc/systemd/timesyncd.conf.d
sudo tee /etc/systemd/timesyncd.conf.d/99-ntp.conf >/dev/null <<'EOF'
[Time]
NTP=0.id.pool.ntp.org 1.id.pool.ntp.org
FallbackNTP=ntp.ubuntu.com
EOF
sudo systemctl restart systemd-timesyncd
sudo timedatectl set-ntp true
```

## 31. File dan akun: pengecekan lanjutan

```bash
# file backup akun tidak boleh terbaca umum
sudo chmod 600 /etc/passwd- /etc/group- /etc/shadow- /etc/gshadow- 2>/dev/null
sudo chmod 644 /etc/shells /etc/hosts /etc/issue /etc/issue.net
sudo chmod 600 /etc/ssh/sshd_config.d/*.conf

# banner tidak boleh membocorkan versi OS
sudo truncate -s 0 /etc/motd
sudo chmod -x /etc/update-motd.d/* 2>/dev/null
grep -E '\\[mrsv]' /etc/issue /etc/issue.net || echo "banner bersih"

# dotfile milik user tidak boleh bisa ditulis orang lain
sudo find /home /root -maxdepth 2 -name '.*' -type f -perm /022 -exec chmod go-w {} \; 2>/dev/null

# PATH root tidak boleh berisi '.' atau direktori world-writable
sudo bash -c 'echo $PATH' | tr ':' '\n' | while read d; do [ -z "$d" -o "$d" = "." ] && echo "BAHAYA: PATH kosong atau ."; [ -d "$d" ] && [ -n "$(find "$d" -maxdepth 0 -perm -0002)" ] && echo "WORLD-WRITABLE: $d"; done

# hapus SUID dari program yang tidak perlu, permanen walau paket di-update
for f in /usr/bin/chfn /usr/bin/chsh /usr/bin/newgrp /usr/bin/pkexec; do
  [ -u "$f" ] && sudo dpkg-statoverride --update --add root root 0755 "$f" 2>/dev/null && echo "SUID dicabut: $f"
done

# shell yang valid di /etc/shells
cat /etc/shells

# batasi jumlah login dan proses
sudo tee /etc/security/limits.d/98-limits.conf >/dev/null <<'EOF'
* hard maxlogins 10
* hard nproc 4096
* soft nproc 2048
EOF
grep -q pam_limits /etc/pam.d/common-session || echo 'session required pam_limits.so' | sudo tee -a /etc/pam.d/common-session >/dev/null

# file hardening dikunci agar tidak berubah tanpa sengaja (lepas dengan chattr -i)
sudo chattr +i /etc/ssh/sshd_config.d/00-hardening.conf /etc/sudoers.d/00-hardening
lsattr /etc/ssh/sshd_config.d/00-hardening.conf
```

## 32. APT dan integritas paket

```bash
sudo tee /etc/apt/apt.conf.d/99-hardening >/dev/null <<'EOF'
APT::Get::AllowUnauthenticated "false";
Acquire::AllowInsecureRepositories "false";
Acquire::AllowDowngradeToInsecureRepositories "false";
APT::Install-Recommends "false";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Mail "root";
EOF

# repo tidak boleh memakai trusted=yes atau http polos
grep -rnE 'trusted=yes|allow-insecure|\[trusted' /etc/apt/ || echo "repo bersih"
grep -rn '^deb http://' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null

# paket yang filenya berubah dari aslinya
sudo apt-get install -y debsums
sudo debsums -s 2>/dev/null | head

# anti-malware dan rootkit
sudo apt-get install -y rkhunter chkrootkit
sudo rkhunter --update >/dev/null 2>&1; sudo rkhunter --propupd >/dev/null 2>&1
sudo rkhunter -c --sk --rwo
sudo chkrootkit 2>/dev/null | grep -i infected || echo "chkrootkit: bersih"
```

## 33. Logging: permission dan konfigurasi

```bash
# file log baru dibuat 0640
grep -R 'FileCreateMode' /etc/rsyslog.conf /etc/rsyslog.d/ || echo '$FileCreateMode 0640' | sudo tee /etc/rsyslog.d/99-filemode.conf
sudo systemctl restart rsyslog

# journald
sudo tee /etc/systemd/journald.conf.d/98-extra.conf >/dev/null <<'EOF'
[Journal]
ForwardToSyslog=yes
Seal=yes
MaxRetentionSec=1month
EOF
sudo systemctl restart systemd-journald

# auditd
sudo sed -i 's/^log_group.*/log_group = adm/;s/^max_log_file .*/max_log_file = 32/;s/^num_logs.*/num_logs = 5/;s/^space_left_action.*/space_left_action = email/;s/^action_mail_acct.*/action_mail_acct = root/' /etc/audit/auditd.conf
sudo chmod 750 /var/log/audit
sudo chmod 640 /var/log/audit/*.log* 2>/dev/null
sudo service auditd restart

# permission file log lain
sudo find /var/log -type f -name '*.log' -exec chmod g-wx,o-rwx {} \; 2>/dev/null
sudo chmod 640 /var/log/auth.log /var/log/syslog /var/log/kern.log 2>/dev/null
```

## 34. DAC lanjutan: default ACL dan role per direktori

```bash
sudo groupadd -f role-dev; sudo groupadd -f role-ops; sudo groupadd -f role-audit
sudo mkdir -p /srv/app/{src,config,logs}
sudo chown -R root:root /srv/app

# dev: tulis di src, tidak bisa lihat config
sudo setfacl -m g:role-dev:rwx /srv/app/src
sudo setfacl -d -m g:role-dev:rwx /srv/app/src
# ops: baca/tulis config dan logs
sudo setfacl -m g:role-ops:rwx /srv/app/config /srv/app/logs
sudo setfacl -d -m g:role-ops:rwx /srv/app/config /srv/app/logs
# audit: hanya baca
sudo setfacl -m g:role-audit:rx /srv/app/src /srv/app/config /srv/app/logs
sudo setfacl -d -m g:role-audit:rx /srv/app/src /srv/app/config /srv/app/logs
sudo chmod 750 /srv/app/config
sudo chmod -R o-rwx /srv/app

# default ACL untuk direktori rahasia dari langkah sebelumnya
sudo setfacl -d -m o::--- /srv/lab/internal /srv/lab/secret
getfacl -p /srv/app/config
```

Separation of duties: auditor tidak boleh punya hak ubah.

```bash
for u in auditor; do id "$u" >/dev/null 2>&1 && { sudo gpasswd -d "$u" sudo 2>/dev/null; sudo gpasswd -d "$u" admin 2>/dev/null; sudo gpasswd -d "$u" docker 2>/dev/null; sudo gpasswd -d "$u" lxd 2>/dev/null; sudo gpasswd -a "$u" role-audit; }; done
sudo grep -rn 'ALL=(ALL' /etc/sudoers /etc/sudoers.d/ | grep -v '^#'
```

## 35. ABAC ringan: batasi akun berdasarkan waktu dan asal

Demo atribut waktu dan lokasi untuk satu akun contoh saja. Akun `ubuntu`, `auditor`, dan `root` tidak terkena, dan tidak ada aturan catch-all yang menolak semua akun.

```bash
sudo useradd -m -s /bin/bash -G role-dev abacdemo 2>/dev/null
sudo passwd -l abacdemo

sudo tee -a /etc/security/time.conf >/dev/null <<'EOF'
login;*;abacdemo;Wk0800-1700
sshd;*;abacdemo;Wk0800-1700
EOF

sudo tee -a /etc/security/access.conf >/dev/null <<'EOF'
+:ubuntu:ALL
+:root:ALL
+:abacdemo:LOCAL 10.0.0.0/24
-:abacdemo:ALL
EOF

for f in /etc/pam.d/login /etc/pam.d/sshd; do
  [ -f "$f" ] && ! grep -q pam_access "$f" && sudo sed -i '0,/^@include common-account/s//account required pam_access.so\naccount required pam_time.so\n@include common-account/' "$f"
done
grep -n 'pam_access\|pam_time' /etc/pam.d/login /etc/pam.d/sshd 2>/dev/null
```

## 36. MAC lanjutan: AppArmor

```bash
sudo apt-get install -y apparmor-profiles-extra
sudo systemctl enable --now apparmor
sudo aa-enforce /etc/apparmor.d/* 2>/dev/null | tail -2
sudo aa-status | grep -E 'profiles are in|processes are unconfined'
sudo aa-unconfined 2>/dev/null | head -15
sudo aa-complain --help >/dev/null 2>&1 && sudo aa-status --complaining
```

Kalau `aa-status --complaining` masih menampilkan profil, jalankan `sudo aa-enforce /etc/apparmor.d/NAMA_PROFIL` untuk masing-masing.

## 37. Kalau lab menjalankan web server atau database

Setiap blok hanya jalan bila softwarenya ada, jadi aman dicopas semua.

```bash
# Apache
if [ -d /etc/apache2 ]; then
  sudo sed -i 's/^ServerTokens.*/ServerTokens Prod/;s/^ServerSignature.*/ServerSignature Off/;s/^TraceEnable.*/TraceEnable Off/' /etc/apache2/conf-available/security.conf
  sudo a2dismod -f autoindex status info 2>/dev/null
  sudo apache2ctl configtest && sudo systemctl reload apache2
fi

# Nginx
if [ -d /etc/nginx ]; then
  grep -q server_tokens /etc/nginx/nginx.conf && sudo sed -i 's/^\s*#\?\s*server_tokens.*/    server_tokens off;/' /etc/nginx/nginx.conf
  sudo nginx -t && sudo systemctl reload nginx
fi

# Postfix hanya localhost
command -v postconf >/dev/null && sudo postconf -e 'inet_interfaces = loopback-only' && sudo systemctl reload postfix

# MySQL/MariaDB, Redis, PostgreSQL hanya localhost
[ -d /etc/mysql ] && sudo grep -rn 'bind-address' /etc/mysql/ 
[ -f /etc/redis/redis.conf ] && sudo sed -i 's/^bind .*/bind 127.0.0.1 -::1/;s/^protected-mode.*/protected-mode yes/' /etc/redis/redis.conf
[ -d /etc/postgresql ] && sudo grep -rn "^listen_addresses" /etc/postgresql/
```

## Urutan paling aman

Kerjakan langkah 25, 27, 28, 30, 31, 32, 33, 34, 36 lebih dulu, lalu 26, 29, 35.

Setelah langkah 27, 29, dan 35 (PAM dan firewall), pastikan lab tetap bisa diakses:

```bash
systemctl is-active teleport && sudo ufw status | head -3
```

Buka tab Teleport baru sebelum menutup tab lama. Kalau ada yang bermasalah, backup ada di `/root/backup-hardening/` (dari langkah 0).