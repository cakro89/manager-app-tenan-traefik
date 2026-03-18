🚀 VNET-Management Infrastructure Documentation (ARM64/Armbian)
Dokumentasi resmi infrastruktur VNET MEDIA. Sistem ini mengotomatisasi deployment tenant aplikasi (Mikhmon/PHP) menggunakan Docker, Traefik, dan Cloudflare Tunnel.

---

## 
🏗️ Arsitektur Sistem (3-Layer)
Cloudflare Tunnel: Menghubungkan VPS ke internet tanpa IP Publik/Buka Port.

Traefik Proxy: Mengatur perutean otomatis subdomain ke masing-masing kontainer tenant.

Manager App: Dashboard PHP Native untuk kontrol deploy, monitor CPU/RAM, dan manajemen user.
3. **Manager App**
   Dashboard PHP Native untuk:

   * Deploy tenant
   * Monitoring CPU/RAM
   * Manajemen user

---

## 📂 Struktur Folder

# 🚀 VNET-Management Infrastructure Documentation

Dokumentasi resmi infrastruktur **VNET MEDIA**. Sistem ini mengotomatisasi deployment tenant aplikasi (Mikhmon/PHP) menggunakan Docker, Traefik, dan Cloudflare Tunnel.

---

## 🏗️ Arsitektur Sistem (3-Layer)
Sistem ini berjalan dengan skema modular:
1. **Cloudflare Tunnel**: Menghubungkan VPS ke internet tanpa IP Publik/Buka Port.
2. **Traefik Proxy**: Mengatur perutean otomatis subdomain ke masing-masing kontainer tenant.
3. **Manager App**: Dashboard PHP Native untuk kontrol deploy, monitor CPU/RAM, dan manajemen user.

---

## 📂 Struktur Folder Wajib
Pastikan lokasi file di VPS sesuai dengan jalur berikut agar skrip tidak error:

```plaintext
/home/ubuntu/app-manager/manager/
├── docker-compose.yml       # Konfigurasi orkestrasi kontainer
├── Dockerfile-manager       # Docker Image khusus Dashboard (ARM64)
├── kontrol/                 # Source code Dashboard VNET
├── vnet-404/                # Source code EROR 404 DI SEMUA TENAN KOSONG
├── user-app/                # Tempat penyimpanan data tiap Tenant
├── mikhmon-pppoe/           # Template Master 1 (PPPOE)
└── mikhmon-modem/           # Template Master 2 (MODEM)
```


## 🛠️ Instalasi VPS Baru (Armbian)
### 1. Update & Install Docker

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose -y
```
CEK DOCKER 
```bash
docker --version
docker-compose --version
```
sampai disini docker harusnya jalan Running 
---

### Tambahan Kalau mau tambah GUI install Portainer Klau Suka CLI skip bagian ini  lanjut 2. Setup Network
Buat Volume untuk penyimpanan database Portainer
```bash
sudo docker volume create portainer_data
```
### 2. Jalankan Container Portainer
> Paste di terminal pkai sudo kalau user 

```bash
docker run -d -p 8000:8000 -p 9000:9000 -p 9443:9443 \
  --name portainer \
  --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```
> Port 9000 untuk HTTP (Lokal/Web)
> Port 9443 untuk HTTPS (Aman)
>✅ Portainer berhasil diinstal!"
>🌐 Akses via browser: http://IP-VPS-KAMU:9000 atau https://IP-VPS-KAMU:9443"
---

### 2. Setup Network

```bash
docker network create vnet-network
```

---

### 3. Setup Awal (Opsional) buat folder

```bash

sudo mkdir -p /home/ubuntu/app-manager/manager/user-app
sudo mkdir -p /home/ubuntu/app-manager/manager/kontrol
sudo mkdir -p /home/ubuntu/app-manager/manager/vnet-404
sudo mkdir -p /home/ubuntu/app-manager/manager/app1
sudo mkdir -p /home/ubuntu/app-manager/manager/app2
sudo mkdir -p /home/ubuntu/app-manager/manager/app3

sudo docker network create vnet-network || true

sudo chmod 666 /var/run/docker.sock
sudo chown -R 33:33 /home/ubuntu/app-manager/manager/user-app
```


---

## ☁️ 4. Konfigurasi Cloudflare Dashboard (Zero Trust)

> **PENTING:** Pengaturan ini wajib dilakukan agar semua tenant yang kamu buat otomatis aktif tanpa perlu setting manual di Cloudflare satu per satu.

### Langkah-langkah:
1. Masuk ke **Cloudflare Zero Trust** > **Networks** > **Tunnels**.
2. Buat Nama Tunnel bebas create CLOUDFLARE TUNNEL SAVE
4. habis itu copy token nya
kira kira sperti ini nanti
```yaml
cloudflared.exe service install eyJhIjoiYTY2NmM2MmExMTk4OWZmYTU5M2E1MTU4MDljM2YxNzAiLCJ0IjoiNTE0NmJiMmMtNWU1MC00MzViLTk1NmMtMjhkMTAxYWIyNmFjicyI6Ik0yUXpNamc1WTnpWakxUaGpOakV0TWpkaFpEVXpZbUprT0RaaSJ9
```
Ambil bagian ini saja
```yaml
eyJhIjoiYTY2NmM2MmExMTk4OWZmYTU5M2E1MTU4MDljM2YxNzAiLCJ0IjoiNTE0NmJiMmMtNWU1MC00MzViLTk1NmMtMjhkMTAxYWIyNmFjicyI6Ik0yUXpNamc1WTnpWakxUaGpOakV0TWpkaFpEVXpZbUprT0RaaSJ9
```
5. Pilih Tunnel yang aktif > Klik tombol **Configure**.
6. Pilih Tab **Public Hostname**, lalu tambahkan dua aturan berikut:
7. ganti domain.id sesui domain  aktiv di tunnels

| Subdomain | Domain | Type | URL (Service) |
| :--- | :--- | :--- | :--- |
| `*` | `domain.id` | HTTP | `traefik:80` |



### 💡 Penjelasan Teknis:
* **Tanda Bintang (`*`)**: Berfungsi sebagai *Wildcard*. Artinya, apapun nama tenant yang kamu buat di dashboard (misal: `budi`, `vnet`, `candra`), semuanya akan otomatis diarahkan ke container **Traefik**.
* **Service traefik:80**: Mengarahkan trafik dari Cloudflare langsung ke pintu masuk Proxy Docker kamu.
---
### 🌐 4. Konfigurasi DNS & Cloudflare Tunnel
A. Pengaturan DNS (Dashboard DNS)
Tambahkan record CNAME di menu DNS Cloudflare:

|Type | Name | Content |Proxy Status |
| :--- | :--- | :--- | :--- |
| CNAME |* | ID-TUNNEL-KAMU.cfargotunnel.com | 🟠 Proxy (Orange) | 

Note: Ganti ID-TUNNEL-KAMU dengan ID unik tunnel Anda. MISAL ( 166af089-73ea-412a-b185-46b90d0f tambah cfargotunnel.com
> jadi sperti ini kira kira 166af089-73ea-412a-b185-46b90d0f.cfargotunnel.com

### contoh di bawah ini 
|Type | Name | Content |Proxy Status |
| :--- | :--- | :--- | :--- |
| CNAME | * | 166af089-73ea-412a-b185-46b90d0f.cfargotunnel.com | 🟠 Proxy (Orange) | 

---

## ⚙️ Masuk ke dir 
ketik 
```yaml
sudo nano  /home/ubuntu/app-manager/manager/docker-compose.yml
```
masukan compose di bawah ini ganti dengan domain sesuiakan 


```yaml
version: '3.3'

services:
  # --- CLOUDFLARE TUNNEL (Konektor ke internet) ---
  cloudflare-tunnel:
    image: cloudflare/cloudflared:latest
    container_name: vnet-tunnel
    restart: always
    command: tunnel --no-autoupdate run --protocol http2 --token GANTI DENGAN TOKEN 
    networks:
      - vnet-network

  # --- TRAEFIK (Reverse Proxy / Auto-subdomain) ---
  traefik:
    image: traefik:v2.10
    container_name: vnet-proxy
    restart: always
    command:
      - "--api.insecure=false"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--providers.docker.network=vnet-network" # <--- Beri tahu Traefik network defaultnya
      - "--entrypoints.web.address=:80"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    networks:
      - vnet-network

  # --- MANAGER APP (Dashboard Utama) ---
  manager-app:
    build:
      context: .
      dockerfile: Dockerfile-manager
    container_name: vnet-manager
    user: root
    restart: always
    volumes:
      - ./kontrol:/var/www/html
      - /home/ubuntu/app-manager/manager:/manager-root
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - vnet-network
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.vnet-manager.rule=Host(`manager.domain.id`)"  # <--- GANTI DENGAN DOMAIN MASING MASING) ---
      - "traefik.http.routers.vnet-manager.entrypoints=web"
      - "traefik.http.services.vnet-manager.loadbalancer.server.port=80"

# --- CUSTOM ERROR PAGE (PAKAI APACHE) ---
  vnet-error:
    image: php:8.1-apache-bullseye
    container_name: vnet-404
    restart: always
    volumes:
      - ./vnet-404:/var/www/html
    networks:
      - vnet-network
    labels:
      - "traefik.enable=true"
      # Gunakan nama router vnet-404 secara konsisten
      - "traefik.http.routers.vnet-404.rule=HostRegexp(`{host:.+}`)"
      - "traefik.http.routers.vnet-404.priority=1"
      - "traefik.http.services.vnet-404.loadbalancer.server.port=80"


networks:
  vnet-network:
    driver: bridge



```

janagn lupa masukan token di atas tadi
```yaml
 eyJhIjoiYTY2NmM2MmExMTk4OWZmYTU5M2E1MTU4MDljM2YxNzAiLCJ0IjoiNTE0NmJiMmMtNWU1MC00MzViLTk1NmMtMjhkMTAxYWIyNmFjicyI6Ik0yUXpNamc1WTnpWakxUaGpOakV0TWpkaFpEVXpZbUprT0RaaSJ9
```
ubah jadi sperti ini paste di bawah compose
```yaml
tunnel run --token eyJhIjoiYTY2NmM2MmExMTk4OWZmYTU5M2E1MTU4MDljM2YxNzAiLCJ0IjoiNTE0NmJiMmMtNWU1MC00MzViLTk1NmMtMjhkMTAxYWIyNmFjicyI6Ik0yUXpNamc1WTnpWakxUaGpOakV0TWpkaFpEVXpZbUprT0RaaSJ9
```
> kalau sudah ketik CTRL + O , ENTER, CTRL + X
---
## 2. Cek Angka Sakti (GID Docker)
###  PENTING: Di Armbian, ID grup Docker seringkali berbeda. Jalankan ini dan catat angkanya:
> getent group docker | cut -d: -f3
> # Contoh hasil: 998

## 🐳  Tambah Dockerfile-manager ubuntu/debian
### sudo nano  /home/ubuntu/app-manager/manager/Dockerfile-manager
masukan di bawah ini
kalau sudah ketik CTRL + O , ENTER, CTRL + X


```dockerfile
# Pakai Bullseye (Debian 11) agar stabil di ARM64
FROM php:8.1-apache-bullseye

# Step 1: Install Docker CLI
RUN apt-get update && apt-get install -y docker.io && apt-get clean

# Step 2: Sinkronisasi GID Docker (Sesuaikan angka 998 dengan hasil cek tadi)
RUN groupmod -g 998 docker || groupadd -g 998 docker

# Step 3: Setup User & Extension
RUN usermod -aG docker www-data
RUN docker-php-ext-install mysqli pdo pdo_mysql && a2enmod rewrite

WORKDIR /var/www/html

```

---
# Bersihkan cache lama (jika ada error build)
```bash
sudo docker builder prune -f
```
# Jalankan sistem
```bash
sudo docker-compose up -d --build
```


## 🔐 Izin Docker Socket

```bash
sudo chmod 666 /var/run/docker.sock
```

---

## ⚠️ Troubleshooting

### Deployment gagal

```bash
sudo chmod 666 /var/run/docker.sock
```

---

### Subdomain loading terus

```bash
docker network inspect vnet-network
```

```bash
docker network connect vnet-network vnet-proxy
docker network connect vnet-network vnet-tunnel
```

---

### Cek penggunaan resource

```bash
docker stats
```

---

### Cek log Traefik

```bash
docker logs vnet-proxy
```
---
## © 2026 VNET MEDIA

Managed by Cakro



