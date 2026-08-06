# RateLimiting
Rate limit berdasarkan IP Address menggunakan Stick Table

## 1. Target
Misalnya kita ingin aturan seperti ini:
- maksimal 20 request dalam 10 detik untuk setiap IP
- jika melebihi limit, HAProxy mengembalikan HTTP 429 Too Many Requests
- berlaku untuk semua endpoint
- flownya:
```bash
Client
   │
   ▼
HAProxy
   │
   ├── Hitung request dari IP
   │
   ├── <= 20 / 10 detik ?
   │         │
   │         ├── Ya → Backend
   │         └── Tidak → 429
   │
   ▼
Backend
``` 

## 2. Konfigurasi
Di frontend docapi_front, tambahkan konfigurasi berikut:

```bash
frontend docapi_front
    bind :80
    bind :443 ssl crt /etc/haproxy/certs/haproxy.pem

    # Redirect HTTP ke HTTPS
    http-request redirect scheme https unless { ssl_fc }

    # Stick Table
    stick-table type ip size 100k expire 30s store http_req_rate(10s)

    # Simpan IP client ke stick table
    http-request track-sc0 src

    # Tolak jika request > 20 dalam 10 detik
    http-request deny deny_status 429 if { sc_http_req_rate(0) gt 20 }

    default_backend docapi_backend
```

Stick table artinya:
- type ip → key yang disimpan adalah alamat IP client.
- size 100k → maksimal 100.000 IP disimpan.
- expire 30s → jika IP tidak aktif selama 30 detik, entry dihapus.
- http_req_rate(10s) → simpan rata-rata request selama 10 detik terakhir.

## 3. Tes
- restart haproxy
```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl restart haproxy
```
- lalu gunakan curl di host
```bash
for i in {1..30}; do
    curl -k https://192.168.122.10/health
done
```
- output yang diharapkan
```bash
200
200
200
...
200
429
429
429
```