1. Buat sebuah diagram sebuah jaringan komputer dengan 4 device dengan kondisi :

- IP Class C : 192.168.4.xxx

- CIDR Block : 192.168.4.0/24

![alt text](ip4device.drawio.png)


2. Jelaskan perbedaan antara SH (Shell) dan BASH (Bourne-Again Shell)

SH (Shell) itu shell yang udah lam dan basic banget, fiturnya sedikit tapi ringan. 

BASH (Bourne-Again Shell) itu upgrade dari SH, jadi fiturnya lebih lengkap BASH punya tambahan kayak command history, tab completion, array variables commands yang lebih banyak. dan BASH tetap compatible sama SH, jadi pokonya ini yang lebih baru dan perintah SH masih bisa di pake di bash 

3. Buat dokumentasi/kumpulan command linux yang kalian ketahui! (Command diluar materi akan diberi nilai ++)

pwd buat liat posisi ladi di mana 

![alt text](image-16.png)

ls buat liat direktori 
ls              # list file biasa
ls -l           # list dengan detail
ls -la          # list semua termasuk hidden file
ls -lh          # list dengan ukuran 

![alt text](image-17.png)


cd - Pindah direktori

![alt text](image-18.png)

cd /home        # pindah ke /home
cd ..           # naik 1 level
cd ~            # ke home directory
cd -            # balik ke direktori sebelumnya

mkdir - Buat direktori baru

![alt text](image-19.png)

mkdir folder1
mkdir -p folder1/folder2/folder3   # bikin folder di dalam folder di dalam folder 

ini buat hapus direktori kosong 
rmdir folder1

ini buat bikin file 
touch newfile.txt


ini buat hapus file 
rm file.txt           # hapus file
rm -r folder1         # hapus folder dan isinya
rm -rf folder1        # paksa hapus

ini buat copy 
cp file1.txt file2.txt           # copy file
cp -r folder1 folder2            # copy folder
cp file.txt /home/user/          # copy ke lokasi lain


ini buat di pindahin 
mv file1.txt file2.txt           # rename
mv file.txt /home/user/          # pindah lokasi

cat file.txt
cat file1.txt file2.txt          # gabung beberapa file

buat write file bisa pake nano atau vim 
buat liat isinya bisa pake cat 

head file.txt
head -n 5 file.txt       # 5 baris pertama
tail file.txt

find . -name "file.txt"       # cari file dengan nama
find /home -name "*.txt"      # cari semua file .txt

grep "kata" file.txt           # cari kata dalam file
grep -i "kata" file.txt        # case insensitive
grep -r "kata" /home/          # cari di semua file dalam folder




