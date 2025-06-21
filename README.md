# a. Setup Direktori dan Pembuatan User

pertama membuat direktori dahulu
### Buat source directory dan subfoldernya
```bash
sudo mkdir -p /home/shared_files


sudo mkdir -p /home/shared_files/public
sudo mkdir -p /home/shared_files/private_yuadi
sudo mkdir -p /home/shared_files/private_irwandi
```

### TAMBAHKAN USER yuadi dan irwandi
```bash
sudo adduser yuadi
sudo adduser irwandi
```

### Set permission agar hanya owner bisa baca file private
```bash
sudo chown yuadi:yuadi /home/shared_files/private_yuadi
sudo chmod 700 /home/shared_files/private_yuadi

sudo chown irwandi:irwandi /home/shared_files/private_irwandi
sudo chmod 700 /home/shared_files/private_irwandi
```
### direktori public bebas di akses
```bash
sudo chmod 755 /home/shared_files/public
```

# b. Akses Mount Point
Buat direktori kosong untuk mount point:
```bash
sudo mkdir -p /mnt/secure_fs
```

mount point ini untuk lokasi fuse nanti. Setelah itu aku membuat fusecure.c di dalam direktori sendiri yaitu `fuse_secure_fs`.
dan jangan lupa untuk menginstall libfuse terlebih dahulu.
```bash
sudo apt-get update
sudo apt-get install libfuse-dev
```

setelah itu kompile fusecure yang telah dibuat menggunakan perintah ini.
```bash
gcc -Wall `pkg-config fuse --cflags` fusecure.c -o fusecure `pkg-config fuse --libs`
```

Setelah itu menjalankan FUSE mount agar `source directory` ter-mount di `/mnt/secure_fs`.
```bash
sudo ./fusecure /mnt/secure_fs -o allow_other
```
`-o allow_other` agar semua user (yuadi dan irwandi) bisa melihat kontennya.

Setelah itu uji mount point menggunakan `ls -l /mnt/secure_fs`, maka akan menampilkan `public/`, `private_yuadi/`, dan `private_irwandi/`.


# d. Akses Public Folder
Pastikan permission di source direktori sudah benar
```bash
sudo chmod 755 /home/shared_files/public
```

misal jika ingin membuat file di dalam soure direktori contohnya seperti ini
```bash
echo "Ini materi kuliah algoritma" > /home/shared_files/public/materi_kuliah.txt
```

dan jangan lupa untuk menjalankan
```bash
sudo chmod 777 /home/shared_files/public
```

(777 membuat siapa pun bisa membuat file di dalam public.)
maka otomatis di dalam direktori `public` akan terdapat file `materi_kuliah.txt` yang berisi tulisan `Ini materi kuliah algoritma`

Kemudian cobalah melihat isinya misalnya dengan menggunakan `cat /mnt/secure_fs/public/materi_kuliah.txt`

# e. Akses Private Folder yang Terbatas

coba login pada masing-masing user yang sudah dibuat
```bash
su - yuadi
su - irwandi
```
kemudian check apakah sudah berhasil
```bash
cat /mnt/secure_fs/public/materi_kuliah.txt
cat /mnt/secure_fs/private_yuadi/jawaban.c
```

# JIKA INGIN UNMOUNT
```bash
sudo fusermount -u /mnt/secure_fs
```
# Jika ingin membuat file di yaudi atau irwandi harus pindah direktori dahulu
```bash
cat /mnt/secure_fs/private_yuadi/jawaban_praktikum1.c
```
kemudian nano [nama_file]
