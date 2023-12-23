## Fungsi

Fungsi sering digunakan dalam kode Cairo. Anda sudah melihat salah satu fungsi paling penting dalam bahasa ini: fungsi `main`, yang merupakan titik masuk banyak program. Anda juga sudah melihat kata kunci `fn`, yang memungkinkan Anda mendeklarasikan fungsi baru.

Kode Cairo menggunakan _snake case_ sebagai gaya konvensional untuk nama fungsi dan variabel, di mana semua huruf huruf kecil dan garis bawah memisahkan kata-kata. Berikut adalah contoh program yang berisi definisi fungsi:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_15_functions/src/lib.cairo}}
```

Kami mendefinisikan fungsi di Cairo dengan mengetikkan `fn` diikuti oleh nama fungsi dan sepasang tanda kurung. Kurung kurawal memberi tahu kompiler di mana tubuh fungsi dimulai dan berakhir.

Kami dapat memanggil setiap fungsi yang telah kami definisikan dengan mengetikkan namanya diikuti oleh sepasang tanda kurung. Karena `another_function` didefinisikan dalam program, itu dapat dipanggil dari dalam fungsi `main`. Perhatikan bahwa kami mendefinisikan `another_function` _sebelum_ fungsi `main` dalam kode sumber; kami juga bisa mendefinisikannya setelahnya. Cairo tidak peduli di mana Anda mendefinisikan fungsi Anda, asalkan mereka didefinisikan di dalam cakupan yang dapat dilihat oleh pemanggil.

Mari mulai proyek baru dengan Scarb yang diberi nama _functions_ untuk menjelajahi fungsi lebih lanjut. Tempatkan contoh `another_function` di _src/lib.cairo_ dan jalankannya. Anda seharusnya melihat keluaran berikut:

```shell
$ scarb cairo-run --available-gas=200000000
[DEBUG] Hello, world!                (raw: 5735816763073854953388147237921)
[DEBUG] Another function.            (raw: 22265147635379277118623944509513687592494)
```

Baris-baris dieksekusi sesuai dengan urutan mereka muncul dalam fungsi `main`. Pertama-tama pesan "Hello, world!" mencetak, dan kemudian `another_function` dipanggil dan pesannya dicetak.

### Parameter

Kita dapat mendefinisikan fungsi agar memiliki _parameter_, yaitu variabel khusus yang merupakan bagian dari tanda tangan fungsi. Ketika sebuah fungsi memiliki parameter, Anda dapat memberikan nilai konkret untuk parameter tersebut. Secara teknis, nilai konkret disebut _argumen_, tetapi dalam percakapan santai, orang cenderung menggunakan kata _parameter_ dan _argumen_ secara bergantian untuk variabel dalam definisi fungsi atau nilai konkret yang dilewatkan saat Anda memanggil fungsi.

Pada versi ini dari `another_function`, kita menambahkan satu parameter:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_16_single_param/src/lib.cairo}}
```

Coba jalankan program ini; Anda seharusnya mendapatkan keluaran berikut:

```shell
$ scarb cairo-run --available-gas=200000000
x = 5
Run completed successfully, returning []
Remaining gas: 1999829300
```

Deklarasi `another_function` memiliki satu parameter bernama `x`. Tipe `x` ditentukan sebagai `felt252`. Ketika kami memasukkan `5` ke `another_function`, makro `println!` menempatkan `5` di tempat sepasang kurung kurawal yang berisi `x` di dalam string format.

Dalam tanda tangan fungsi, Anda _harus_ mendeklarasikan tipe setiap parameter. Ini adalah keputusan yang disengaja dalam desain Cairo: mensyaratkan anotasi tipe dalam definisi fungsi berarti kompiler hampir tidak pernah perlu Anda menggunakannya di tempat lain dalam kode untuk mengetahui tipe apa yang Anda maksudkan. Kompiler juga dapat memberikan pesan kesalahan yang lebih membantu jika ia tahu tipe apa yang diharapkan oleh fungsi.

Saat mendefinisikan beberapa parameter, pisahkan deklarasi parameter dengan koma, seperti ini:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_17_multiple_params/src/lib.cairo}}
```

Contoh ini membuat fungsi bernama `print_labeled_measurement` dengan dua parameter. Parameter pertama dinamai `value` dan bertipe `u128`. Parameter kedua dinamai `unit_label` dan bertipe `ByteArray` - tipe internal Cairo untuk mewakili literal string. Fungsi kemudian mencetak teks yang berisi baik `value` maupun `unit_label`.

Mari coba jalankan kode ini. Gantilah program yang saat ini ada di file _src/lib.cairo_ proyek _functions_ Anda dengan contoh sebelumnya dan jalankan dengan `scarb cairo-run --available-gas=200000000`:

```shell
$ scarb cairo-run --available-gas=200000000
The measurement is: 5h
Run completed successfully, returning []
Remaining gas: 1999755400
```

Karena kami memanggil fungsi dengan `5` sebagai nilai untuk `value` dan `"h"` sebagai nilai untuk `unit_label`, keluaran program berisi nilai-nilai tersebut.

#### Parameter bernama

Dalam Cairo, parameter bernama memungkinkan Anda menentukan nama argumen saat Anda memanggil fungsi. Hal ini membuat pemanggilan fungsi lebih mudah dibaca dan deskriptif. Jika Anda ingin menggunakan parameter bernama, Anda perlu menentukan nama parameter dan nilai yang ingin Anda lewatkan. Sintaksnya adalah `nama_parameter: nilai`. Jika Anda melewati variabel yang memiliki nama yang sama dengan parameter, Anda dapat menulis `:nama_parameter` daripada `nama_parameter: nama_variabel`.

Berikut adalah contoh:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_30_named_parameters/src/lib.cairo}}
```

### Pernyataan dan Ekspresi

Tubuh fungsi terdiri dari serangkaian pernyataan yang opsionalnya diakhiri oleh sebuah ekspresi. Sejauh ini, fungsi yang telah kita bahas belum mencakup ekspresi penutup, tetapi Anda telah melihat sebuah ekspresi sebagai bagian dari sebuah pernyataan. Karena Cairo adalah bahasa berbasis ekspresi, ini adalah perbedaan penting untuk dipahami. Bahasa lain tidak memiliki perbedaan yang sama, jadi mari kita lihat apa itu pernyataan dan ekspresi dan bagaimana perbedaan mereka mempengaruhi tubuh fungsi.

- **Pernyataan** adalah instruksi yang melakukan suatu tindakan dan tidak mengembalikan nilai.
- **Ekspresi** mengevaluasi menjadi nilai hasil. Mari kita lihat beberapa contoh.

Sebenarnya, kita sudah menggunakan pernyataan dan ekspresi. Membuat variabel dan memberikan nilai kepadanya dengan kata kunci `let` adalah suatu pernyataan. Dalam Listing 2-1, `let y = 6;` adalah sebuah pernyataan.

```rust
{{#include ../listings/ch02-common-programming-concepts/listing_01_statement/src/lib.cairo}}
```

<span class="caption">Listing 2-1: Deklarasi fungsi `main` yang berisi satu pernyataan</span>

Definisi fungsi juga merupakan pernyataan; seluruh contoh sebelumnya adalah pernyataan itu sendiri.

Pernyataan tidak mengembalikan nilai. Oleh karena itu, Anda tidak dapat menugaskan pernyataan `let` ke variabel lain, seperti yang dicoba oleh kode berikut; Anda akan mendapatkan kesalahan:

```rust, noplayground
{{#include ../listings/ch02-common-programming-concepts/no_listing_18_statements_dont_return_values/src/lib.cairo}}
```

Ketika Anda menjalankan program ini, kesalahan yang akan Anda dapatkan terlihat seperti ini:

```shell
$ scarb cairo-run --available-gas=200000000
error: Missing token TerminalRParen.
 --> src/lib.cairo:2:14
    let x = (let y = 6);
             ^

error: Missing token TerminalSemicolon.
 --> src/lib.cairo:2:14
    let x = (let y = 6);
             ^

error: Missing token TerminalSemicolon.
 --> src/lib.cairo:2:14
    let x = (let y = 6);
                      ^

error: Skipped tokens. Expected: statement.
 --> src/lib.cairo:2:14
    let x = (let y = 6);
```

Pernyataan `let y = 6` tidak mengembalikan nilai, sehingga tidak ada yang dapat diikat oleh `x`. Ini berbeda dengan apa yang terjadi dalam bahasa lain, seperti C dan Ruby, di mana penugasan mengembalikan nilai dari penugasan tersebut. Dalam bahasa-bahasa itu, Anda dapat menulis `x = y = 6` dan memiliki kedua `x` dan `y` memiliki nilai `6`; itu tidak terjadi dalam Cairo.

Ekspresi mengevaluasi menjadi nilai dan membentuk sebagian besar kode yang akan Anda tulis di Cairo. Pertimbangkan operasi matematika, seperti `5 + 6`, yang merupakan ekspresi yang mengevaluasi menjadi nilai `11`. Ekspresi dapat menjadi bagian dari pernyataan: dalam Listing 2-1, `6` dalam pernyataan `let y = 6;` adalah ekspresi yang mengevaluasi menjadi nilai `6`. Memanggil suatu fungsi adalah ekspresi. Blok cakupan baru yang dibuat dengan tanda kurung kurawal adalah ekspresi, misalnya:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_19_blocks_are_expressions/src/lib.cairo:all}}
```

Ekspresi ini:

```rust, noplayground
{{#include ../listings/ch02-common-programming-concepts/no_listing_19_blocks_are_expressions/src/lib.cairo:block_expr}}
```

adalah blok yang, dalam hal ini, mengevaluasi menjadi `4`. Nilai tersebut diikatkan ke `y` sebagai bagian dari pernyataan `let`. Perhatikan bahwa baris `x + 1` tidak memiliki titik koma di akhir, yang berbeda dari sebagian besar baris yang telah Anda lihat sejauh ini. Ekspresi tidak mencakup titik koma di akhir. Jika Anda menambahkan titik koma di akhir ekspresi, Anda mengubahnya menjadi pernyataan, dan itu tidak akan mengembalikan nilai. Ingatlah ini saat Anda menjelajahi nilai kembalian fungsi dan ekspresi selanjutnya.

### Fungsi dengan Nilai Kembalian

Fungsi dapat mengembalikan nilai ke kode yang memanggilnya. Kita tidak memberi nama pada nilai kembalian, tetapi kita harus mendeklarasikan tipe mereka setelah tanda panah (`->`). Di Cairo, nilai kembalian fungsi adalah sinonim dengan nilai dari ekspresi terakhir dalam blok tubuh fungsi. Anda dapat mengembalikan secara cepat dari fungsi dengan menggunakan kata kunci `return` dan menentukan nilai, tetapi sebagian besar fungsi mengembalikan ekspresi terakhir secara implisit. Berikut adalah contoh fungsi yang mengembalikan nilai:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_20_function_return_values/src/lib.cairo}}
```

Tidak ada pemanggilan fungsi, atau bahkan pernyataan `let` dalam fungsi `five`—hanya angka `5` itu sendiri. Itu adalah fungsi yang sepenuhnya valid di Cairo. Perhatikan bahwa tipe pengembalian fungsi juga ditentukan, yaitu `-> u32`. Coba jalankan kode ini; keluarannya seharusnya terlihat seperti ini:

```shell
$ scarb cairo-run --available-gas=200000000
x = 5
Run completed successfully, returning []
Remaining gas: 1999847760
```

Angka `5` dalam fungsi `five` adalah nilai kembalian fungsi, itulah mengapa tipe pengembalian adalah `u32`. Mari kita periksa ini lebih detail. Ada dua bagian penting: pertama, baris `let x = five();` menunjukkan bahwa kita menggunakan nilai kembalian suatu fungsi untuk menginisialisasi sebuah variabel. Karena fungsi `five` mengembalikan `5`, baris tersebut sama dengan yang berikut:

```rust, noplayground
let x = 5;
```

Kedua, fungsi `five` tidak memiliki parameter dan mendefinisikan tipe nilai kembali, tetapi tubuh fungsi tersebut adalah angka `5` yang sepi tanpa titik koma karena itu adalah ekspresi yang nilainya ingin kita kembalikan.

Mari kita lihat contoh lain:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_21_function_return_values_2/src/lib.cairo}}
```

Menjalankan kode ini akan mencetak `x = 6`. Tetapi jika kita menempatkan titik koma di akhir baris yang berisi `x + 1`, mengubahnya dari ekspresi menjadi pernyataan, kita akan mendapatkan kesalahan:

```rust,does_not_compile
{{#include ../listings/ch02-common-programming-concepts/no_listing_22_function_return_invalid/src/lib.cairo}}
```

Mengompilasi kode ini menghasilkan kesalahan, seperti berikut:

```shell
error: Unexpected return type. Expected: "core::integer::u32", found: "()".
```

Pesan kesalahan utama, `Unexpected return type`, mengungkapkan isu inti dengan kode ini. Definisi fungsi `plus_one` menyatakan bahwa itu akan mengembalikan `u32`, tetapi pernyataan tidak mengevaluasi menjadi nilai, yang dinyatakan oleh `()`, tipe unit. Oleh karena itu, tidak ada yang dikembalikan, yang bertentangan dengan definisi fungsi dan menghasilkan kesalahan.