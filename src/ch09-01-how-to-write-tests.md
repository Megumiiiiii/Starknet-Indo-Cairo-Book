# Cara Menulis Uji Coba

## Anatomi Fungsi Uji Coba

Uji coba adalah fungsi Cairo yang memverifikasi bahwa kode non-uji berfungsi sesuai dengan yang diharapkan. Tubuh fungsi uji coba biasanya melakukan tiga tindakan ini:

- Menyiapkan data atau keadaan yang diperlukan.
- Menjalankan kode yang ingin diuji.
- Memastikan hasil sesuai dengan yang diharapkan.

Mari kita lihat fitur-fitur yang disediakan Cairo khusus untuk menulis uji coba yang melibatkan tindakan-tindakan ini, termasuk atribut `test`, fungsi `assert`, dan atribut `should_panic`.

### Anatomi Fungsi Uji Coba

Pada dasarnya, uji coba di Cairo adalah fungsi yang dianotasi dengan atribut `test`. Atribut adalah metadata tentang potongan kode Cairo; salah satu contohnya adalah atribut derive yang kita gunakan dengan struct di Bab 5. Untuk mengubah fungsi menjadi fungsi uji coba, tambahkan `#[test]` pada baris sebelum `fn`. Saat Anda menjalankan uji coba dengan perintah `scarb cairo-test`, Scarb menjalankan binary pelari uji coba Cairo yang menjalankan fungsi-fungsi yang dianotasi dan melaporkan apakah setiap fungsi uji coba lulus atau gagal.

Mari buat proyek baru bernama `adder` yang akan menambahkan dua angka menggunakan Scarb dengan perintah `scarb new adder`:

```shell
adder
├── Scarb.toml
└── src
    └── lib.cairo
```

Di _lib.cairo_, mari hapus konten yang ada dan tambahkan uji coba pertama, seperti yang ditunjukkan dalam Listing 9-1.

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch09-testing-cairo-programs/listing_08_01_02/src/lib.cairo:it_works}}
```

<span class="caption">Listing 9-1: Modul dan fungsi uji coba</span>

Saat ini, mari abaikan dua baris teratas dan fokus pada fungsi. Perhatikan anotasi `#[test]`: atribut ini menunjukkan bahwa ini adalah fungsi uji coba, sehingga pelari uji coba tahu untuk memperlakukan fungsi ini sebagai uji coba. Kita juga mungkin memiliki fungsi-fungsi non-uji coba di modul uji coba untuk membantu menyiapkan skenario umum atau melakukan operasi umum, jadi kita selalu perlu menunjukkan fungsi mana yang merupakan uji coba.

Tubuh fungsi contoh menggunakan fungsi `assert`, yang berisi hasil penambahan 2 dan 2, sama dengan 4. Pernyataan ini berfungsi sebagai contoh format uji coba yang khas. Mari jalankan untuk melihat bahwa uji coba ini lulus.

Perintah `scarb cairo-test` menjalankan semua uji coba yang ditemukan di proyek kita, seperti yang ditunjukkan dalam Listing 9-2.

```shell
$ scarb cairo-test
testing adder...
running 1 tests
test adder::lib::tests::it_works ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out;
```

<span class="caption">Listing 9-2: Keluaran dari menjalankan uji coba</span>

`scarb cairo-test` mengompilasi dan menjalankan uji coba. Kami melihat baris `running 1 tests`. Baris berikutnya menunjukkan nama fungsi uji coba, disebut `it_works`, dan bahwa hasil dari menjalankan uji coba tersebut adalah `ok`. Ringkasan keseluruhan `test result: ok.` berarti bahwa semua uji coba berhasil, dan bagian yang membaca `1 passed; 0 failed` menjumlahkan jumlah uji coba yang berhasil atau gagal.

Mungkin untuk menandai uji coba sebagai diabaikan sehingga tidak berjalan dalam suatu instance tertentu; kita akan membahasnya dalam bagian [Mengabaikan Beberapa Uji Coba Kecuali Diminta Secara Khusus](#ignoring-some-tests-unless-specifically-requested) lebih lanjut dalam bab ini. Karena kita belum melakukannya di sini, ringkasan menunjukkan `0 ignored`. Kami juga dapat menyertakan argumen ke perintah `scarb cairo-test` untuk menjalankan hanya uji coba yang namanya cocok dengan string tertentu; ini disebut penyaringan dan kita akan membahasnya dalam bagian [Menjalankan Uji Coba Tunggal](#running-single-tests). Kami juga belum menyaring uji coba yang dijalankan, sehingga akhir ringkasan menunjukkan `0 filtered out`.

Mari mulai menyesuaikan uji coba sesuai kebutuhan kita. Pertama, ubah nama fungsi `it_works` menjadi nama yang berbeda, seperti `eksplorasi`, seperti ini:

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch09-testing-cairo-programs/listing_08_01_02/src/lib.cairo:eksplorasi}}
```

Kemudian jalankan `scarb cairo-test` lagi. Keluaran sekarang menunjukkan `eksplorasi` bukan `it_works`:

```shell
$ scarb cairo-test
running 1 tests
test adder::lib::tests::eksplorasi ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out;
```

Sekarang kita akan menambahkan uji coba lain, tapi kali ini kita akan membuat uji coba yang gagal! Uji coba gagal ketika ada yang panic dalam fungsi uji coba. Setiap uji coba dijalankan dalam thread baru, dan ketika thread utama melihat bahwa thread uji coba telah mati, uji coba ditandai sebagai gagal. Masukkan uji coba baru sebagai fungsi bernama `lainnya`, sehingga file _src/lib.cairo_ Anda terlihat seperti Listing 9-3.

```rust
{{#include ../listings/ch09-testing-cairo-programs/listing_08_03/src/lib.cairo:lainnya}}
```

<span class="caption">Listing 9-3: Menambahkan uji coba kedua yang akan gagal</span>

```shell
$ scarb cairo-test
running 2 tests
test adder::lib::tests::eksplorasi ... ok
test adder::lib::tests::lainnya ... fail
failures:
    adder::lib::tests::lainnya - panicked with [1725643816656041371866211894343434536761780588 ('Make this test fail'), ].
Error: test result: FAILED. 1 passed; 1 failed; 0 ignored
```

<span class="caption">Listing 9-4: Hasil uji coba ketika satu uji coba lulus dan satu uji coba gagal</span>

Daripada `ok`, baris `adder::lib::tests::lainnya` menunjukkan `fail`. Bagian baru muncul di antara hasil individual dan ringkasan. Ini menampilkan alasan rinci untuk setiap kegagalan uji coba. Dalam kasus ini, kita mendapatkan detail bahwa `lainnya` gagal karena panic dengan `[1725643816656041371866211894343434536761780588 ('Make this test fail'), ]` di file _src/lib.cairo_.

Baris ringkasan ditampilkan di akhir: secara keseluruhan, hasil uji coba kita adalah `FAILED`. Kami memiliki satu uji coba yang lulus dan satu uji coba yang gagal.

Sekarang setelah Anda melihat seperti apa hasil uji coba dalam skenario yang berbeda, mari kita lihat beberapa fungsi yang berguna dalam uji coba.

## Memeriksa Hasil dengan Fungsi assert

Fungsi `assert`, yang disediakan oleh Cairo, berguna ketika Anda ingin memastikan bahwa suatu kondisi dalam uji coba mengevaluasi menjadi `true`. Kami memberikan fungsi `assert` argumen pertama yang mengevaluasi menjadi Boolean. Jika nilai tersebut adalah `true`, tidak ada yang terjadi dan uji coba lulus. Jika nilai tersebut adalah `false`, fungsi assert memanggil `panic()` untuk menyebabkan uji coba gagal dengan pesan yang kita tentukan sebagai argumen kedua fungsi `assert`. Menggunakan fungsi `assert` membantu kita memeriksa bahwa kode kita berfungsi sesuai yang diinginkan.

Pada [Bab 5, Listing 5-15](ch05-03-method-syntax.md#multiple-impl-blocks), kita menggunakan struktur `Rectangle` dan metode `can_hold`, yang diulang di Listing 9-5. Mari letakkan kode ini di file _src/lib.cairo_, lalu tulis beberapa uji coba untuknya menggunakan fungsi `assert`.

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch09-testing-cairo-programs/listing_08_06/src/lib.cairo:trait_impl}}
```

<span class="caption">Listing 9-5: Menggunakan struktur `Rectangle` dan metode `can_hold` dari Bab 5</span>

Metode `can_hold` mengembalikan `bool`, yang berarti ini adalah kasus penggunaan yang sempurna untuk fungsi assert. Pada Listing 9-6, kita menulis uji coba yang menggunakan metode `can_hold` dengan membuat instans `Rectangle` yang memiliki lebar `8` dan tinggi `7` dan menegaskan bahwa itu dapat menahan instans `Rectangle` lain yang memiliki lebar `5` dan tinggi `1`.

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#rustdoc_include ../listings/ch09-testing-cairo-programs/listing_08_06/src/lib.cairo:test1}}
```

<span class="caption">Listing 9-6: Sebuah uji coba untuk `can_hold` yang memeriksa apakah suatu persegi panjang yang lebih besar dapat benar-benar menahan persegi panjang yang lebih kecil</span>

Perhatikan bahwa kita telah menambahkan dua baris baru di dalam modul uji coba: `use super::Rectangle;` dan `use super::RectangleTrait;`. Modul uji coba adalah modul biasa yang mengikuti aturan visibilitas biasa. Karena modul uji coba adalah modul dalam, kita perlu membawa kode yang diuji di modul luar ke dalam cakupan modul dalam.

Kami menamai uji coba kami `larger_can_hold_smaller`, dan kami telah membuat dua instans `Rectangle` yang kami butuhkan. Kemudian kami memanggil fungsi assert dan meneruskannya hasil dari pemanggilan `larger.can_hold(@smaller)`. Ekspresi ini seharusnya mengembalikan `true`, jadi uji coba kita seharusnya lulus. Mari kita cari tahu!

```shell
$ scarb cairo-test
running 1 tests
test adder::lib::tests::larger_can_hold_smaller ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out;
```

Itu berhasil! Mari tambahkan uji coba lain, kali ini menegaskan bahwa persegi panjang yang lebih kecil tidak dapat menahan persegi panjang yang lebih besar:

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#rustdoc_include ../listings/ch09-testing-cairo-programs/listing_08_06/src/lib.cairo:test2}}
```

Karena hasil yang benar dari fungsi `can_hold` dalam kasus ini adalah `false`, kita perlu membalik hasil tersebut sebelum kami meneruskannya ke fungsi assert. Akibatnya, uji coba kita akan lulus jika `can_hold` mengembalikan false:

```shell
$ scarb cairo-test
    running 2 tests
    test adder::lib::tests::smaller_cannot_hold_larger ... ok
    test adder::lib::tests::larger_can_hold_smaller ... ok
    test result: ok. 2 passed; 0 failed; 0 ignored; 0 filtered out;
```

Dua uji coba yang lulus! Sekarang mari lihat apa yang terjadi pada hasil uji coba kita ketika kita memperkenalkan bug dalam kode kita. Kami akan mengubah implementasi metode `can_hold` dengan mengganti tanda lebih dari dengan tanda kurang dari saat membandingkan lebar:

```rust
{{#include ../listings/ch09-testing-cairo-programs/no_listing_02_wrong_can_hold_impl/src/lib.cairo:wrong_impl}}
```

Menjalankan uji coba sekarang menghasilkan hal berikut:

```shell
$ scarb cairo-test
running 2 tests
test adder::lib::tests::smaller_cannot_hold_larger ... ok
test adder::lib::tests::larger_can_hold_smaller ... fail
failures:
   adder::lib::tests::larger_can_hold_smaller - panicked with [167190012635530104759003347567405866263038433127524 ('rectangle cannot hold'), ].

Error: test result: FAILED. 1 passed; 1 failed; 0 ignored
```

Uji coba kita menangkap bug tersebut! Karena `larger.width` adalah `8` dan `smaller.width` adalah `5`, perbandingan lebar dalam `can_hold` sekarang mengembalikan `false`: `8` tidak kurang dari `5`.

## Memeriksa Panic dengan `should_panic`

Selain memeriksa nilai kembalian, penting untuk memastikan bahwa kode kita menangani kondisi kesalahan sebagaimana yang diharapkan. Sebagai contoh, pertimbangkan tipe `Guess` pada Listing 9-8. Kode lain yang menggunakan `Guess` bergantung pada jaminan bahwa instance `Guess` hanya akan berisi nilai antara `1` dan `100`. Kita dapat menulis uji yang memastikan mencoba membuat instance `Guess` dengan nilai di luar rentang tersebut menyebabkan panic.

Ini dapat dilakukan dengan menambahkan atribut `should_panic` pada fungsi uji kita. Uji tersebut lulus jika kode di dalam fungsi menyebabkan panic; uji tersebut gagal jika kode di dalam fungsi tidak menyebabkan panic.

Listing 9-8 menunjukkan uji yang memeriksa bahwa kondisi kesalahan dari `GuessTrait::new` terjadi seperti yang diharapkan.

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch09-testing-cairo-programs/listing_08_08/src/lib.cairo}}
```

<span class="caption">Listing 9-8: Menguji bahwa suatu kondisi akan menyebabkan panic</span>

Kita menempatkan atribut `#[should_panic]` setelah atribut `#[test]` dan sebelum fungsi uji yang diterapkannya. Mari lihat hasilnya ketika uji ini berhasil:

```shell
$ scarb cairo-test
running 1 tests
test adder::lib::tests::greater_than_100 ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out;
```

Tampak bagus! Sekarang mari kita perkenalkan bug dalam kode kita dengan menghapus kondisi bahwa fungsi baru akan panic jika nilai lebih dari `100`:

```rust
{{#rustdoc_include ../listings/ch09-testing-cairo-programs/no_listing_03_wrong_new_impl/src/lib.cairo:here}}
```

Ketika kita menjalankan uji pada Listing 9-8, itu akan gagal:

```shell
$ scarb cairo-test
running 1 tests
test adder::lib::tests::greater_than_100 ... fail
failures:
   adder::lib::tests::greater_than_100 - expected panic but finished successfully.
Error: test result: FAILED. 0 passed; 1 failed; 0 ignored
```

Kita tidak mendapatkan pesan yang sangat membantu dalam kasus ini, tetapi ketika kita melihat fungsi uji, kita melihat bahwa itu di-annotasi dengan `#[should_panic]`. Kegagalan yang kita dapatkan berarti bahwa kode di dalam fungsi uji tidak menyebabkan panic.

Uji yang menggunakan `should_panic` dapat tidak akurat. Uji `should_panic` akan lulus bahkan jika uji menyebabkan panic karena alasan yang berbeda dari yang kita harapkan. Untuk membuat uji `should_panic` lebih tepat, kita dapat menambahkan parameter opsional yang diharapkan ke atribut `should_panic`. Harness uji akan memastikan bahwa pesan kegagalan berisi teks yang diberikan. Sebagai contoh, pertimbangkan kode yang dimodifikasi untuk `Guess` pada Listing 9-9 di mana fungsi baru menyebabkan panic dengan pesan yang berbeda tergantung pada apakah nilai terlalu kecil atau terlalu besar.

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#rustdoc_include ../listings/ch09-testing-cairo-programs/listing_08_09/src/lib.cairo:test_panic}}
```

<span class="caption">Listing 9-9: Menguji panic dengan pesan panic yang berisi string pesan kesalahan</span>

Uji ini akan lulus karena nilai yang kita masukkan pada parameter yang diharapkan atribut `should_panic` adalah array string dari pesan bahwa fungsi `Guess::new` panic. Kita perlu menentukan seluruh pesan panic yang kita harapkan.

Untuk melihat apa yang terjadi ketika uji `should_panic` dengan pesan yang diharapkan gagal, mari kita lagi memasukkan bug ke dalam kode kita dengan menukar tubuh blok if `value < 1` dan blok else if `value > 100`:

```rust
{{#include ../listings/ch09-testing-cairo-programs/no_listing_04_new_bug/src/lib.cairo:here}}
```

Kali ini ketika kita menjalankan uji `should_panic`, itu akan gagal:

```shell
$ scarb cairo-test
running 1 tests
test adder::lib::tests::greater_than_100 ... fail
failures:
   adder::lib::tests::greater_than_100 - panicked with [6224920189561486601619856539731839409791025 ('Guess must be >= 1'), ].

Error: test result: FAILED. 0 passed; 1 failed; 0 ignored
```

Pesan kegagalan menunjukkan bahwa uji ini memang panic sesuai yang kita harapkan, tetapi pesan panic tidak menyertakan string yang diharapkan. Pesan panic yang kita dapatkan dalam kasus ini adalah `Guess must be >= 1`. Sekarang kita dapat mulai mencari di mana letak bug kita!

## Menjalankan Uji Tunggal

Terkadang, menjalankan seluruh rangkaian uji bisa memakan waktu lama. Jika Anda sedang bekerja pada kode di area tertentu, Anda mungkin hanya ingin menjalankan uji yang berkaitan dengan kode tersebut. Anda dapat memilih uji mana yang akan dijalankan dengan melewatkan opsi `-f` (untuk "filter") kepada `scarb cairo-test`, diikuti dengan nama uji yang ingin dijalankan sebagai argumen.

Untuk mendemonstrasikan cara menjalankan uji tunggal, pertama-tama kita akan membuat dua fungsi uji, seperti yang ditunjukkan pada Listing 9-10, dan memilih mana yang ingin dijalankan.

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch09-testing-cairo-programs/listing_08_10/src/lib.cairo}}
```

<span class="caption">Listing 9-10: Dua uji dengan dua nama berbeda</span>

Kita dapat melewatkan nama fungsi uji apa pun kepada `cairo-test` untuk menjalankan hanya uji itu menggunakan opsi `-f`:

```shell
$ scarb cairo-test -f add_two_and_two
running 1 tests
test adder::lib::tests::add_two_and_two ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 1 filtered out;
```

Hanya uji dengan nama `add_two_and_two` yang dijalankan; uji lainnya tidak cocok dengan nama tersebut. Output uji memberi tahu kita bahwa ada satu uji lagi yang tidak dijalankan dengan menampilkan 1 filtered out di akhir.

Kita juga dapat menentukan sebagian dari nama uji, dan semua uji yang namanya mengandung nilai tersebut akan dijalankan.

## Mengabaikan Beberapa Uji Kecuali Diminta Secara Khusus

Terkadang beberapa uji tertentu dapat sangat memakan waktu untuk dieksekusi, sehingga Anda mungkin ingin mengeluarkannya selama sebagian besar jalankan `scarb cairo-test`. Alih-alih menyebutkan sebagai argumen semua uji yang ingin dijalankan, Anda dapat memberi tanda uji yang memakan waktu menggunakan atribut `ignore` untuk mengeluarkannya, seperti yang ditunjukkan di sini:

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch09-testing-cairo-programs/no_listing_05_ignore_tests/src/lib.cairo}}
```

Setelah `#[test]`, kita tambahkan baris `#[ignore]` pada uji yang ingin kita eksklusikan. Sekarang ketika kita menjalankan uji kita, `it_works` berjalan, tetapi `expensive_test` tidak:

```shell
$ scarb cairo-test
running 2 tests
test adder::lib::tests::expensive_test ... ignored
test adder::lib::tests::it_works ... ok
test result: ok. 1 passed; 0 failed; 1 ignored; 0 filtered out;
```

Fungsi `expensive_test` dicatat sebagai diabaikan.

Ketika Anda sudah pada titik di mana masuk akal untuk memeriksa hasil uji yang diabaikan dan Anda memiliki waktu untuk menunggu hasilnya, Anda dapat menjalankan `scarb cairo-test --include-ignored` untuk menjalankan semua uji apakah diabaikan atau tidak.

## Menguji fungsi rekursif atau perulangan

Saat menguji fungsi rekursif atau perulangan, Anda harus memberikan jumlah gas maksimum yang dapat dikonsumsi oleh uji. Ini mencegah terjadinya perulangan tak terbatas atau penggunaan gas yang terlalu banyak, dan dapat membantu Anda melakukan pengukuran efisiensi implementasi Anda. Untuk melakukannya, Anda harus menambahkan atribut `#[available_gas(<Number>)]` pada fungsi uji. Contoh berikut menunjukkan cara menggunakannya:

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch09-testing-cairo-programs/no_listing_08_test_gas/src/lib.cairo}}
```

## Benchmark penggunaan gas dari suatu operasi tertentu

Saat Anda ingin melakukan benchmark penggunaan gas dari suatu operasi tertentu, Anda dapat menggunakan pola berikut dalam fungsi uji Anda.

```rust
let initial = testing::get_available_gas();
gas::withdraw_gas().unwrap();
    /// kode yang ingin kita benchmark.
(testing::get_available_gas() - x).print();
```

Contoh berikut menunjukkan cara menggunakannya untuk menguji fungsi gas dari fungsi `sum_n` di atas.

```rust
{{#include ../listings/ch09-testing-cairo-programs/no_listing_09_benchmark_gas/src/lib.cairo}}
```

Nilai yang dicetak ketika menjalankan `scarb cairo-test` adalah jumlah gas yang dikonsumsi oleh operasi yang di-benchmark.

```shell
$ scarb cairo-test
testing no_listing_09_benchmark_gas ...
running 1 tests
[DEBUG]	                               	(raw: 0x179f8

test no_listing_09_benchmark_gas::benchmark_sum_n_gas ... ok (gas usage est.: 98030)
test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out;

```

Di sini, penggunaan gas dari fungsi `sum_n` adalah 96760 (representasi desimal dari angka heksadesimal). Jumlah total yang dikonsumsi oleh uji sedikit lebih tinggi pada 98030, karena beberapa langkah ekstra diperlukan untuk menjalankan seluruh fungsi uji.
