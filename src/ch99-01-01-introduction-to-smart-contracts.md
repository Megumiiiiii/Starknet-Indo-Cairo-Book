# Pengantar tentang Kontrak Pintar

Bab ini akan memberikan pengantar tingkat tinggi tentang apa itu kontrak pintar, untuk apa mereka digunakan, dan mengapa pengembang blockchain menggunakan Cairo dan Starknet.
Jika Anda sudah familiar dengan pemrograman blockchain, silakan lewati bab ini. Bagian terakhir mungkin tetap menarik.

## Kontrak Pintar

Kontrak pintar menjadi populer dan semakin tersebar luas dengan lahirnya Ethereum. Kontrak pintar pada dasarnya adalah program-program yang diterapkan pada blockchain. Istilah "kontrak pintar" agak menyesatkan, karena mereka tidak benar-benar "pintar" atau "kontrak," melainkan kode dan instruksi yang dieksekusi berdasarkan input tertentu. Mereka terutama terdiri dari dua komponen: penyimpanan dan fungsi. Setelah diterapkan, pengguna dapat berinteraksi dengan kontrak pintar dengan memulai transaksi blockchain yang berisi data eksekusi (fungsi apa yang harus dipanggil dan dengan input apa). Kontrak pintar dapat memodifikasi dan membaca penyimpanan dari blockchain yang mendasarinya. Kontrak pintar memiliki alamat sendiri dan dianggap sebagai akun blockchain, yang berarti dapat menyimpan token.

Bahasa pemrograman yang digunakan untuk menulis kontrak pintar bervariasi tergantung pada blockchain. Misalnya, di Ethereum dan [ekosistem yang kompatibel dengan EVM](https://ethereum.org/en/developers/docs/evm/), bahasa yang paling umum digunakan adalah Solidity, sedangkan di Starknet, menggunakan bahasa Cairo. Cara kode dikompilasi juga berbeda berdasarkan blockchain. Di Ethereum, Solidity dikompilasi menjadi bytecode. Di Starknet, Cairo dikompilasi menjadi Sierra, kemudian menjadi Cair Assembly (casm).

Kontrak pintar memiliki beberapa karakteristik unik. Mereka **tanpa izin**, artinya siapa pun dapat menerapkan kontrak pintar di jaringan (dalam konteks blockchain terdesentralisasi, tentu saja). Kontrak pintar juga **transparan**; data yang disimpan oleh kontrak pintar dapat diakses oleh siapa pun. Kode yang menyusun kontrak juga dapat transparan, memungkinkan **komposabilitas**. Ini memungkinkan pengembang menulis kontrak pintar yang menggunakan kontrak pintar lainnya. Kontrak pintar hanya dapat mengakses dan berinteraksi dengan data dari blockchain tempat mereka diterapkan. Mereka memerlukan perangkat lunak pihak ketiga (yang disebut `oracle`) untuk mengakses data eksternal (seperti harga token, misalnya).

Untuk memungkinkan pengembang membangun kontrak pintar yang dapat berinteraksi satu sama lain, diperlukan pengetahuan tentang bagaimana kontrak lainnya terlihat. Oleh karena itu, pengembang Ethereum mulai membangun standar untuk pengembangan kontrak pintar, yang dikenal sebagai `ERCxx`. Dua standar yang paling banyak digunakan dan terkenal adalah `ERC20`, digunakan untuk membangun token seperti `USDC`, `DAI`, atau `STARK`, dan `ERC721`, untuk NFT (Token Non-fungible) seperti `CryptoPunks` atau `Everai`.

## Kasus Penggunaan

Ada banyak kasus penggunaan potensial untuk kontrak pintar. Satu-satunya batasan adalah kendala teknis dari blockchain dan kreativitas para pengembang.

#### DeFi

Saat ini, penggunaan utama untuk kontrak pintar serupa dengan Ethereum atau Bitcoin, yaitu pada dasarnya menangani uang. Dalam konteks sistem pembayaran alternatif yang dijanjikan oleh Bitcoin, kontrak pintar di Ethereum memungkinkan pembuatan aplikasi keuangan terdesentralisasi yang tidak lagi bergantung pada perantara keuangan tradisional. Ini yang disebut sebagai DeFi (keuangan terdesentralisasi). DeFi terdiri dari berbagai proyek seperti aplikasi peminjaman/pinjaman, pertukaran terdesentralisasi (DEX), derivatif on-chain, stablecoin, dana lindung terdesentralisasi, asuransi, dan banyak lagi.

#### Tokenisasi

Kontrak pintar dapat memfasilitasi tokenisasi aset dunia nyata, seperti properti, seni, atau logam mulia. Tokenisasi membagi suatu aset menjadi token digital, yang dapat dengan mudah diperdagangkan dan dikelola di platform blockchain. Ini dapat meningkatkan likuiditas, memungkinkan kepemilikan fraksional, dan menyederhanakan proses jual beli.

#### Voting

Kontrak pintar dapat digunakan untuk membuat sistem pemilihan yang aman dan transparan. Suara dapat dicatat di blockchain, memastikan ketidakubahannya dan transparansi. Kontrak pintar kemudian dapat secara otomatis menghitung suara dan mengumumkan hasilnya, meminimalkan potensi kecurangan atau manipulasi.

#### Royalti

Kontrak pintar dapat mengotomatisasi pembayaran royalti bagi seniman, musisi, dan pencipta konten lainnya. Ketika suatu konten dikonsumsi atau dijual, kontrak pintar dapat secara otomatis menghitung dan mendistribusikan royalti kepada pemilik yang berhak, memastikan kompensasi yang adil dan mengurangi kebutuhan akan perantara.

#### Identitas Terdesentralisasi (DID)

Kontrak pintar dapat digunakan untuk membuat dan mengelola identitas digital, memungkinkan individu mengontrol informasi pribadi mereka dan berbagi dengan pihak ketiga secara aman. Kontrak pintar dapat memverifikasi otentisitas identitas pengguna dan secara otomatis memberikan atau mencabut akses ke layanan tertentu berdasarkan kredensial pengguna.

<br/>
<br/>
Seiring berlanjutnya kedewasaan Ethereum, kita dapat mengharapkan kasus penggunaan dan aplikasi kontrak pintar berkembang lebih jauh, membawa peluang baru yang menarik dan membentuk kembali sistem tradisional menjadi lebih baik.

## Munculnya Starknet dan Cairo

Ethereum, sebagai platform kontrak pintar yang paling banyak digunakan dan tangguh, menjadi korban kesuksesannya sendiri. Dengan adopsi cepat beberapa kasus penggunaan yang telah disebutkan sebelumnya, terutama DeFi, biaya untuk melakukan transaksi menjadi sangat tinggi, membuat jaringan hampir tidak dapat digunakan. Insinyur dan peneliti di ekosistem ini mulai bekerja pada solusi untuk mengatasi masalah skalabilitas ini.

Trilema terkenal ([The Blockchain Trilemma](https://vitalik.ca/general/2021/04/07/sharding.html#the-scalability-trilemma)) dalam ruang blockchain menyatakan bahwa tidak mungkin mencapai tingkat skalabilitas, desentralisasi, dan keamanan yang tinggi secara bersamaan; harus ada kompromi. Ethereum berada pada persimpangan desentralisasi dan keamanan. Akhirnya, diputuskan bahwa tujuan Ethereum akan menjadi sebagai lapisan penyelesaian yang aman, sementara perhitungan kompleks akan dipindahkan ke jaringan lain yang dibangun di atas Ethereum. Ini disebut sebagai Layer 2 (L2).

Dua jenis utama dari L2 adalah optimistic rollups dan validity rollups. Kedua pendekatan melibatkan kompresi dan pengelompokan sejumlah transaksi bersama-sama, menghitung status baru, dan menyelesaikan hasilnya di Ethereum (L1). Perbedaannya terletak pada cara hasilnya diselesaikan di L1. Untuk optimistic rollups, status baru dianggap valid secara default, tetapi ada jendela 7 hari bagi node untuk mengidentifikasi transaksi yang bersifat berbahaya.

Sebaliknya, validity rollups, seperti Starknet, menggunakan kriptografi untuk membuktikan bahwa status baru telah dihitung dengan benar. Ini adalah tujuan dari STARKs, teknologi kriptografi ini dapat memungkinkan validity rollups untuk dapat meningkatkan skalabilitas secara signifikan lebih dari optimistic rollups. Anda dapat mempelajari lebih lanjut tentang STARKs dari [artikel](https://medium.com/starkware/starks-starkex-and-starknet-9a426680745a) Medium Starkware, yang berfungsi sebagai panduan yang baik.

> Arsitektur Starknet dijelaskan secara rinci dalam [Buku Starknet](https://book.starknet.io/chapter_4/index.html), yang merupakan sumber yang bagus untuk mempelajari lebih lanjut tentang jaringan Starknet.

Ingat Cairo? Sebenarnya, ini adalah bahasa yang dikembangkan khusus untuk bekerja dengan STARKs dan membuatnya menjadi tujuan umum. Dengan Cairo, kita dapat menulis **kode yang dapat dibuktikan**. Dalam konteks Starknet, ini memungkinkan membuktikan kebenaran perhitungan dari satu status ke status lainnya.

Berbeda dengan sebagian besar (jika tidak semua) pesaing Starknet yang memilih menggunakan EVM (entah itu apa adanya atau disesuaikan) sebagai lapisan dasar, Starknet menggunakan VM sendiri. Ini membebaskan pengembang dari batasan EVM, membuka berbagai kemungkinan. Bersamaan dengan penurunan biaya transaksi, kombinasi Starknet dan Cairo menciptakan tempat bermain yang menarik bagi para pengembang. Abstraksi akun asli memungkinkan logika yang lebih kompleks untuk akun, yang disebut "Smart Accounts," dan alur transaksi. Kasus penggunaan yang muncul termasuk **kecerdasan buatan yang transparan** dan aplikasi pembelajaran mesin. Akhirnya, **game blockchain** dapat dikembangkan sepenuhnya **on-chain**. Starknet telah dirancang khusus untuk memaksimalkan kemampuan bukti STARK untuk skalabilitas yang optimal.

> Pelajari lebih lanjut tentang Abstraksi Akun di [Buku Starknet](https://book.starknet.io/chapter_5/index.html).

## Program Cairo dan Kontrak Starknet: Apa Perbedaannya?

Kontrak Starknet adalah superset khusus dari program Cairo, sehingga konsep-konsep yang telah dipelajari sebelumnya dalam buku ini masih berlaku untuk menulis kontrak Starknet.
Seperti yang mungkin telah Anda perhatikan, program Cairo harus selalu memiliki fungsi `main` yang berfungsi sebagai titik masuk untuk program ini:

```rust
fn main() {}
```

Kontrak Starknet pada dasarnya adalah program-program yang dapat berjalan di sistem operasi Starknet, dan sebagai program semacam itu, mereka memiliki akses ke status Starknet. Agar suatu modul dapat dianggap sebagai kontrak oleh kompiler, modul tersebut harus dianotasi dengan atribut `#[starknet::contract]`.
