# Cara Konfigurasi GluserFS 3Nodes
Berikut adalah langkah-langkah untuk konfigurasi **GlusterFS 3-node** di **Ubuntu 24.04 LTS** dengan IP yang telah Anda tentukan:

### Persiapan Awal
1. **Install GlusterFS di semua node** (Node 1, Node 2, dan Node 3).
   Pastikan semua node terupdate dan install paket yang diperlukan.

   Di **Node 1**, **Node 2**, dan **Node 3**, jalankan:
   ```bash
   sudo apt update
   sudo apt install glusterfs-server -y
   ```

2. **Aktifkan dan mulai layanan GlusterFS** di setiap node:
   ```bash
   sudo systemctl enable glusterd
   sudo systemctl start glusterd
   ```

3. **Verifikasi bahwa GlusterFS berjalan** di semua node:
   ```bash
   sudo systemctl status glusterd
   ```

4. **Edit file `/etc/hosts`** untuk memastikan bahwa setiap node dapat berkomunikasi dengan baik.
   Tambahkan IP dan hostname masing-masing node di **Node 1**, **Node 2**, dan **Node 3**.

   ```bash
   sudo nano /etc/hosts
   ```

   Tambahkan baris berikut:
   ```
   157.66.56.53   node1
   157.66.56.54   node2
   157.66.56.31   node3
   ```

5. **Nonaktifkan firewall atau pastikan port GlusterFS (24007 dan 24008) dibuka**.
   Jika menggunakan UFW (Uncomplicated Firewall), jalankan:
   ```bash
   sudo ufw allow 24007,24008/tcp
   sudo ufw allow from 157.66.56.0/24
   ```

### Konfigurasi Cluster GlusterFS
6. **Di Node 1**, tambahkan Node 2 dan Node 3 ke dalam cluster GlusterFS:
   ```bash
   sudo gluster peer probe node2
   sudo gluster peer probe node3
   ```

7. **Cek status peer** untuk memastikan semua node terhubung:
   ```bash
   sudo gluster peer status
   ```

   Anda harus melihat Node 2 dan Node 3 terhubung dengan Node 1.

### Membuat dan Mengkonfigurasi Volume
8. **Siapkan direktori penyimpanan** di masing-masing node.
   Pada **Node 1**, **Node 2**, dan **Node 3**, buat direktori untuk brick:
   ```bash
   sudo mkdir -p /data/glusterfs/brick1
   ```

9. **Buat volume GlusterFS** di **Node 1** dengan replikasi 3:
   ```bash
   sudo gluster volume create gv0 replica 3 node1:/data/glusterfs/brick1 node2:/data/glusterfs/brick1 node3:/data/glusterfs/brick1
   ```

10. **Start volume**:
    ```bash
    sudo gluster volume start gv0
    ```

11. **Cek status volume**:
    ```bash
    sudo gluster volume status
    ```

### Mount Volume GlusterFS
12. **Mount volume** di salah satu node, atau di client yang ingin mengakses storage.
    
    Install paket untuk mount GlusterFS:
    ```bash
    sudo apt install glusterfs-client -y
    ```

    Buat direktori untuk mount:
    ```bash
    sudo mkdir /mnt/glusterfs
    ```

    Mount volume:
    ```bash
    sudo mount -t glusterfs node1:/gv0 /mnt/glusterfs
    ```

13. **Verifikasi mount** dengan menggunakan perintah `df -h`:
    ```bash
    df -h
    ```

14. Untuk **otomatis mount saat boot**, tambahkan baris berikut di `/etc/fstab`:
    ```bash
    node1:/gv0 /mnt/glusterfs glusterfs defaults,_netdev 0 0
    ```

### Langkah Tambahan (Opsional)
- Untuk memastikan semua perubahan disinkronkan dan diterapkan, Anda bisa melakukan **self-heal** di GlusterFS:
  ```bash
  sudo gluster volume heal gv0 full
  ```

Dengan langkah-langkah di atas, GlusterFS 3-node harus sudah terkonfigurasi dan siap digunakan sebagai sistem penyimpanan terdistribusi dengan replikasi di Ubuntu 24.04 LTS.
