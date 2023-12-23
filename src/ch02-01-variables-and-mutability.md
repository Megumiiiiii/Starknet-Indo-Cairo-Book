## Variabel dan Mutabilitas

Cairo menggunakan model memori yang tidak dapat diubah (immutable), yang berarti bahwa setelah sebuah sel memori ditulis, sel tersebut tidak dapat ditimpa (overwrite) tetapi hanya dapat dibaca. Untuk mencerminkan model memori yang tidak dapat diubah ini, variabel dalam Cairo secara default tidak dapat diubah (immutable).
Namun, bahasa ini menyederhanakan model ini dan memberikan opsi untuk membuat variabel Anda dapat diubah (mutable). Mari kita jelajahi bagaimana dan mengapa Cairo menerapkan ketidakmutabilan (immutability), dan bagaimana Anda dapat membuat variabel Anda menjadi mutable.

Ketika sebuah variabel tidak dapat diubah, setelah sebuah variabel terikat pada sebuah nilai, Anda tidak dapat mengubah variabel tersebut. Untuk mengilustrasikannya, buat proyek baru bernama _variables_ dalam direktori _cairo_projects_ Anda dengan menggunakan `scarb new variables`.

Kemudian, di direktori _variables_ baru Anda, buka _src/lib.cairo_ yang baru dan gantikan kode di dalamnya dengan kode berikut, yang belum akan berhasil dikompilasi:

```rust,does_not_compile
{{#include ../listings/ch02-common-programming-concepts/no_listing_01_variables_are_immutable/src/lib.cairo}}

```

Simpan dan jalankan program menggunakan `scarb cairo-run --available-gas=200000000`. Anda seharusnya menerima pesan kesalahan yang berkaitan dengan kesalahan ketidakmutabilan, seperti yang ditunjukkan dalam keluaran berikut:

```shell
error: Cannot assign to an immutable variable.
 --> lib.cairo:5:5
    x = 6;
    ^***^

Error: failed to compile: src/lib.cairo
```

Contoh ini menunjukkan bagaimana kompiler membantu Anda menemukan kesalahan dalam program Anda. Kesalahan kompiler dapat menjengkelkan, tetapi sebenarnya itu hanya berarti bahwa program Anda belum melakukan dengan aman apa yang ingin Anda lakukan; itu _bukan_ berarti bahwa Anda bukan seorang programmer yang baik! Para Caironautes berpengalaman tetap mendapatkan kesalahan kompiler.

Anda menerima pesan kesalahan `Cannot assign to an immutable variable.` karena Anda mencoba untuk memberikan nilai kedua ke variabel `x` yang tidak dapat diubah.

Penting bagi kita untuk mendapatkan kesalahan pada waktu kompilasi ketika kita mencoba untuk mengubah nilai yang ditetapkan sebagai tidak dapat diubah karena situasi spesifik ini dapat menyebabkan bug. Jika salah satu bagian kode kita beroperasi dengan asumsi bahwa sebuah nilai tidak akan pernah berubah dan bagian lain dari kode kita mengubah nilai tersebut, mungkin bagian pertama kode tersebut tidak akan melakukan apa yang dirancang untuk dilakukan. Penyebab bug semacam ini dapat sulit dilacak setelahnya, terutama ketika bagian kedua kode tersebut hanya mengubah nilai itu _kadang-kadang_.

Cairo, berbeda dengan kebanyakan bahasa lain, memiliki memori yang tidak dapat diubah. Hal ini membuat sejumlah bug menjadi tidak mungkin, karena nilai tidak akan berubah secara tak terduga. Hal ini membuat kode menjadi lebih mudah untuk dianalisis.

Namun, mutabilitas dapat sangat berguna, dan dapat membuat kode menjadi lebih nyaman untuk ditulis. Meskipun variabel secara default tidak dapat diubah, Anda dapat membuatnya menjadi mutable dengan menambahkan `mut` di depan nama variabel. Menambahkan `mut` juga menyampaikan niat kepada pembaca kode di masa depan dengan menunjukkan bahwa bagian lain dari kode akan mengubah nilai yang terkait dengan variabel ini.

<!-- TODO: tambahkan ilustrasi mengenai ini -->

Namun, pada titik ini mungkin Anda bertanya-tanya apa yang sebenarnya terjadi ketika sebuah variabel dinyatakan sebagai `mut`, karena sebelumnya kami menyebutkan bahwa memori Cairo tidak dapat diubah. Jawabannya adalah _nilai_ tidak dapat diubah, tetapi _variabel_-nya dapat. Nilai yang ditunjuk oleh variabel dapat diubah. Menugaskan ke variabel yang mutable dalam Cairo pada dasarnya setara dengan mendeklarasikan ulang variabel untuk merujuk pada nilai lain di sel memori lain, tetapi kompiler menangani hal tersebut untuk Anda, dan kata kunci `mut` membuatnya eksplisit. Setelah memeriksa kode Assembly Cairo tingkat rendah, menjadi jelas bahwa mutasi variabel diimplementasikan sebagai gula sintaksis, yang menerjemahkan operasi mutasi menjadi serangkaian langkah yang setara dengan variable shadowing. Satu-satunya perbedaan adalah bahwa pada level Cairo, variabel tidak dideklarasikan ulang sehingga tipe variabel tidak dapat berubah.

Sebagai contoh, mari ubah _src/lib.cairo_ menjadi seperti berikut:

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_02_adding_mut/src/lib.cairo}}
```

Ketika kita menjalankan program sekarang, kita akan mendapatkan keluaran ini:

```shell
$ scarb cairo-run --available-gas=200000000
[DEBUG]                                (raw: 5)

[DEBUG]                                (raw: 6)

Run completed successfully, returning []
```

Kita diizinkan untuk mengubah nilai yang terikat pada `x` dari `5` menjadi `6` ketika `mut` digunakan. Pada akhirnya, keputusan untuk menggunakan mutabilitas atau tidak tergantung pada Anda dan tergantung pada apa yang menurut Anda paling jelas dalam situasi tertentu tersebut.

### Konstan

Seperti variabel yang tidak dapat diubah, _konstan_ adalah nilai yang terikat pada sebuah nama dan tidak diizinkan untuk berubah, tetapi ada beberapa perbedaan antara konstan dan variabel.

Pertama, Anda tidak diizinkan untuk menggunakan `mut` dengan konstan. Konstan tidak hanya tidak dapat diubah secara default—mereka selalu tidak dapat diubah. Anda mendeklarasikan konstan menggunakan kata kunci `const` alih-alih kata kunci `let`, dan tipe nilai _harus_ diannotasikan. Kita akan membahas tentang tipe dan annotasi tipe pada bagian selanjutnya, [“Tipe Data”][data-types], jadi jangan khawatir tentang detail-detailnya sekarang. Yang penting, Anda harus selalu menambahkan annotasi tipe.

Konstan hanya dapat dideklarasikan pada scope global, yang membuatnya berguna untuk nilai-nilai yang banyak bagian kode perlu mengetahuinya.

Perbedaan terakhir adalah bahwa konstan hanya dapat diatur ke ekspresi konstan, bukan hasil dari nilai yang hanya dapat dihitung saat runtime. Hanya konstan literal yang didukung saat ini.

Berikut contoh deklarasi konstan:

```rust, noplayground
const SATU_JAM_DALAM_DETIK: u32 = 3600;
```

Konvensi penamaan Cairo untuk konstan adalah dengan menggunakan huruf besar semua dengan garis bawah di antara kata-kata.

Konstan valid selama program berjalan, dalam scope di mana mereka dideklarasikan. Properti ini membuat konstan berguna untuk nilai-nilai dalam domain aplikasi Anda yang mungkin dibutuhkan oleh banyak bagian program, seperti jumlah maksimum poin yang dapat diperoleh oleh pemain dalam sebuah permainan, atau kecepatan cahaya.

Memberi nama nilai-nilai yang dikodekan secara kaku yang digunakan di seluruh program Anda sebagai konstan berguna dalam menyampaikan makna dari nilai tersebut kepada para pemelihara kode di masa depan. Ini juga membantu jika hanya ada satu tempat dalam kode Anda yang perlu Anda ubah jika nilai yang dikodekan secara kaku tersebut perlu diperbarui di masa depan.

### Shadowing

Variable shadowing mengacu pada deklarasi variabel baru dengan nama yang sama seperti variabel sebelumnya. Caironautes mengatakan bahwa variabel pertama _disandingkan_ (shadowed) oleh yang kedua, yang berarti bahwa variabel kedua yang akan dilihat oleh kompiler ketika Anda menggunakan nama variabel tersebut. Pada dasarnya, variabel kedua menutupi (overshadow) yang pertama, mengambil penggunaan nama variabel itu sendiri sampai entah itu sendiri yang disandingkan atau scope berakhir. Kita dapat menutupi variabel dengan menggunakan nama variabel yang sama dan mengulangi penggunaan kata kunci `let` sebagai berikut:

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_03_shadowing/src/lib.cairo}}
```

Program ini pertama kali mengikat `x` pada nilai `5`. Kemudian, ia membuat variabel baru `x` dengan mengulangi `let x =`, mengambil nilai asli dan menambahkan `1` sehingga nilai `x` kemudian menjadi `6`. Kemudian, dalam scope dalam dengan menggunakan kurung kurawal, pernyataan `let` ketiga juga menutupi `x` dan membuat variabel baru, mengalikan nilai sebelumnya dengan `2` sehingga memberikan nilai `x` menjadi `12`. Ketika scope itu selesai, penutupan dalam berakhir dan `x` kembali menjadi `6`. Ketika kita menjalankan program ini, akan mengeluarkan keluaran berikut:

```shell
scarb cairo-run --available-gas=200000000
[DEBUG] Nilai x pada scope dalam adalah:    (raw: 7033328135641142205392067879065573688897582790068499258)

[DEBUG]
                                       (raw: 12)

[DEBUG] Nilai x pada scope luar adalah:    (raw: 7610641743409771490723378239576163509623951327599620922)

[DEBUG]                                (raw: 6)

Run completed successfully, returning []
```

Shadowing berbeda dengan menandai variabel sebagai `mut` karena kita akan mendapatkan kesalahan waktu kompilasi jika secara tidak sengaja kita mencoba untuk melakukan penugasan ulang pada variabel ini tanpa menggunakan kata kunci `let`. Dengan menggunakan `let`, kita dapat melakukan beberapa transformasi pada sebuah nilai tetapi variabelnya tetap tidak dapat diubah setelah transformasi tersebut selesai.

Perbedaan lain antara `mut` dan shadowing adalah bahwa ketika kita menggunakan kata kunci `let` lagi, kita pada dasarnya membuat variabel baru, yang memungkinkan kita untuk mengubah tipe nilai sambil menggunakan kembali nama yang sama. Seperti yang disebutkan sebelumnya, variabel shadowing dan variabel yang dapat diubah (mutable variables) setara pada level yang lebih rendah.
Perbedaan utamanya adalah bahwa dengan menutupi (shadowing) sebuah variabel, kompiler tidak akan mengeluh jika Anda mengubah tipe variabel tersebut. Sebagai contoh, katakanlah program kami melakukan konversi tipe antara tipe `u64` dan `felt252`.

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_04_shadowing_different_type/src/lib.cairo}}
```

Variabel `x` pertama memiliki tipe `u64` sementara variabel `x` kedua memiliki tipe `felt252`. Shadowing ini menghemat kita dari harus membuat nama yang berbeda, seperti `x_u64` dan `x_felt252`; sebaliknya, kita dapat menggunakan kembali nama yang lebih sederhana, yaitu `x`. Namun, jika kita mencoba menggunakan `mut` untuk ini, seperti yang ditunjukkan di sini, kita akan mendapatkan kesalahan waktu kompilasi:

```rust,does_not_compile
{{#include ../listings/ch02-common-programming-concepts/no_listing_05_mut_cant_change_type/src/lib.cairo}}
```

Kesalahan tersebut mengatakan kita mengharapkan sebuah `u64` (tipe asli) tetapi kita mendapatkan tipe yang berbeda:

```shell
$ scarb cairo-run --available-gas=200000000
error: Unexpected argument type. Expected: "core::integer::u64", found: "core::felt252".
 --> lib.cairo:9:9
    x = 100_felt252;
        ^*********^

Error: failed to compile: src/lib.cairo
```

Sekarang setelah kita telah menjelajahi bagaimana variabel bekerja, mari kita lihat lebih banyak jenis data yang dapat mereka miliki.

[data-types]: ch02-02-data-types.md