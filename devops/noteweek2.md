# Dokumentasi Jenkins CI/CD untuk Aplikasi Staging & Production

## 1. Tujuan
Dokumen ini menjelaskan langkah-langkah implementasi Jenkins CI/CD untuk aplikasi yang telah dideploy pada tugas sebelumnya, dengan skenario:

- Install Jenkins di atas Docker
- Setup SSH key dari server Jenkins ke server deploy
- Reverse proxy Jenkins dengan domain
- Membuat job staging dan production
- Menjalankan flow CI/CD: pull repository, build/dockerize, test application, deploy aplikasi on top Docker, dan auto trigger

Domain yang digunakan:
- https://jenkins.team1.studentdumbways.my.id

---

## 2. Topologi yang Digunakan

- Server Jenkins: server tempat Jenkins berjalan
- Server Deploy: server tujuan untuk men-deploy aplikasi menggunakan Docker
- SCM: repository Git (misalnya GitHub/GitLab/Bitbucket)
- Reverse Proxy: Nginx

---

## 3. Install Jenkins di atas Docker

### 3.1 Persiapan server
```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin curl git nginx certbot python3-certbot-nginx
sudo systemctl enable docker
sudo systemctl start docker
```

### 3.2 Buat volume Jenkins
```bash
docker volume create jenkins_home
```

### 3.3 Jalankan Jenkins container
```bash
docker run -d \
  --name jenkins \
  --restart always \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

### 3.4 Ambil initial admin password
```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 3.5 Setup awal Jenkins
1. Buka browser ke http://IP_SERVER:8080
2. Masukkan initial admin password
3. Install plugin yang direkomendasikan
4. Buat user admin
5. Simpan konfigurasi

> Bukti instalasi Jenkins dapat dilihat pada [imageweek2/1. Installasi Jenkins on top Docker.png](imageweek2/1.%20Installasi%20Jenkins%20on%20top%20Docker.png)

---

## 4. Setup SSH Key dari Jenkins ke Server Deploy

Tujuan: agar Jenkins dapat login ke server deploy tanpa password melalui SSH.

### 4.1 Generate SSH key di server Jenkins
```bash
ssh-keygen -t ed25519 -C "jenkins@jenkins.team1.studentdumbways.my.id"
```

### 4.2 Copy public key ke server deploy
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub deployuser@DEPLOY_SERVER_IP
```

### 4.3 Verifikasi koneksi SSH
```bash
ssh -i ~/.ssh/id_ed25519 deployuser@DEPLOY_SERVER_IP
```

### 4.4 Tambahkan private key ke Jenkins Credentials
1. Masuk ke Jenkins -> Manage Jenkins -> Credentials -> System -> Global credentials
2. Tambah credential jenis "SSH Username with private key"
3. Isi:
   - Scope: Global
   - Username: deployuser
   - Private Key: isi private key dari server Jenkins (contoh isi file ~/.ssh/id_ed25519)
   - ID: deploy-ssh-key
4. Simpan

> Bukti setup SSH key dapat dilihat pada [imageweek2/2. Setup SSH-KEY di Jenkins.png](imageweek2/2.%20Setup%20SSH-KEY%20di%20Jenkins.png)

---

## 5. Reverse Proxy Jenkins dengan Nginx

### 5.1 Konfigurasi Nginx
Buat file konfigurasi reverse proxy:

```bash
sudo tee /etc/nginx/sites-available/jenkins <<'EOF'
server {
    listen 80;
    server_name jenkins.team1.studentdumbways.my.id;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 90;
    }
}
EOF
```

Aktifkan konfigurasi:
```bash
sudo ln -s /etc/nginx/sites-available/jenkins /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 5.2 Enable HTTPS dengan Let’s Encrypt
```bash
sudo certbot --nginx -d jenkins.team1.studentdumbways.my.id
```

Setelah selesai, Jenkins dapat diakses melalui:
- https://jenkins.team1.studentdumbways.my.id

> Bukti reverse proxy dapat dilihat pada [imageweek2/3. Reverse Proxy Jenkins.png](imageweek2/3.%20Reverse%20Proxy%20Jenkins.png)

---

## 6. Plugin yang Diperlukan di Jenkins

Install plugin berikut:

- Git Plugin
- GitHub Integration Plugin
- Docker Plugin
- Docker Pipeline
- SSH Agent / Publish Over SSH
- Credentials Binding Plugin
- Workspace Cleanup Plugin

Path: Jenkins -> Manage Jenkins -> Plugins -> Available Plugins

---

## 7. Konfigurasi Credential di Jenkins

### 7.1 Credential untuk Git Repository (opsional)
Jika repository private, tambahkan credential GitHub/Bitbucket dengan username dan token.

### 7.2 Credential untuk SSH deploy
Gunakan credential yang sudah dibuat pada langkah 4.4 dengan ID `deploy-ssh-key`.

---

## 8. Struktur Job CI/CD

Dibuat dua job freestyle:

1. `staging-deploy`
2. `production-deploy`

Kedua job wajib mencakup flow berikut:
- Pull dari repository
- Dockerize/Build aplikasi
- Test application
- Deploy aplikasi on top Docker
- Auto trigger setiap ada perubahan di SCM

---

## 9. Konfigurasi Job Staging

### 9.1 Buat Freestyle Project
- Nama job: `staging-deploy`
- Deskripsi: Deploy aplikasi ke environment staging

### 9.2 Source Code Management
- Pilih Git
- Repository URL: URL repository aplikasi yang digunakan sebelumnya
- Branches to build: `staging` atau `main` (sesuaikan branch yang dipakai)
- Credentials: credential Git yang valid

### 9.3 Build Triggers
- Centang `GitHub hook trigger for GITScm polling` (jika repository GitHub)
- Atau centang `Poll SCM` dengan schedule:
```cron
H/5 * * * *
```

### 9.4 Build Environment
- Centang `Delete workspace before build starts`

### 9.5 Build Steps
Tambahkan `Execute shell` dengan script berikut:

```bash
export IMAGE_NAME="staging-app:latest"
export APP_NAME="staging-app"
export PORT="3000"

cd $WORKSPACE

echo "[1] Pull repository"
git pull origin staging

echo "[2] Build Docker image"
docker build -t $IMAGE_NAME .

echo "[3] Test application"
# Contoh test sederhana, sesuaikan dengan stack aplikasi
if [ -f package.json ]; then
  npm install
  npm test
fi

echo "[4] Deploy ke server target"
docker save $IMAGE_NAME | ssh -o StrictHostKeyChecking=no -i ~/.ssh/id_ed25519 deployuser@DEPLOY_SERVER_IP "docker load && docker stop $APP_NAME || true && docker rm $APP_NAME || true && docker run -d --name $APP_NAME -p $PORT:$PORT $IMAGE_NAME"
```

> Bukti job staging dapat dilihat pada [imageweek2/4. Job Staging.png](imageweek2/4.%20Job%20Staging.png)

---

## 10. Konfigurasi Job Production

### 10.1 Buat Freestyle Project
- Nama job: `production-deploy`
- Deskripsi: Deploy aplikasi ke environment production

### 10.2 Source Code Management
- Git
- Repository URL: repository yang sama
- Branches to build: `main`
- Credentials: Git credential yang valid

### 10.3 Build Triggers
- `GitHub hook trigger for GITScm polling` atau `Poll SCM` dengan schedule:
```cron
H/5 * * * *
```

### 10.4 Build Environment
- `Delete workspace before build starts`

### 10.5 Build Steps
Tambah `Execute shell` dengan script berikut:

```bash
export IMAGE_NAME="production-app:latest"
export APP_NAME="production-app"
export PORT="3001"

cd $WORKSPACE

echo "[1] Pull repository"
git pull origin main

echo "[2] Build Docker image"
docker build -t $IMAGE_NAME .

echo "[3] Test application"
if [ -f package.json ]; then
  npm install
  npm test
fi

echo "[4] Deploy ke server target"
docker save $IMAGE_NAME | ssh -o StrictHostKeyChecking=no -i ~/.ssh/id_ed25519 deployuser@DEPLOY_SERVER_IP "docker load && docker stop $APP_NAME || true && docker rm $APP_NAME || true && docker run -d --name $APP_NAME -p $PORT:$PORT $IMAGE_NAME"
```

> Bukti job deploy production dapat dilihat pada [imageweek2/9. Deploy aplikasi on top Docker.png](imageweek2/9.%20Deploy%20aplikasi%20on%20top%20Docker.png)

---

## 11. Integrasi Auto Trigger dari SCM

Agar Jenkins dapat otomatis menjalankan job setiap ada perubahan di repository:

### 11.1 GitHub Webhook
1. Masuk ke repository GitHub
2. Settings -> Webhooks
3. Tambah webhook:
   - Payload URL: https://jenkins.team1.studentdumbways.my.id/github-webhook/
   - Content type: application/json
   - Events: Just the push event
4. Simpan

### 11.2 Jenkins Build Trigger
Pada setiap job:
- Centang `GitHub hook trigger for GITScm polling`

Dengan konfigurasi ini, setiap push ke branch yang sesuai akan otomatis memicu Jenkins.

> Bukti auto trigger dapat dilihat pada [imageweek2/10. Auto Trigger Setiap Ada Perubahan di SCM.png](imageweek2/10.%20Auto%20Trigger%20Setiap%20Ada%20Perubahan%20di%20SCM.png)

---

## 12. Flow CI/CD yang Sudah Diimplementasikan

1. Developer push code ke repository
2. Jenkins mendeteksi perubahan melalui webhook/polling
3. Jenkins melakukan pull repository
4. Jenkins build Docker image aplikasi
5. Jenkins menjalankan test aplikasi
6. Jenkins SSH ke server deploy
7. Jenkins men-transfer image hasil build dan menjalankan container

---

## 14. Hasil yang Diharapkan

- Jenkins berhasil diakses melalui HTTPS di https://jenkins.team1.studentdumbways.my.id
- Terdapat job `staging-deploy` dan `production-deploy`
- Setiap push ke repository memicu build secara otomatis
- Image berhasil dibuat, diuji, dan dideploy ke server deploy

---

## 15. Bukti Pengerjaan

Berikut screenshot yang menjadi bukti pelaksanaan tugas:

### 15.1 Installasi Jenkins on top Docker
![Installasi Jenkins on top Docker](imageweek2/1.%20Installasi%20Jenkins%20on%20top%20Docker.png)

### 15.2 Setup SSH Key di Jenkins
![Setup SSH Key di Jenkins](imageweek2/2.%20Setup%20SSH-KEY%20di%20Jenkins.png)

### 15.3 Reverse Proxy Jenkins
![Reverse Proxy Jenkins](imageweek2/3.%20Reverse%20Proxy%20Jenkins.png)

### 15.4 Job Staging
![Job Staging](imageweek2/4.%20Job%20Staging.png)

### 15.5 Pull Repository
![Pull Repository](imageweek2/5.%20Pull%20Repository.png)

### 15.6 Dockerize / Build Aplikasi
![Dockerize Build aplikasi](imageweek2/6.%20Dockerize%20Build%20aplikasi.png)

### 15.7 Test Application
![Test Application](imageweek2/7.%20Test%20Application.png)

### 15.8 Deploy Aplikasi on top Docker
![Deploy aplikasi on top Docker](imageweek2/9.%20Deploy%20aplikasi%20on%20top%20Docker.png)

### 15.9 Auto Trigger Setiap Ada Perubahan di SCM
![Auto Trigger Setiap Ada Perubahan di SCM](imageweek2/10.%20Auto%20Trigger%20Setiap%20Ada%20Perubahan%20di%20SCM.png)

---

