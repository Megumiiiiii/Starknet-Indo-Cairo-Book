## Referensi dan Cuplikan

Permasalahan pada kode tuple dalam Listing 4-5 adalah kita harus mengembalikan `Array` ke fungsi pemanggil sehingga kita masih bisa menggunakan `Array` setelah pemanggilan ke `calculate_length`, karena `Array` telah dipindahkan ke dalam `calculate_length`.

### Cuplikan

Pada bab sebelumnya, kita membahas bagaimana sistem kepemilikan Cairo mencegah kita menggunakan variabel setelah kita telah memindahkannya, melindungi kita dari potensi menulis dua kali ke sel memori yang sama. Namun, ini tidak begitu nyaman. Mari kita lihat bagaimana kita dapat mempertahankan kepemilikan variabel dalam fungsi pemanggil dengan menggunakan snapshot.

Di Cairo, snapshot adalah tampilan tidak berubah dari suatu nilai pada titik waktu tertentu. Ingatlah bahwa memori tidak berubah, sehingga memodifikasi nilai sebenarnya membuat sel memori baru. Sel memori lama masih ada, dan snapshot adalah variabel yang merujuk pada nilai "lama" tersebut. Dalam hal ini, snapshot adalah tampilan "ke masa lalu".

Berikut adalah bagaimana Anda akan mendefinisikan dan menggunakan fungsi `calculate_length` yang mengambil snapshot ke array sebagai parameter daripada mengambil kepemilikan dari nilai yang mendasarinya. Pada contoh ini, fungsi `calculate_length` mengembalikan panjang array yang dilewatkan sebagai parameter. Karena kita melewatinya sebagai snapshot, yang merupakan tampilan tidak berubah dari array, kita dapat yakin bahwa fungsi `calculate_length` tidak akan mengubah array, dan kepemilikan array tetap ada di dalam fungsi utama.

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch04-understanding-ownership/no_listing_09_snapshots/src/lib.cairo}}
```

> Catatan: Hanya mungkin untuk memanggil metode `len()` pada snapshot array karena itu didefinisikan seperti itu dalam trait `ArrayTrait`. Jika Anda mencoba memanggil metode yang tidak didefinisikan untuk snapshot pada snapshot, Anda akan mendapatkan kesalahan kompilasi. Namun, Anda dapat memanggil metode yang mengharapkan snapshot pada tipe non-snapshot.

Keluaran dari program ini adalah:

```shell
[DEBUG]	                               	(raw: 0)

[DEBUG]	                              	(raw: 1)

Pengembalian berhasil, mengembalikan []
```

Pertama, perhatikan bahwa semua kode tuple dalam deklarasi variabel dan nilai kembalian fungsi telah hilang. Kedua, perhatikan bahwa kita melewatkan `@arr1` ke `calculate_length` dan, dalam definisinya, kita menggunakan `@Array<u128>` daripada `Array<u128>`.

Mari kita perhatikan lebih dekat panggilan fungsi di sini:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no_listing_09_snapshots/src/lib.cairo:function_call}}
```

Sintaks `@arr1` memungkinkan kita membuat snapshot dari nilai dalam `arr1`. Karena snapshot adalah tampilan tidak berubah dari nilai pada titik waktu tertentu, aturan biasa dari sistem tipe linear tidak diberlakukan. Khususnya, variabel snapshot selalu `Drop`, tidak pernah `Destruct`, bahkan snapshot kamus.

Sama halnya, tanda fungsi menggunakan `@` untuk menunjukkan bahwa tipe dari parameter `arr` adalah snapshot. Mari tambahkan beberapa anotasi penjelas:

```rust, noplayground
fn calculate_length(
    array_snapshot: @Array<u128>
) -> usize { // array_snapshot adalah snapshot dari Array
    array_snapshot.len()
} // Di sini, array_snapshot keluar dari cakupan dan di-drop.
// Namun, karena ini hanya merupakan tampilan dari apa yang asli array `arr` berisi, `arr` asli masih bisa digunakan.
```

Cakupan di mana variabel `array_snapshot` valid adalah sama dengan cakupan parameter fungsi apa pun, tetapi nilai mendasar dari snapshot tidak di-drop saat `array_snapshot` berhenti digunakan. Ketika fungsi memiliki snapshot sebagai parameter alih-alih nilai sebenarnya, kita tidak perlu mengembalikan nilai tersebut untuk memberikan kepemilikan kembali dari nilai asli, karena kita tidak pernah memiliki kepemilikannya.

#### Operator Desnap

Untuk mengonversi snapshot kembali ke variabel reguler, Anda dapat menggunakan operator `desnap` `*`, yang berfungsi sebagai kebalikan dari operator `@`.

Hanya tipe `Copy` yang dapat didesnap. Namun, dalam kasus umum, karena nilai tidak dimodifikasi, variabel baru yang dibuat oleh operator `desnap` menggunakan kembali nilai lama, sehingga proses desnapping adalah operasi yang sepenuhnya gratis, sama seperti `Copy`.

Pada contoh berikut, kita ingin menghitung luas persegi panjang, tetapi kita tidak ingin mengambil kepemilikan persegi panjang dalam fungsi `calculate_area`, karena kita mungkin ingin menggunakan persegi panjang tersebut lagi setelah pemanggilan fungsi. Karena fungsi kita tidak mengubah instance persegi panjang, kita dapat melewatkan snapshot dari persegi panjang ke fungsi, dan kemudian mengubah snapshot kembali menjadi nilai menggunakan operator `desnap` `*`.

```rust
{{#include ../listings/ch04-understanding-ownership/no_listing_10_desnap/src/lib.cairo}}
```

Namun, apa yang terjadi jika kita mencoba memodifikasi sesuatu yang kita lewati sebagai snapshot? Cobalah kode pada
Listing 4-6. SPOILER: Ini tidak berhasil!

<span class="filename">Nama file: src/lib.cairo</span>

```rust,does_not_compile
{{#include ../listings/ch04-understanding-ownership/listing_03_06/src/lib.cairo}}
```

<span class="caption">Listing 4-6: Mencoba memodifikasi nilai snapshot</span>

Berikut adalah kesalahannya:

```shell
error: Invalid left-hand side of assignment.
 --> ownership.cairo:15:5
    rec.height = rec.width;
    ^********^
```

Kompilator mencegah kita untuk memodifikasi nilai yang terkait dengan snapshot.

### Referensi Mutable

Kita bisa mencapai perilaku yang kita inginkan pada Listing 4-6 dengan menggunakan _referensi mutable_ alih-alih snapshot. Referensi mutable sebenarnya adalah nilai yang dapat diubah yang dilewatkan ke fungsi yang secara implisit dikembalikan pada akhir fungsi, mengembalikan kepemilikan ke konteks pemanggil. Dengan cara ini, mereka memungkinkan Anda untuk mengubah nilai yang dilewatkan sambil tetap memegang kepemilikannya dengan mengembalikannya secara otomatis pada akhir eksekusi.
Di Cairo, sebuah parameter dapat dilewati sebagai _referensi mutable_ menggunakan modifier `ref`.

> **Catatan**: Di Cairo, sebuah parameter hanya dapat dilewati sebagai _referensi mutable_ menggunakan modifier `ref` jika variabel dideklarasikan sebagai mutable dengan `mut`.

Pada Listing 4-7, kita menggunakan referensi mutable untuk mengubah nilai bidang `height` dan `width` dari instan `Rectangle` dalam fungsi `flip`.

```rust
{{#include ../listings/ch04-understanding-ownership/listing_03_07/src/lib.cairo}}
```

<span class="caption">Listing 4-7: Penggunaan referensi mutable untuk mengubah nilai</span>

Pertama, kita ubah `rec` menjadi `mut`. Kemudian kita lewatkan referensi mutable dari `rec` ke `flip` dengan `ref rec`, dan perbarui tanda tangan fungsi untuk menerima referensi mutable dengan `ref rec: Rectangle`. Ini membuatnya sangat jelas bahwa fungsi `flip` akan mengubah nilai dari instan `Rectangle` yang dilewatkan sebagai parameter.

Keluaran dari program ini adalah:

```shell
[DEBUG]
                                (raw: 10)

[DEBUG]	                        (raw: 3)
```

Seperti yang diharapkan, bidang `height` dan `width` dari variabel `rec` telah ditukar.

### Ringkasan Kecil

Mari kita ringkas apa yang telah kita diskusikan tentang sistem tipe linear, kepemilikan, snapshot, dan referensi:

- Pada suatu waktu tertentu, sebuah variabel hanya dapat dimiliki oleh satu pemilik.
- Anda dapat melewati sebuah variabel berdasarkan nilai, snapshot, atau referensi ke fungsi.
- Jika Anda melewatinya berdasarkan nilai, kepemilikan variabel dipindahkan ke fungsi.
- Jika Anda ingin tetap memegang kepemilikan variabel dan tahu bahwa fungsi Anda tidak akan memutasi variabel tersebut, Anda dapat melewatinya sebagai snapshot dengan `@`.
- Jika Anda ingin tetap memegang kepemilikan variabel dan tahu bahwa fungsi Anda akan memutasi variabel tersebut, Anda dapat melewatinya sebagai referensi mutable dengan `ref`.
