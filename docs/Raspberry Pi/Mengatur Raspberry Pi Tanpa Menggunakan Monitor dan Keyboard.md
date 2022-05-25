Raspberry Pi adalah komputer kecil berdaya rendah yang serbaguna. Kita bisa memanfaatkannya untuk berbagai macam kebutuhan, mulai dari PC Desktop, Retro Game Emulator, Server, hingga Wi-Fi Extender. The possibilities are endless. Tapi untuk kebutuhan seperti server yang tidak begitu membutuhkan tampilan desktop, setupnya kadang suka bikin pusing. Kita harus menyiapkan mouse, keyboard, dan monitor untuk sekedar masuk ke Terminal.

Sekarang, dengan Raspberry Pi Imager versi terbaru, semua itu tidak diperlukan lagi. Kita bisa langsung mengakses Raspberry Pi setelah booting, selama kita terkoneksi ke jaringan yang sama. Alat-alat yang dibutuhkan adalah:

* Raspberry Pi & Power Adapter
* Kartu MicroSD
* PC/Laptop (Windows, macOS ataupun Linux)
* Card reader (opsional)

## Setting up
1. Siapkan PC/Laptop, kemudian download dan install aplikasi **Raspberry Pi Imager** melalui website [Raspberry Pi](https://www.raspberrypi.com/software/). Untuk pengguna Linux selain Ubuntu dan turunannya, cek ketersediaan aplikasi pada masing-masing package manager/distro.
	![](https://miro.medium.com/max/1400/1*4kC7bhCqhrc1iuWU5jYKsQ.png)

2. Masukkan Kartu MicroSD pada slot yang tersedia di PC/Laptop. Jika tidak ada, bisa menggunakan card reader.

3. Buka aplikasi **Raspberry Pi Imager**, klik **Choose OS**, pilih **Raspberry Pi OS (other)**, kemudian pilih **Raspberry Pi OS Lite (32-bit)**. Ini adalah varian Raspberry Pi OS yang lebih ringan karena tidak disediakan Desktop GUI.
	![](https://miro.medium.com/max/1368/1*xk5WNbMEn7AUcFU-dze1VQ.png)
	![](https://miro.medium.com/max/1368/1*p_vm2KrRrF8qOrwfMMT-qQ.png)
	![](https://miro.medium.com/max/1368/1*QJxquzmSzBkbya5uY0CCMw.png)

4. Pilih **Choose Storage**, kemudian pilih lokasi MicroSD. Pastikan drive yang dipilih sudah benar, karena seluruh data akan dihapus dan diganti dengan OS yang sudah dipilih.
	![](https://miro.medium.com/max/1368/1*CANBWiJ2QnmC5aZQlOCqTw.png)
	![](https://miro.medium.com/max/1368/1*fdyR22RGZdXlEcAeDxsvtg.png)

5. Klik **Settings** (icon gear), kemudian beri tanda centang pada **Enable SSH**, Set username dan password dan **Configure Wi-Fi**. Masukkan password Wi-Fi pada kolom password, kemudian pastikan SSID Wi-Fi yang dipilih merupakan jaringan yang sama dengan yang digunakan PC/Laptop. Seluruh Settings ini harus diaktifkan agar kita bisa melakukan konfigurasi pada Raspberry Pi tanpa menggunakan monitor dan keyboard.
	![](https://miro.medium.com/max/1368/1*2LtyOdevxs0Ektugc0yOsg.png)
	![](https://miro.medium.com/max/1368/1*pujUX5T0DhUwezlThULdRg.png)
	![](https://miro.medium.com/max/1368/1*L7EZeqwXuqFntgqIEhN85Q.png)

6. Klik **Write**, pilih **Yes** pada perintah yang muncul, kemudian tunggu hingga seluruh proses selesai. Setelah proses selesai, tekan **Continue**, kemudian cabut kartu MicroSD dari PC/Laptop dan masukkan kartu pada slot MicroSD yang ada pada Raspberry Pi.
	![](https://miro.medium.com/max/1368/1*hKZY-PGIGNMAo8qJbZvvFA.png)
	![](https://miro.medium.com/max/1368/1*UUyAenIxuC36myMjbVCPBQ.png)
	![](https://miro.medium.com/max/1368/1*l3v6bSIb1o3BsJ5Gp2zsjw.png)
	![](https://miro.medium.com/max/1368/1*FXiCR4jh1ixx_63PHr06mQ.png)

7. Nyalakan Raspberry Pi dengan menyambungkan ke sumber listrik menggunakan Power Adapter.

8. Tunggu sekitar 1-2 menit hingga Raspberry Pi selesai booting, kemudian buka aplikasi **Terminal** pada PC/Laptop, lalu ketik `ssh pi@raspberrypi`. Ketik `yes` pada prompt yang muncul, kemudian masukkan password yang sudah dibuat melalui menu Settings pada Raspberry Pi Imager tadi.

> Tidak perlu bingung jika password tidak muncul saat diketik, Linux menyembunyikan password secara default.

![](https://miro.medium.com/max/1400/1*GS2kpEMZaPUjiutgHL958A.png)

Jika tidak bisa masuk menggunakan perintah `ssh pi@raspberrypi`, maka kita harus masuk menggunakan perintah `ssh pi@[Alamat IP]`. Untuk mengetahui alamat IP perangkat Raspberry Pi, kita bisa menggunakan aplikasi seperti [Angry IP Scanner](https://angryip.org/), [Advanced IP Scanner](https://www.advanced-ip-scanner.com/), ataupun [mRemoteNG](https://mremoteng.org/). Kita juga bisa mengetahui alamat IP dengan cara login ke web interface pada router Wi-Fi.

Sekarang, kita sudah masuk ke dalam perangkat Raspberry Pi yang ditandai dengan berubahnya posisi path menjadi `pi@raspberrypi:~ $`. Saatnya bereksperimen!