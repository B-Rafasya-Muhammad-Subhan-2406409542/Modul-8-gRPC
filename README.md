# Reflection - Module 08 (gRPC in Rust)

**1. What are the key differences between unary, server streaming, and bi-directional streaming RPC methods, and in what scenarios would each be most suitable?**
* **Unary:** Klien mengirim satu request dan menunggu satu response dari server. Cocok untuk mengambil satu data spesifik dari database atau melakukan autentikasi.
* **Server Streaming:** Klien mengirim satu request, lalu server membalas dengan stream data yang banyak/beruntun. Cocok untuk mengirim file besar secara bertahap atau memberikan update real-time seperti harga saham.
* **Bi-directional Streaming:** Klien dan server dapat saling mengirim banyak pesan secara terus-menerus dan interaktif di waktu yang bersamaan. Cocok untuk aplikasi chat atau analitik data real-time.

**2. What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?**
Untuk mengamankan gRPC, kita perlu menggunakan TLS/SSL untuk mengenkripsi transmisi data agar tidak bisa disadap di tengah jalan. Untuk autentikasi dan otorisasi, kita bisa menggunakan mekanisme token-based (seperti JWT) yang disisipkan melalui gRPC metadata (mirip seperti headers di HTTP biasa) menggunakan fitur interceptor di Tonic.

**3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?**
Tantangan utamanya adalah mengelola concurrency atau proses yang berjalan bersamaan. Di tutorial, kita harus menggunakan `tokio::spawn` untuk menjalankan background task dan menggunakan `mpsc::channel` untuk berkomunikasi antar proses. Menentukan ukuran buffer channel yang tepat (seperti batas 10 atau 32) juga penting agar program tidak macet jika pesan masuk terlalu cepat.

**4. What are the advantages and disadvantages of using `tokio_stream::wrappers::ReceiverStream` for streaming responses in Rust gRPC services?**
* **Kelebihan:** Sangat praktis untuk mengonversi penerima dari Tokio (channel receiver atau `rx`) menjadi format stream yang kompatibel dan bisa langsung direturn oleh gRPC/Tonic. Ini mempermudah pengiriman respons asinkron dari dalam loop.
* **Kekurangan:** Membutuhkan pemahaman ekstra mengenai ekosistem asynchronous Tokio, terutama bagaimana channel dan task berinteraksi, yang bisa jadi membingungkan bagi pemula di Rust.

**5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?**
Struktur dasar sudah baik dengan memisahkan definisi kontrak di folder `proto/` dan kode di folder `src/`. Untuk lebih modular, kita bisa memisahkan setiap service (`PaymentService`, `TransactionService`, `ChatService`) ke dalam file `.rs` masing-masing (contoh: `src/services/payment.rs`), daripada menumpuk semuanya di dalam satu file `grpc_server.rs` atau `grpc_client.rs`.

**6. In the `MyPaymentService` implementation, what additional steps might be necessary to handle more complex payment processing logic?**
Pada implementasi saat ini, fungsi hanya langsung me-return `success: true` sebagai simulasi. Untuk sistem nyata, kita perlu menambahkan beberapa hal, yaitu validasi input (apakah `amount` lebih dari 0), memanggil API payment gateway pihak ketiga, menyimpan status transaksi ke dalam database, serta memberikan penanganan error (mengembalikan status gRPC gagal jika saldo tidak cukup).

**7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?**
gRPC sangat menguntungkan untuk arsitektur microservices karena mendukung pembuatan kode otomatis di banyak bahasa pemrograman, sehingga servis di Rust bisa mudah berkomunikasi dengan servis di Java atau Python. Namun, gRPC kurang optimal jika diakses langsung dari browser web (frontend) karena limitasi dukungannya, sehingga biasanya butuh gateway tambahan jika dibanding REST.

**8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?**
* **Kelebihan:** HTTP/2 menggunakan format biner dan mendukung multiplexing, artinya banyak request/response bisa berjalan bersamaan dalam satu koneksi TCP tanpa harus menunggu satu sama lain.
* **Kekurangan:** HTTP/2 tidak se-universal HTTP/1.1 yang plain-text. Implementasi debugging HTTP/2 lebih sulit karena format biner tidak bisa dibaca langsung oleh manusia seperti format HTTP/1.1 biasa.

**9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?**
REST API harus melalui siklus buka-tutup koneksi (satu request untuk satu response). Jika ingin real-time, REST harus melakukan polling (menarik data terus-menerus) yang boros resource. Sebaliknya, bidirectional streaming pada gRPC membuka satu koneksi konstan di mana kedua pihak bisa saling lempar data kapan saja tanpa jeda (overhead), membuatnya jauh lebih responsif.

**10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?**
Dengan Protocol Buffers, setiap pesan otomatis divalidasi dengan sangat ketat sesuai kontrak API (`.proto`) dan diubah ke format biner yang ringan. Ini membuat program lebih aman dari error salah tipe data. Sebaliknya, format JSON pada REST memang lebih fleksibel dan mudah dibaca, tetapi ukurannya lebih besar dan mewajibkan kita membuat validasi data manual pada setiap endpoint.