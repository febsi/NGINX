# Optimasi dan Keamanan Server Nginx di Linux: Dari Hardening hingga Load Testing

keamanan dan kinerja server web sangat penting untuk menjaga integritas dan ketersediaan layanan. Proyek ini bertujuan untuk mengoptimalkan dan mengamankan server Nginx yang berjalan di sistem operasi Linux melalui serangkaian langkah yang mencakup hardening, instalasi Nginx dengan modul Brotli, pembuatan sertifikat SSL, dan pengujian beban menggunakan alat k6.
 
### 1. Hardening linux

Hardening Linux adalah proses meningkatkan keamanan sistem operasi Linux dengan mengurangi potensi kerentanan dan mengamankan konfigurasi sistem. Dalam bagian ini, kita akan membahas langkah-langkah yang diambil untuk mengamankan server Linux, termasuk pengaturan SSH, pengelolaan pengguna, dan perlindungan terhadap serangan brute-force. Hardering yang saya lakuakn saya lakuakn adalah Passwodless, Ganti port Default untuk ssh, menonaktifkan PING, dan menggunakan Fail2ban.  

#### a. Passwordless.

1. Pertama kita buat linux mengupdate secara otomatis.
    ```bash
    sudo apt install unattended-upgardes
    sudo dpkg-reconfigure --priority=low unattended-upgardes
    ```

2. Tambahkan user dan masukan user yang sudah dibuat ke dalam user sudo.

    - Tambah user.
    ```bash
    sudo adduser <user>
    ```

    - Masukan user ke user sudo.
    ```bash
    sudo usermod -aG sudo <user>
    ```

    - Tukar user.
    ```bash
    sudo su - febri
    ```

3. Setalah melakukan otomatisa update pada serever baru kitamembuat  posswordless.

    - Kita masuk ke file ~/.ssh pada linux dan kita buat izin 700 dimana cuman hanya pemilik direktori yang dapat mengaksesnya.
    ```bash
     cd ~/.ssh
     sudo chmod 700 ~/.ssh
    ```

    - Setelah itu kita create publik/privet key pada komputer yang ingin mengakses linux server kita dengan ssh dengan perintah.
    ```bash
    ssh-keygen -b 4096
    ```
    Dapat kita lihat bahwa public key dan privet key sudah dibuat.
    
    ![baru](Gambar/gambar1.png)

    Dapat kita lihat pada folder ~/.ssh sudah terdapat file id_rsa(privet key) dan id_rsa.pub(public key).

    - Uplaoud file public key ke dalam linux server dengan perintah.
    ```bash
    scp $env:USERPROFILE/.ssh/id_rsa.pub <user@ip ubuntu>:~/ssh/authorized_keys
    ```
    ![baru](Gambar/gambar2.png)

    Kita coba ssh dari komputer kita.

    ![baru](Gambar/gambar3.png)

#### b. Ganti Port DEfault.

1. Buka file sshd_config dengan perintah.
    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

    Ubah configurasi sesuai dengan gambar dibawah

![baru](Gambar/gambar4.png)


2. Restart ssh server nya dengan perintah.
    ```bash
    sudo systemctl restart sshd
    ```

3. Kita coba mengakses ssh nya dengan perintah yang sama.

![baru](Gambar/gambar5.png)

Pasti tidak bisa sebab port yang kita gunakan sudah kita ganti.

4. Kita coba dengan dengan menambahkan port yang sudah kita masukan kedalam konfigurasi. sebelum itu pastikan firewall menngizinkan koneksi port yang diatur dengan perintah.

    ```bash 
    sudo ufw allow 717/tcp
    ```

![baru](Gambar/gambar6.png)

Setalh itu baru buka ssh dengan port yang ditentukan.

![baru](Gambar/gambar7.png)

#### c. PING server

Kita buat ip addres kita tidak bisa di ping agar server ip kita tidak terdeteksi dari server lain.

- Buka file /etc/ufw/before.rules setelah itu tambah commandi pada bagian icmp INPUT dengan peintah.

    ```bash
    sudo nano /etc/ufw/before.rules
    ```
    
    Tambahkan
    ```bash
    -A ufw-before-input -p icmp --icmp-type echo-request -j DROP
    ```
    Reload Firewall
    ```bash
    sudo ufw reload
    ```

    ping ip server linux dari server lain.
    
    ![baru](Gambar/gambar8.png)

    Akan tetapi saat server masih dapat melakukan ssh.
    
    ![baru](Gambar/gambar9.png)

#### d. Fail2ban
Fail2ban adalah software yang menggunakan bahasa  pyhton untuk melindungi sistem kita dari serangan brute-force. berikut cara pengunaan fail2ban di linux ubuntu.

- Instal fail2band dengan perintah.
    ```bash 
    sudo apt install fail2ban
    ```

- Setelah itu jalankan fail2ban dan lihat statusnya dengan perintah.
    ```bash
    sudo systemctl start fail2ban
    sudo systemctl status fail2ban
    ```

    ![baru](Gambar/gambar10.png)

- Setelah itu kita aktifkan fail2ban agar otomatis running meski sistem kita mengalami restart atau reboot.
    ```bash
    sudo systemctl enable fail2ban
    ```

- Copy file jail.conf 
    ```bash
    sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
    ```

- Setelah itu kita configurasi pada jail.conf.
    ```bash
    sudo nano /etc/fail2ban/jail.local
    ```

    Pada bagian ssh tambahkan command

    ```bash
    enable  = true // aktifkan fail2ban
    findtime = 10m //waktu lama nya untuk melakukan login
    maxretry = 4 //batas mencoba 
    bandtime = 2h //waktu ban ip yang mencoba paksa masuk
    ```
- Setelah itu kita uji konfigurasi fail2ban yang sudah di buat valid atau tidak dengan perintah.
    ```bash 
    sudo fail2ban-client -d
    ```

- Restart fail2ban dan lihat status.
    ```bash 
    sudo systemctl restart fail2ban
    sudo systemctl status fail2ban
    ```
![baru](Gambar/gambar11.png)

- Kita cek status status konfigurasi yang dikerjaan oleh fail2ban
    ```bash
    sudo fail2ban-client status
    ```
![baru](Gambar/gambar12.png)

Uji coba fail2ban pada sshd yang sudah kita atur. Dengan cara memasukan user yang salah tapi dengan ip dan port yang sama.

![baru](Gambar/gambar13.png)

Lihat statsu fail2ban.

![baru](Gambar/gambar14.png)


### 2. Install Nginx dengan module Brotil

Nginx adalah server web yang populer dan efisien. Dalam bagian ini, kita menginstal Nginx dan menambahkan modul Brotli untuk meningkatkan kompresi konten, yang dapat mempercepat waktu muat halaman dan mengurangi penggunaan bandwidth. Proses ini mencakup pengunduhan, kompilasi, dan konfigurasi Nginx untuk memanfaatkan modul Brotli.

1. Update server kita dan install depedensi yang diperlukan.
    ```bash
    sudo apt update
    sudo apt install -y build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev wget
    ```

2. Unduh Source code nginx.
    ```bash
    wget https://nginx.org/download/nginx-1.25.3.tar.gz
    ```

3. Ekstrak File Source Code
    ```bash
     tar -xvzf nginx-1.25.3.tar.gz
     cd nginx-1.25.3
    ```

4. Lakukan konfigurasi dan komplikasi.
    ```bash
      ./configure --prefix=/usr/local/nginx --with-http_ssl_modul --with-http_gzip_static_module
    ```

    Kompilasi dan instal.
    ```bash
     make
     sudomake install
    ```

5. Tambahkan Nginx ke PATH
    ```bash
     echo 'export PATH=/usr/local/nginx/sbin:$PATH' >> ~/.bashrc
     source ~/.bashrc
    ```

6. Setelah itu cek konfigurasi.
    ```bash
     sudo nginx -t
    ```

7. Menambahkan module brotil.
    ```bash
    sudo git clone https://github.com/google/ngx_brotli.git
    cd ngx_brotli
    sudo git submodule update --init
    cd ..
    ```

8. Konfigurasi ulang nginx dengan modul Brotli.
    ```bash
     ./configure --prefix=/usr/local/nginx -with-http_ssl_module -with-http_realip_module  --add-module=./ngx_brotli
    ```

9. Kompilasi dan install.
    ```bash
     make 
     sudo make install
    ```

10. Konfigurasi nginx untuk menggunakan brotli.
    ```bash
    sudo nano /usr/local/nginx/conf/nginx.conf
    ```

    Tambahkan konfigurasi berikut pada blok 'http'.
    ```bash
    brotli on;
    brotli_comp_level 6; 
    brotli_types text/plain text/css application/javascript application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
    ```

11. Restart nginx
    ```bash
    sudo nginx -s reload
    ```

12. Verifikasi Brotli berhasil atau tidak.
    ```bash
    curl -H "Accept-Encoding: br" -I http://localhost
    ```
    ![baru](Gambar/gambar15.png)

    pengujian brotli pada nginx.

- Buat direktori web.
    ```bash
     sudo mkdir -p /usr/local/nginx/html
     cd /usr/local/nginx/html
     sudo nano index.html
    ```
 ![baru](Gambar/gambar16.png)

- Verifikasi brotli.
    ```bash
     curl -H "Accept-Encoding: br" -I http://localhost
    ```
    ![baru](Gambar/gambar17.png)
    brotli berhasil mengkompresi file html.

### 3 buat ssl certificate dengan self signed.

Keamanan data yang ditransmisikan antara server dan klien sangat penting. Dengan membuat sertifikat SSL self-signed, kita dapat mengenkripsi komunikasi dan melindungi informasi sensitif. Konfigurasi SSL di Nginx memastikan bahwa semua data yang dikirimkan aman dan terlindungi dari penyadapan.

- Kita buat SSL directory.
    ```bash
     cd /usr/local/nginx
     sudo mkdir SSL
    ```

- Buat file infoemasi tentang SSL
    ```bash
     sudo nano self-info.txt
     [req]
     default_bits       = 2048
     prompt      = no
     default_keyfile    = localhost.key
     distinguished_name = dn
     req_extensions     = req_ext
     x509_extensions    = v3_ca

     [ dn ]
     C = PH
     ST = NCR
     L = Manila
     O = localhost
     OU = Development
     CN = localhost

     [req_ext]
     subjectAltName = @alt_names

     [v3_ca]
     subjectAltName = @alt_names

     [alt_names]
     DNS.1   = localhost
     DNS.2   = 127.0.0.1
    ```

- Jalankan petintah OpenSSl.
    ```bash 
     sudo openssl req -x509 -nodes -days 3652 -new key rsa:2048 -keyout localhost.key -out localhost.crt -config ssl-info.txt
    ```

    Cek direktori ssl.
    ![baru](Gambar/gambar18.png)


- Buka konfigurasi nginx dan tambahkan printah beriku pada bagian server.
    ```bash
     sudo nano /usr/local/nginx/conf/nginx.conf

     listen 443 ssl;
     listen [::]:443 ssl;

     ssl_certificate /usr/local/nginx/ssl/localhost.crt;
     ssl_certificate_key /usr/local/nginx/ssl/localhost.key;

     ssl_protocols TLSv1.2 TLSv1.1 TLSv1;
     ```

- Kita izinkan port 80 dan 443 melalui firewall agar dapat diakses oleh  server.
    ```bash
     sudo ufw allow 80
     sudo ufw allow 443
    ```

- Setelah itu restar nginx dengan perintah.
    ```bash
     sudo systemctl reload nginx
    ```

- Kita lihat tampilan web yang sudah kita buat.
    ![baru](Gambar/gambar19.png)

    dapat kita lihat bahwa ssl berhasil kita tambahkan.

### 4. Load Testing dengan K6

Setelah server dikonfigurasi dan dioptimalkan, penting untuk menguji kinerjanya di bawah beban. Dengan menggunakan alat k6, kita dapat mensimulasikan banyak pengguna virtual yang mengakses server secara bersamaan. Pengujian ini membantu kita memahami bagaimana server menangani permintaan tinggi dan mengidentifikasi potensi masalah kinerja.

- Update server danInstal K6 dengan perintah 
    ```bash
     sudo apt update
     sudo apt install k6
    ```

- Buat direktori k6 dan buat file baru untuk konfigurasi k6.
    ```bash
     mkdir k6
     sudo nano <nama_file>.js
     ```

- Run file k6 yang sudah kita buat dengan perintah.
    ```bash
     k6 run <nama_file>.js
    ```
    ![baru](Gambar/gambar20.png)

Kita coba web yang sudah kita buat dengam 100 virtual user yang masuk ke web kita dengan duration 40 detik dan kita lihat usage pada server kita dengan perintah.
    ```bash 
    k6 run --vus 100 --duration 40s main.js
    ```

![baru](Gambar/gambar21.png)

Dapat kita lihat usage resorce yang terjadi saat load testing yang dilakukan k6.
![baru](Gambar/gambar22.png)

Inilah hasil dari log load test dari k6.
![baru](Gambar/gambar23.png)



