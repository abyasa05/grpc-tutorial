## Refleksi Tutorial 8

1. Perbedaan utama dari ketiga jenis RPC methods terletak pada volume data yang dikirimkan pada sisi klien dan server. Pada metode _unary_, klien hanya mengirimkan satu data (_request_) dan server mengirimkan satu data kembali sebagai _response_. Skenario ini cocok untuk interaksi yang hanya membutuhkan _one-time response_ seperti pemrosesan transaksi dan autentikasi. Sementara itu, pada _server streaming_, klien mengirimkan satu data dan server mengirimkan _response_ dengan jumlah data yang besar. Metode ini cocok untuk digunakan pada skenario di mana klien memerlukan _request_ akses terhadap data dalam jumlah yang besar dalam satu waktu, misalnya mengecek riwayat transaksi atau melihat katalog produk pada aplikasi _e-commerce_. Lalu, _bidirectional streaming_ merupakan metode yang memungkinkan klien dan server untuk saling bertukar data secara bersamaan (_concurrent_). Hal ini berguna dalam skenario yang memerlukan interaksi _real-time_ yang responsif antara klien dan server, misalnya pada aplikasi _chat_.

2.  Beberapa aspek keamanan yang perlu dikonsiderasi tentang implementasi gRPC di Rust di antaranya:
    - Penggunaan TLS (_Transport Security Layer_) untuk pengiriman data kredensial secara aman.
    - Penggunaan mekanisme _hashing_ yang kuat untuk menjaga keamanan data dan mendeteksi adanya tampering.
    - Proses autentikasi dengan token (seperti OAuth atau JWT) dengan menyimpan token dalam metadata gRPC untuk tiap _request_.

3.  Beberapa tantangan yang bisa muncul dalam menangani _bidirectional streaming_ gRPC di Rust di antaranya:
    - Menjaga keseimbangan antara volume/frekuensi data yang dikirimkan klien dengan kapabilitas server dalam meng-handle data.
    - Pentingnya manajemen konkurensi data yang diproses untuk mencegah skenario seperti _race condition_.
    - Manajemen _stream_ yang benar untuk menjaga efisiensi penggunaan _resource_ dan mencegah _memory leak_ (misalnya implementasi _timeout_ untuk mendeteksi _stream_ yang tidak digunakan).

4.  Kelebihan:
    - `ReceiverStream` menggunakan trait `Stream` sehingga bisa digunakan secara langsung untuk mengirimkan respons (dalam bentuk _stream_) melalui gRPC.
    - Memungkinkan proses komunikasi yang efisien melalui pengiriman dan penerimaan _message_ secara asinkronus.
    - Bisa digunakan dengan `tokio::spawn` untuk membuat respons secara asinkronus.

    Kelemahan:
    - Penggunaan `mpsc` _channel_ bisa meningkatkan latensi.
    - Potensi adanya _memory leak_ jika _sender_ mengirimkan data ketika _receiver_ memiliki performa rendah atau sedang inaktif.

5. Cara yang pertama bisa dilakukan untuk meningkatkan modularitas kode yaitu dengan membagi tiap _service_ dalam _file-file_ tersendiri (misalnya membuat _file_ `service/payment.rs`, `service/transaction.rs`, dan `service/chat.rs`). Selain itu, _trait_ juga bisa digunakan untuk mengabstraksi _behavior_ dari masing-masing `service`. Hal ini akan membuat implementasi kode menjadi fleksibel dan mudah untuk dimodifikasi. Lalu, konfigurasi dari variabel-variabel tertentu (misalnya alamat server gRPC) bisa disimpan dalam file TOML untuk mempermudah pengaturan dan modifikasi. Untuk meningkatkan _mantainability_ kode, _unit test_ juga bisa dibuat untuk menjaga integritas dan fungsionalitas dari masing-masing _service_.

6.  Berikut merupakan beberapa langkah yang bisa ditambahkan untuk menangani logika pemrosesan transaksi yang lebih kompleks:
    - Implementasi validasi input untuk `user_id` dan `amount`.
    - Penambahan autentikasi, misalnya dengan menggunakan OAuth untuk memvalidasi token user.
    - Mengenkripsi data pada proses transaksi seperti melalui penggunaan TLS (Transport Layer Service).
    - Pembatasan frekuensi _request_ per user (_rate-limiting_) untuk mencegah penyalahgunaan (misalnya serangan DDoS).
    - Implementasi _atomic transaction_ untuk memastikan bahwa keseluruhan transaksi di-_rollback_ jika terjadi kegagalan dalam prosesnya.

7. Penggunaan gRPC bisa membawa sejumlah efek terhadap arsitektur dan desain dari sistem yang digunakan. Terkait interoperabilitas, perlu diperhatikan bahwa penggunaan gRPC bisa membawa isu kompatibilitas jika ingin digunakan untuk mengintegrasikan _service_ yang masih bergantung pada HTTP/1.1 dan REST API. Selain itu, penggunaan gRPC juga mengharuskan _service-service_ yang menggunakannya untuk mendefinisikan skema data melalui Protocol Buffer. Hal ini berpotensi mengakibatkan _coupling_ pada sistem yang bisa berefek kompleks jika kemudian dilakukan perubahan/modifikasi yang signifikan pada _service_. Lalu, meskipun _bidirectional streaming_ memungkinkan dilakukannya komunikasi antar-sistem secara cepat, diperlukan penanganan yang tepat untuk menjaga performa klien dan server dalam bertukar data. Hal ini mencakup manajemen _stream_ serta kontrol atas ukuran data yang ditransimikan.

8.  Kelebihan HTTP/2:
    - Memungkinkan klien dan server untuk mengirimkan data dalam volume besar secara bersamaan (berguna untuk pengiriman data dalam bentuk _stream_).
    - Memungkinkan beberapa _stream_ (_request_) untuk dikirimkan melalui satu koneksi TCP saja.
    - Memiliki _header_ yang berukuran ringan karena menggunakan kompresi HPACK.
    - Data dikirimkan dalam bentuk _binary frame_ yang membuatnya lebih ringan dan efisien untuk di-_parse_.

    Kelemahan HTTP/2:
    - _Support_ oleh browser-browser yang ada masih relatif terbatas dibandingkan HTTP/1.1.
    - Penggunaan _binary frame_ sebagai format data cenderung lebih sulit untuk diimplementasikan dan di-_debug_.
    - Kapabilitas untuk memuat data dalam jumlah besar secara langsung membuatnya rentan terhadap ketidakstabilan jaringan.

9. Model _request-response_ dari REST API melakukan komunikasi antara klien dan server secara sinkronus. Dalam kata lain, klien mengirimkan suatu _request_ dan menunggu _server_ untuk mengirim kembali satu respons dalam satu koneksi TCP. Sementara itu, model _bidirectional streaming_ oleh gRPC memungkinkan klien dan server untuk berkomunikasi secara asinkronus. Dalam kata lain, jika suatu koneksi TCP sudah terbangun, klien bisa mengirimkan data-data secara berangsur (_stream of data_) dan pada saat yang bersamaan, bisa menerima data secara bertahap dari server sebagai respons terhadap data-data yang dikirimkan tadi.

10. Pada gRPC, penggunaan Protocol Buffer mengharuskan klien dan server untuk mendefinisikan skema dari data yang ingin ditransmisikan secara eksplisit. Pendekatan seperti ini bisa berguna untuk memastikan validitas dan konsistensi dari data yang akan dikirim dan diterima antara kedua sisi. Sementara itu, format JSON pada REST API bersifat lebih fleksibel dan tidak memiliki _constraint_ tertentu mengenai format data yang bisa dikirimkan. Maka dari itu, biasanya diperlukan validasi tambahan dari sisi klien dan server untuk menerima data dengan aman.