# MPI-Numerik
# Panduan Pembuatan Master dan Slave untuk Open MPI pada Ubuntu Desktop

Laporan ini menjelaskan langkah-langkah pembuatan master dan slave, konfigurasi SSH, konfigurasi NFS, instalasi MPI, dan langkah-langkah terkait untuk menjalankan kodingan numerik pada Ubuntu Desktop.

## Daftar Isi
- [Device dan Tools yang Perlu Disiapkan](#device-dan-tools-yang-perlu-disiapkan)
- [Topologi Bridged](#topologi-bridged)
- [Pembuatan Master dan Slave](#pembuatan-master-dan-slave)
- [Konfigurasi SSH](#konfigurasi-ssh)
- [Konfigurasi NFS](#konfigurasi-nfs)
- [Instalasi MPI](#instalasi-mpi)
- [Running Code Python](#running-code-python)

## Device dan Tools yang Perlu Disiapkan
1. Ubuntu Desktop
   - Ubuntu Desktop Master
   - Ubuntu Desktop Slave 1
   - Ubuntu Desktop Slave 2
   - Ubuntu Desktop Slave 3
2. MPI (Master dan Slave)
3. SSH (Master dan Slave)
4. NFS (Master dan Slave)
5. Code Bubble Sort Python

## Topologi Bridged
![Topologi](https://github.com/NauvalPerdana/MPI-Numerik/blob/main/Topologi.png)

## Pembuatan Master dan Slave

1. Pastikan setiap master dan slave menggunakan Network Bridge Adapter dan terhubung ke internet.
2. Tentukan perangkat mana yang akan dijadikan master, slave1, slave2, dan slave3.
3. Buat user baru dengan perintah berikut pada master dan setiap slave:

    ```bash
    sudo adduser mpiuser
    ```

    Gantilah bagian 'master' menjadi 'slave1', 'slave2', dan seterusnya untuk slave.

4. Berikan akses kepada root dengan perintah:

    ```bash
    sudo usermod -aG sudo mpiuser
    ```

    Lakukan langkah di atas untuk setiap slave dengan mengganti pengguna 'master' menjadi 'slave1', 'slave2', dan seterusnya.

5. Masuk ke server dengan pengguna `mpiuser`:

    ```bash
    su - mpiuser
    ```

6. Update Ubuntu Desktop dan install tools:

    ```bash
    sudo apt update && sudo apt upgrade
    sudo apt install net-tools vim
    ```

7. Konfigurasi file `/etc/hosts` pada master, slave1, slave2, dan slave3. Daftarkan IP dan hostname masing-masing komputer.

## Konfigurasi SSH

1. Install OpenSSH pada master dan semua slave:

    ```bash
    sudo apt install openssh-server
    ```

2. Generate key pada master:

    ```bash
    ssh-keygen -t rsa
    ```

3. Copy key public ke setiap slave. Gunakan perintah berikut pada direktori `.ssh`:

    ```bash
    cd .ssh
    cat id_rsa.pub | ssh mpiuser@slave1 "mkdir .ssh; cat >> .ssh/authorized_keys"
    ```

    Ulangi perintah di atas untuk setiap slave.

## Konfigurasi NFS

1. Buat shared folder pada master dan setiap slave:

    ```bash
    mkdir numeric
    ```

2. Install NFS pada master:

    ```bash
    sudo apt install nfs-kernel-server
    ```

3. Konfigurasi file `/etc/export` pada master. Tambahkan baris berikut pada akhir file:

    ```plaintext
    /home/mpiuser/numeric *(rw,sync,no_root_squash,no_subtree_check)
    ```

    Lokasi Shared Folder adalah direktori tempat file numeric di atas dibuat.

4. Restart NFS Server:

    ```bash
    sudo exportfs -a
    sudo systemctl restart nfs-kernel-server
    ```

5. Install NFS pada setiap slave:

    ```bash
    sudo apt install nfs-common
    ```

6. Mount shared folder dari master ke setiap slave:

    ```bash
    sudo mount master:/home/mpiuser/numeric /home/mpiuser/numeric
    ```

    Ulangi perintah di atas untuk setiap slave.

## Instalasi MPI

1. Install Open MPI pada master dan semua slave:

    ```bash
    sudo apt install openmpi-bin libopenmpi-dev
    ```

2. Install library MPI melalui pip:

    ```bash
    sudo apt install python3-pip
    pip install mpi4py
    ```

3. Install Numpy pada master dan setiap slave:

    ```bash
    pip3 install numpy
    ```

## Running Code Python

1. Buka direktori numeric:

    ```bash
    cd numeric
    ```

2. Edit file Python (misalnya `num.py`) dengan code numerik.
   [num.py code](https://github.com/NauvalPerdana/MPI-Numerik/blob/main/num.py)

3. Jalankan code Python dengan MPI:

    ```bash
    mpirun -np 3 -hosts master,slave1,slave2 python3 num.py
    ```
   ![Output](https://github.com/NauvalPerdana/MPI-Numerik/blob/main/output.png)
   Hasil eksekusi akan ditampilkan dengan runtime yang diukur.
