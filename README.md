# Module 9 - High Level Networking

> ##### Muhammad Daffa Grahito Triharsanto - 2206820075 - Pemrograman Lanjut B

## Reflection
1. ***What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?***
   - **Unary RPC** adalah bentuk yang paling simple di RPC dimana client mengirimkan request ke server dan menunggu mendapatkan response kembali dari server. Biasanya ini digunakan untuk simple request-response scenario saja dimana tidak ada interaksi tambahan dengan request data diperlukan seperti pengambilan data tertentu dari server. 
   - **Server streaming RPC** adalah saat dimana client mengirimkan request ke server dan server merespons dengan serangkaian pesan yang berurutan hingga tidak ada pesan lagi. Biasanya ini digunakan saat client perlu server me-*return* data yang besar, seperti pengambilan data dalam jumlah besar dari database.
   - **Bi-Directional Streaming RPC** adalah saat dimana kedua sisi, client dan server, mengirimkan serangkaian pesan secara independen menggunakan *read-write* stream dalam arah yang sama atau berlawanan secara bersamaan. Biasanya ini digunakan saat client dan server perlu berinteraksi untuk mengirimkan dan menerima data dalam waktu nyata, seperti Chat Service.

2. ***What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?***
    - **Authentication**, gRPC mendukung berbagai mekanisme autentikasi, termasuk TLS (Transport Layer Security), OAuth2, autentikasi berbasis token, dan yang paling simple yaitu username dan password saja.
    - **Authorization**, Setelah terautentikasi juga perlu ada penentuan permission yang tepat untuk requested operationnya. Bisa dengan menggunakan Role Based Access Control (RBAC) atau Attribute-Based Access Control (ABAC) dan di-set dengan mengecek secara manual di service method nya serta menggunaakan pola middleware atau interceptor.
    - **Data Encryption**, Lalu mengenkripsi data yang ditransmisikan di network harus dilindungi dari *interception* dan *eavesdropping*. Secara default gRPC menggunakan HTTP/2, yang mendukung enkripsi melalui TLS.
   
3. ***What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?***
Problem yang mungkin timbul yaitu seperti **Error Handling** saat read-write stream, **Connection Management** seperti seperti menangani pemutusan koneksi yang tidak terduga dan mengembalikan koneksi dengan cepat karena client dapat connect dan disconnect kapanpun, dan juga **Concurrency Management (Message Synchronization)** karena banyak client dapat mengirim dan menerima pesan secara bersamaan sehingga perlu manage responses dari client yang berbeda-beda secara tepat.
   
4. ***What are the advantages and disadvantages of using the `tokio_stream::wrappers::ReceiverStream` for streaming responses in Rust gRPC services?***
    - ### Advantages
      - **Simple**, `ReceiverStream` bisa untuk mengonversi Receiver menjadi Stream, yang dapat digunakan langsung dalam gRPC.
      - **Kompatibilitas**: `ReceiverStream` kompatibel dengan trait Stream, yang digunakan secara luas dalam Rust sehingga dapat digunakan dengan function dan library lain yang bekerja dengan Stream.
      - **Penanganan Backpressure**: `ReceiverStream` secara alami menyediakan penanganan backpressure. Jika receiver penuh (yaitu, telah mencapai kapasitasnya), pengirim akan dijeda sampai ada ruang kosong, mencegah sistem dari kelebihan pesan.
    - ### Disadvantages
      - **Error Handling**, `ReceiverStream` tidak punya built-in mechanism untuk menangani kesalahan yang terjadi saat menerima pesan.
      - **Komunikasi Satu Arah**, `ReceiverStream` hanya mendukung komunikasi satu arah dari pengirim ke penerima.
      - **Kontrol Terbatas atas Polling**, `ReceiverStream` memberikan kontrol terbatas atas kapan dan bagaimana underlying Receiver di poll.

   
5. ***In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?***
   
    Beberapa caranya bisa dengan memisahkan *business logic* dari implementasi gRPC itu sendiri, kita menciptakan modularitas yang memungkinkan penggunaan kembali kode dalam konteks yang berbeda tanpa terikat pada infrastruktur spesifik. Lalu, dengan menggunakan Protobuf untuk define services sehingga memperjelas *service contracts* yang dapat diimplementasikan secara independen. Selain itu, penggunaan Traits dalam Rust memungkinkan solid abstraction antara *services* dan *concrete implementations*, memisahkan business logic dari infrastruktur gRPC. Adapun cara lain lagi seperti *code modularization*, memperhatikan *dependency injection*, dan pemastian *error handling*.

6. ***In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?***
   
    Additional step yang bisa dilakukan adalah:
    - **Validasi request**, apakah PaymentRequest mengandung informasi yang diperlukan dan valid dengan mengecek apakah id nya exist atau amountnya > 0
    - **Koneksi dengan External Services**, Mungkin perlu external service seperti fraud detection service atau payment gateway.
    - **Handle error**, Perlu handle error yang berpotensi seperti return appropriate gRPC statuses, yaitu seperti mapping database error maupun validation error.
    - **Interaksi dengan database**, Mungkin juga perlu berinteraksi dengan database secara langsung untuk mengecek balance user, update balance setelah payment, dan merecord transaction tersebut. Ini mungkin perlu async database library agar langsung menghandle database error yang juga bisa timbul.
   
7. ***What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?***

    Penggunaan gRPC sebagai *communication protocol* memungkinkan interoperabilitas antar layanan yang ditulis dalam berbagai bahasa pemrograman dan menghasilkan kinerja yang lebih efisien dengan menggunakan *Protocol Buffers* (protobuf) sebagai format pesan. Jadi ada dukungan untuk streaming dan desain berbasis kontrak juga yang memperkuat interaksi antara client dan server. Namun, kompleksitas setup dan keterbatasan browser dalam mendukung gRPC bisa menjadi hambatan, terutama jika perlu berinteraksi dengan sistem yang hanya mendukung REST.
   
8. ***What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?***

    - ### Advantages dari menggunakan HTTP/2:
      - **Multiplexing**, bisa menerima multiple request dan response sekaligus dalam sebuah TCP connection.
      - **Header compression**, HTTP/2 menggunakan HPACK compression untuk headers, sehingga mengurangi overhead terutama request dan response yang memiliki headers yang besar atau repetitif.
      - **Server Push**, Server push di HTTP/2 memungkinkan server untuk mengirimkan resource ke client secara proaktif sehingga meningkatkan performance dengan me-*reduce* keperluan round-trip requests.
    - ### Disadvantages dari menggunakan HTTP/2:
      - **Complexity**, HTTP/2 lebih complex dari HTTP/1.1 sehingga lebih sulit diimplementasikan dan debug. Tools dan librariesnya juga belum sebaik HTTP/1.1.
      - **Take more resource**, HTTP/2 dapat mengonsumsi lebih banyak resource server, seperti CPU dan memori, dibandingkan dengan HTTP/1.1 karena overhead multiplexing dan kompresi header.
      - **Cache usage**, Penggunaan multiplexing dan server push oleh HTTP/2 dapat mempersulit strategi caching perantara, sehingga mengurangi efektivitas caching dibandingkan dengan HTTP/1.1.
   
9.  ***How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?***
Keduanya berbeda terutama dalam hal komunikasi *real-time* dan responsif. REST API mengikuti pola *response-request*, di mana klien mengirim permintaan ke server dan server memberikan respons dengan data yang diminta. Sementara itu, gRPC mendukung bidirectional streaming dimana client dan server dapat saling bertukar pesan secara asinkron melalui satu koneksi sehingga memungkinkan komunikasi real-time di mana client dan server dapat bertukar data tanpa menunggu request atau response. Dalam konteks ini, gRPC lebih responsif dan cocok untuk aplikasi yang membutuhkan pertukaran data kontinu dan pembaruan yang cepat.
    
10.  ***What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?***
Pendekatan berbasis skema dari gRPC menggunakan Protocol Buffers menawarkan validasi data yang ketat, otomatisasi kode, dan kesesuaian yang kuat antara client dan server. Sementara itu, sifat yang lebih fleksibel dari JSON dalam REST API memungkinkan penanganan data yang lebih dinamis dan perubahan struktur tanpa perlu pembaruan skema atau kode. Pemilihan antara keduanya tergantung *spesific needs* masing-masing.