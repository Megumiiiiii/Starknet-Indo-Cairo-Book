## Pemilikan Menggunakan Sistem Tipe Linear

Cairo menggunakan sistem tipe linear. Dalam sistem tipe seperti ini, setiap nilai (tipe dasar, struktur data, enum) harus digunakan dan hanya boleh digunakan sekali. 'Digunakan' di sini berarti bahwa nilai tersebut entah _dihancurkan_ atau _dipindahkan_.

_Penjelasan_ dapat terjadi dalam beberapa cara:

- sebuah variabel keluar dari cakupan (out of scope)
- sebuah struktur data di-destrukturisasi
- penghancuran eksplisit menggunakan destruct()

_Memindahkan_ nilai hanya berarti meneruskan nilai itu ke fungsi lain.

Hal ini menghasilkan batasan yang agak mirip dengan model kepemilikan Rust, tetapi ada beberapa perbedaan.
Secara khusus, model kepemilikan Rust ada (sebagian) untuk menghindari perlombaan data (data races) dan akses mutabel konkuren ke nilai memori. Ini jelas tidak mungkin dalam Cairo karena memori bersifat tak berubah (immutable).
Sebagai gantinya, Cairo memanfaatkan sistem tipe linear-nya untuk dua tujuan utama:

- Memastikan bahwa semua kode dapat dibuktikan (provable) dan dengan demikian dapat diverifikasi.
- Abstraksi memori yang tak berubah (immutable) dari VM Cairo.

#### Kepemilikan (Ownership)

Dalam Cairo, kepemilikan berlaku untuk _variabel_ dan bukan untuk _nilai (values)_. Sebuah nilai dapat dengan aman dirujuk oleh banyak variabel yang berbeda (bahkan jika mereka adalah variabel mutabel), karena nilai itu sendiri selalu tidak berubah (immutable).
Namun variabel dapat bersifat mutabel, sehingga kompilator harus memastikan bahwa variabel konstan tidak secara tidak sengaja diubah oleh programmer.
Hal ini membuat mungkin untuk berbicara tentang kepemilikan dari sebuah variabel: pemilik (owner) adalah kode yang dapat membaca (dan menulis jika mutabel) variabel tersebut.

Ini berarti bahwa variabel (bukan nilai) mengikuti aturan yang mirip dengan nilai-nilai Rust:

- Setiap variabel di Cairo memiliki seorang pemilik (owner).
- Hanya boleh ada satu pemilik pada satu waktu.
- Ketika pemilik keluar dari cakupan (out of scope), variabel itu dihancurkan.

Sekarang setelah kita melewati sintaks dasar Cairo, kita tidak akan menyertakan semua contoh `fn main() {` di dalam fungsi `main` secara manual. Sebagai hasilnya, contoh-contoh kita akan menjadi kode dalam contoh-contoh, jadi jika Anda mengikuti, pastikan untuk menambahkan sedikit yang berikut ini lebih ringkas, memungkinkan kita fokus pada detail sebenarnya daripada kode boilerplate.

### Cakupan Variabel

Sebagai contoh pertama dari sistem tipe linear, kita akan melihat _cakupan_ dari beberapa variabel. Cakupan adalah rentang dalam sebuah program di mana suatu item valid. Ambil contoh variabel berikut:

```rust,noplayground
let s = 'hello';
```

Variabel `s` merujuk pada string pendek. Variabel ini valid mulai dari titik deklarasinya hingga akhir _cakupan_ (_scope_) saat ini. Listing 4-1 menunjukkan sebuah program dengan komentar yang menandai dimana variabel `s` akan valid.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing_03_01/src/lib.cairo:here}}
```

<span class="caption">Listing 4-1: Sebuah variabel dan cakupan di mana variabel tersebut valid</span>

Dengan kata lain, ada dua poin penting dalam waktu ini:

- Ketika `s` masuk ke dalam _cakupan_, itu valid.
- Itu tetap valid hingga keluar dari _cakupan_.

Pada titik ini, hubungan antara cakupan dan kapan variabel valid serupa dengan dalam bahasa pemrograman lainnya. Sekarang kita akan membangun di atas pemahaman ini dengan menggunakan tipe `Array` yang kita perkenalkan dalam [bab sebelumnya](./ch03-01-arrays.md).

### Memindahkan Nilai - Contoh dengan Array

Seperti yang disebutkan sebelumnya, _memindahkan_ nilai hanya berarti meneruskan nilai tersebut ke fungsi lain. Ketika itu terjadi, variabel yang merujuk pada nilai tersebut dalam cakupan asli dihancurkan dan tidak dapat lagi digunakan, dan variabel baru dibuat untuk menyimpan nilai yang sama.

Array adalah contoh tipe kompleks yang dipindahkan ketika dilewatkan ke fungsi lain.
Berikut adalah pengingat singkat tentang tampilan array:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no_listing_01_array/src/lib.cairo:2:4}}
```

Bagaimana sistem tipe memastikan bahwa program Cairo tidak pernah mencoba menulis ke sel memori yang sama dua kali?
Pertimbangkan kode berikut, di mana kita mencoba untuk menghapus bagian depan array dua kali:

```rust,does_not_compile
{{#include ../listings/ch04-understanding-ownership/no_listing_02_pass_array_by_value/src/lib.cairo}}
```

Dalam kasus ini, kita mencoba meneruskan nilai yang sama (array dalam variabel `arr`) ke kedua panggilan fungsi. Ini berarti kode kita mencoba untuk menghapus elemen pertama dua kali, yang akan mencoba untuk menulis ke sel memori yang sama dua kali - yang dilarang oleh VM Cairo, menyebabkan kesalahan saat runtime.
Untungnya, kode ini sebenarnya tidak dikompilasi. Setelah kita meneruskan array ke fungsi `foo`, variabel `arr` tidak lagi dapat digunakan. Kita mendapatkan kesalahan pada waktu kompilasi ini, memberitahu kita bahwa kita perlu Array mengimplementasikan Trait Copy:

```shell
error: Variabel sebelumnya telah dipindahkan. Trait tidak memiliki implementasi dalam konteks: core::traits::Copy::<core::array::Array::<core::integer::u128>>
 --> array.cairo:6:9
    let mut arr = ArrayTrait::<u128>::new();
        ^*****^
```

#### Trait Copy

Jika suatu tipe mengimplementasikan trait `Copy`, mempassing nilai dari tipe tersebut ke fungsi tidak akan memindahkan nilai tersebut. Sebagai gantinya, variabel baru dibuat, merujuk pada nilai yang sama.
Hal penting yang perlu dicatat di sini adalah bahwa ini adalah operasi yang benar-benar gratis, karena variabel hanyalah abstraksi Cairo dan karena _nilai_ di Cairo selalu tidak berubah (immutable). Hal ini, khususnya, konseptual berbeda dari versi Rust dari trait `Copy`, di mana nilai potensialnya disalin di memori.

Anda dapat mengimplementasikan trait `Copy` pada tipe Anda dengan menambahkan anotasi `#[derive(Copy)]` pada definisi tipe Anda. Namun, Cairo tidak akan mengizinkan tipe untuk dianotasi dengan Copy jika tipe itu sendiri atau salah satu komponennya tidak mengimplementasikan trait Copy.
Meskipun Array dan Dictionary tidak dapat disalin, tipe kustom yang tidak mengandung keduanya bisa.

```rust,ignore_format
{{#include ../listings/ch04-understanding-ownership/no_listing_03_copy_trait/src/lib.cairo}}
```

Dalam contoh ini, kita dapat meneruskan `p1` dua kali ke fungsi foo karena tipe `Point` mengimplementasikan trait `Copy`. Ini berarti bahwa ketika kita meneruskan `p1` ke `foo`, kita sebenarnya meneruskan salinan dari `p1`, sehingga `p1` tetap valid. Dalam istilah kepemilikan, ini berarti bahwa kepemilikan `p1` tetap berada pada fungsi utama.
Jika Anda menghapus turunan trait `Copy` dari tipe `Point`, Anda akan mendapatkan kesalahan waktu kompilasi saat mencoba mengompilasi kode tersebut.

_Jangan khawatir tentang kata kunci `Struct`. Kami akan memperkenalkannya dalam [Bab 5](ch05-00-using-structs-to-structure-related-data.md)._

### Menghancurkan Nilai - Contoh dengan FeltDict

Cara lain tipe linear dapat _digunakan_ adalah dengan dihancurkan. Penghancuran harus memastikan bahwa 'sumber daya' sekarang dilepaskan dengan benar. Dalam Rust misalnya, ini bisa menjadi menutup akses ke file, atau mengunci sebuah mutex.
Di Cairo, satu tipe yang memiliki perilaku seperti ini adalah `Felt252Dict`. Untuk kepastian, dictionary harus 'disquash' ketika mereka dihancurkan.
Ini bisa sangat mudah dilupakan, jadi ini ditegakkan oleh sistem tipe dan kompilator.

#### Penghancuran yang tidak berpengaruh: Trait Drop

Anda mungkin telah memperhatikan bahwa tipe `Point` pada contoh sebelumnya juga mengimplementasikan trait `Drop`.
Misalnya, kode berikut tidak akan dikompilasi, karena struktur `A` tidak dipindahkan atau dihancurkan sebelum keluar dari cakupan:

```rust,does_not_compile
{{#include ../listings/ch04-understanding-ownership/no_listing_04_no_drop_derive_fails/src/lib.cairo}}
```

Namun, tipe yang mengimplementasikan trait `Drop` secara otomatis dihancurkan saat keluar dari cakupan. Penghancuran ini tidak melakukan apa-apa, ini hanya sebuah hint kepada kompilator bahwa tipe ini dapat dengan aman dihancurkan begitu tidak lagi berguna. Kami menyebut ini "meng-drop" sebuah nilai.

Saat ini, implementasi `Drop` dapat diturunkan untuk semua tipe, memungkinkan mereka di-drop saat keluar dari cakupan, kecuali untuk kamus (`Felt252Dict`) dan tipe yang berisi kamus.
Sebagai contoh, kode berikut akan dikompilasi:

```rust
{{#include ../listings/ch04-understanding-ownership/no_listing_05_drop_derive_compiles/src/lib.cairo}}
```

#### Penghancuran dengan Efek Samping: trait Destruct

Ketika sebuah nilai dihancurkan, kompilator pertama-tama mencoba memanggil metode `drop` pada tipe tersebut. Jika tidak ada, maka kompilator mencoba memanggil `destruct` sebagai gantinya. Metode ini disediakan oleh trait `Destruct`.

Seperti yang disebutkan sebelumnya, kamus (dictionaries) di Cairo adalah tipe yang harus "disquash" saat dihancurkan, sehingga urutan akses dapat dibuktikan. Hal ini mudah dilupakan oleh para pengembang, jadi sebagai gantinya kamus mengimplementasikan trait `Destruct` untuk memastikan bahwa semua kamus _disquash_ ketika keluar dari cakupan.
Dengan demikian, contoh berikut tidak akan dikompilasi:

```rust,does_not_compile
{{#include ../listings/ch04-understanding-ownership/no_listing_06_no_destruct_compile_fails/src/lib.cairo}}
```

Jika Anda mencoba menjalankan kode ini, Anda akan mendapatkan kesalahan waktu kompilasi:

```shell
error: Variable not dropped. Trait has no implementation in context: core::traits::Drop::<temp7::temp7::A>. Trait has no implementation in context: core::traits::Destruct::<temp7::temp7::A>.
 --> temp7.cairo:7:5
    A {
    ^*^
```

Ketika `A` keluar dari cakupan, itu tidak dapat di-drop karena tidak mengimplementasikan trait `Drop` (karena berisi kamus dan tidak bisa `derive(Drop)`) maupun trait `Destruct`. Untuk memperbaiki ini, kita dapat menurunkan implementasi trait `Destruct` untuk tipe `A`:

```rust
{{#include ../listings/ch04-understanding-ownership/no_listing_07_destruct_compiles/src/lib.cairo}}
```

Sekarang, ketika `A` keluar dari cakupan, kamusnya akan secara otomatis _disquash_, dan program akan berhasil dikompilasi.

### Menyalin Data Array dengan Clone

Jika kita _ingin_ menyalin data dari sebuah `Array`, kita dapat menggunakan metode umum yang disebut `clone`. Kita akan membahas sintaks metode dalam Bab 6, tetapi karena metode adalah fitur umum dalam banyak bahasa pemrograman, Anda mungkin sudah pernah melihatnya sebelumnya.

Berikut adalah contoh dari metode `clone` dalam aksi.

```rust
{{#include ../listings/ch04-understanding-ownership/no_listing_08_array_clone/src/lib.cairo}}
```

Ketika Anda melihat panggilan ke `clone`, Anda tahu bahwa ada beberapa kode sembarang yang sedang dieksekusi dan kode tersebut mungkin mahal. Itu merupakan indikator visual bahwa ada sesuatu yang berbeda terjadi.
Dalam kasus ini, _nilai_ sedang disalin, menghasilkan penggunaan sel memori baru, dan variabel baru diciptakan, merujuk pada nilai yang baru, disalin.

### Nilai Kembali dan Cakupan

Mengembalikan nilai setara dengan _memindahkan_ mereka. Listing 4-4 menunjukkan contoh sebuah
fungsi yang mengembalikan beberapa nilai, dengan anotasi serupa seperti pada Listing
4-3.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch04-understanding-ownership/listing_03_04/src/lib.cairo}}
```

<span class="caption">Listing 4-4: Memindahkan nilai kembali</span>

Meskipun ini berfungsi, memindahkan masuk dan keluar dari setiap fungsi agak membosankan. Bagaimana jika kita ingin membiarkan sebuah fungsi menggunakan nilai tetapi tidak memindahkan nilainya? Sangat menjengkelkan bahwa apa pun yang kita lewatkan juga perlu dilewatkan kembali jika kita ingin menggunakannya lagi, selain dari data apapun yang dihasilkan dari tubuh fungsi yang mungkin ingin kita kembalikan juga.

Cairo memungkinkan kita untuk mengembalikan beberapa nilai menggunakan tuple, seperti yang ditunjukkan dalam Listing 4-5.

<span class="filename">Nama File: src/lib.cairo</span>

```rust
{{#include ../listings/ch04-understanding-ownership/listing_03_05/src/lib.cairo}}
```

<span class="caption">Listing 4-5: Mengembalikan banyak nilai</span>

Tetapi ini terlalu banyak cakupan dan banyak pekerjaan untuk konsep yang seharusnya umum. Untungnya untuk kita, Cairo memiliki dua fitur untuk melewatkan nilai tanpa menghancurkannya atau memindahkannya, disebut _referensi_ dan _snapshot_.

[data-types]: ch02-02-data-types.html#data-types
[method-syntax]: ch04-03-method-syntax.html#method-syntax