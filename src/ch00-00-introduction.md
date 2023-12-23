# Pengantar

## Apa itu Cairo?

Cairo adalah bahasa pemrograman yang dirancang untuk CPU virtual dengan nama yang sama. Aspek unik dari prosesor ini adalah bahwa ia tidak diciptakan untuk batasan fisik dunia kita, tetapi untuk batasan kriptografi, sehingga mampu membuktikan secara efisien eksekusi dari setiap program yang berjalan di atasnya. Ini berarti Anda dapat melakukan operasi yang memakan waktu pada mesin yang tidak Anda percayai, dan memeriksa hasilnya dengan cepat pada mesin yang lebih murah.
Sementara Cairo 0 dulunya langsung dikompilasi ke CASM, perakitan CPU Cairo, Cairo 1 adalah bahasa yang lebih tinggi. Cairo 1 pertama-tama dikompilasi ke Sierra, representasi intermediate dari Cairo yang akan dikompilasi nantinya ke subset aman dari CASM. Tujuan dari Sierra adalah memastikan CASM Anda selalu dapat dibuktikan, bahkan ketika komputasi gagal.

## Apa yang bisa Anda lakukan dengan itu?

Cairo memungkinkan Anda menghitung nilai-nilai yang dapat dipercaya pada mesin-mesin yang tidak dipercayai. Salah satu penggunaan utama adalah Starknet, solusi untuk skalabilitas Ethereum. Ethereum adalah platform blockchain terdesentralisasi yang memungkinkan pembuatan aplikasi terdesentralisasi di mana setiap interaksi antara pengguna dan d-app diverifikasi oleh semua partisipan. Starknet adalah Layer 2 yang dibangun di atas Ethereum. Alih-alih memiliki semua partisipan jaringan untuk memverifikasi semua interaksi pengguna, hanya satu node, yang disebut prover, yang menjalankan program dan menghasilkan bukti bahwa komputasi dilakukan dengan benar. Bukti-bukti ini kemudian diverifikasi oleh kontrak pintar Ethereum, membutuhkan daya komputasi yang jauh lebih sedikit dibandingkan dengan mengeksekusi interaksi itu sendiri. Pendekatan ini memungkinkan peningkatan throughput dan pengurangan biaya transaksi sambil tetap menjaga keamanan Ethereum.

## Apa perbedaannya dengan bahasa pemrograman lain?

Cairo cukup berbeda dari bahasa pemrograman tradisional, terutama dalam hal biaya overhead dan keuntungan utamanya. Program Anda dapat dieksekusi dengan dua cara yang berbeda:

- Ketika dieksekusi oleh prover, ini mirip dengan bahasa lainnya. Karena Cairo tervirtualisasi, dan karena operasi tidak dirancang secara khusus untuk efisiensi maksimum, ini dapat menyebabkan overhead kinerja tetapi bukan bagian yang paling relevan untuk dioptimalkan.

- Ketika bukti yang dihasilkan diverifikasi oleh verifier, ini agak berbeda. Ini harus semurah mungkin karena bisa saja diverifikasi pada banyak mesin yang sangat kecil. Untungnya, verifikasi lebih cepat daripada perhitungan dan Cairo memiliki beberapa keunggulan unik untuk meningkatkannya lebih jauh. Salah satunya adalah non-determinisme. Ini adalah topik yang akan Anda bahas lebih detail nanti dalam buku ini, tetapi ideanya adalah bahwa secara teoretis Anda dapat menggunakan algoritma yang berbeda untuk verifikasi daripada untuk perhitungan. Saat ini, menulis kode non-deterministik kustom tidak didukung untuk para pengembang, tetapi pustaka standar memanfaatkan non-determinisme untuk meningkatkan kinerja. Misalnya, mengurutkan sebuah array dalam Cairo memiliki biaya yang sama dengan mengkopi array tersebut. Karena verifier tidak mengurutkan array, hanya memeriksa apakah array tersebut terurut, yang lebih murah.

Aspek lain yang membedakan bahasa ini adalah model memorinya. Dalam Cairo, akses memori bersifat tidak dapat diubah, artinya setelah nilai ditulis ke memori, itu tidak dapat diubah. Cairo 1 menyediakan abstraksi yang membantu pengembang dalam bekerja dengan batasan-batasan ini, tetapi tidak sepenuhnya mensimulasikan mutabilitas. Oleh karena itu, pengembang harus memikirkan dengan hati-hati bagaimana mereka mengelola memori dan struktur data dalam program mereka untuk mengoptimalkan kinerja.

## Referensi

- Arsitektur CPU Cairo: <https://eprint.iacr.org/2021/1063>
- Cairo, Sierra, dan Casm: <https://medium.com/nethermind-eth/under-the-hood-of-cairo-1-0-exploring-sierra-7f32808421f5>
- Status non-determinism: <https://twitter.com/PapiniShahar/status/1638203716535713798>