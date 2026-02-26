# 📦 Install Docker on Ubuntu 24.04 LTS

## 🖥 Server Specification

* **CPU:** 2 vCPU
* **RAM:** 4GB
* **Storage:** 60GB SSD
* **OS:** Ubuntu Server 24.04 LTS 64bit
* **Use Case:** Odoo 18 SaaS (Docker Deployment)

---

# 🎯 Objective

Install Docker using the **official Docker repository** to ensure stability and production readiness for Odoo 18 deployment.

---

# ✅ Step 1 — Update System

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

Login kembali setelah reboot selesai.

---

# ✅ Step 2 — Remove Old Docker Versions (If Any)

```bash
sudo apt remove docker docker-engine docker.io containerd runc -y
```

---

# ✅ Step 3 — Install Required Dependencies

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg -y
```

---

# ✅ Step 4 — Add Docker Official GPG Key

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo tee /etc/apt/keyrings/docker.asc > /dev/null

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

---

# ✅ Step 5 — Add Docker Repository

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

# ✅ Step 6 — Install Docker Engine & Compose Plugin

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

---

# ✅ Step 7 — Enable & Start Docker Service

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

# ✅ Step 8 — Verify Installation

Check Docker version:

```bash
docker --version
```

Check Docker Compose version:

```bash
docker compose version
```

Check service status:

```bash
sudo systemctl status docker
```

Expected output:

```
Active: active (running)
```

---

# ✅ Step 9 — Allow Non-Root Docker Usage (Optional)

Agar tidak perlu menggunakan `sudo` setiap menjalankan docker:

```bash
sudo usermod -aG docker $USER
```

Logout dan login kembali agar group aktif.

---

# ✅ Step 10 — Test Docker

```bash
docker run hello-world
```

Jika muncul pesan sukses → Docker siap digunakan.

---

# 🔒 Optional (Recommended for Production)

Limit Docker log size agar tidak memenuhi storage:

Edit daemon config:

```bash
sudo nano /etc/docker/daemon.json
```

Isi dengan:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

---

# 🚀 Final Status Checklist

Server sekarang sudah:

* ✅ Docker Official Installed
* ✅ Docker Compose Plugin Installed
* ✅ Service Auto-Start Enabled
* ✅ Log Rotation Configured
* ✅ Production-Ready Base for Odoo 18

---

Kalau kamu mau, saya bisa langsung lanjutkan buatkan:

* `INSTALL_ODOO_18_DOCKER.md`
* `DOCKER_COMPOSE_PRODUCTION_ODOO.md`
* `SETUP_NGINX_SSL.md`
* `AUTO_BACKUP_CRON.md`

Kita bikin dokumentasi SaaS kamu rapi dari awal 👌
