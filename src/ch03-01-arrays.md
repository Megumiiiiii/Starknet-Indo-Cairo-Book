## Array

Sebuah array adalah kumpulan elemen yang memiliki tipe yang sama. Anda dapat membuat dan menggunakan metode array dengan menggunakan `ArrayTrait` trait dari pustaka inti.

Hal penting yang perlu dicatat adalah bahwa array memiliki opsi modifikasi yang terbatas. Array sebenarnya adalah antrian nilai yang tidak dapat diubah.

Ini terkait dengan fakta bahwa setelah suatu slot memori ditulis, itu tidak dapat ditimpa, tetapi hanya dapat dibaca. Anda hanya dapat menambahkan item ke ujung array dan menghapus item dari depan menggunakan `pop_front`.

### Membuat Array

Membuat array dilakukan dengan panggilan `ArrayTrait::new()`. Berikut adalah contoh pembuatan array yang kami tambahkan 3 elemen ke dalamnya:

```rust
{{#include ../listings/ch03-common-collections/no_listing_00_array_new_append/src/lib.cairo}}
```

Jika diperlukan, Anda dapat menyertakan tipe yang diharapkan dari item di dalam array saat menginstansiasi array seperti ini, atau secara eksplisit menentukan tipe variabelnya.

```rust, noplayground
let mut arr = ArrayTrait::<u128>::new();
```

```rust, noplayground
let mut arr:Array<u128> = ArrayTrait::new();
```

### Memperbarui Array

#### Menambahkan Elemen

Untuk menambahkan elemen ke ujung array, Anda dapat menggunakan metode `append()`:

```rust
{{#rustdoc_include ../listings/ch03-common-collections/no_listing_00_array_new_append/src/lib.cairo:5}}
```

#### Menghapus Elemen

Anda hanya dapat menghapus elemen dari depan array dengan menggunakan metode `pop_front()`. Metode ini mengembalikan `Option` yang berisi elemen yang dihapus, atau `Option::None` jika array kosong.

```rust
{{#include ../listings/ch03-common-collections/no_listing_01_array_pop_front/src/lib.cairo}}
```

Kode di atas akan mencetak `10` karena kita menghapus elemen pertama yang ditambahkan.

Di Cairo, memori bersifat tidak dapat diubah, yang berarti tidak mungkin mengubah elemen-elemen array setelah ditambahkan. Anda hanya dapat menambahkan elemen ke ujung array dan menghapus elemen dari depan array. Operasi-operasi ini tidak memerlukan mutasi memori, karena melibatkan pembaruan pointer daripada langsung mengubah sel-sel memori.

### Membaca Elemen dari Array

Untuk mengakses elemen-elemen array, Anda dapat menggunakan metode array `get()` atau `at()` yang mengembalikan tipe yang berbeda. Menggunakan `arr.at(index)` setara dengan menggunakan operator subscripting `arr[index]`.

Fungsi `get` mengembalikan `Option<Box<@T>>`, yang berarti mengembalikan opsi untuk tipe Box (tipe smart-pointer Cairo) yang berisi snapshot elemen pada indeks yang ditentukan jika elemen tersebut ada dalam array. Jika elemen tidak ada, `get` mengembalikan `None`. Metode ini berguna ketika Anda berharap mengakses indeks yang mungkin tidak berada dalam batas array dan ingin menangani kasus tersebut dengan lembut tanpa panic. Snapshot akan dijelaskan lebih detail dalam bab [Referensi dan Snapshot](ch04-02-references-and-snapshots.md).

Fungsi `at`, di sisi lain, langsung mengembalikan snapshot elemen pada indeks yang ditentukan menggunakan operator `unbox()` untuk mengekstrak nilai yang disimpan dalam kotak. Jika indeks di luar batas, terjadi kesalahan panic. Anda sebaiknya hanya menggunakan `at` ketika Anda ingin program panic jika indeks yang diberikan berada di luar batas array, yang dapat mencegah perilaku yang tidak terduga.

Secara ringkas, gunakan `at` ketika Anda ingin panic pada percobaan akses di luar batas, dan gunakan `get` ketika Anda lebih suka menangani kasus tersebut dengan lembut tanpa panic.

```rust
{{#include ../listings/ch03-common-collections/no_listing_02_array_at/src/lib.cairo}}
```

Dalam contoh ini, variabel yang dinamai `first` akan mendapatkan nilai `0` karena itu adalah nilai pada indeks `0` dalam array. Variabel yang dinamai `second` akan mendapatkan nilai `1` dari indeks `1` dalam array.

Berikut adalah contoh dengan metode `get()`:

```rust
{{#include ../listings/ch03-common-collections/no_listing_03_array_get/src/lib.cairo}}
```

### Metode Terkait Ukuran

Untuk menentukan jumlah elemen dalam sebuah array, gunakan metode `len()`. Hasilnya adalah tipe `usize`.

Jika Anda ingin memeriksa apakah sebuah array kosong atau tidak, Anda dapat menggunakan metode `is_empty()`, yang mengembalikan `true` jika array kosong dan `false` sebaliknya.

### Menyimpan Tipe-Tipe Berbeda dengan Enums

Jika Anda ingin menyimpan elemen-elemen dengan tipe yang berbeda dalam sebuah array, Anda dapat menggunakan `Enum` untuk mendefinisikan tipe data kustom yang dapat menyimpan tipe-tipe berbeda. Enums akan dijelaskan lebih detail dalam bab [Enums and Pattern Matching](ch06-00-enums-and-pattern-matching.md).

```rust
{{#include ../listings/ch03-common-collections/no_listing_04_array_with_enums/src/lib.cairo}}
```

### Span

`Span` adalah struktur data yang mewakili snapshot dari sebuah `Array`. Dirancang untuk memberikan akses aman dan terkontrol ke elemen-elemen array tanpa memodifikasi array asli. Span sangat berguna untuk memastikan integritas data dan menghindari masalah peminjaman saat melewatkan array antar fungsi atau saat melakukan operasi hanya baca (lihat [References and Snapshots](ch04-02-references-and-snapshots.md)).

Semua metode yang disediakan oleh `Array` juga dapat digunakan dengan `Span`, kecuali metode `append()`.

#### Mengubah Array menjadi Span

Untuk membuat `Span` dari sebuah `Array`, panggil metode `span()`:

```rust
{{#rustdoc_include ../listings/ch03-common-collections/no_listing_05_array_span/src/lib.cairo:3}}
```
