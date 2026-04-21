## Commit 1 Reflection notes

Di dalam fungsi `handle_connection`, kita memproses koneksi TCP yang masuk dari klien (seperti web browser). Berikut adalah pemahaman saya mengenai apa yang terjadi di dalamnya:

1. **Membaca Stream:** Fungsi ini menerima `mut stream: TcpStream`. Ini adalah aliran data (stream) antara server dan klien.
2. **Menggunakan BufReader:** Kita membungkus `stream` ke dalam `BufReader`. Ini sangat membantu karena `BufReader` mengelola *buffer* internal, sehingga kita bisa membaca data baris demi baris secara efisien (menggunakan metode `lines()`), daripada harus mengurus pembacaan *byte* per *byte* secara manual.
3. **Memparsing HTTP Request:** Kita menggunakan *iterator* dari `BufReader` untuk mengumpulkan baris-baris teks yang dikirim oleh browser sampai kita menemukan baris kosong. Baris kosong ini adalah penanda bahwa bagian *header* dari HTTP Request sudah selesai.
4. **Melihat Output:** Hasilnya, kita bisa mengumpulkan baris-baris tersebut ke dalam sebuah `Vec<_>` dan mencetaknya. Dari *output* terminal, saya bisa melihat wujud asli dari HTTP Request yang dikirimkan oleh browser saya (seperti metode `GET`, `Host`, `User-Agent`, dll).