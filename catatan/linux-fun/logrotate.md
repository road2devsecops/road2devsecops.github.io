# Manajemen Log Menggunakan Logrotate

![](images/log.png)

## Apa itu logrotate?
Menurut manual page dari logrotate `man logrotate`, Logrotate adalah sebuah program yang berfungsi untuk mengotomatissai proses rotasi, kompresi, dan penghapusan log file. Logrotate menghandle (rotate, compress, delete) file log secara satuan waktu (daily, weekly, monthly), atau ukuran besar log. 


## Bagaimana Logrotate Bekerja?
Logrotate bekerja dengan memanfaatkan file konfigurasi yang bisa kita kustomisasi `/etc/logrotate.conf`. Dalam file konfigurasi kita bisa mengatur berapa lama kita menyimpan log tersebut sebelum log tersebut akan otomatis terhapus. Selain itu kita bisa mengatus agar file log yang ada dirotasi berdasarkan satuan waktu (daily, weekly, monthly).

`Contoh :` log file yang dirotate setiap hari selama 7 hari.
```
# ls /var/log
syslog  syslog.1  syslog.2.gz  syslog.3.gz syslog.4.gz syslog.5.gz syslog.6.gz
``` 

Log akan dirotate, kemudian di-compress dan setelah 7 hari maka log akan otomatis dihapus.
## Installasi
Kebanyakan distribusi linux sudah terinstall logrotate secara default. Tapi jika di sistem linux kalian belum terinstall, logrotate bisa diinstall dengan perintah :

**RHEL/CentOS**
```
sudo yum -y install logrotate
```

**Debian/Ubuntu**
```
sudo apt -y install logrotate
```

**Fedora**
```
sudo dnf install logrotate
```

## Konfigurasi
Contoh file konfigurasi logrotate untuk log `httpd access.log & error.log`.

```
compress

"/var/log/httpd/access.log" /var/log/httpd/error.log {
    rotate 5
    mail recipient@example.org
    size 100k
    sharedscripts
    postrotate
        /usr/bin/killall -HUP httpd
    endscript
}
```
**Penjelasan**
```
compress
```
Baris pertama merupakan global options. Di contoh ini, log akan dicompress setelah log tersebut dirotasi.

```
"/var/log/httpd/access.log" /var/log/httpd/error.log
```
Baris berikutnya mendefine parameter untuk log httpd `access.log` dan `error.log`.

```
rotate 5
```
Log akan dirotasi maksimal 5 kali, setelah itu log akan dihapus.
```
mail recipient@example.org
```
Log yang melebihi 5 kali rotasi akan dikirim ke email `recipient@example.org`.
```
size 100k
```
Log akan dirotasi setelah besar file 100k.
```
sharedscripts
postrotate
    /usr/bin/killall -HUP httpd
endscript
```
`sharedscripts` berarti `postrotate` script akan dijalankan sekali setelah log lama di-compress.

Untuk lebih detailnya kalian bisa cek manual page dari logrotate dengan perintah: `man logrotate`