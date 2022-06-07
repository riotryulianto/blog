Jika sinyal Wi-Fi di rumah jangkauannya lemah, dan kebetulan punya Raspberry Pi menganggur yang tidak tau mau dibuat apa, kita bisa manfaatkan komputer kecil ini sebagai Wireless Access Point. Dengan memanfaatkan port Ethernet yang tersedia, koneksi internet bisa menjadi jauh lebih stabil dibandingkan dengan menggunakan dongle Wi-Fi.

Alat-alat yang dibutuhkan adalah:

- Raspberry Pi & Power Adapter
- Kabel LAN (Ethernet/RJ45)
- Kartu MicroSD
- PC/Laptop (Windows, macOS ataupun Linux)

Pastikan Kartu MicroSD yang akan dipasang pada Raspberry Pi sudah dipasang Raspbian OS Lite (32-bit), diaktifkan ssh-nya, dan sudah disambungkan ke jaringan yang sama dengan PC/Laptop yang kita gunakan. Lihat [[Mengatur Raspberry Pi Tanpa Menggunakan Monitor dan Keyboard]]

Untuk langkah awal, pastikan bahwa Raspberry Pi sudah tersambung ke listrik dan sudah selesai booting. Pastikan juga bahwa kabel LAN sudah dicolok ke port Ethernet/RJ45 yang ada pada Raspberry Pi. Kemudian masuk ke Terminal pada Raspberry Pi melalui `ssh` menggunakan PC/Laptop.

Setelah masuk ke Raspberry Pi, jalankan **update** dan **upgrade**.
```bash
sudo apt update
sudo apt upgrade
```

Kemudian install **dnsmasq** dan **hostapd**. Kedua tools ini dibutuhkan untuk manajemen DNS dan mem-broadcast perangkat kita sebagai hotspot.
```bash
sudo apt install dnsmasq hostapd iptables
```


## Konfigurasi dhcpcd
Selanjutnya, set IP static pada dongle Wi-Fi USB yang nantinya akan diatur sebagai pemancar hotspot. Jadi, Wi-Fi internal berfungsi sebagai penangkap sinyal Wi-Fi, dan dongle berfungsi sebagai pemancar hotspot. Untuk mengatur IP Static, kita edit file **dhcpcd.conf** menggunakan perintah ini:
```bash
sudo nano /etc/dhcpcd.conf
```

Kemudian masukkan teks di bawah ini:
```
interface wlan1
static ip_address=192.168.4.1
nohook wpa_supplicant
```

Simpan file dengan menekan tombol **Ctrl + X**, **Y**, kemudian **Enter**.

Jalankan perintah ini untuk me-restart **dhcpd**:
```shell
sudo systemctl restart dhcpcd
```


## Konfigurasi hostapd
Kemudian, ubah konfigurasi pada **hostapd.conf** dengan memasukkan perintah berikut:
```shell
sudo nano /etc/hostapd/hostapd.conf
```

Tambahkan teks di bawah ini pada file tersebut:
```
interface=wlan0
hw_mode=g
channel=6
ieee80211n=1
wmm_enabled=0
macaddr_acl=0
ignore_broadcast_ssid=0

auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP

ssid=WifiName
wpa_passphrase=WifiPassword
```

Ganti ***ssid*** dan ***wpa_passphrase*** sesuai dengan nama yang diinginkan.

Simpan file dengan menekan tombol **Ctrl + X**, **Y**, kemudian **Enter**.

Kemudian, ubah konfigurasi default agar **hostapd** dapat mengenali konfigurasi yang baru saja ditambahkan.

Jalankan perintah berikut:
```shell
sudo nano /etc/default/hostapd
```

Cari teks berikut:
```
#DAEMON_CONF=""
```

Kemudian ubah menjadi:
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

Simpan file dengan menekan tombol **Ctrl + X**, **Y**, kemudian **Enter**.

Ubah juga konfigurasi init.d dengan menjalankan:
```shell
sudo nano /etc/init.d/hostapd
```

Cari teks berikut:
```
#DAEMON_CONF=
```

Kemudian ubah menjadi:
```
DAEMON_CONF=/etc/hostapd/hostapd.conf
```

Simpan file dengan menekan tombol **Ctrl + X**, **Y**, kemudian **Enter**.


## Konfigurasi dnsmasq
Selanjutnya, ubah konfigurasi **dnsmasq**. Sebelum mengubah konfigurasi tersebut, ada baiknya jika konfigurasi tersebut di-backup terlebih dahulu. Jalankan perintah berikut:

```shell
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
```

Ubah file **dnsmasq.conf** dengan menjalankan perintah berikut:
```shell
sudo nano /etc/dnsmasq.conf
```

Tambahkan teks di bawah ini:
```
interface=wlan0
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h 
```

Simpan file dengan menekan tombol **Ctrl + X**, **Y**, kemudian **Enter**.


## Konfigurasi sysctl
Langkah selanjutnya adalah meneruskan traffic dari koneksi **wlan1** ke **wlan0**.

Jalankan perintah berikut:
```shell
sudo nano /etc/sysctl.conf
```

Hapus tanda pagar pada line berikut:
```
#net.ipv4.ip_forward=1
```

Sehingga menjadi seperti ini:
```
net.ipv4.ip_forward=1
```

Simpan file dengan menekan tombol **Ctrl + X**, **Y**, kemudian **Enter**

Agar pengaturan tersebut tetap tersimpan saat perangkat booting, jalankan perintah berikut:
```shell
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
```


## Konfigurasi iptables
Untuk menambahkan aturan firewall pada **iptables**, jalankan perintah berikut:
```shell
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

Untuk menyimpan aturan iptables secara permanen, jalankan perintah berikut:
```shell
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

Kemudian jalankan perintah berikut:
```shell
sudo nano /etc/rc.local
```

Tambahkan teks di bawah ini , tempatkan di atas tulisan `exit 0`:
```
iptables-restore < /etc/iptables.ipv4.nat
```

Simpan file dengan menekan tombol **Ctrl + X**, **Y**, kemudian **Enter**


## Konfigurasi autostart menggunakan systemctl
Jalankan kedua perintah di bawah:
```shell
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd
sudo systemctl enable dnsmasq
sudo systemctl start dnsmasq
```

Reboot untuk memastikan bahwa Wi-Fi Extender telah berfungsi dengan baik.

### Referensi:
- https://pimylifeup.com/raspberry-pi-wireless-access-point/comment-page-1/

* https://www.maketecheasier.com/how-to-turn-raspberry-pi-into-wireless-access-point/

- https://raspberrypi-guide.github.io/networking/create-wireless-access-point

- https://forums.fogproject.org/topic/15357/dnsmasq-won-t-start-automatically-solved/2