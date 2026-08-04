# instalasi server-3

## 1. Update Sistem
- sudo apt update
- sudo apt upgrade -y
- sudo reboot

## 2. Install Dependency
```bash
sudo apt install -y \
ca-certificates \
curl \
gnupg \
lsb-release
```

## 3. Tambahkan GPG Key Docker
```bash
sudo install -m 0755 -d /etc/apt/keyrings
```
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

## 4. Tambahkan Repository Docker
```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## 5. Install Docker
```bash
sudo apt update

sudo apt install -y \
docker-ce \
docker-ce-cli \
containerd.io \
docker-buildx-plugin \
docker-compose-plugin
```

## 6. Jalankan Docker Saat Boot
```bash
sudo systemctl enable --now docker
```
- cek status
```bash
sudo systemctl status docker
```

## 7. Tambahkan User ke Group Docker
```bash
sudo usermod -aG docker $USER
```
- lalu reboot

## 8. Test Docker
- docker version
- docker compose version

## 9. Install Git
- sudo apt install git

## 10. Clone Project
```bash
git clone https://github.com/aditya3232/deployment-doc-api-di-vm.git
```

## 11. Build Dockerfile (Jika mau build dulu)
```bash
cd ./deployment-doc-api-di-vm/doc-api-server
docker build -t doc-api-service .
```

## 12. Run
```bash
cd ./deployment-doc-api-di-vm/doc-api-server/deployment
docker compose up -d --build
```
- cek apakah container berjalan
```bash
docker compose ps
```
- lihat logs jika ada yang gagal
```bash
docker compose logs {container_name}
```