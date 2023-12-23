## Path untuk Merujuk ke Item dalam Pohon Modul

Untuk menunjukkan kepada Cairo di mana menemukan suatu item dalam pohon modul, kita menggunakan path dengan cara yang sama seperti kita menggunakan path saat menjelajahi sistem file. Untuk memanggil sebuah fungsi, kita perlu tahu path-nya.

Sebuah path dapat memiliki dua bentuk:

- Sebuah _absolute path_ adalah path lengkap yang dimulai dari root kardus. Absolute path dimulai dengan nama kardus.
- Sebuah _relative path_ dimulai dari modul saat ini.

  Keduanya, absolute dan relative paths diikuti oleh satu atau lebih identifikasi
  yang dipisahkan oleh titik dua ganda (`::`).

Untuk mengilustrasikan konsep ini, mari kita ambil contoh Listing 7-1 untuk restoran yang kita gunakan dalam bab terakhir. Kita memiliki sebuah kardus bernama `restaurant` di dalamnya terdapat modul bernama `front_of_house` yang berisi modul bernama `hosting`. Modul `hosting` berisi fungsi bernama `add_to_waitlist`. Kita ingin memanggil fungsi `add_to_waitlist` dari fungsi `eat_at_restaurant`. Kita perlu memberi tahu Cairo path ke fungsi `add_to_waitlist` agar bisa menemukannya.

<span class="filename">Nama File: src/lib.cairo</span>

```rust,noplayground
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_03/src/lib.cairo}}
```

<span class="caption">Listing 7-3: Memanggil fungsi `add_to_waitlist` menggunakan absolute dan relative paths</span>

Pertama kali kita memanggil fungsi `add_to_waitlist` di dalam `eat_at_restaurant`,
kita menggunakan absolute path. Fungsi `add_to_waitlist` didefinisikan dalam kardus yang sama dengan `eat_at_restaurant`. Dalam Cairo, absolute paths dimulai dari root kardus, yang perlu Anda sebutkan dengan menggunakan nama kardus.

Kedua kalinya kita memanggil `add_to_waitlist`, kita menggunakan relative path. Path dimulai dengan `front_of_house`, nama modul
yang didefinisikan pada tingkat yang sama dalam pohon modul seperti `eat_at_restaurant`. Di sini, ekuivalen sistem file akan menggunakan path
`./front_of_house/hosting/add_to_waitlist`. Memulai dengan nama modul berarti
bahwa path ini bersifat relatif terhadap modul saat ini.

### Memulai Relative Paths dengan `super`

Memilih apakah akan menggunakan `super` atau tidak adalah keputusan yang akan Anda ambil
berdasarkan proyek Anda, dan tergantung pada apakah Anda lebih mungkin memindahkan definisi item
terpisah dari atau bersamaan dengan kode yang menggunakan item tersebut.

<span class="filename">Nama File: src/lib.cairo</span>

```rust,noplayground
{{#include ../listings/ch07-managing-cairo-projects-with-packages-crates-and-modules/listing_06_04/src/lib.cairo}}
```

<span class="caption">Listing 7-4: Memanggil fungsi menggunakan relative path yang dimulai dengan super</span>

Di sini Anda dapat melihat secara langsung bahwa Anda dapat mengakses modul induk dengan mudah menggunakan `super`, yang tidak mungkin sebelumnya.