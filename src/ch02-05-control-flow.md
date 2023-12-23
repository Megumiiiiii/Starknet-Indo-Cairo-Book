## Alur Kontrol

Kemampuan untuk menjalankan beberapa kode tergantung pada apakah suatu kondisi benar dan untuk menjalankan beberapa kode secara berulang saat suatu kondisi benar adalah blok dasar dalam kebanyakan bahasa pemrograman. Konstruksi paling umum yang memungkinkan Anda mengontrol alur eksekusi kode Cairo adalah ekspresi `if` dan perulangan.

### Ekspresi `if`

Ekspresi `if` memungkinkan Anda bercabang dalam kode Anda tergantung pada kondisi. Anda memberikan kondisi dan kemudian menyatakan, "Jika kondisi ini terpenuhi, jalankan blok kode ini. Jika kondisinya tidak terpenuhi, jangan jalankan blok kode ini."

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_24_if/src/lib.cairo}}
```

Semua ekspresi `if` dimulai dengan kata kunci `if`, diikuti oleh suatu kondisi. Dalam kasus ini, kondisinya memeriksa apakah variabel `number` memiliki nilai yang sama dengan 5. Kami menempatkan blok kode yang akan dieksekusi jika kondisinya `true` langsung setelah kondisi di dalam kurung kurawal.

Opsional, kita juga dapat menyertakan ekspresi `else`, yang kita pilih lakukan di sini, untuk memberikan program blok kode alternatif untuk dieksekusi jika kondisinya dinilai `false`. Jika Anda tidak menyediakan ekspresi `else` dan kondisinya `false`, program hanya akan melewati blok `if` dan melanjutkan ke bit kode berikutnya.

Coba jalankan kode ini; Anda seharusnya melihat output berikut:

```shell
$ cairo-run main.cairo
[DEBUG]	kondisi salah
```

Mari coba ubah nilai `number` menjadi nilai yang membuat kondisinya menjadi `true` untuk melihat apa yang terjadi:

```rust, noplayground
    let number = 5;
```

```shell
$ cairo-run main.cairo
kondisi benar
```

Perlu dicatat bahwa kondisi dalam kode ini harus berupa tipe data bool. Jika kondisinya bukan bool, kita akan mendapatkan kesalahan.

```shell
$ cairo-run main.cairo
thread 'main' panicked at 'Failed to specialize: `enum_match<felt252>`. Error: Could not specialize libfunc `enum_match` with generic_args: [Type(ConcreteTypeId { id: 1, debug_name: None })]. Error: Provided generic argument is unsupported.', crates/cairo-lang-sierra-generator/src/utils.rs:256:9
```

### Menangani Kondisi Berganda dengan `else if`

Anda dapat menggunakan beberapa kondisi dengan menggabungkan `if` dan `else` dalam ekspresi `else if`. Contohnya:

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_25_else_if/src/lib.cairo}}
```

Program ini memiliki empat jalur yang mungkin dapat diambil. Setelah dijalankan, Anda seharusnya melihat output berikut:

```shell
number is 3
Run completed successfully, returning []
Remaining gas: 1999937120
```

Ketika program ini dijalankan, itu memeriksa setiap ekspresi `if` secara bergantian dan mengeksekusi blok pertama di mana kondisinya dinilai `true`. Perlu diperhatikan bahwa meskipun `number - 2 == 1` adalah `true`, kita tidak melihat output `number minus 2 is 1'` dan juga tidak melihat teks `number not found` dari blok `else`. Hal ini karena Cairo hanya mengeksekusi blok untuk kondisi pertama yang benar, dan begitu menemukannya, ia bahkan tidak memeriksa yang lain. Menggunakan terlalu banyak ekspresi `else if` dapat membuat kode Anda berantakan, jadi jika Anda memiliki lebih dari satu, Anda mungkin ingin melakukan refaktor pada kode Anda. [Bab 6](./ch06-02-the-match-control-flow-construct.md) menjelaskan suatu konstruksi percabangan Cairo yang kuat yang disebut `match` untuk kasus-kasus seperti ini.

### Menggunakan `if` dalam pernyataan `let`

Karena `if` adalah suatu ekspresi, kita dapat menggunakannya di sisi kanan dari pernyataan `let` untuk menetapkan hasilnya ke suatu variabel.

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_26_if_let/src/lib.cairo}}
```

```shell
$ cairo-run main.cairo
kondisi benar dan number adalah 5
Run completed successfully, returning []
Remaining gas: 1999780390
```

Variabel `number` akan diikat ke suatu nilai berdasarkan hasil ekspresi `if`. Yang akan menjadi 5 dalam contoh ini.

### Pengulangan dengan Perulangan

Seringkali berguna untuk menjalankan blok kode lebih dari sekali. Untuk tugas ini, Cairo menyediakan sintaks loop sederhana, yang akan menjalankan kode di dalam tubuh loop hingga selesai, lalu segera memulainya kembali dari awal. Untuk bereksperimen dengan pengulangan, mari buat proyek baru yang disebut loops.

Saat ini, Cairo hanya memiliki satu jenis loop: `loop`.

#### Mengulang Kode dengan `loop`

Kata kunci `loop` memberi tahu Cairo untuk menjalankan blok kode berulang kali
selamanya atau sampai Anda secara eksplisit memberi tahu untuk berhenti.

Sebagai contoh, ubah file _src/lib.cairo_ di direktori _loops_ Anda agar terlihat
seperti ini:

<span class="filename">Nama file: src/lib.cairo</span>

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_27_loop/src/lib.cairo}}
```

Ketika kita menjalankan program ini, kita akan melihat `again!` dicetak terus-menerus
hingga kita menghentikan program secara manual, karena kondisi berhenti tidak pernah tercapai.
Meskipun kompiler mencegah kita menulis program tanpa kondisi berhenti (`break` statement),
kondisi berhenti mungkin tidak pernah tercapai, menghasilkan pengulangan tak terbatas.
Sebagian besar terminal mendukung pintasan keyboard <span class="keystroke">ctrl-c</span> untuk menghentikan program yang terjebak dalam pengulangan terus-menerus. Cobalah:

```shell
$ scarb cairo-run --available-gas=20000000
[DEBUG]	again                          	(raw: 418346264942)

[DEBUG]	again                          	(raw: 418346264942)

[DEBUG]	again                          	(raw: 418346264942)

[DEBUG]	again                          	(raw: 418346264942)

Run panicked with err values: [375233589013918064796019]
Remaining gas: 1050
```

> Catatan: Cairo mencegah kita menjalankan program dengan pengulangan tak terbatas dengan menyertakan meteran gas. Meteran gas adalah mekanisme yang membatasi jumlah komputasi yang dapat dilakukan dalam suatu program. Dengan mengatur nilai ke flag `--available-gas`, kita dapat menetapkan jumlah gas maksimum yang tersedia untuk program. Gas adalah satuan pengukuran yang mengekspresikan biaya komputasi dari suatu instruksi. Ketika meteran gas habis, program akan berhenti. Dalam kasus ini, program mengalami kegagalan karena kehabisan gas, karena kondisi berhenti tidak pernah tercapai.
> Hal ini terutama penting dalam konteks kontrak pintar yang diterapkan di Starknet, karena mencegah dari menjalankan pengulangan tak terbatas di jaringan.
> Jika Anda menulis program yang perlu menjalankan pengulangan, Anda perlu menjalankannya dengan flag `--available-gas` diatur ke nilai yang cukup besar untuk menjalankan program.

Untuk keluar dari loop, Anda dapat menempatkan pernyataan `break` dalam loop untuk memberi tahu program kapan harus berhenti
mengeksekusi loop. Mari perbaiki pengulangan tak terbatas dengan membuat kondisi berhenti `i > 10` dapat dicapai.

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_28_loop_break/src/lib.cairo}}
```

Kata kunci `continue` memberi tahu program untuk melanjutkan ke iterasi berikutnya dari loop dan melewati sisa kode dalam iterasi ini. Mari tambahkan pernyataan `continue` ke loop kami untuk melewati pernyataan `print` ketika `i` sama dengan `5`.

```rust
use core::debug::PrintTrait;
fn main() {
    let mut i: usize = 0;
    loop {
        if i > 10 {
            break;
        }
        if i == 5 {
            i += 1;
            continue;
        }
        i.print();
        i += 1;
    }
}
```

Menjalankan program ini tidak akan mencetak nilai `i` ketika `i` sama dengan `5`.

#### Mengembalikan Nilai dari Loop

Salah satu penggunaan dari `loop` adalah untuk mencoba operasi yang Anda tahu mungkin gagal,
seperti memeriksa apakah operasi tersebut berhasil. Anda mungkin juga perlu meneruskan
hasil dari operasi tersebut keluar dari loop ke bagian lain dari kode Anda. Untuk melakukan
ini, Anda dapat menambahkan nilai yang ingin Anda kembalikan setelah ekspresi `break` yang
Anda gunakan untuk menghentikan loop; nilai tersebut akan dikembalikan dari loop sehingga Anda bisa
menggunakannya, seperti yang ditunjukkan di sini:

```rust
{{#include ../listings/ch02-common-programming-concepts/no_listing_29_loop_return_values/src/lib.cairo}}
```

Sebelum loop, kita mendeklarasikan variabel bernama `counter` dan menginisialisasinya dengan
`0`. Kemudian kita mendeklarasikan variabel bernama `result` untuk menyimpan nilai yang dikembalikan dari
loop. Pada setiap iterasi loop, kita memeriksa apakah `counter` sama dengan `10`, dan kemudian menambahkan `1` ke variabel `counter`.
Ketika kondisi terpenuhi, kita menggunakan kata kunci `break` dengan nilai `counter * 2`. Setelah loop, kita menggunakan titik koma untuk mengakhiri pernyataan yang menetapkan nilai ke `result`. Akhirnya, kita
mencetak nilai di `result`, yang dalam hal ini adalah `20`.

## Ringkasan

Anda berhasil! Ini adalah bab yang cukup besar: Anda belajar tentang variabel, tipe data, fungsi, komentar,
ekspresi `if`, dan loop! Untuk berlatih dengan konsep-konsep yang dibahas dalam bab ini,
cobalah membangun program untuk melakukan hal-hal berikut:

- Menghasilkan bilangan Fibonacci ke-_n_-th.
- Menghitung faktorial dari suatu bilangan _n_.

Selanjutnya, kita akan meninjau jenis koleksi umum di Cairo pada bab berikutnya.