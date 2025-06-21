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
cd /home/shared_files/private_yuadi
```
 ```
#define FUSE_USE_VERSION 28
#include <fuse.h>
#include <string.h>
#include <errno.h>
#include <sys/stat.h>

static const char *hello_path = "/hello.txt";
static char file_content[1024] = "Hello, World\n";
static size_t file_size = 13;

static int hello_getattr(const char *path, struct stat *stbuf) {
    memset(stbuf, 0, sizeof(struct stat));
    if (strcmp(path, "/") == 0) {
        stbuf->st_mode = S_IFDIR | 0755;
        stbuf->st_nlink = 2;
    } else if (strcmp(path, hello_path) == 0) {
        stbuf->st_mode = S_IFREG | 0666;
        stbuf->st_nlink = 1;
        stbuf->st_size = file_size;
    } else {
        return -ENOENT;
    }
    return 0;
}

static int hello_readdir(const char *path, void *buf, fuse_fill_dir_t filler,
                         off_t offset, struct fuse_file_info *fi) {
    (void) offset;
    (void) fi;
    if (strcmp(path, "/") != 0) return -ENOENT;
    filler(buf, ".", NULL, 0);
    filler(buf, "..", NULL, 0);
    filler(buf, "hello.txt", NULL, 0);
    return 0;
}

static int hello_open(const char *path, struct fuse_file_info *fi) {
    if (strcmp(path, hello_path) != 0) return -ENOENT;
    return 0;
}

static int hello_read(const char *path, char *buf, size_t size, off_t offset,
                      struct fuse_file_info *fi) {
    (void) fi;
    if (strcmp(path, hello_path) != 0) return -ENOENT;
    if (offset < file_size) {
        if (offset + size > file_size) size = file_size - offset;
        memcpy(buf, file_content + offset, size);
    } else {
        size = 0;
    }
    return size;
}

static int hello_write(const char *path, const char *buf, size_t size,
                       off_t offset, struct fuse_file_info *fi) {
    (void) fi;
    if (strcmp(path, hello_path) != 0) return -ENOENT;
    if (offset + size > sizeof(file_content)) return -EFBIG;
    memcpy(file_content + offset, buf, size);
    if (offset + size > file_size) file_size = offset + size;
    return size;
}

static int hello_truncate(const char *path, off_t size) {
    if (strcmp(path, hello_path) != 0) return -ENOENT;
    if (size < file_size) file_size = size;
    else {
        memset(file_content + file_size, 0, size - file_size);
        file_size = size;
    }
    return 0;
}

static struct fuse_operations hello_ops = {
    .getattr = hello_getattr,
    .readdir = hello_readdir,
    .open    = hello_open,
    .read    = hello_read,
    .write   = hello_write,
    .truncate= hello_truncate,
};

int main(int argc, char *argv[]) {
    return fuse_main(argc, argv, &hello_ops, NULL);
}

```
