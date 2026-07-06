# TUGAS AUTOMATION: ANSIBLE, DOCKER, MONITORING & SSL

## 1. Membuat User Baru (SSH Key + Password)
Buatlah user baru menggunakan Ansible dengan login menggunakan SSH key dan password.

![Screenshot 1](imageweek3/ansiblle_create_user.png)

---

## 2. Instalasi Docker
Instalasi Docker dan Docker Compose Plugin di semua server menggunakan Ansible.

![Screenshot 2](imageweek3/install_docker.png)

---

## 3. Deploy Frontend Application
Deploy aplikasi frontend yang sudah digunakan sebelumnya menggunakan Ansible dengan Docker container.

![Screenshot 3](imageweek3/deploy_frontend.png)

---

## 4. Instalasi Node Exporter
Setup Node Exporter di semua server untuk monitoring metrics (CPU, Memory, Disk, Network).

![Screenshot 4](imageweek3/node_exporter_pake_nginx_domain.png)

---

## 5. Setup Prometheus
Konfigurasi Prometheus untuk scraping metrics dari Node Exporter, Docker container, dan aplikasi frontend.

![Screenshot 5](imageweek3/instalasi_hasil_prometeus.png)

---

## 6. Setup Grafana
Instalasi Grafana dengan datasource Prometheus dan persiapan untuk membuat dashboard monitoring.

![Screenshot 6](imageweek3/instalasi_grafana.png)

---

## 7. Setup Nginx Reverse Proxy
Konfigurasi Nginx sebagai reverse proxy untuk 3 service:
- `prom-yourname.studentdumbways.my.id` → Prometheus (port 9090)
- `monitoring-yourname.studentdumbways.my.id` → Grafana (port 3000)
- `exporter-yourname.studentdumbways.my.id` → Node Exporter (port 9100)

![Screenshot 7](imageweek3/grafana_nginx_domain.png)

---

## 8. Generate SSL Certificate
Generate SSL certificate menggunakan Certbot (Let's Encrypt) untuk semua domain monitoring.

![Screenshot 8](imageweek3/akses_doker_graf_prm_node.png)

---

## 9. PromQL Queries (Rumus Monitoring)
Dokumentasi rumus PromQL yang digunakan untuk monitoring:
- CPU Usage > 20%
- Memory Usage > 75%
- Disk Usage
- Container Status
- Uptime monitoring

![Screenshot 9](imageweek3/prom_pake_nginx_domain.png)

---

## 10. Grafana Dashboard
Membuat dashboard Grafana untuk monitoring resource server:
- CPU Usage gauge
- Memory Usage gauge
- Disk Usage gauge
- Container Status table
- 24-hour trend graph

![Screenshot 10](imageweek3/dashbord_grafana.png)

---

## 11. Monitoring Specific Container
Setup monitoring khusus untuk container aplikasi frontend dengan metrics tracking.
![Screenshot 10](imageweek3/dashbord_grafana.png)
