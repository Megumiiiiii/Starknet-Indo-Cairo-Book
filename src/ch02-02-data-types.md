## Jenis Data

Setiap nilai dalam Cairo memiliki _jenis data_ tertentu, yang memberi tahu Cairo jenis data apa yang di spesifikasikan sehingga Cairo tahu bagaimana cara bekerja dengan data tersebut. Bagian ini mencakup dua subset dari jenis data: skalar dan gabungan.

Ingatlah bahwa Cairo adalah bahasa _berjenis statis_, yang berarti bahwa ia harus mengetahui jenis dari semua variabel pada waktu kompilasi. Compiler biasanya dapat menyimpulkan jenis yang diinginkan berdasarkan nilai dan penggunaannya. Dalam kasus di mana banyak jenis mungkin, kita dapat menggunakan metode cast di mana kita menentukan jenis keluaran yang diinginkan.

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_06_data_types/src/lib.cairo}}
```

Anda akan melihat anotasi tipe yang berbeda untuk jenis data lainnya.

### Jenis Scalar

Jenis scalar mewakili sebuah nilai tunggal. Cairo memiliki tiga jenis skalar utama: felts, integers, dan booleans. Anda mungkin mengenali ini dari bahasa pemrograman lain. Mari kita bahas bagaimana cara kerjanya di Cairo.

#### Jenis Felt

Di Cairo, jika Anda tidak menentukan jenis variabel atau argumen, tipe defaultnya adalah elemen lapangan, yang direpresentasikan oleh kata kunci `felt252`. Dalam konteks Cairo, ketika kita mengatakan "sebuah elemen lapangan", kita maksudkan sebuah bilangan bulat dalam rentang `0 <= x < P`, di mana `P` adalah bilangan prima yang sangat besar saat ini sama dengan `P = 2^{251} + 17 * 2^{192}+1`. Ketika menambah, mengurangi, atau mengalikan, jika hasilnya jatuh di luar rentang yang ditentukan oleh bilangan prima, maka akan terjadi overflow, dan kelipatan P yang sesuai akan ditambahkan atau dikurangkan untuk membawa hasil kembali ke dalam rentang (artinya, hasilnya dihitung modulo P).

Perbedaan terpenting antara bilangan bulat dan elemen lapangan adalah pada pembagian: Pembagian elemen lapangan (dan oleh karena itu pembagian di Cairo) tidak seperti pembagian CPU reguler, di mana pembagian bilangan bulat `x / y` didefinisikan sebagai `[x/y]` di mana bagian bilangan bulat dari hasil bagi dikembalikan (sehingga Anda mendapatkan `7 / 3 = 2`) dan mungkin atau mungkin tidak memenuhi persamaan `(x / y) * y == x`, tergantung pada kelipatan `x` oleh `y`.

Di Cairo, hasil dari `x/y` didefinisikan untuk selalu memenuhi persamaan `(x / y) * y == x`. Jika y membagi x sebagai bilangan bulat, Anda akan mendapatkan hasil yang diharapkan di Cairo (misalnya `6 / 2` akan menghasilkan `3`). Tetapi ketika y tidak membagi x, Anda mungkin mendapatkan hasil yang mengejutkan: Sebagai contoh, karena `2 * ((P+1)/2) = P+1 ≡ 1 mod[P]`, nilai dari `1 / 2` di Cairo adalah `(P+1)/2` (dan bukan 0 atau 0.5), karena memenuhi persamaan di atas.

#### Jenis Integer

Jenis felt252 adalah jenis fundamental yang berfungsi sebagai dasar untuk membuat semua jenis dalam pustaka inti.
Namun, sangat disarankan bagi para pemrogram untuk menggunakan jenis integer alih-alih jenis `felt252`, karena jenis integer dilengkapi dengan fitur keamanan tambahan yang memberikan perlindungan ekstra terhadap kerentanan potensial dalam kode, seperti pemeriksaan overflow. Dengan menggunakan jenis integer ini, para pemrogram dapat memastikan bahwa program-program mereka lebih aman dan kurang rentan terhadap serangan atau ancaman keamanan lainnya.
Sebuah _integer_ adalah sebuah bilangan tanpa komponen pecahan. Deklarasi jenis ini menunjukkan jumlah bit yang dapat digunakan pemrogram untuk menyimpan integer tersebut.
Tabel 3-1 menunjukkan
jenis integer bawaan dalam Cairo. Kita dapat menggunakan salah satu variasi ini untuk mendeklarasikan
jenis dari nilai integer.

<span class="caption">Tabel 3-1: Jenis Integer dalam Cairo</span>

| Panjang  | Tidak Bertanda |
| ------- | -------- |
| 8-bit   | `u8`     |
| 16-bit  | `u16`    |
| 32-bit  | `u32`    |
| 64-bit  | `u64`    |
| 128-bit | `u128`   |
| 256-bit | `u256`   |
| 32-bit  | `usize`  |

Setiap variasi memiliki ukuran eksplisit. Perlu diperhatikan bahwa untuk saat ini, jenis `usize` hanya merupakan alias untuk `u32`; namun, ini mungkin berguna ketika pada masa depan Cairo dapat dikompilasi ke MLIR.
Karena variabel bersifat tidak bertanda, mereka tidak dapat menyimpan angka negatif. Kode berikut akan menyebabkan program panic:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_07_integer_types/src/lib.cairo}}
```

Semua jenis integer yang disebutkan sebelumnya cocok ke dalam `felt252`, kecuali untuk `u256` yang memerlukan 4 bit tambahan untuk disimpan. Di balik layar, `u256` pada dasarnya adalah sebuah struktur dengan 2 bidang: `u256 {low: u128, high: u128}`

Anda dapat menulis literal integer dalam bentuk apapun yang ditunjukkan dalam Tabel 3-2. Perlu
dicatat bahwa literal angka yang dapat menjadi beberapa jenis numerik memungkinkan penambahan tipe,
seperti `57_u8`, untuk menunjukkan tipe.

<span class="caption">Tabel 3-2: Literal Integer dalam Cairo</span>

| Literal numerik | Contoh   |
| ---------------- | --------- |
| Desimal          | `98222`   |
| Heksadesimal     | `0xff`    |
| Oktales          | `0o04321` |
| Biner            | `0b01`    |

Jadi bagaimana Anda tahu jenis integer mana yang akan digunakan? Cobalah untuk memperkirakan nilai maksimum yang dapat dipegang oleh integer Anda dan pilih ukuran yang tepat.
Situasi utama di mana Anda akan menggunakan `usize` adalah saat melakukan pengindeksan pada koleksi data tertentu.

#### Operasi Numerik

Cairo mendukung operasi matematika dasar yang diharapkan untuk semua jenis integer:
penambahan, pengurangan, perkalian, pembagian, dan sisa bagi. Pembagian integer memotong ke nol ke bilangan bulat terdekat. Kode berikut menunjukkan
bagaimana cara menggunakan setiap operasi numerik dalam sebuah pernyataan `let`:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_08_numeric_operations/src/lib.cairo}}
```

Setiap ekspresi dalam pernyataan ini menggunakan operator matematika dan mengevaluasi
menjadi sebuah nilai tunggal, yang kemudian diikat ke sebuah variabel.

[Lampiran B][appendix_b] berisi daftar semua operator yang disediakan oleh Cairo.

#### Jenis Boolean

Seperti dalam kebanyakan bahasa pemrograman lainnya, tipe data Boolean dalam Cairo memiliki dua nilai yang mungkin: `true` dan `false`. Boolean memiliki ukuran satu felt252. Tipe data Boolean dalam
Cairo ditentukan menggunakan `bool`. Sebagai contoh:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_09_boolean_type/src/lib.cairo}}
```

Cara utama menggunakan nilai Boolean adalah melalui kondisional, seperti ekspresi `if`. Kami akan menutup bagaimana ekspresi `if` bekerja di Cairo pada bagian [“Aliran Kontrol”][control-flow].

#### Jenis String Pendek

Cairo tidak memiliki jenis data asli untuk string, tetapi Anda dapat menyimpan karakter yang membentuk apa yang kami sebut sebagai "string pendek" di dalam `felt252`. Sebuah string pendek memiliki panjang maksimum 31 karakter. Hal ini dilakukan untuk memastikan bahwa dapat muat dalam sebuah felt tunggal (sebuah felt adalah 252 bit, satu karakter ASCII adalah 8 bit).
Berikut adalah beberapa contoh deklarasi nilai dengan meletakkannya di antara tanda kutip satu:

```rust
{{#rustdoc_include ../listings/ch02-common-programming-concepts/no_listing_10_short_string_type/src/lib.cairo:2:3}}
```

### Jenis Casting

Di Cairo, Anda dapat mengonversi jenis data skalar dari satu jenis ke jenis lainnya dengan menggunakan metode `try_into` dan `into` yang disediakan oleh trait `TryInto` dan `Into` dari pustaka inti.

Metode `try_into` memungkinkan pencast tipe yang aman ketika tipe tujuan mungkin tidak cocok dengan nilai sumber. Perlu diingat bahwa `try_into` mengembalikan tipe `Option<T>`, yang perlu Anda bungkus (unwrap) untuk mengakses nilai baru.

Di sisi lain, metode `into` dapat digunakan untuk pencast tipe ketika kesuksesan dijamin, seperti saat tipe sumber lebih kecil dari tipe tujuan.

Untuk melakukan konversi, panggil `var.into()` atau `var.try_into()` pada nilai sumber untuk mencastingnya ke tipe lain. Tipe variabel baru harus didefinisikan secara eksplisit, seperti yang ditunjukkan dalam contoh di bawah ini.

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_11_type_casting/src/lib.cairo}}
```

### Jenis Tuple

Sebuah _tuple_ adalah cara umum untuk mengelompokkan sejumlah nilai dengan berbagai jenis ke dalam satu jenis gabungan. Tuple memiliki panjang tetap: setelah dideklarasikan, mereka tidak dapat tumbuh atau menyusut dalam ukuran.

Kita membuat tuple dengan menuliskan daftar nilai yang dipisahkan koma di dalam
tanda kurung. Setiap posisi dalam tuple memiliki tipe, dan tipe-tipe
nilai yang berbeda di dalam tuple tidak harus sama. Kami menambahkan anotasi
tipe opsional dalam contoh ini:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_12_tuple_type/src/lib.cairo}}
```

Variabel `tup` terikat ke seluruh tuple karena tuple dianggap sebagai
elemen tunggal. Untuk mendapatkan nilai-nilai individu dari sebuah tuple, kita dapat
menggunakan pola pencocokan untuk mendestruksi nilai tuple, seperti ini:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_13_tuple_destructuration/src/lib.cairo}}
```

Program ini pertama-tama membuat sebuah tuple dan mengikatnya ke variabel `tup`. Kemudian
menggunakan pola dengan `let` untuk mengambil `tup` dan mengubahnya menjadi tiga variabel terpisah, `x`, `y`, dan `z`. Ini disebut _destructuring_ karena memecah
tuple tunggal menjadi tiga bagian. Akhirnya, program mencetak `y adalah enam` karena nilai dari
`y` adalah `6`.

Kita juga dapat mendeklarasikan tuple dengan nilai dan tipe pada saat yang sama.
Sebagai contoh:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_14_tuple_types/src/lib.cairo}}
```

### Jenis unit ()

Sebuah _jenis unit_ adalah jenis yang hanya memiliki satu nilai `()`.
Ini direpresentasikan oleh sebuah tuple tanpa elemen.
Ukurannya selalu nol, dan dijamin tidak ada dalam kode yang dikompilasi.

[aliran-kontrol]: ch02-05-control-flow.md
[appendix_b]: appendix-02-operators-and-symbols.md#operators