# Pengantar

Pada tahun 2020, StarkWare merilis Cairo 0, sebuah bahasa pemrograman yang mendukung komputasi yang dapat diverifikasi dan berbasis Turing lengkap. Cairo dimulai sebagai bahasa rakitan (assembly language) dan secara bertahap menjadi lebih ekspresif. Kurva pembelajarannya pada awalnya curam, karena Cairo 0.x merupakan bahasa tingkat rendah yang tidak sepenuhnya mengabstraksi primitif kriptografi yang mendasari yang diperlukan untuk membangun bukti untuk eksekusi suatu program.

Dengan rilis Cairo 1, pengalaman pengembangan telah sangat ditingkatkan, dengan mengabstraksi model memori yang tidak berubah dari arsitektur Cairo sebisa mungkin. Terinspirasi kuat oleh Rust, Cairo 1 dibangun untuk membantu Anda membuat program-program yang dapat dibuktikan tanpa pengetahuan khusus tentang arsitektur dasarnya sehingga Anda dapat fokus pada program itu sendiri, meningkatkan keselamatan keseluruhan dari program-program Cairo. Didukung oleh Rust VM, eksekusi program-program Cairo sekarang sangat cepat, memungkinkan Anda untuk membangun rangkaian pengujian yang luas tanpa mengorbankan kinerja.

Pengembang blockchain yang ingin mendeploy kontrak pada Starknet akan menggunakan bahasa pemrograman Cairo untuk mengkodekan kontrak pintar mereka. Hal ini memungkinkan Starknet OS untuk menghasilkan jejak eksekusi untuk transaksi yang akan dibuktikan oleh suatu pembuktian, yang kemudian diverifikasi pada Ethereum L1 sebelum memperbarui akar keadaan Starknet.

Namun, Cairo bukan hanya untuk pengembang blockchain. Sebagai bahasa pemrograman tujuan umum, ia dapat digunakan untuk komputasi apa pun yang akan mendapatkan manfaat dari bukti di satu komputer dan diverifikasi pada mesin lain dengan persyaratan perangkat keras yang lebih rendah.

Buku ini dirancang untuk para pengembang dengan pemahaman dasar tentang konsep pemrograman. Ini adalah teks yang ramah dan mudah dipahami yang dimaksudkan untuk membantu Anda meningkatkan pengetahuan Anda tentang Cairo, tetapi juga membantu Anda mengembangkan keterampilan pemrograman Anda secara umum. Jadi, mulailah dan bersiaplah untuk belajar semua yang perlu Anda ketahui tentang Cairo!

— Komunitas Cairo