## 1. Generate Certificate
- ssh ke node haproxy-1
- jalankan perintah ini untuk generate certificate
```bash
sudo mkdir -p /etc/haproxy/certs

openssl req -x509 \
-newkey rsa:4096 \
-nodes \
-keyout /etc/haproxy/certs/haproxy.key \
-out /etc/haproxy/certs/haproxy.crt \
-days 365
```
- kemudian gabungkan
```bash
sudo sh -c 'cat /etc/haproxy/certs/haproxy.crt /etc/haproxy/certs/haproxy.key > /etc/haproxy/certs/haproxy.pem'
```

## 2. Copy Sertifikat dari HAProxy-1 ke HAProxy-2
- karena kedua node melayani VIP yang sama dan domain yang sama, keduanya sebaiknya menggunakan sertifikat yang sama jadi:
1. generate seritifkat hanya di HAProxy-1
2. salin haproxy.pem ke HAProxy-2

- jalankan perintah ini di node HAProxy-1
```bash
scp /etc/haproxy/certs/haproxy.pem paimon@<IP_HAPROXY_2>:/tmp/
```

- lalu di HAProxy-2
```bash
sudo mkdir -p /etc/haproxy/certs
sudo mv /tmp/haproxy.pem /etc/haproxy/certs/
sudo chown root:root /etc/haproxy/certs/haproxy.pem
sudo chmod 600 /etc/haproxy/certs/haproxy.pem
```

## 3. Ubah Frontend
- pastikan haproxy.pem ada di: /etc/haproxy/certs/haproxy.pem
- ubah haproxy.cfg pada kedua node menjadi:
```bash
frontend docapi_front
    bind :80
    bind :443 ssl crt /etc/haproxy/certs/haproxy.pem
    http-request redirect scheme https unless { ssl_fc } # otomatis redirect http -> https
    default_backend docapi_backend
```
- cek konfigurasi
```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```
- jika valid
```bash
sudo systemctl restart haproxy
```
- pastikan service berjalan
```bash
sudo systemctl status haproxy
```