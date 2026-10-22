# Riset PortSwigger — HTTP Request Smuggling (TE.CL)

Write-up artikel tentang eksploitasi kerentanan HTTP Request Smuggling tipe TE.CL untuk melembangi kontrol keamanan front-end, berdasarkan lab PortSwigger Web Security Academy.

## File Utama

- `write-up.md` — Artikel lengkap dalam Bahasa Indonesia, siap disalin dan dipublikasikan.

## Struktur Artikel

1. Judul Utama (dengan kata kunci TE.CL dan Bypass Front-End)
2. Pendahuluan Teoretis (2 paragraf tentang konflik RFC CL vs TE)
3. Metodologi Eksploitasi (langkah demi langkah dengan code block HTTP payload asli dari lab)
4. Dampak Bisnis (cache poisoning, session hijacking, bypass WAF)
5. Mitigasi (rekomendasi teknis untuk pengembang)
6. Kesimpulan

## Cara Publikasi di Medium

1. Masuk ke [https://medium.com/](https://medium.com/).
2. Klik gambar profil → **Write a story**.
3. Salin **seluruh isi** `write-up.md` dan tempelkan ke editor Medium.
4. Beri judul sesuai yang tercantum di atas.
5. Tambahkan tag: `cybersecurity`, `web-security`, `http-request-smuggling`, `portswigger`, `burp-suite`.
6. Klik **Publish**.

## Cara Publikasi di GitHub Pages

1. Aktifkan GitHub Pages pada repositori ini (Settings → Pages → Source: `main` / folder `/docs` atau root).
2. Salin `write-up.md` ke folder `docs/write-up.md` (atau root).
3. Jika ingin tampilan web, buat file `docs/index.md` yang mengandung link ke `write-up.md`, atau gunakan Jekyll theme.
4. Tautan artikel akan tersedia di: `https://betzq.github.io/<nama-repo>/write-up.md`

## URL Lab Referensi

`https://0a13007404bed62a81f2fc9b0062005c.web-security-academy.net/`
