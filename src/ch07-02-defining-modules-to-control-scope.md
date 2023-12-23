## Mendefinisikan Modul untuk Mengendalikan Cakupan

Pada bagian ini, kita akan membicarakan tentang modul dan bagian-bagian lain dari sistem modul, yaitu _paths_ yang memungkinkan Anda untuk memberi nama pada item-item dan kata kunci `use` yang membawa sebuah path ke dalam cakupan.

Pertama, kita akan memulai dengan daftar aturan sebagai referensi mudah saat Anda mengorganisir kode Anda di masa mendatang. Kemudian, kita akan menjelaskan setiap aturan tersebut secara rinci.

### Cheet Sheet Modul

Di sini, kami memberikan referensi cepat tentang bagaimana modul, paths, dan kata kunci `use` bekerja dalam kompiler, serta bagaimana sebagian besar pengembang mengorganisir kode mereka. Kami akan melalui contoh-contoh dari setiap aturan ini sepanjang bab ini, namun ini adalah tempat yang bagus untuk merujuk sebagai pengingat tentang cara kerja modul. Anda dapat membuat proyek Scarb baru dengan `scarb new backyard` untuk mengikuti penjelasan ini.

- **Mulai dari crate root**: Saat mengompilasi sebuah crate, kompiler pertama-tama melihat di file crate root (_src/lib.cairo_) untuk kode yang akan dikompilasi.
- **Deklarasi modul**: Di file crate root, Anda dapat mendeklarasikan modul-modul baru; misalnya, Anda mendeklarasikan modul "garden" dengan `mod garden;`. Kompiler akan mencari kode modul tersebut di tempat-tempat berikut:

  - Secara inline, dalam kurung kurawal yang menggantikan titik koma setelah `mod garden;`.

    ```rust,noplayground
      // file crate root (src/lib.cairo)
        mod garden {
        // kode yang mendefinisikan modul garden ada di sini
        }
    ```

  - Di dalam file _src/garden.cairo_

- **Deklarasi submodul**: Di file selain crate root, Anda dapat mendeklarasikan submodul. Sebagai contoh, Anda mungkin mendeklarasikan `mod vegetables;` di _src/garden.cairo_. Kompiler akan mencari kode submodul tersebut di dalam direktori yang dinamai sesuai dengan modul induk pada tempat-tempat berikut:

  - Secara inline, langsung mengikuti `mod vegetables`, di dalam kurung kurawal sebagai pengganti titik koma.

    ```rust,noplayground
    // file src/garden.cairo
    mod vegetables {
        // kode yang mendefinisikan submodul vegetables ada di sini
    }
    ```

  - Di dalam file _src/garden/vegetables.cairo_

- **Paths menuju kode dalam modul**: Setelah sebuah modul menjadi bagian dari crate Anda, Anda dapat merujuk pada kode dalam modul tersebut dari mana pun di dalam crate yang sama, menggunakan path ke kode tersebut. Sebagai contoh, sebuah tipe `Asparagus` dalam modul sayuran taman akan ditemukan pada `backyard::garden::vegetables::Asparagus`.
- **Kata kunci `use`**: Dalam sebuah cakupan, kata kunci `use` menciptakan pintasan ke item untuk mengurangi pengulangan dari path yang panjang. Di dalam cakupan yang dapat merujuk ke `backyard::garden::vegetables::Asparagus`, Anda dapat membuat pintasan dengan `use backyard::garden::vegetables::Asparagus;` dan setelah itu Anda hanya perlu menulis `Asparagus` untuk menggunakan tipe tersebut dalam cakupan tersebut.

Berikut ini kami membuat sebuah kardus yang diberi nama `backyard` yang menggambarkan aturan-aturan ini. Direktori kardus, juga bernama `backyard`, berisi file dan direktori-direktori berikut:

```text
backyard/
├── Scarb.toml
└── src
    ├── garden
    │   └── vegetables.cairo
    ├── garden.cairo
    └── lib.cairo
```

File root kardus dalam hal ini adalah _src/lib.cairo_, dan berisi:

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/no_listing_01_lib/src/lib.cairo}}
```

Baris `mod garden;` memberi tahu kompiler untuk menyertakan kode yang ditemukan di _src/garden.cairo_, yang berisi:

<span class="filename">Nama File: src/garden.cairo</span>

```rust,noplayground
mod vegetables;
```

Di sini, `mod vegetables;` berarti kode di _src/garden/vegetables.cairo_ juga disertakan. Kode itu adalah:

```rust,noplayground
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/no_listing_02_garden/src/lib.cairo}}
```

Baris `use garden::vegetables::Asparagus;` memungkinkan kita membawa tipe `Asparagus` ke dalam lingkup, sehingga kita dapat menggunakannya di dalam fungsi `main`.

Sekarang mari kita masuk ke dalam rincian aturan-aturan ini dan tunjukkan mereka dalam tindakan!

### Mengelompokkan Kode yang Berkaitan dalam Modul

_Modul_ memungkinkan kita mengorganisir kode dalam sebuah kardus untuk keterbacaan dan penggunaan ulang yang mudah. Sebagai contoh, mari kita tulis kardus pustaka yang menyediakan fungsionalitas sebuah restoran. Kita akan menentukan tanda tangan fungsi tetapi meninggalkan tubuhnya kosong untuk fokus pada organisasi kode, bukan implementasi restoran.

Dalam industri restoran, beberapa bagian dari restoran disebut sebagai _front of house_ dan yang lain sebagai _back of house_. Front of house adalah tempat pelanggan berada; ini mencakup tempat tuan rumah duduk, pelayan mengambil pesanan dan pembayaran, dan barman membuat minuman. Back of house adalah tempat koki dan karyawan dapur bekerja, pencuci piring membersihkan, dan manajer melakukan pekerjaan administratif.

Untuk mengorganisir kardus kita dengan cara ini, kita dapat mengatur fungsinya ke dalam modul-modul bertingkat. Buat paket baru dengan nama `restaurant` dengan menjalankan `scarb new restaurant`; kemudian masukkan kode pada Listing 7-1 ke dalam _src/lib.cairo_ untuk mendefinisikan beberapa modul dan tanda tangan fungsi. Berikut adalah bagian front of house:

<span class="filename">Nama File: src/lib.cairo</span>

```rust,noplayground
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_01/src/lib.cairo}}
```

<span class="caption">Listing 7-1: Modul `front_of_house` yang berisi modul-modul lain yang kemudian berisi fungsi-fungsi</span>

Kita mendefinisikan modul dengan kata kunci `mod` diikuti oleh nama modul
(dalam hal ini, `front_of_house`). Tubuh modul kemudian ditempatkan di dalam kurung kurawal. Di dalam modul, kita dapat menempatkan modul-modul lain, seperti dalam hal ini dengan modul-modul `hosting` dan `serving`. Modul juga dapat berisi definisi untuk item lain, seperti struktur, enumerasi, konstan, trait, dan—seperti pada Listing
6-1—fungsi.

Dengan menggunakan modul, kita dapat mengelompokkan definisi yang berkaitan bersama dan memberi nama mengapa
mereka berkaitan. Programer yang menggunakan kode ini dapat menavigasi kode berdasarkan
kelompok daripada harus membaca semua definisi, membuatnya lebih mudah
untuk menemukan definisi yang relevan bagi mereka. Programer yang menambahkan fungsionalitas baru
ke dalam kode ini akan tahu di mana menempatkan kode untuk menjaga agar program terorganisir.

Sebelumnya, kami menyebutkan bahwa _src/lib.cairo_ disebut sebagai root kardus.
Nama ini diberikan karena isi file ini membentuk modul yang dinamai sesuai nama kardus di root struktur modul kardus tersebut,
dikenal sebagai _pohon modul_.

Listing 7-2 menunjukkan pohon modul untuk struktur pada Listing 7-1.

```text
restaurant
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

<span class="caption">Listing 7-2: Pohon modul untuk kode pada Listing
6-1</span>

Pohon ini menunjukkan bagaimana beberapa modul bersarang di dalam yang lain; misalnya,
`hosting` bersarang di dalam `front_of_house`. Pohon ini juga menunjukkan bahwa beberapa modul
adalah _sibling_ satu sama lain, yang berarti mereka didefinisikan di dalam modul yang sama;
`hosting` dan `serving` adalah sibling yang didefinisikan di dalam `front_of_house`. Jika modul
A berada di dalam modul B, kita katakan bahwa modul A adalah _child_ dari modul B
dan bahwa modul B adalah _parent_ dari modul A. Perhatikan bahwa seluruh pohon modul berakar di bawah nama eksplisit kardus `restaurant`.

Pohon modul mungkin mengingatkan Anda pada pohon direktori sistem file di komputer Anda;
ini adalah perbandingan yang sangat tepat! Sama seperti direktori dalam sistem file,
Anda menggunakan modul untuk mengorganisir kode Anda. Dan sama seperti file dalam sebuah direktori,
kita membutuhkan cara untuk menemukan modul-modul kita.