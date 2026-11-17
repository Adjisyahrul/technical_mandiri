# 4. Detail Design Screen Behavior


## Berlaku untuk semua layar

- Saat memuat data, app menampilkan **skeleton loading**.
- Kalau koneksi terputus, muncul pesan **"Tidak ada koneksi internet"** dengan tombol **Coba Lagi**.
- Tombol yang mengirim data langsung **disabled setelah ditekan** supaya tidak terkirim dua kali.
- Kalau app tidak dipakai selama **5 menit**, user harus membuka kunci lagi dengan biometrik atau password.

## Splash

- Mengecek versi app. Kalau versi terlalu lama, user diminta update.
- Mengecek apakah device di-root/jailbreak. Kalau iya, app tidak bisa dipakai.
- Kalau sudah ada sesi login, arahkan ke login biometrik/password. Kalau belum, ke halaman registrasi.

## Registrasi Data Diri

- NIK wajib 16 digit angka. App juga mengecek digit ke-7 sampai 12 NIK, yang berisi tanggal lahir (untuk perempuan tanggalnya ditambah 40). Kalau tidak cocok dengan tanggal lahir yang diisi, muncul pesan error. Cara ini menangkap salah ketik sebelum data dikirim.
- Email divalidasi formatnya. No. HP diawali +62.
- Tombol **Lanjut** baru aktif setelah semua field valid.
- Kalau NIK, email, atau no. HP sudah terdaftar, muncul pesan dan pilihan untuk login.

## Verifikasi OTP

- Kode OTP 6 digit. Di Android kode terisi otomatis dari SMS.
- Tombol kirim ulang aktif setelah 60 detik.
- Salah 5 kali, verifikasi diblokir selama 30 menit.

## Foto KTP dan Selfie

- Foto KTP diambil langsung dari kamera dengan bingkai panduan. Upload dari galeri tidak disediakan untuk mengurangi pemakaian foto KTP orang lain.
- Setelah foto, user bisa melihat hasilnya dan memilih **Foto Ulang** atau **Gunakan**.
- Selfie memakai liveness detection dari SDK vendor (misalnya diminta berkedip).
- Foto dikompres sekitar 1 MB sebelum di-upload. Kalau upload gagal, ada tombol **Coba Lagi**.

## Login

- Kalau biometrik sudah diaktifkan, prompt sidik jari/wajah muncul otomatis saat app dibuka.
- Gagal 3 kali atau dibatalkan, user diarahkan ke login password.
- Login password gagal 5 kali, akun dikunci selama 30 menit.
- Ada link **Lupa Password** yang mengirim OTP ke no. HP.
- Setelah login berhasil, app mengirim push token device ke `PUT /users/me/push-token`. Token dikirim ulang setiap kali FCM/APNs memberi token baru.

## Lupa Password

- User memasukkan no. HP terdaftar, lalu OTP 6 digit dikirim. Aturan OTP sama dengan layar Verifikasi OTP.
- Setelah OTP benar, user membuat password baru (minimal 8 karakter, kombinasi huruf dan angka) dan mengulanginya di field konfirmasi.
- Setelah reset berhasil, semua sesi di device lain diakhiri dan biometrik dinonaktifkan. User diarahkan ke Login dan bisa mengaktifkan biometrik lagi dari Profil.
- Supaya tidak bisa dipakai untuk mengecek nomor terdaftar, pesan yang tampil selalu sama: "Jika nomor terdaftar, kode OTP akan dikirim".

## Home

- Menampilkan sisa hutang dan tagihan bulan ini: nominal, tanggal jatuh tempo, dan status (Belum Bayar / Terlambat / Lunas), sesuai status cicilan `UNPAID` / `OVERDUE` / `PAID`.
- Kalau status Terlambat, kartu tagihan berwarna merah dan menampilkan denda yang sudah ditambahkan ke tagihan.
- Kalau belum ada pinjaman, tampil pesan "Belum ada pinjaman" dan tombol **Ajukan Pinjaman**.
- Tombol **Ajukan Pinjaman** tetap terlihat. Kalau user belum boleh mengajukan (KYC belum selesai, masih ada pinjaman berjalan, atau masih dalam masa tunggu setelah ditolak), tombolnya nonaktif dan alasannya ditampilkan di bawah tombol. Status ini diambil dari field `can_apply_loan` dan `reason` di `GET /users/me/summary`.
- Data bisa di-refresh dengan pull to refresh.

## Ajukan Pinjaman

- Nominal dipilih dengan slider dari Rp 500.000 sampai Rp 12.000.000 (kelipatan Rp 500.000), atau diketik manual.
- Tenor dipilih: 1, 3, 6, 9, atau 12 bulan.
- Cicilan per bulan, total biaya, dan total yang harus dibayar langsung berubah setiap nominal atau tenor diganti.

## Konfirmasi Pinjaman

- Menampilkan rincian biaya, jadwal cicilan, dan rekening tujuan pencairan.
- Nama pemilik rekening harus sama dengan nama di KTP.
- User harus mencentang persetujuan syarat dan ketentuan, lalu konfirmasi dengan biometrik atau password sebelum pengajuan dikirim.

## Rekening Bank

- Dibuka dari layar Konfirmasi Pinjaman. Kalau user belum punya rekening, layar ini langsung terbuka untuk menambah rekening.
- User memilih bank dari daftar, lalu mengetik nomor rekening (angka saja).
- Saat **Simpan** ditekan, app menampilkan nama pemilik rekening hasil pengecekan ke bank. Kalau nama tidak sama dengan nama di KTP, muncul pesan error dan rekening tidak disimpan.
- Kalau user punya lebih dari satu rekening, user memilih salah satu sebagai rekening tujuan pencairan. Nomor rekening ditampilkan tersamar (contoh: ******7890).

## Status Pengajuan

- Setelah submit, tampil status "Pengajuan sedang diproses".
- App mengecek status setiap beberapa detik. User juga boleh keluar dari layar ini karena hasilnya akan dikirim lewat notifikasi.
- **Diterima:** tampil nominal yang dicairkan, rekening tujuan, dan tanggal cicilan pertama. Email dan SMS dikirim ke user.
- **Ditolak:** tampil pesan dan kapan user bisa mengajukan lagi, diambil dari `reapply_at`.

## Detail Tagihan dan Pembayaran

- Daftar cicilan per bulan dengan tanggal jatuh tempo, nominal, dan status.
- Tombol **Bayar** membuat nomor Virtual Account. Nomor VA dan nominal bisa disalin dengan sekali tap, dan ada cara bayar per bank.
- Setelah user membayar, status berubah otomatis dan muncul notifikasi pembayaran berhasil.

## Profil

- Menampilkan data diri dengan NIK dan nomor rekening yang disamarkan.
- Toggle untuk mengaktifkan atau mematikan biometrik. Mengaktifkan biometrik memerlukan input password terlebih dahulu. Mematikan biometrik memanggil `DELETE /auth/biometric` dan menghapus key pair dari Keystore/Keychain.
- Logout menghapus token dari device.
