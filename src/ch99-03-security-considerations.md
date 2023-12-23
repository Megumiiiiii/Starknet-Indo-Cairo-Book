# Pertimbangan Keamanan

Ketika mengembangkan perangkat lunak, memastikan bahwa perangkat lunak tersebut berfungsi sebagaimana yang dimaksud biasanya cukup mudah. Namun, mencegah penggunaan yang tidak diinginkan dan kerentanan bisa menjadi lebih menantang.

Dalam pengembangan kontrak pintar (smart contract), keamanan sangatlah penting. Sebuah kesalahan kecil saja dapat mengakibatkan kehilangan aset berharga atau berfungsinya fitur-fitur tertentu secara tidak benar.

Kontrak pintar dijalankan dalam lingkungan publik di mana siapa pun dapat memeriksa kode dan berinteraksi dengannya. Setiap kesalahan atau kerentanan dalam kode dapat dieksploitasi oleh pihak yang jahat.

Bab ini menyajikan rekomendasi umum untuk menulis kontrak pintar yang aman. Dengan menggabungkan konsep-konsep ini selama pengembangan, Anda dapat membuat kontrak pintar yang kuat dan dapat diandalkan. Hal ini mengurangi kemungkinan perilaku atau kerentanan yang tidak terduga.

## Penyangkalan

Bab ini tidak memberikan daftar lengkap dari semua kemungkinan masalah keamanan, dan tidak menjamin bahwa kontrak Anda akan sepenuhnya aman.

Jika Anda sedang mengembangkan kontrak pintar untuk digunakan secara produksi, sangat disarankan untuk melakukan audit eksternal yang dilakukan oleh ahli keamanan.

## Pola Pikir

Cairo adalah bahasa yang sangat aman yang terinspirasi oleh Rust. Dirancang sedemikian rupa untuk memaksa Anda menutupi semua kemungkinan kasus. Masalah keamanan pada Starknet sebagian besar muncul dari cara alur kontrak pintar dirancang, bukan begitu banyak dari bahasa itu sendiri.

Mengadopsi pola pikir keamanan adalah langkah awal dalam menulis kontrak pintar yang aman. Cobalah untuk selalu mempertimbangkan semua skenario yang mungkin saat menulis kode.

### Melihat Kontrak Pintar sebagai Mesin Keadaan Terbatas

Transaksi dalam kontrak pintar bersifat atomik, artinya mereka either berhasil atau gagal tanpa membuat perubahan apapun.

Pikirkan kontrak pintar sebagai mesin keadaan: mereka memiliki kumpulan keadaan awal yang ditentukan oleh batasan konstruktor, dan fungsi eksternal mewakili kumpulan transisi keadaan yang mungkin. Sebuah transaksi hanyalah sebuah transisi keadaan.

Fungsi `assert` atau `panic` dapat digunakan untuk memvalidasi kondisi sebelum melakukan tindakan tertentu. Anda dapat mempelajari lebih lanjut mengenai hal ini di halaman [Unrecoverable Errors with panic](./ch10-01-unrecoverable-errors-with-panic.md).

Validasi ini bisa mencakup:

- Masukan yang diberikan oleh pemanggil
- Persyaratan eksekusi
- Invarian (kondisi yang harus selalu benar)
- Nilai kembalian dari panggilan fungsi lainnya

Sebagai contoh, Anda dapat menggunakan fungsi `assert` untuk memvalidasi bahwa seorang pengguna memiliki cukup dana untuk melakukan transaksi penarikan. Jika kondisinya tidak terpenuhi, transaksi akan gagal dan keadaan kontrak tidak akan berubah.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_10_assert_balance/src/lib.cairo:withdraw}}
```

Menggunakan fungsi-fungsi ini untuk memeriksa kondisi menambahkan batasan yang membantu menentukan dengan jelas batas-batas transisi keadaan yang mungkin untuk setiap fungsi dalam kontrak pintar Anda. Pemeriksaan ini memastikan bahwa perilaku kontrak tetap berada dalam batas yang diharapkan.

## Rekomendasi

### Pola Checks Effects Interactions

Pola Checks Effects Interactions adalah pola desain umum yang digunakan untuk mencegah serangan reentrancy pada Ethereum. Meskipun reentrancy lebih sulit dicapai di Starknet, masih disarankan untuk menggunakan pola ini dalam kontrak pintar Anda.

<!-- TODO tambahkan referensi ke halaman CairoByExample mengenai reentrancy -->

Pola ini terdiri dari mengikuti urutan operasi tertentu dalam fungsi-fungsi Anda:

1. **Pengecekan**: Memvalidasi semua kondisi dan masukan sebelum melakukan perubahan keadaan apapun.
2. **Efek**: Melakukan semua perubahan keadaan.
3. **Interaksi**: Semua panggilan eksternal ke kontrak lain sebaiknya dilakukan di akhir fungsi.

### Kontrol Akses

Kontrol akses adalah proses membatasi akses ke fitur atau sumber daya tertentu. Ini adalah mekanisme keamanan umum yang digunakan untuk mencegah akses tidak sah ke informasi atau tindakan sensitif. Dalam kontrak pintar, beberapa fungsi mungkin sering dibatasi kepada pengguna atau peran tertentu.

Anda dapat menerapkan pola kontrol akses untuk mengelola izin dengan mudah. Pola ini terdiri dari mendefinisikan kumpulan peran dan menetapkannya kepada pengguna tertentu. Setiap fungsi kemudian dapat dibatasi kepada peran-peran tertentu.

```rust,noplayground
{{#include ../listings/ch99-starknet-smart-contracts/listing_99_11_simple_access_control/src/lib.cairo}}
```

### Alat Analisis Statis

Analisis statis merujuk pada proses pemeriksaan kode tanpa eksekusinya, berfokus pada struktur, sintaksis, dan properti kode tersebut. Ini melibatkan analisis kode sumber untuk mengidentifikasi masalah potensial, kerentanan, atau pelanggaran aturan yang ditentukan.

Dengan mendefinisikan aturan, seperti konvensi penulisan kode atau pedoman keamanan, pengembang dapat menggunakan alat analisis statis untuk secara otomatis memeriksa kode sesuai dengan standar-standar tersebut.

Referensi:

- [Dukungan Semgrep Cairo 1.0](https://semgrep.dev/blog/2023/semgrep-now-supports-cairo-1-0)
- [Caracal, penganalisis statis Starknet](https://github.com/crytic/caracal)
