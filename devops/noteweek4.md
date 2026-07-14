**Kubernetes Tasks**

1. Buatlah sebuah kubernetes cluster yang di dalamnya terdapat 3 buah node sebagai master dan worker.

   ![Cluster 3 node master dan worker](<imageweek4/1.cluster 3 node master and worker.png>)

2. Install ingress nginx menggunakan helm atau manifest.

   ![Install ingress nginx menggunakan helm](<imageweek4/2. Install ingress nginx using helm.png>)
   ![Bukti install ingress nginx](<imageweek4/2.1 bukti Install ingress nginx.png>)

3. Deploy aplikasi yang kalian gunakan di Docker Swarm ke dalam kubernetes cluster yang telah dibuat pada point nomor 1.

4. Setup persistent volume untuk database kalian.

   ![Setup persistent volume untuk database](<imageweek4/4. Setup persistent volume untuk database.png>)
   ![Hasil persistent volume untuk database](<imageweek4/4.1 hasil persistent volume untuk database.png>)

5. Deploy database MySQL dengan StatefulSet dan gunakan secrets.

   ![Deploy database MySQL dengan StatefulSet dan secrets](<imageweek4/5. Deploy database mysql with statefullset and use secrets.png>)

6. Install cert-manager ke kubernetes cluster kalian, lalu buat wildcard SSL certificate.

   ![Install cert-manager dan wildcard SSL](<imageweek4/6. Install cert-manager ke kubernetes cluster kalian, lalu buat lah wildcard ssl certificate..png>)
   ![Kubernetes wildcard](<imageweek4/6.1 kubernetes wildcard.png>)

7. Ingress

   - fe : [name.kubernetes.studentdumbways.my.id](http://name.kubernetes.studentdumbways.my.id/)
   - be : [api.name.kubernetes.studentdumbways.my.id](http://api.name.kubernetes.studentdumbways.my.id/)

   ![Ingress](<imageweek4/7. Ingress.png>)
   ![Frontend domain](<imageweek4/7.1 ade.kubernetes.ademulyana.org.png>)
   ![Backend domain](<imageweek4/7.2 api.ade.kubernetes.ademulyana.org.png>)

