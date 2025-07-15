# Nginx

1. Setup linux di Virtual Box.

2. Setup SSH pake passwordless, ganti port default, hardening, bila perlu coba test

- Penginstalan SSH pada linux.
    ```bash
    sudo apt update 
    sudo apt install openssh-server
    ```
!(gambar1.png)

3. Instal Nginx sama module brotil(compile buka pake apt), terus coba test setup simple aplikasi, terus buat ssl certs nya pake yang self-signed juga gpp, terus kalau udah nanti coba load test pake k6s atau locus, atau apalah bebas buat mastiin konfigurasi mu udah ok atau blm, pastiin config nginx nya  juga udah well-turned.

 fasfas
