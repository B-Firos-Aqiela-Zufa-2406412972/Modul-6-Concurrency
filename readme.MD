## Commit 1 Reflection notes

Di dalam fungsi `handle_connection`, kita memproses koneksi TCP yang masuk dari klien (seperti web browser). Berikut adalah pemahaman saya mengenai apa yang terjadi di dalamnya:

1. **Membaca Stream:** Fungsi ini menerima `mut stream: TcpStream`. Ini adalah aliran data (stream) antara server dan klien.
2. **Menggunakan BufReader:** Kita membungkus `stream` ke dalam `BufReader`. Ini sangat membantu karena `BufReader` mengelola *buffer* internal, sehingga kita bisa membaca data baris demi baris secara efisien (menggunakan metode `lines()`), daripada harus mengurus pembacaan *byte* per *byte* secara manual.
3. **Memparsing HTTP Request:** Kita menggunakan *iterator* dari `BufReader` untuk mengumpulkan baris-baris teks yang dikirim oleh browser sampai kita menemukan baris kosong. Baris kosong ini adalah penanda bahwa bagian *header* dari HTTP Request sudah selesai.
4. **Melihat Output:** Hasilnya, kita bisa mengumpulkan baris-baris tersebut ke dalam sebuah `Vec<_>` dan mencetaknya. Dari *output* terminal, saya bisa melihat wujud asli dari HTTP Request yang dikirimkan oleh browser saya (seperti metode `GET`, `Host`, `User-Agent`, dll).

## Commit 2 Reflection notes

Pada tahap ini, kita memodifikasi `handle_connection` agar server dapat mengembalikan respons (balasan) berupa file HTML. Berikut adalah beberapa hal baru yang saya pelajari:

1. **Membaca File HTML:** Kita menggunakan `fs::read_to_string("hello.html")` untuk membaca seluruh isi file HTML menjadi tipe data `String`.
2. **Format HTTP Response:** Agar browser mengerti balasan kita, kita tidak bisa hanya mengirim teks HTML-nya saja. Kita harus menggunakan format standar HTTP Response, yaitu:
   - **Status Line:** `HTTP/1.1 200 OK` yang menandakan *request* berhasil.
   - **Headers:** Dalam hal ini kita menambahkan `Content-Length` yang diisi dengan panjang (ukuran byte) dari konten HTML kita (`contents.len()`). Header ini penting agar browser tahu seberapa besar data yang akan ia terima sebelum koneksi ditutup.
   - **Blank Line (CRLF):** `\r\n\r\n` memisahkan antara bagian *headers* dan isi konten (*body*).
   - **Body:** Isi dari file HTML itu sendiri.
3. **Mengirim ke Stream:** Kita menggabungkan semua bagian di atas menggunakan makro `format!`, mengubahnya menjadi *byte* (`as_bytes()`), lalu mengirimkannya kembali ke *stream* menggunakan `stream.write_all()`.

Berikut adalah screenshot hasil *run* di komputer saya:
![Commit 2 screen capture](assets/images/commit2.png)

## Commit 3 Reflection notes

Pada tahap ini, saya memodifikasi server agar bisa memvalidasi *request* dan memberikan respons yang berbeda (misalnya halaman 404 jika rute tidak ditemukan). 

**Bagaimana cara memisahkan respons (splitting response)?**
Saya memisahkannya dengan memeriksa baris pertama dari HTTP Request (`request_line`). Dengan menggunakan blok `if/else` di Rust, saya mengecek apakah request tersebut persis sama dengan `"GET / HTTP/1.1"`. Jika benar, variabel penampung akan diisi dengan status `200 OK` dan file `hello.html`. Jika tidak cocok (URL selain `/`), maka statusnya menjadi `404 NOT FOUND` dan file yang dipanggil adalah `404.html`.

**Kenapa refactoring dibutuhkan?**
Sebelum di-*refactor*, blok kode `if` dan `else` sama-sama berisi perintah untuk membaca file, menghitung panjang konten, memformat HTTP *response*, dan mengirimkannya ke *stream*. Hal ini menyebabkan duplikasi kode yang melanggar prinsip *Don't Repeat Yourself* (DRY). 
Dengan refactoring, blok `if/else` hanya bertugas menentukan *nilai* dari `status_line` dan `filename`. Setelah itu, proses membaca file dan mengirim respons cukup ditulis **satu kali** saja di bawahnya. Ini membuat kode jauh lebih bersih, mudah dibaca, dan mudah dimodifikasi di kemudian hari.

Berikut adalah screenshot hasil *run* di komputer saya:
![Commit 3 screen capture](assets/images/commit3.png)