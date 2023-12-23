## Kamus

Cairo menyediakan dalam perpustakaan intinya tipe data mirip kamus. Tipe data `Felt252Dict<T>` mewakili kumpulan pasangan kunci-nilai di mana setiap kunci bersifat unik dan terkait dengan nilai yang sesuai. Tipe struktur data ini dikenal dengan nama berbeda dalam berbagai bahasa pemrograman seperti peta, tabel hash, larik asosiatif, dan banyak lainnya.

Tipe `Felt252Dict<T>` berguna ketika Anda ingin mengorganisir data Anda dengan cara tertentu di mana menggunakan `Array<T>` dan pengindeksan tidak cukup. Kamus Cairo juga memungkinkan pemrogram untuk dengan mudah mensimulasikan keberadaan memori yang dapat diubah ketika tidak ada.

### Penggunaan Dasar Kamus

Biasanya dalam bahasa lain ketika membuat kamus baru, kita harus mendefinisikan tipe data baik untuk kunci maupun nilai. Di Cairo, tipe kunci dibatasi menjadi `felt252`, meninggalkan hanya kemungkinan untuk menentukan tipe data nilai, yang direpresentasikan oleh `T` dalam `Felt252Dict<T>`.

Fungsionalitas inti dari `Felt252Dict<T>` diimplementasikan dalam trait `Felt252DictTrait` yang mencakup semua operasi dasar. Di antaranya, kita dapat menemukan:

1. `insert(felt252, T) -> ()` untuk menulis nilai ke instance kamus dan
2. `get(felt252) -> T` untuk membaca nilai darinya.

Fungsi-fungsi ini memungkinkan kita untuk memanipulasi kamus seperti dalam bahasa pemrograman lain. Pada contoh berikut, kita membuat kamus untuk merepresentasikan pemetaan antara individu dan saldo mereka:

```rust
{{#include ../listings/ch03-common-collections/no_listing_07_intro/src/lib.cairo}}
```

Kita dapat membuat instance baru dari `Felt252Dict<u64>` dengan menggunakan metode `default` dari trait `Default` dan menambahkan dua individu, masing-masing dengan saldo mereka sendiri, menggunakan metode `insert`. Akhirnya, kita memeriksa saldo pengguna dengan metode `get`. Metode-metode ini didefinisikan dalam trait `Felt252DictTrait` dalam perpustakaan inti.

Sepanjang buku ini, kita telah membahas bagaimana memori Cairo bersifat tidak dapat diubah, yang berarti Anda hanya dapat menulis ke sel memori sekali, tetapi tipe `Felt252Dict<T>` mewakili cara untuk mengatasi hambatan ini. kita akan menjelaskan bagaimana hal ini diimplementasikan nanti pada bagian [Dictionaries Underneath](#dictionaries-underneath).

Berlanjut dari contoh sebelumnya, mari tunjukkan contoh kode di mana saldo pengguna yang sama berubah:

```rust
{{#include ../listings/ch03-common-collections/no_listing_08_intro_rewrite/src/lib.cairo}}
```

Perhatikan bagaimana dalam contoh ini kita menambahkan individu _Alex_ dua kali, setiap kali menggunakan saldo yang berbeda dan setiap kali kita memeriksa saldo, itu memiliki nilai terakhir yang dimasukkan! `Felt252Dict<T>` efektif memungkinkan kita "menulis ulang" nilai yang disimpan untuk kunci apa pun.

Sebelum melanjutkan dan menjelaskan bagaimana kamus diimplementasikan, penting untuk dicatat bahwa begitu Anda menginisiasi `Felt252Dict<T>`, di belakang layar semua kunci memiliki nilai yang terkait diinisialisasi sebagai nol. Ini berarti bahwa jika misalnya Anda mencoba mendapatkan saldo pengguna yang tidak ada, Anda akan mendapatkan 0 daripada kesalahan atau nilai yang tidak terdefinisi. Ini juga berarti tidak ada cara untuk menghapus data dari kamus. Sesuatu yang perlu diperhatikan ketika menggabungkan struktur ini ke dalam kode Anda.

Hingga saat ini, kita telah melihat semua fitur dasar dari `Felt252Dict<T>` dan bagaimana ia meniru perilaku yang sama seperti struktur data yang sesuai dalam bahasa pemrograman lainnya, yaitu, secara eksternal tentu saja. Cairo pada intinya adalah bahasa pemrograman yang non-deterministik dan dapat diselesaikan oleh Turing, sangat berbeda dari bahasa populer lainnya yang ada, yang sebagai konsekuensinya berarti bahwa kamus diimplementasikan dengan cara yang sangat berbeda juga!

Pada bagian-bagian berikutnya, kita akan memberikan beberapa wawasan tentang mekanisme internal `Felt252Dict<T>` dan kompromi yang diambil untuk membuatnya berfungsi. Setelah itu, kita akan melihat bagaimana menggunakan kamus dengan struktur data lain serta menggunakan metode `entry` sebagai cara lain untuk berinteraksi dengan mereka.

### Kamus di Dalamnya

Salah satu kendala dari desain non-deterministik Cairo adalah bahwa sistem memorinya bersifat tidak dapat diubah, sehingga untuk mensimulasikan mutabilitas, bahasa ini mengimplementasikan `Felt252Dict<T>` sebagai daftar entri. Setiap entri mewakili waktu ketika kamus diakses untuk tujuan membaca/memperbarui/menulis. Sebuah entri memiliki tiga bidang:

1. Bidang `key` yang mengidentifikasi nilai untuk pasangan kunci-nilai kamus ini.
2. Bidang `previous_value` yang menunjukkan nilai sebelumnya yang dipegang di `key`.
3. Bidang `new_value` yang menunjukkan nilai baru yang dipegang di `key`.

Jika kita mencoba mengimplementasikan `Felt252Dict<T>` menggunakan struktur tingkat tinggi, kita akan mendefinisikannya secara internal sebagai `Array<Entry<T>>` di mana setiap `Entry<T>` memiliki informasi tentang pasangan kunci-nilai yang direpresentasikannya dan nilai-nilai sebelumnya dan baru yang dipegangnya. Definisi dari `Entry<T>` akan menjadi sebagai berikut:

```rust,noplayground
{{#include ../listings/ch03-common-collections/no_listing_09_entries/src/lib.cairo:struct}}
```

Setiap kali kita berinteraksi dengan `Felt252Dict<T>`, entri baru `Entry<T>` akan terdaftar:

- Sebuah `get` akan mendaftarkan entri di mana tidak ada perubahan dalam status, dan nilai-nilai sebelumnya dan baru disimpan dengan nilai yang sama.
- Sebuah `insert` akan mendaftarkan `Entry<T>` baru di mana `new_value` akan menjadi elemen yang dimasukkan, dan `previous_value` adalah elemen terakhir yang dimasukkan sebelumnya. Jika ini adalah entri pertama untuk kunci tertentu, maka nilai sebelumnya akan menjadi nol.

Penggunaan daftar entri ini menunjukkan bagaimana tidak ada penulisan ulang, hanya pembuatan sel memori baru per interaksi `Felt252Dict<T>`. Mari tunjukkan contoh penggunaan kamus `balances` dari bagian sebelumnya dan menyisipkan pengguna 'Alex' dan 'Maria':

```rust
{{#rustdoc_include ../listings/ch03-common-collections/no_listing_09_entries/src/lib.cairo:inserts}}
```

Instruksi-instruksi ini kemudian akan menghasilkan daftar entri berikut:

|  kunci | sebelumnya | baru |
| :----: | --------- | --- |
| Alex   | 0         | 100 |
| Maria  | 0         | 50  |
| Alex   | 100       | 200 |
| Maria  | 50        | 50  |

Perhatikan bahwa karena 'Alex' dimasukkan dua kali, itu muncul dua kali dan nilai `sebelumnya` dan `saat ini` diatur dengan benar. Juga membaca dari 'Maria' mendaftarkan entri tanpa perubahan dari nilai sebelumnya ke nilai saat ini.

Pendekatan ini untuk mengimplementasikan `Felt252Dict<T>` berarti bahwa untuk setiap operasi baca/tulis, ada pemindaian seluruh daftar entri dalam pencarian entri terakhir dengan kunci yang sama. Begitu entri telah ditemukan, `new_value`-nya diekstrak dan digunakan pada entri baru yang akan ditambahkan sebagai `previous_value`. Ini berarti berinteraksi dengan `Felt252Dict<T>` memiliki kompleksitas waktu terburuk `O(n)` di mana `n` adalah jumlah entri dalam daftar.

Jika Anda memikirkan cara alternatif untuk mengimplementasikan `Felt252Dict<T>`, Anda pasti akan menemukannya, mungkin bahkan meninggalkan sepenuhnya kebutuhan untuk bidang `previous_value`, meskipun, karena Cairo bukanlah bahasa normal Anda, hal ini tidak akan berfungsi. Salah satu tujuan Cairo adalah, dengan sistem bukti STARK, menghasilkan bukti integritas komputasional. Ini berarti Anda perlu memverifikasi bahwa eksekusi program benar dan berada dalam batas pembatasan Cairo. Salah satu pemeriksaan batas tersebut terdiri dari "penghancuran kamus" dan itu memerlukan informasi tentang nilai-nilai sebelumnya dan baru untuk setiap entri.

### Penghancuran Kamus

Untuk memverifikasi bahwa bukti yang dihasilkan oleh eksekusi program Cairo yang menggunakan `Felt252Dict<T>` benar, kita perlu memeriksa bahwa tidak ada manipulasi ilegal dengan kamus. Ini dilakukan melalui metode yang disebut `squash_dict` yang meninjau setiap entri dari daftar entri dan memeriksa bahwa akses ke kamus tetap konsisten sepanjang eksekusi.

Proses penghancuran dilakukan sebagai berikut: diberikan semua entri dengan kunci tertentu `k`, diambil dalam urutan yang sama dengan saat dimasukkan, verifikasi bahwa nilai `new_value` pada entri ke-i sama dengan nilai `previous_value` pada entri ke-i + 1.

Sebagai contoh, diberikan daftar entri berikut:

|   kunci   | sebelumnya | baru |
| :-------: | ---------- | --- |
|   Alex    | 0          | 150 |
|   Maria   | 0          | 100 |
| Charles   | 0          | 70  |
|   Maria   | 100        | 250 |
|   Alex    | 150        | 40  |
|   Alex    | 40         | 300 |
|   Maria   | 250        | 190 |
|   Alex    | 300        | 90  |

Setelah penghancuran, daftar entri akan dikurangi menjadi:

|   kunci   | sebelumnya | baru |
| :-------: | ---------- | --- |
|   Alex    | 0          | 90  |
|   Maria   | 0          | 190 |
| Charles   | 0          | 70  |

Jika terjadi perubahan pada salah satu nilai tabel pertama, penghancuran akan gagal selama runtime.

### Penghancuran Kamus

Jika Anda menjalankan contoh dari [Penggunaan Dasar Kamus](#basic-use-of-dictionaries), Anda akan melihat bahwa tidak pernah ada panggilan untuk menyusun kamus, namun program berhasil dikompilasi. Apa yang terjadi di belakang layar adalah bahwa penyusunan secara otomatis dipanggil melalui implementasi `Felt252Dict<T>` dari trait `Destruct<T>`. Panggilan ini terjadi tepat sebelum kamus `balance` keluar dari cakupan.

Trait `Destruct<T>` mewakili cara lain untuk mengeluarkan instansi dari cakupan selain `Drop<T>`. Perbedaan utama antara keduanya adalah bahwa `Drop<T>` dianggap sebagai operasi no-op, yang berarti tidak menghasilkan CASM baru sementara `Destruct<T>` tidak memiliki batasan ini. Satu-satunya tipe yang secara aktif menggunakan trait `Destruct<T>` adalah `Felt252Dict<T>`, untuk setiap tipe lainnya, `Destruct<T>` dan `Drop<T>` adalah sinonim. Anda dapat membaca lebih lanjut tentang trait ini di [Drop and Destruct](/appendix-03-derivable-traits.md#drop-and-destruct).

Nanti, pada bagian [Kamus sebagai Anggota Struktur](#dictionaries-as-struct-members), kita akan memiliki contoh praktis di mana kita mengimplementasikan trait `Destruct<T>` untuk tipe kustom.

### Lebih Banyak Kamus

Hingga saat ini, kita telah memberikan gambaran menyeluruh tentang fungsionalitas `Felt252Dict<T>` serta bagaimana dan mengapa itu diimplementasikan dengan cara tertentu. Jika Anda belum memahami semuanya, jangan khawatir karena dalam bagian ini kita akan memberikan beberapa contoh lebih lanjut menggunakan kamus.

kita akan memulai dengan menjelaskan metode `entry` yang merupakan bagian dari fungsionalitas dasar kamus yang disertakan dalam `Felt252DictTrait<T>` yang tidak kita sebutkan di awal. Segera setelah itu, kita akan melihat contoh bagaimana `Felt252Dict<T>` [berinteraksi](#dictionaries-of-complex-types) dengan jenis kompleks lain seperti `Array<T>` dan bagaimana [mengimplementasikan](#dictionaries-as-struct-members) sebuah struktur dengan kamus sebagai anggota.

### Entry dan Finalisasi

Pada bagian [Dictionaries Underneath](#dictionaries-underneath), kita menjelaskan bagaimana `Felt252Dict<T>` bekerja secara internal. Ini adalah daftar entri untuk setiap kali kamus diakses dengan cara apa pun. Pertama, ia akan menemukan entri terakhir dengan memberikan `key` tertentu, dan kemudian memperbarui sesuai dengan operasi apa pun yang sedang dijalankan. Bahasa Cairo memberi kita alat untuk mereplikasikan ini sendiri melalui metode `entry` dan `finalize`.

Metode `entry` datang sebagai bagian dari `Felt252DictTrait<T>` dengan tujuan membuat entri baru dengan memberikan kunci tertentu. Begitu dipanggil, metode ini mengambil kepemilikan kamus dan mengembalikan entri untuk diperbarui. Tanda tangan metodenya adalah sebagai berikut:

```rust,noplayground
fn entry(self: Felt252Dict<T>, key: felt252) -> (Felt252DictEntry<T>, T) nopanic
```

Parameter masukan pertama mengambil kepemilikan kamus sementara yang kedua digunakan untuk membuat entri yang sesuai. Ini mengembalikan tuple yang berisi `Felt252DictEntry<T>`, yang merupakan tipe yang digunakan oleh Cairo untuk mewakili entri kamus, dan `T` yang mewakili nilai yang dipegang sebelumnya.

Langkah berikutnya adalah memperbarui entri dengan nilai baru. Untuk ini, kita menggunakan metode `finalize` yang menyisipkan entri dan mengembalikan kepemilikan kamus:

```rust,noplayground
fn finalize(self: Felt252DictEntry<T>, new_value: T) -> Felt252Dict<T> {
```

Metode ini menerima entri dan nilai baru sebagai parameter dan mengembalikan kamus yang diperbarui.

Mari kita lihat contoh penggunaan `entry` dan `finalize`. Bayangkan kita ingin mengimplementasikan versi kustom dari metode `get` dari kamus. Maka kita harus melakukan hal berikut:

1. Buat entri baru untuk ditambahkan menggunakan metode `entry`.
2. Sisipkan kembali entri di mana `new_value` sama dengan `previous_value`.
3. Kembalikan nilai.

Implementasi `get` kustom kita akan terlihat seperti ini:

```rust,noplayground
{{#include ../listings/ch03-common-collections/no_listing_10_custom_methods/src/lib.cairo:imports}}

{{#include ../listings/ch03-common-collections/no_listing_10_custom_methods/src/lib.cairo:custom_get}}
```

Implementasi metode `insert` akan mengikuti alur kerja yang serupa, kecuali untuk menyisipkan nilai baru saat finalisasi. Jika kita hendak mengimplementasikannya, itu akan terlihat seperti berikut:

```rust,noplayground
{{#include ../listings/ch03-common-collections/no_listing_10_custom_methods/src/lib.cairo:imports}}

{{#include ../listings/ch03-common-collections/no_listing_10_custom_methods/src/lib.cairo:custom_insert}}
```

Sebagai catatan penutup, kedua metode ini diimplementasikan dengan cara yang mirip dengan bagaimana `insert` dan `get` diimplementasikan untuk `Felt252Dict<T>`. Kode ini menunjukkan beberapa penggunaan contoh:

```rust
{{#rustdoc_include ../listings/ch03-common-collections/no_listing_10_custom_methods/src/lib.cairo:main}}
```

### Kamus dari Jenis yang Tidak Didukung Secara Asli

Salah satu pembatasan dari `Felt252Dict<T>` yang belum kita bahas adalah trait `Felt252DictValue<T>`.
Trait ini mendefinisikan metode `zero_default` yang dipanggil ketika nilai tidak ada dalam kamus.
Ini diimplementasikan oleh beberapa tipe data umum, seperti sebagian besar bilangan bulat tak bertanda, `bool`, dan `felt252`, tetapi tidak diimplementasikan untuk tipe yang lebih kompleks seperti larik, struktur (termasuk `u256`), dan tipe lain dari perpustakaan inti.
Ini berarti membuat kamus dari jenis yang tidak didukung secara alami tidaklah tugas yang mudah, karena Anda perlu menulis beberapa implementasi trait untuk membuat tipe data menjadi tipe nilai kamus yang valid.
Sebagai gantinya, Anda dapat melingkupi tipe Anda dalam `Nullable<T>`.

`Nullable<T>` adalah tipe penunjuk pintar yang dapat menunjuk pada nilai atau menjadi `null` dalam ketiadaan nilai. Biasanya digunakan dalam Bahasa Pemrograman Berorientasi Objek ketika referensi tidak menunjuk ke mana-mana. Perbedaannya dengan `Option` adalah bahwa nilai yang dilingkupi disimpan di dalam tipe data `Box<T>`. Tipe `Box<T>`, terinspirasi oleh Rust, memungkinkan kita mengalokasikan segmen memori baru untuk tipe kita, dan mengakses segmen ini menggunakan penunjuk yang hanya dapat dimanipulasi di satu tempat pada satu waktu.

Mari tunjukkan dengan contoh. kita akan mencoba menyimpan `Span<felt252>` di dalam kamus. Untuk itu, kita akan menggunakan `Nullable<T>` dan `Box<T>`. Juga, kita menyimpan `Span<T>` dan bukan `Array<T>` karena yang terakhir tidak mengimplementasikan trait `Copy<T>` yang diperlukan untuk membaca dari kamus.

```rust,noplayground
{{#include ../listings/ch03-common-collections/no_listing_11_dict_of_complex/src/lib.cairo:imports}}

{{#include ../listings/ch03-common-collections/no_listing_11_dict_of_complex/src/lib.cairo:header}}

//...
```

Dalam potongan kode ini, hal pertama yang kita lakukan adalah membuat kamus baru `d`. kita ingin kamus ini menyimpan `Nullable<Span>`. Setelah itu, kita membuat larik dan mengisinya dengan nilai.

Langkah terakhir adalah menyisipkan larik sebagai span ke dalam kamus. Perhatikan bahwa kita tidak melakukan itu secara langsung, tetapi sebaliknya, kita mengambil beberapa langkah di antaranya:

1. kita melingkupi larik dalam `Box` menggunakan metode `new` dari `BoxTrait`.
2. kita melingkupi `Box` dalam bentuk yang dapat dinyatakan nol menggunakan fungsi `nullable_from_box`.
3. Akhirnya, kita menyisipkan hasilnya.

Setelah elemen berada di dalam kamus, dan kita ingin mendapatkannya, kita ikuti langkah yang sama tetapi dalam urutan terbalik. Kode berikut menunjukkan bagaimana melakukannya:

```rust,noplayground
//...

{{#include ../listings/ch03-common-collections/no_listing_11_dict_of_complex/src/lib.cairo:footer}}
```

Di sini kita:

1. Membaca nilai menggunakan `get`.
2. Memverifikasi bahwa itu tidak null menggunakan fungsi `match_nullable`.
3. Membuka nilai di dalam kotak dan memastikan bahwa itu benar.

Skrip lengkap akan terlihat seperti ini:

```rust
{{#include ../listings/ch03-common-collections/no_listing_11_dict_of_complex/src/lib.cairo:all}}
```

### Kamus sebagai Anggota Struct

Menentukan kamus sebagai anggota struct memungkinkan di Cairo, tetapi berinteraksi dengan mereka mungkin tidak sepenuhnya lancar. Mari mencoba mengimplementasikan _database pengguna_ kustom yang akan memungkinkan kita menambahkan pengguna dan mengajukannya. Kita perlu menentukan struct untuk mewakili tipe baru dan trait untuk menentukan fungsinya:

```rust,noplayground
{{#include ../listings/ch03-common-collections/no_listing_12_dict_struct_member/src/lib.cairo:struct}}

{{#include ../listings/ch03-common-collections/no_listing_12_dict_struct_member/src/lib.cairo:trait}}
```

Tipe baru kita `UserDatabase<T>` mewakili database pengguna. Ini generic terhadap saldo pengguna, memberikan fleksibilitas utama kepada siapa pun yang menggunakan tipe data kita. Dua anggotanya adalah:

- `users_amount`, jumlah pengguna yang saat ini dimasukkan, dan
- `balances`, pemetaan setiap pengguna ke saldo mereka.

Fungsionalitas inti database didefinisikan oleh `UserDatabaseTrait`. Metode-metode berikut didefinisikan:

- `new` untuk membuat tipe `UserDatabase` baru dengan mudah.
- `add_user` untuk menyisipkan pengguna ke dalam database.
- `get_balance` untuk menemukan saldo pengguna di dalam database.

Langkah terakhir yang tersisa adalah mengimplementasikan masing-masing metode di `UserDatabaseTrait`, tetapi karena kita bekerja dengan [jenis generik](/src/ch08-00-generic-types-and-traits.md), kita juga perlu menetapkan persyaratan `T` dengan benar sehingga itu dapat menjadi nilai tipe `Felt252Dict<T>` yang valid:

1. `T` harus mengimplementasikan `Copy<T>` karena diperlukan untuk mendapatkan nilai dari `Felt252Dict<T>`.
2. Semua jenis nilai dari kamus mengimplementasikan `Felt252DictValue<T>`, tipe generic kita juga harus melakukannya.
3. Untuk menyisipkan nilai, `Felt252DictTrait<T>` mengharuskan semua jenis nilai untuk dapat dihancurkan.

Implementasinya, dengan semua pembatasan di tempat, akan sebagai berikut:

```rust,noplayground
{{#include ../listings/ch03-common-collections/no_listing_12_dict_struct_member/src/lib.cairo:imports}}

{{#include ../listings/ch03-common-collections/no_listing_12_dict_struct_member/src/lib.cairo:impl}}
```

Implementasi database kita hampir selesai, kecuali satu hal: kompiler tidak tahu bagaimana membuat `UserDatabase<T>` keluar dari cakupan, karena tidak mengimplementasikan trait `Drop<T>` atau trait `Destruct<T>`. Karena memiliki `Felt252Dict<T>` sebagai anggota, itu tidak dapat dihapus, jadi kita terpaksa mengimplementasikan trait `Destruct<T>` secara manual (lihat bab [Milik](ch04-01-what-is-ownership.md#the-drop-trait) untuk informasi lebih lanjut). Menggunakan `#[derive(Destruct)]` di atas definisi `UserDatabase<T>` tidak akan berhasil karena penggunaan [generik](/src/ch08-00-generic-types-and-traits.md) dalam definisi struct. Kita perlu menulis implementasi trait `Destruct<T>` sendiri:

```rust,noplayground
{{#include ../listings/ch03-common-collections/no_listing_12_dict_struct_member/src/lib.cairo:destruct}}
```

Mengimplementasikan `Destruct<T>` untuk `UserDatabase` adalah langkah terakhir kita untuk mendapatkan database yang sepenuhnya fungsional. Sekarang kita bisa mencobanya:

```rust
{{#rustdoc_include ../listings/ch03-common-collections/no_listing_12_dict_struct_member/src/lib.cairo:main}}
```

## Ringkasan

Selamat! Anda telah menyelesaikan bab ini tentang array dan kamus di Cairo. Struktur data ini mungkin sedikit sulit dipahami, tetapi mereka sangat berguna.

Ketika Anda siap untuk melanjutkan, kita akan membahas konsep yang Cairo bagikan dengan Rust dan _tidak biasa_ ada di bahasa pemrograman lain: kepemilikan.
