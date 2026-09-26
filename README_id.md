# 🚀 Panduan Setup Auto-Backup Jaringan (Cisco, MikroTik, Huawei, Juniper) dengan n8n & Telegram

Selamat datang! Workflow ini akan menyulap Telegram Anda menjadi asisten jaringan pribadi. Anda bisa menambahkan router/switch, menghapusnya, dan meminta sistem melakukan backup otomatis atau manual—semuanya hanya lewat chat Telegram!

Sistem ini menggunakan n8n (platform otomatisasi) dan mendukung 4 brand perangkat jaringan terpopuler: MikroTik, Cisco, Huawei, dan Juniper.

Mari kita mulai langkah-langkah setup-nya! Tidak perlu jago coding, cukup ikuti panduan santai di bawah ini.

---

## 📋 Persiapan Awal (Yang Kamu Butuhkan)

Sebelum mulai, pastikan kamu sudah menyiapkan 3 hal ini:
1. **Aplikasi n8n:** Sudah terinstal dan berjalan (bisa di Docker, Cloud, atau Desktop).
2. **Bot Telegram:** Buat bot baru melalui [@BotFather](https://t.me/BotFather) di Telegram dan simpan API Token-nya.
3. **Server FTP:** Sebuah tempat (server/PC) yang membuka layanan FTP untuk menyimpan file backup dari router.

---

## 🛠️ Langkah 1: Buat Database (Data Table)

Sistem butuh tempat untuk mencatat daftar router yang kamu miliki. Kita akan menggunakan fitur database bawaan n8n.

1. Buka n8n kamu. Di menu sebelah kiri, klik **Data tables**.
2. Klik tombol **New data table**. Beri nama bebas (misalnya: `data auto backup`).
3. Kamu bisa mengimpor file contoh (dummy) yang sudah saya sediakan, atau membuat kolomnya secara manual. Jika manual, buat kolom-kolom berikut ini (perhatikan huruf kecilnya):
   * `address` (Tipe: String)
   * `port` (Tipe: Number)
   * `username` (Tipe: String)
   * `password` (Type: String)
   * `brand` (Tipe: String)
   * `portftp` (Tipe: Number)
   * `date_last_backup` (Tipe: Date & Time)

<img width="837" height="73" alt="image" src="https://github.com/user-attachments/assets/12264ad5-b2fd-4a5f-a53a-524a0b8152b5" />

---

## 📥 Langkah 2: Import Workflow ke n8n

Sekarang kita masukkan "otak" otomatisasinya ke dalam n8n.

1. Di menu n8n sebelah kiri, klik **Workflows**, lalu klik **Add workflow**.
2. Di pojok kanan atas, klik tombol opsi (titik tiga `...`), lalu pilih **Import from File**.
3. Pilih file `.json` workflow ini yang sudah kamu download.

Tadaa! Jaring-jaring otomatisasi akan muncul di layarmu.

<img width="626" height="735" alt="image" src="https://github.com/user-attachments/assets/88eb96d0-5a05-4ecf-bac4-5349d282278f" />

---

## 🔗 Langkah 3: Menghubungkan Akun (Credentials) & Tabel

Karena ini adalah workflow baru, kamu harus "memperkenalkan" akun Telegram dan server FTP-mu ke n8n, serta menyambungkan tabel yang dibuat di Langkah 1.

### A. Menyambungkan Tabel
1. Cari semua node yang bernama **Data Table** (ikon berbentuk tabel warna oranye).
2. Klik dua kali (buka) node tersebut satu per satu.
3. Pada bagian **Data table**, klik menu dropdown dan pilih nama tabel yang kamu buat di Langkah 1 (misal: `data auto backup`).
4. Ulangi untuk semua node Data Table agar sistem tahu di mana harus membaca dan menyimpan data.

### B. Memasukkan Token Telegram
1. Cari node bernama **Telegram Trigger** (paling kiri) dan klik dua kali.
2. Di bagian **Credential**, klik tanda panah ke bawah, pilih **Create New Credential**.
3. Masukkan API Token yang kamu dapatkan dari `@BotFather`. Simpan.
4. Ulangi pemilihan nama kredensial Telegram ini di semua node yang berlogo Telegram (warna biru).

### C. Mengisi Kredensial FTP Eksternal
1. Cari node FTP bernama **Upload Backup to Server** (atau **Download Backup from Server**).
2. Buat kredensial FTP baru. Masukkan IP Address, Username, dan Password server FTP tempat kamu ingin menyimpan kumpulan backup akhirnya.

<img width="807" height="375" alt="image" src="https://github.com/user-attachments/assets/50cbb6dd-6cd5-4d6c-acf5-35e5b7763e0c" />

---

## ⚙️ Pengaturan Kredensial SSH dan FTP Dinamis (Dynamic Credentials)

Hebatnya dari workflow ini, Anda tidak perlu membuat kredensial SSH dan FTP secara manual satu per satu untuk setiap perangkat. Sistem sudah dirancang menggunakan fitur Dynamic Credentials di n8n, di mana informasi akses (Host, Port, Username, dan Password) akan ditarik secara otomatis langsung dari baris data yang sedang diproses di database tabel.

Saat Anda mengatur kredensial SSH dan FTP di n8n, ubah kolom input menjadi mode Expression (`fx`) dan masukkan variabel berikut:

* **Untuk Node SSH:**
  * Host: `{{ $json.address }}`
  * Port: `{{ $json.port }}`
  * Username: `{{ $json.username }}`
  * Password: `{{ $json.password }}`

* **Untuk Node FTP:**
  * Host: `{{ $('Get Credential').item.json.address }}`
  * Port: `{{ $('Get Credential').item.json.portftp }}`
  * Username: `{{ $('Get Credential').item.json.username }}`
  * Password: `{{ $('Get Credential').item.json.username }}` *(atau sesuaikan dengan password FTP Anda)*

<img width="423" height="460" alt="image" src="https://github.com/user-attachments/assets/5169e1e0-320b-48a2-9ba0-523cb43095e0" />
<img width="583" height="573" alt="image" src="https://github.com/user-attachments/assets/873f83fe-e6e8-4d7b-8b38-27fe73eac969" />
<img width="539" height="570" alt="image" src="https://github.com/user-attachments/assets/f02bb63b-d578-4158-a2a3-4753864cbc80" />

> ⚠️ *Abaikan Error yang ditunjukkan pada hasil Expression pada langkah ini dikarenakan memang akan seperti itu, namun secara fungsional akan berjalan 100% ketika node aktif.*

---

## 🚀 Langkah 4: Aktifkan Workflow!

Sudah selesai setup? Saatnya menghidupkan mesinnya!

1. Di pojok kanan atas kanvas n8n, klik tombol **Publish** (dari menu dropdown Publish).
2. n8n sekarang sudah bersiaga 24 jam untuk mendengarkan perintah dari Telegram-mu.

<img width="702" height="135" alt="image" src="https://github.com/user-attachments/assets/7b453e9f-1a27-41dd-9ab2-d748ac6d1fcd" />

---

## 📱 Cara Penggunaan di Telegram

Sekarang buka aplikasi Telegram dan buka chat dengan Bot kamu. Ketikkan perintah-perintah sakti ini:

### 1. Menambahkan Device Baru (`/add`)
* Kirim pesan ke bot dengan format berikut (pisahkan dengan spasi):
  `/add [IP_Address] [Port_SSH] [Username] [Password] [Brand] [Port_FTP]`
* **Contoh:**
  `/add 192.168.1.1 22 admin rahasia123 mikrotik 21`
* Bot akan langsung membalas bahwa device berhasil ditambahkan dan menyajikan daftar device yang aktif. *(Catatan: Brand yang didukung hanya mikrotik, cisco, huawei, dan juniper).*

<img width="705" height="246" alt="image" src="https://github.com/user-attachments/assets/c9f7c22f-8a55-4a8b-9ae8-51fa9c09865f" />

### 2. Melihat Menu Utama (`/menu`)
* Ketik `/menu`. Bot akan menampilkan teks sapaan beserta dua tombol interaktif:
  * **Backup Sekarang**: Akan memaksa sistem melakukan backup ke semua device di daftar saat itu juga!
  * **Download Backup Terakhir**: Memintakan file backup terbaru untuk dikirimkan langsung ke Telegram-mu.

<img width="704" height="230" alt="image" src="https://github.com/user-attachments/assets/b1ede717-da52-46be-bd79-66fdf90e6d22" />
 
### 3. Menghapus Device (`/remove`)
* Ingin melihat daftar device beserta nomor ID-nya? Ketik `/remove` saja.
* Ingin menghapus device nomor 2? Ketik `/remove 2`. Bot akan langsung menghapusnya dari database.

<img width="713" height="612" alt="image" src="https://github.com/user-attachments/assets/ca0f6e4d-7bef-4c0e-8705-e3e607229ebc" />

---

## 💡 Catatan Tambahan (Penting!)

1. **Jadwal Otomatis:** Sistem ini otomatis melakukan backup setiap tanggal 1 jam 02:00 pagi. Kamu bisa mengubah jadwal ini dengan mengedit node **Schedule Trigger** (logo jam di bagian atas).
2. **Khusus Cisco & Huawei:** Pastikan username yang kamu gunakan untuk SSH memiliki hak akses level tertinggi (Privilege Level 15 / Super Admin) agar sistem n8n bisa langsung mengeksekusi perintah backup tanpa terhalang password tambahan.
3. **Fitur FTP di Router:** Pastikan fitur FTP Server di dalam setiap router/switch yang kamu daftarkan sudah dalam keadaan aktif (Enable), karena n8n akan mengambil file backup dari perangkat menggunakan jalur FTP.

Selamat mencoba! Jika terjadi error, bot Telegram akan secara otomatis mengirimkan error notification beserta nama langkah yang bermasalah agar mudah diperbaiki.
