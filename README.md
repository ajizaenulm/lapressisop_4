# a.

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

# b.

