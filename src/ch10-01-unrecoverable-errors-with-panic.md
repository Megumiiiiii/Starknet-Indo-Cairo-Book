# Kesalahan yang Tidak Dapat Dipulihkan dengan panic

Di Cairo, masalah yang tidak terduga dapat muncul selama eksekusi program, menghasilkan kesalahan waktu eksekusi. Meskipun fungsi panic dari pustaka inti tidak memberikan penyelesaian untuk kesalahan-kesalahan ini, namun ia mengakui keberadaan mereka dan mengakhiri program. Ada dua cara utama yang dapat memicu panic di Cairo: secara tidak sengaja, melalui tindakan yang menyebabkan kode menjadi panic (misalnya, mengakses array melebihi batasnya), atau sengaja, dengan memanggil fungsi panic.

Ketika panic terjadi, ini menyebabkan penghentian mendadak dari program. Fungsi `panic` mengambil sebuah array sebagai argumen, yang dapat digunakan untuk memberikan pesan kesalahan dan melakukan proses unwind di mana semua variabel di-drop dan kamus-kamus diremas untuk memastikan keamanan program dalam menghentikan eksekusi dengan aman.

Berikut adalah contoh bagaimana kita dapat menggunakan `panic` dari dalam sebuah program dan mengembalikan kode kesalahan `2`:

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch10-error-handling/no_listing_01_panic/src/lib.cairo}}
```

Menjalankan program akan menghasilkan output berikut:

```shell
$ scarb cairo-run --available-gas=200000000
Program berhenti secara mendadak dengan [2 (''), ].
```

Seperti yang dapat Anda perhatikan dalam output, pernyataan cetak tidak pernah tercapai, karena program menghentikan eksekusi setelah menemui pernyataan `panic`.

Pendekatan alternatif dan lebih idiomatik untuk panic di Cairo adalah menggunakan fungsi `panic_with_felt252`. Fungsi ini berfungsi sebagai abstraksi dari proses mendefinisikan array dan sering lebih disukai karena ungkapan niatnya yang lebih jelas dan ringkas. Dengan menggunakan `panic_with_felt252`, pengembang dapat memicu panic dalam satu baris dengan memberikan pesan kesalahan felt252 sebagai argumen, membuat kode lebih mudah dibaca dan mudah dipelihara.

Mari kita pertimbangkan sebuah contoh:

```rust
{{#include ../listings/ch10-error-handling/no_listing_02_with_felt252/src/lib.cairo}}
```

Menjalankan program ini akan menghasilkan pesan kesalahan yang sama seperti sebelumnya. Dalam hal ini, jika tidak ada kebutuhan akan sebuah array dan beberapa nilai yang harus dikembalikan dalam kesalahan, maka `panic_with_felt252` menjadi alternatif yang lebih ringkas.

## Notasi nopanic

Anda dapat menggunakan notasi `nopanic` untuk menunjukkan bahwa sebuah fungsi tidak akan pernah panic. Hanya fungsi-fungsi `nopanic` yang dapat dipanggil dalam fungsi yang dianotasi sebagai `nopanic`.

Contoh:

```rust,noplayground
{{#include ../listings/ch10-error-handling/no_listing_03_nopanic/src/lib.cairo}}
```

Contoh yang salah:

```rust,noplayground
{{#include ../listings/ch10-error-handling/no_listing_04_nopanic_wrong/src/lib.cairo}}
```

Jika Anda menulis fungsi berikut yang mencakup pemanggilan fungsi yang mungkin panic, Anda akan mendapatkan pesan kesalahan berikut:

```shell
error: Fungsi dideklarasikan sebagai nopanic tetapi memanggil fungsi yang mungkin panic.
 --> test.cairo:2:12
    assert(1 == 1, 'what');
           ^****^
Fungsi dideklarasikan sebagai nopanic tetapi memanggil fungsi yang mungkin panic.
 --> test.cairo:2:5
    assert(1 == 1, 'what');
    ^********************^
```

Perhatikan bahwa ada dua fungsi yang mungkin panic di sini, yaitu assert dan equality.

## Atribut panic_with

Anda dapat menggunakan atribut `panic_with` untuk menandai sebuah fungsi yang mengembalikan `Option` atau `Result`. Atribut ini mengambil dua argumen, yaitu data yang dilewatkan sebagai alasan panic serta nama untuk sebuah fungsi pembungkus. Ini akan membuat sebuah pembungkus untuk fungsi yang Anda anotasi yang akan panic jika fungsi mengembalikan `None` atau `Err`, fungsi panic akan dipanggil dengan data yang diberikan.

Contoh:

```rust
{{#include ../listings/ch10-error-handling/no_listing_05_panic_with/src/lib.cairo}}
```

## Menggunakan assert

Fungsi assert dari pustaka inti Cairo sebenarnya adalah fungsi utilitas berdasarkan panic. Ia memastikan bahwa suatu ekspresi boolean benar pada waktu eksekusi, dan jika tidak benar, ia memanggil fungsi panic dengan nilai kesalahan. Fungsi assert mengambil dua argumen: ekspresi boolean yang akan diverifikasi, dan nilai kesalahan. Nilai kesalahan ditentukan sebagai felt252, sehingga string yang dilewatkan harus dapat muat di dalam felt252.

Berikut adalah contoh penggunaannya:

```rust
{{#include ../listings/ch10-error-handling/no_listing_06_assert/src/lib.cairo}}
```

Kami melakukan assert di dalam fungsi utama bahwa `my_number` bukan nol untuk memastikan bahwa kita tidak melakukan pembagian dengan 0. 
Pada contoh ini, `my_number` adalah nol sehingga asertion akan gagal, dan program akan panic
dengan string 'number is zero' (sebagai felt252) dan pembagian tidak akan tercapai.
