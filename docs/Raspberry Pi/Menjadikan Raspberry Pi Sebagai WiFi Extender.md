Jika sinyal Wi-Fi di rumah jangkauannya lemah, dan punya Raspberry Pi nganggur yang nggak tau mau dibuat apa, kita bisa manfaatkan komputer kecil ini sebagai Wi-Fi Extender.

Bahan-bahan yang dibutuhkan:
- Raspberry Pi & Power Adapter
- Dongle Wi-Fi USB
- Kartu MicroSD
- PC/Laptop (Windows, macOS ataupun Linux)

Pastikan Kartu MicroSD yang akan dipasang pada Raspberry Pi sudah diinstal Raspbian OS Lite (32-bit), diaktifkan ssh-nya, dan sudah disambungkan ke jaringan Wi-Fi yang sama. Lihat [[Mengatur Raspberry Pi Tanpa Menggunakan Monitor dan Keyboard]]

Untuk langkah pertama, kita pastikan bahwa Raspberry Pi sudah tersambung ke listrik dan sudah selesai booting. Pastikan juga bahwa Dongle Wi-Fi USB sudah kita colokkan ke salah satu port USB yang ada pada Raspberry Pi. Kemudian kita masuk ke Terminal pada Raspberry Pi melalui ssh menggunakan PC/Laptop.

Setelah masuk ke Raspberry Pi, kita jalankan **update** dan **upgrade**.
```bash
sudo apt update
sudo apt upgrade
```

Kemudian install **dnsmasq** dan **hostapd**. Kedua tools ini dibutuhkan untuk manajemen dns dan mem-broadcast perangkat kita sebagai hotspot.
```bash
sudo apt install dnsmasq hostapd iptables
```


## Konfigurasi wpa_supplicant
Sebelum kita mengatur hotspot, pastikan saat ini perangkat Raspberry Pi kita telah terkoneksi ke jaringan Wi-Fi yang kita inginkan untuk di-extend.
Untuk mengubah konfigurasi wpa_supplicant, jalankan perintah berikut:
```shell
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

Pastikan pengaturan ssid dan psk (password Wi-Fi) sudah sesuai.

Simpan file dengan tombol Ctrl + X, Y, kemudian Enter

## Konfigurasi dhcpcd
Selanjutnya, kita set IP static pada Dongle Wi-Fi USB yang nantinya akan kita atur sebagai pemancar hotspot. Jadi, Wi-Fi internal berfungsi sebagai penangkap sinyal Wi-Fi, dan Dongle berfungsi sebagai pemancar hotspot. Untuk mengatur IP Static, kita edit file dhcpcd.conf menggunakan perintah ini:
```bash
sudo nano /etc/dhcpcd.conf
```

Kemudian masukkan teks di bawah ini:
```
interface wlan1
static ip_address=192.168.220.1/24
static routers=192.168.220.0
```

Simpan file dengan tombol Ctrl + X, Y, kemudian Enter

Jalankan perintah ini untuk merestart dhcpd
```shell
sudo service dhcpcd restart
```


## Konfigurasi hostapd
Kemudian, kita akan mengubah konfigurasi pada hostapd.conf dengan memasukkan perintah berikut:

```shell
sudo nano /etc/hostapd/hostapd.conf
```

Tambahkan teks di bawah ini pada file tersebut:

```
interface=wlan1
hw_mode=g
channel=6
ieee80211n=1
wmm_enabled=1
HT_CAPAB=[HT40+][SHORT-GI-20][DSSS_CCK-40]
macaddr_acl=0
ignore_broadcast_ssid=0
country_code=ID

auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP

ssid=WifiName
wpa_passphrase=WifiPassword
```

interface wlan1 pada konfigurasi di atas adalah interface untuk dongle Wi-Fi, sedangkan interface wlan0 adalah perangkat Wi-Fi bawaan Raspberry Pi.
Ganti **ssid** dan **wpa_passphrase** sesuai dengan nama yang diinginkan.

Simpan file dengan tombol Ctrl + X, Y, kemudian Enter

Kemudian, kita juga harus mengubah konfigurasi default agar hostapd dapat mengenali konfigurasi yang kita baru saja tambahkan.

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

Simpan file dengan tombol Ctrl + X, Y, kemudian Enter

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

Simpan file dengan tombol Ctrl + X, Y, kemudian Enter


## Konfigurasi dnsmasq
Selanjutnya, kita akan mengubah konfigurasi **dnsmasq**. Sebelum mengubah konfigurasi tersebut, ada baiknya jika kita backup terlebih dahulu. Jalankan perintah berikut:

```shell
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
```

Ubah file dnsmasq.conf dengan menjalankan perintah berikut:
```shell
sudo nano /etc/dnsmasq.conf
```

Tambahkan teks di bawah ini:
```
interface=wlan1
listen-address=192.168.220.1 
domain-needed
bogus-priv 
dhcp-range=192.168.220.50,192.168.220.150,12h  
```

Simpan file dengan tombol Ctrl + X, Y, kemudian Enter

## Konfigurasi sysctl
Langkah selanjutnya adalah mem-forward (meneruskan) traffic dari koneksi wlan1 ke wlan0

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

Simpan file dengan tombol Ctrl + X, Y, kemudian Enter

Agar pengaturan tersebut tetap tersimpan saat perangkat booting, jalankan perintah berikut:
```shell
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
```

## Konfigurasi iptables
Untuk menambahkan aturan iptables, jalankan perintah berikut:
```shell
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE  
sudo iptables -A FORWARD -i wlan0 -o wlan1 -m state --state RELATED,ESTABLISHED -j ACCEPT  
sudo iptables -A FORWARD -i wlan1 -o wlan0 -j ACCEPT
```

Untuk menyimpan aturan iptables secara permanen, jalankan perintah berikut:
```shell
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

kemudian jalankan perintah berikut:
```shell
sudo nano /etc/rc.local
```

tambahkan teks di bawah ini , tempatkan diatas tulisan `exit 0`:
```
iptables-restore < /etc/iptables.ipv4.nat
```

Simpan file dengan tombol Ctrl + X, Y, kemudian Enter

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

Referensi:
- https://pimylifeup.com/raspberry-pi-wifi-extender/
- https://github.com/iotJumpway/RPI-Examples/blob/master/_DOCS/4-Securing-Your-Raspberry-Pi-With-IPTables.md
- https://forums.fogproject.org/topic/15357/dnsmasq-won-t-start-automatically-solved/2