# Tutorial Module 8 Advanced Programming

Nama: Rizqya Az Zahra Putri

NPM: 2306244936

Kelas: B

## Reflection

1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?
   - **Unary RPC** adalah model RPC yang paling sederhana, di mana client mengirim satu permintaan dan server memberikan satu respons. Cocok digunakan ketika client hanya membutuhkan respons tunggal, seperti permintaan data pengguna berdasarkan ID.
   - **Server Streaming RPC** berbeda dengan unary karena client mengirim satu permintaan dan server mengirimkan banyak respons secara bertahap dalam bentuk stream. Dapat digunakan saat client ingin menerima aliran data seperti daftar transaksi atau log _real-time_ dari server.
   - **Bi-directional Streaming RPC** memungkinkan client dan server saling mengirim banyak pesan secara independen dalam satu koneksi. Model RPC ini memungkinkan komunikasi dua arah secara bersamaan. Digunakan pada aplikasi chat, sistem monitoring _real-time_, atau _game multiplayer_.

2. What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?
   - Authentication: Pastikan hanya client yang sah yang dapat mengakses layanan. Misalnya dengan menggunakan TLS untuk sertifikat client, atau sistem token seperti JWT untuk memverifikasi identitas pengguna.
   - Authorization: Setelah otentikasi berhasil, pastikan client hanya dapat mengakses data dan operasi yang diizinkan. Hal ini dapat diatur menggunakan role-based access control (RBAC) atau _policy_ yang lebih rinci.
   - Data Encryption: Gunakan TLS untuk mengenkripsi komunikasi antara client dan server. Hal ini penting agar data yang dikirim tidak dapat disadap oleh pihak ketiga.

3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?

   Beberapa tantangan yang mungkin muncul saat menangani bidirectional streaming di Rust gRPC, terutama dalam skenario seperti aplikasi chat, meliputi kompleksitas dalam menangani sinkronisasi dan konkurensi. Karena komunikasi berlangsung dua arah secara simultan, developer harus mengelola aliran pesan masuk dan keluar secara asinkron menggunakan mekanisme seperti `async/await`, yang rentan terhadap kesalahan jika tidak dikelola dengan baik. Selain itu, terdapat tantangan dalam manajemen koneksi, misalnya jika client tiba-tiba terputus, sistem harus dapat menanganinya tanpa menyebabkan kebocoran memori. Masalah _backpressure_ juga bisa terjadi jika salah satu pihak mengirimkan data terlalu cepat, sehingga sistem perlu membatasi kecepatan aliran data agar tetap stabil. Pengelolaan _state_ juga penting dalam aplikasi chat karena sistem harus melacak sesi pengguna dan menjaga konsistensi pesan. Terakhir, perlu ada penanganan kesalahan yang baik saat memproses pesan, karena kegagalan _parsing_ atau error lainnya dapat memengaruhi seluruh sesi komunikasi.

4. What are the advantages and disadvantages of using the `tokio_stream::wrappers::ReceiverStream` for streaming responses in Rust gRPC services?

   Kelebihan:
   - Kemudahan integrasi dengan tokio dan channel: ReceiverStream memungkinkan kita membungkus `tokio::sync::mpsc::Receiver` menjadi stream yang kompatibel dengan gRPC, sehingga mempermudah integrasi antara alur pengiriman data internal (dengan channel) dan sistem streaming eksternal ke client.
   - Responsif dan fleksibel: Dengan menggunakan channel, server dapat mengirim data secara asinkron kepada client kapan saja, tanpa perlu menunggu seluruh data tersedia di awal, sehingga sangat cocok untuk kasus seperti notifikasi atau log streaming.
   - Skalabilitas yang baik: Karena berbasis async, ReceiverStream bisa digunakan dalam aplikasi berskala besar tanpa blocking, dan mampu menangani banyak client secara bersamaan.
     
   Kekurangan:
   - Potensi kebocoran memori atau task yang gantung: Jika channel tidak di-drop dengan benar, receiver bisa menunggu selamanya tanpa ada data yang dikirim, menyebabkan task menggantung.
   - Kesalahan tidak terdeteksi langsung: Error pada sisi sender tidak secara otomatis diteruskan ke stream. Kita perlu membungkus data dalam enum atau Result agar bisa mengirimkan status error ke client.
   - Penggunaan MPSC bisa menjadi bottleneck: Jika digunakan secara berlebihan tanpa pertimbangan kapasitas dan kontrol, channel bisa penuh atau lambat yang berpengaruh pada performa.

5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?

   Untuk mendukung code reuse dan modularitas dalam pengembangan layanan gRPC dengan Rust, struktur kode sebaiknya dibagi berdasarkan tanggung jawab utama. File `.proto` dan hasil generate dapat dipisahkan dalam folder khusus, sementara logika layanan gRPC ditulis dalam modul terpisah yang mengimplementasikan trait dari gRPC. Logika bisnis utama dipisahkan dari akses data dengan menggunakan trait, yang membuat menjadi lebih fleksibel dan pengujian lebih mudah. Setiap fitur atau layanan juga dapat dimodularisasi ke dalam file atau folder tersendiri, sehingga strukturnya bersih dan mudah dikembangkan. Untuk meningkatkan maintainability, error handling sebaiknya dalam satu enum dan semua konfigurasi seperti port dan koneksi database dipisahkan ke file config. Dengan pendekatan ini, proyek menjadi lebih terorganisir, fleksibel terhadap perubahan, dan mudah dikembangkan dalam jangka panjang.

6. In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?

   Untuk menangani payment processing yang lebih kompleks, ada beberapa langkah tambahan yang perlu dilakukan, misalnya menambahkan proses autentikasi dan otorisasi, yaitu memverifikasi apakah pengguna berhak melakukan pembayaran tersebut dan memiliki saldo atau metode pembayaran yang sah. Selain itu, penting juga untuk menangani error handling dan retry mechanism jika terjadi kegagalan pada proses eksternal, serta mencatat transaksi ke dalam log dan database untuk keperluan audit atau histori pembayaran.

7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?

   Adopsi gRPC dalam sistem terdistribusi berdampak signifikan pada arsitektur dan desain sistem dengan meningkatkan efisiensi komunikasi melalui HTTP/2 dan Protobuf yang ringan, serta mendorong pendekatan _contract-first_ menggunakan file `.proto` untuk menjaga konsistensi antar layanan. gRPC mendukung berbagai bahasa pemrograman, sehingga mempermudah komunikasi antar layanan lintas platform, namun integrasi dengan sistem non-gRPC seperti frontend berbasis REST atau layanan lama membutuhkan adaptor tambahan seperti gRPC-Gateway, yang dapat menambah kompleksitas. Dari sisi desain sistem, gRPC memfasilitasi penggunaan arsitektur _microservices_ dengan mendukung komunikasi sinkron dan streaming (server-streaming, client-streaming, dan bidirectional), memungkinkan interaksi yang lebih fleksibel, tetapi juga menuntut desain sistem yang memperhatikan error handling, timeouts, dan fallback karena sifatnya yang _real-time_ dan sensitif terhadap gangguan jaringan.

8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?

   Penggunaan HTTP/2 sebagai protokol dasar gRPC memiliki beberapa keunggulan dibandingkan HTTP/1.1 atau HTTP/1.1 dengan WebSocket dalam konteks REST API. Keunggulan utamanya adalah kemampuan multiplexing, yaitu mengirim banyak permintaan dalam satu koneksi TCP tanpa menunggu satu per satu selesai seperti pada HTTP/1.1, sehingga mengurangi latensi dan meningkatkan efisiensi komunikasi antar layanan. Selain itu, HTTP/2 menggunakan header compression yang lebih efisien, sehingga cocok untuk skenario streaming seperti chat atau _real-time_ monitoring. Dibandingkan WebSocket, HTTP/2 juga menawarkan struktur komunikasi yang lebih aman secara bawaan melalui TLS.

   Sementara itu, kekurangannya adalah gRPC berbasis HTTP/2 tidak didukung secara penuh oleh semua browser, sehingga kurang cocok untuk client berbasis web tanpa bantuan seperti gRPC-Web. Di sisi lain, REST API berbasis HTTP/1.1 atau WebSocket masih lebih kompatibel secara luas, terutama dalam aplikasi frontend. Selain itu, debugging HTTP/2 cenderung lebih kompleks karena biner dan tidak setransparan HTTP/1.1. Dengan demikian, meskipun HTTP/2 memberikan performa dan efisiensi yang lebih baik, pemilihan protokol tetap perlu disesuaikan dengan kebutuhan, lingkungan sistem, dan target client.

9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?

    Model request-response pada REST API bersifat sinkron dan stateless, di mana client mengirim permintaan dan harus menunggu respons sebelum dapat melanjutkan interaksi berikutnya. Pendekatan ini cukup efektif untuk komunikasi satu arah dan operasi sederhana seperti pengambilan data, tetapi kurang ideal untuk skenario komunikasi _real-time_ karena tidak mendukung koneksi terbuka jangka panjang secara native. Sebaliknya, gRPC mendukung bidirectional streaming, yang memungkinkan client dan server saling mengirim pesan secara terus-menerus melalui satu koneksi yang tetap terbuka. Ini membuat gRPC jauh lebih responsif dan efisien untuk aplikasi _real-time_ seperti chat, notifikasi langsung, atau pemantauan data langsung. Dengan demikian, gRPC memberikan fleksibilitas dan performa yang lebih tinggi dalam komunikasi dua arah dan interaksi berbasis peristiwa secara _real-time_ dibandingkan REST.

10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?

    Pendekatan skema pada gRPC dengan Protobuf memastikan data konsisten, efisien, dan cepat diproses berkat format binernya, namun kurang fleksibel karena setiap perubahan memerlukan update skema. Sementara itu, JSON pada REST memungkinkan perubahan struktur data secara lebih bebas dan mudah dibaca manusia, sehingga cocok untuk pengembangan cepat, tetapi cenderung lebih lambat dan rentan terhadap inkonsistensi antar layanan. Maka dari itu, pilihan antara skema Protobuf dan JSON harus mempertimbangkan kebutuhan akan performa, skala sistem, serta tingkat fleksibilitas yang diinginkan.