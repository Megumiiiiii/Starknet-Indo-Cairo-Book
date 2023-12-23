## Hello, World!

Sekarang setelah Anda telah menginstal Cairo melalui Scarb, saatnya untuk menulis program Cairo pertama Anda.
Saat belajar bahasa pemrograman baru, adalah tradisi untuk menulis program kecil yang mencetak teks `Hello, world!` ke layar, jadi kita akan melakukan hal yang sama di sini!

> Catatan: Buku ini mengasumsikan pemahaman dasar tentang baris perintah. Cairo tidak membuat
> permintaan spesifik tentang pengeditan atau alat-alat Anda atau di mana kode Anda berada, jadi
> jika Anda lebih suka menggunakan lingkungan pengembangan terpadu (IDE) daripada
> baris perintah, silakan gunakan IDE favorit Anda. Tim Cairo telah mengembangkan
> ekstensi VSCode untuk bahasa Cairo yang dapat Anda gunakan untuk mendapatkan fitur dari
> server bahasa dan penyorotan kode. Lihat [Lampiran D][devtools]
> untuk lebih banyak detail.

### Membuat Direktori Proyek

Anda akan memulai dengan membuat direktori untuk menyimpan kode Cairo Anda. Tidak masalah
bagi Cairo di mana kode Anda berada, tetapi untuk latihan dan proyek dalam buku ini,
kami menyarankan untuk membuat direktori _cairo_projects_ di direktori rumah Anda dan menyimpan semua
proyek Anda di sana.

Buka terminal dan masukkan perintah berikut untuk membuat direktori _cairo_projects_
dan sebuah direktori untuk proyek "Hello, world!" di dalam direktori _cairo_projects_.

> Catatan: Mulai sekarang, untuk setiap contoh yang ditunjukkan dalam buku ini, kami mengasumsikan bahwa
> Anda akan bekerja dari direktori proyek Scarb. Jika Anda tidak menggunakan Scarb, dan mencoba menjalankan contoh dari direktori yang berbeda, Anda mungkin perlu menyesuaikan perintahnya atau membuat proyek Scarb.

Untuk Linux, macOS, dan PowerShell di Windows, masukkan ini:

```shell
mkdir ~/cairo_projects
cd ~/cairo_projects
```

Untuk Windows CMD, masukkan ini:

```cmd
> mkdir "%USERPROFILE%\cairo_projects"
> cd /d "%USERPROFILE%\cairo_projects"
```

### Membuat Proyek dengan Scarb

Mari buat proyek baru menggunakan Scarb.

Beralihlah ke direktori proyek Anda (atau di mana pun Anda memutuskan untuk menyimpan kode Anda). Kemudian jalankan yang berikut:

```bash
scarb new hello_world
```

Ini membuat direktori dan proyek baru yang disebut `hello_world`. Kami menamai proyek kami `hello_world`, dan Scarb membuat file-filenya di dalam direktori dengan nama yang sama.

Masuklah ke direktori `hello_world` dengan perintah `cd hello_world`. Anda akan melihat bahwa Scarb telah menghasilkan dua file dan satu direktori untuk kita: file `Scarb.toml` dan direktori src dengan file `lib.cairo` di dalamnya.

Ini juga menginisialisasi repositori Git baru bersama dengan file `.gitignore`

> Catatan: Git adalah sistem kontrol versi umum. Anda dapat berhenti menggunakan sistem kontrol versi dengan menggunakan opsi `--vcs` flag.
> Jalankan `scarb new -help` untuk melihat opsi yang tersedia.

Buka _Scarb.toml_ di editor teks pilihan Anda. Seharusnya terlihat mirip dengan kode pada Listing 1-2.

<span class="filename">Nama File: Scarb.toml</span>

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2023_10"

# Lihat lebih banyak kunci dan definisi mereka di https://docs.swmansion.com/scarb/docs/reference/manifest

[dependencies]
# foo = { path = "vendor/foo" }
```

<span class="caption">Listing 1-2: Isi dari Scarb.toml yang dihasilkan oleh `scarb new`</span>

File ini menggunakan format [TOML](https://toml.io/) (Tom's Obvious, Minimal Language), yang merupakan format konfigurasi Scarb.

Baris pertama, `[package]`, adalah heading bagian yang menunjukkan bahwa pernyataan-pernyataan berikutnya mengonfigurasi sebuah paket. Ketika kami menambahkan informasi lebih lanjut ke file ini, kami akan menambahkan bagian-bagian lainnya.

Tiga baris berikutnya mengatur informasi konfigurasi yang dibutuhkan Scarb untuk mengompilasi program Anda: nama dan versi Scarb yang digunakan, dan edisi prelude yang digunakan. Prelude adalah kumpulan item yang paling sering digunakan yang secara otomatis diimpor ke setiap program Cairo. Anda dapat mempelajari lebih lanjut tentang prelude di [Lampiran E](./appendix-05-common-types-and-traits-and-cairo-prelude.md)

Baris terakhir, `[dependencies]`, adalah awal dari sebuah bagian di mana Anda dapat mencantumkan dependensi-proyek Anda. Dalam Cairo, paket kode disebut sebagai crates. Kami tidak akan membutuhkan crates lain untuk proyek ini.

> Catatan: Jika Anda membangun kontrak untuk Starknet, Anda perlu menambahkan dependensi `starknet` sebagaimana disebutkan dalam [dokumentasi Scarb](https://docs.swmansion.com/scarb/docs/extensions/starknet/starknet-package.html).

File lain yang dibuat oleh Scarb adalah `src/lib.cairo`, mari kita hapus semua isinya dan masukkan konten berikut, kita akan jelaskan alasannya nanti.

```rust,noplayground
mod hello_world;
```

Kemudian buat file baru bernama `src/hello_world.cairo` dan masukkan kode berikut di dalamnya:

<span class="filename">Nama File: src/hello_world.cairo</span>

```rust,file=hello_world.cairo
fn main() {
    println!("Hello, World!");
}
```

Kami baru saja membuat file bernama `lib.cairo`, yang berisi deklarasi modul yang merujuk ke modul lain bernama `hello_world`, serta file `hello_world.cairo`, yang berisi detail implementasi dari modul `hello_world`.

Scarb mengharuskan file sumber Anda berada dalam direktori `src`.

Direktori proyek tingkat atas disediakan untuk file README, informasi lisensi, file konfigurasi, dan konten lain yang tidak berhubungan dengan kode.
Scarb memastikan lokasi tertentu untuk semua komponen proyek, menjaga organisasi yang terstruktur.

Jika Anda memulai proyek yang tidak menggunakan Scarb, Anda dapat

 mengonversinya menjadi proyek yang menggunakan Scarb. Pindahkan kode proyek ke dalam direktori src dan buat file `Scarb.toml` yang sesuai.

### Membangun Proyek Scarb

Dari direktori `hello_world` Anda, bangun proyek Anda dengan memasukkan perintah berikut:

```bash
$ scarb build
   Mengerjakan hello_world v0.1.0 (file:///projects/Scarb.toml)
    Menyelesaikan target rilis dalam 0 detik
```

Perintah ini membuat file `sierra` di `target/dev`, mari abaikan file `sierra` untuk saat ini.

Jika Anda telah menginstal Cairo dengan benar, Anda seharusnya dapat menjalankan dan melihat keluaran berikut:

```shell
$ scarb cairo-run --available-gas=200000000
menjalankan hello_world ...
[DEBUG] Hello, World!                   (raw: 0x48656c6c6f2c20776f726c6421

Pekerjaan selesai dengan sukses, mengembalikan []
```

Tidak peduli sistem operasi Anda, string `Hello, world!` harus tercetak di
terminal.

Jika `Hello, world!` tercetak, selamat! Anda telah resmi menulis program Cairo.
Itu membuat Anda seorang pemrogram Cairo—selamat datang!

### Anatomi Program Cairo

Mari kita tinjau program "Hello, world!" ini secara detail. Berikut adalah potongan pertama
dari teka-teki ini:

```rust,noplayground
fn main() {

}
```

Baris ini mendefinisikan fungsi bernama `main`. Fungsi `main` adalah istimewa: ini
selalu adalah kode pertama yang dijalankan dalam setiap program Cairo yang dapat dieksekusi. Di sini, baris pertama mendeklarasikan fungsi bernama `main` yang tidak memiliki parameter dan tidak mengembalikan apa pun. Jika ada parameter, mereka akan dimasukkan ke dalam tanda kurung `()`.

Badan fungsi dibungkus dalam `{}`. Cairo memerlukan kurung kurawal di sekitar semua
badan fungsi. Adalah gaya yang baik untuk menempatkan kurung kurawal pembuka di baris yang sama dengan deklarasi fungsi, menambahkan satu spasi di antaranya.

> Catatan: Jika Anda ingin tetap menggunakan gaya standar di seluruh proyek Cairo, Anda dapat
> menggunakan alat pemformat otomatis yang tersedia dengan `scarb fmt` untuk memformat kode Anda dalam
> gaya tertentu (lebih lanjut tentang `scarb fmt` di
> [Lampiran D][devtools]). Tim Cairo telah menyertakan alat ini
> dengan distribusi standar Cairo, seperti `cairo-run`, jadi seharusnya sudah terinstal di komputer Anda!

Badan fungsi `main` memiliki kode berikut:

```rust,noplayground
    println!("Hello, World!");
```

Baris ini melakukan seluruh pekerjaan dalam program kecil ini: mencetak teks ke
layar. Ada empat detail penting yang perlu diperhatikan di sini.

Pertama, gaya Cairo adalah dengan indentasi empat spasi, bukan tab.

Kedua, `println!` memanggil sebuah makro Cairo. Jika ini memanggil sebuah fungsi, itu akan dimasukkan sebagai `println` (tanpa `!`). Kami akan membahas makro Cairo dengan lebih detail di [Bab Makro](./ch11-02-macros.md). Untuk saat ini, yang perlu Anda ketahui adalah penggunaan `!` berarti Anda memanggil sebuah makro bukan fungsi normal dan bahwa makro tidak selalu mengikuti aturan yang sama dengan fungsi.

Ketiga, Anda melihat string `"Hello, world!"`. Kami meneruskan string ini sebagai argumen ke `println!`, dan string tersebut dicetak ke layar.

Keempat, kami mengakhiri baris dengan titik koma (`;`), yang menunjukkan bahwa ini
ekspresi telah selesai dan yang berikutnya siap dimulai. Sebagian besar baris kode Cairo
diakhiri dengan titik koma.

[devtools]: appendix-04-useful-development-tools.md

### Menjalankan Tes

Untuk menjalankan semua tes yang terkait dengan paket tertentu, Anda dapat menggunakan perintah `scarb test`.
Ini bukanlah runner tes itu sendiri, melainkan mendelegasikan pekerjaan ke solusi pengujian yang dipilih. Scarb dilengkapi dengan ekstensi `scarb cairo-test` yang telah terinstal sebelumnya, yang menggabungkan runner tes asli Cairo. Ini adalah runner tes default yang digunakan oleh scarb test.
Untuk menggunakan runner tes pihak ketiga, silakan lihat [dokumentasi Scarb](https://docs.swmansion.com/scarb/docs/extensions/testing.html#using-third-party-test-runners).

Fungsi-fungsi tes ditandai dengan atribut `#[test]`, dan menjalankan `scarb test` akan menjalankan semua fungsi tes dalam basis kode Anda di bawah direktori `src/`.

```txt
├── Scarb.toml
├── src
│   ├── lib.cairo
│   └── file.cairo
```

<span class="caption"> Struktur proyek Scarb contoh</span>

Mari kita ringkas apa yang telah kita pelajari tentang Scarb sejauh ini:

- Kita dapat membuat proyek menggunakan `scarb new`.
- Kita dapat membangun proyek menggunakan `scarb build` untuk menghasilkan kode Sierra yang sudah dikompilasi.
- Kita dapat mendefinisikan skrip kustom di `Scarb.toml` dan memanggilnya dengan perintah `scarb run`.
- Kita dapat menjalankan tes menggunakan perintah `scarb test`.

Keuntungan tambahan dari menggunakan Scarb adalah bahwa perintahnya sama tidak peduli pada sistem operasi mana Anda bekerja. Jadi, pada titik ini, kami tidak akan lagi memberikan instruksi spesifik untuk Linux dan macOS versus Windows.

# Ringkasan

Anda telah memulai perjalanan Cairo Anda dengan sangat baik! Di bab ini, Anda telah belajar bagaimana:

- Menginstal versi stabil terbaru dari Cairo
- Menulis dan menjalankan program "Hello, Scarb!" menggunakan `scarb` langsung
- Membuat dan menjalankan proyek baru menggunakan konvensi Scarb
- Melakukan tes menggunakan perintah `scarb test`

Ini adalah waktu yang tepat untuk membangun program yang lebih besar untuk terbiasa membaca dan menulis kode Cairo.
