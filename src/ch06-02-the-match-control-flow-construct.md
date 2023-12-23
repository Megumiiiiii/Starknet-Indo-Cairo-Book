# Konstruksi Aliran Kontrol Match

Cairo memiliki sebuah konstruksi aliran kontrol yang sangat kuat yang disebut `match` yang memungkinkan Anda untuk membandingkan sebuah nilai terhadap serangkaian pola kemudian menjalankan kode berdasarkan pola mana yang cocok. Pola-pola dapat terdiri dari nilai literal, nama variabel, wildcard, dan banyak hal lainnya. Keunggulan dari `match` datang dari ekspresivitas pola-pola dan fakta bahwa kompilator memastikan bahwa semua kasus yang mungkin telah ditangani.

Bayangkan ekspresi match seperti mesin penggolong koin: koin-koin meluncur di jalur dengan lubang-lubang berbagai ukuran di sepanjangnya, dan setiap koin jatuh melalui lubang pertama yang cocok. Dengan cara yang sama, nilai-nilai melewati setiap pola dalam match, dan pada pola pertama yang sesuai dengan nilai, nilai tersebut jatuh ke dalam blok kode yang terkait untuk digunakan selama eksekusi.

Berbicara tentang koin, mari gunakan mereka sebagai contoh menggunakan match! Kita dapat menulis sebuah fungsi yang mengambil koin AS yang tidak diketahui dan, dengan cara yang mirip dengan mesin penghitungan, menentukan jenis koin tersebut dan mengembalikan nilainya dalam sen, seperti yang ditunjukkan dalam Listing 6-3.

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/listing_05_03/src/lib.cairo:all}}
```

Listing 6-3: Sebuah enum dan ekspresi match yang memiliki variant dari enum sebagai polanya

Mari kita bahas `match` dalam fungsi `value_in_cents`. Pertama, kita menyebutkan kata kunci `match` diikuti oleh sebuah ekspresi, yang dalam hal ini adalah nilai `coin`. Ini mirip sekali dengan ekspresi kondisional yang digunakan dengan if, tetapi ada perbedaan besar: dengan if, kondisi harus dievaluasi menjadi nilai Boolean, tetapi di sini bisa berupa tipe apa saja. Tipe dari `coin` dalam contoh ini adalah enum `Coin` yang kita definisikan pada baris pertama.

Selanjutnya adalah `arm` dari `match`. Sebuah `arm` memiliki dua bagian: sebuah pola dan beberapa kode. Arm pertama di sini memiliki pola yang merupakan nilai `Coin::Penny(_)` dan kemudian operator `=>` yang memisahkan antara pola dan kode yang akan dijalankan. Kode dalam hal ini hanyalah nilai `1`. Setiap arm dipisahkan dari yang lain dengan tanda koma.

Ketika ekspresi `match` dieksekusi, ia membandingkan nilai yang dihasilkan dengan pola dari setiap arm, secara berurutan. Jika sebuah pola cocok dengan nilai, kode yang terkait dengan pola tersebut dieksekusi. Jika pola tersebut tidak cocok dengan nilai, eksekusi akan melanjutkan ke arm berikutnya, seperti pada mesin penggolong koin. Kita dapat memiliki sebanyak yang kita butuhkan: dalam contoh di atas, match kita memiliki empat arm.

Di Cairo, urutan arm harus mengikuti urutan yang sama dengan enum.

Kode yang terkait dengan setiap arm adalah sebuah ekspresi, dan nilai hasil dari ekspresi pada arm yang cocok adalah nilai yang akan dikembalikan untuk seluruh ekspresi match.

Biasanya, kita tidak menggunakan kurung kurawal jika kode arm match singkat, seperti pada contoh kami di mana setiap arm hanya mengembalikan nilai. Jika Anda ingin menjalankan beberapa baris kode dalam arm match, Anda harus menggunakan kurung kurawal, dengan koma mengikuti setelah arm. Sebagai contoh, kode berikut mencetak "Lucky penny!" setiap kali metode dipanggil dengan `Coin::Penny`, tetapi masih mengembalikan nilai terakhir dari blok, yaitu `1`:

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/no_listing_04_match_arms/src/lib.cairo:here}}
```

## Pola yang Terikat pada Nilai

Fitur lain yang berguna dari arm match adalah bahwa mereka dapat terikat pada bagian-bagian dari nilai yang cocok dengan pola. Inilah cara kita dapat mengekstrak nilai dari variant enum.

Sebagai contoh, mari ubah salah satu variant enum kita untuk menyimpan data di dalamnya. Dari tahun 1999 hingga 2008, Amerika Serikat memprangkai koin seperempat dengan desain yang berbeda untuk setiap dari 50 negara bagian di satu sisi. Koin lain tidak mendapatkan desain negara bagian, jadi hanya seperempat memiliki nilai tambahan ini. Kita dapat menambahkan informasi ini ke enum kita dengan mengubah variant `Quarter` untuk menyertakan nilai `UsState` yang disimpan di dalamnya, yang kita lakukan dalam Listing 6-4.

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/listing_05_04/src/lib.cairo:enum_def}}
```

Listing 6-4: Sebuah enum `Coin` di mana variant `Quarter` juga menyimpan nilai `UsState`

Bayangkan seorang teman mencoba mengumpulkan semua 50 koin negara bagian. Ketika kita menyortir koin kepingan kepingan berdasarkan jenis koinnya, kita juga akan menyebutkan nama negara bagian yang terkait dengan setiap koin seperempat sehingga jika itu salah satu yang teman kita tidak miliki, mereka bisa menambahkannya ke koleksi mereka.

Pada ekspresi match untuk kode ini, kita menambahkan variabel bernama `state` ke dalam pola yang cocok dengan nilai dari variant `Coin::Quarter`. Ketika sebuah `Coin::Quarter` cocok, variabel `state` akan terikat pada nilai dari negara bagian dari koin tersebut. Kemudian kita dapat menggunakan `state` dalam kode untuk arm tersebut, seperti ini:

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/listing_05_04/src/lib.cairo:function}}
```

Untuk mencetak nilai dari sebuah variant dari enum di Cairo, kita perlu menambahkan implementasi untuk fungsi `print` dari `debug::PrintTrait`:

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/listing_05_04/src/lib.cairo:print_impl}}
```

Jika kita memanggil `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` akan menjadi `Coin::Quarter(UsState::Alaska)`. Ketika kita membandingkan nilai tersebut dengan setiap arm match, tidak ada yang cocok sampai kita mencapai `Coin::Quarter(state)`. Pada titik itu, pengikatan untuk state akan menjadi nilai `UsState::Alaska`. Kita kemudian dapat menggunakan pengikatan itu dalam `PrintTrait`, sehingga mendapatkan nilai dalam variant enum `Quarter`.

## Pencocokan dengan Opsi

Pada bagian sebelumnya, kita ingin mengambil nilai `T` dalam kasus `Some` saat menggunakan `Option<T>`; kita juga dapat menangani `Option<T>` menggunakan `match`, seperti yang kita lakukan dengan enum `Coin`! Alih-alih membandingkan koin, kita akan membandingkan variant dari `Option<T>`, tetapi cara kerja ekspresi `match` tetap sama.

Misalkan kita ingin menulis sebuah fungsi yang mengambil `Option<u8>` dan, jika ada nilai di dalamnya, menambahkan `1` ke nilai tersebut. Jika tidak ada nilai di dalamnya, fungsi tersebut harus mengembalikan nilai `None` dan tidak melakukan operasi apa pun.

Fungsi ini sangat mudah ditulis, berkat `match`, dan akan terlihat seperti pada Listing 6-5.

```rust
{{#include ../listings/ch06-enums-and-pattern-matching/listing_05_05/src/lib.cairo:all}}
```

<span class="caption">Listing 6-5: Sebuah fungsi yang menggunakan ekspresi `match` pada `Option<u8>`</span>

Perhatikan bahwa arm-arm Anda harus mengikuti urutan yang sama dengan enum yang didefinisikan di `OptionTrait` dari lib Cairo inti.

```rust,noplayground
enum Option<T> {
    Some: T,
    None,
}
```

Mari kita periksa eksekusi pertama dari `plus_one` lebih detail. Ketika kita panggil `plus_one(five)`, variabel `x` di dalam tubuh `plus_one` akan memiliki nilai `Some(5)`. Kita kemudian membandingkannya dengan setiap arm match:

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/listing_05_05/src/lib.cairo:option_some}}
```

Apakah nilai `Option::Some(5)` cocok dengan pola `Option::Some(val)`? Iya! Kita memiliki variant yang sama. `val` mengikat pada nilai yang terdapat di dalam `Option::Some`, jadi `val` mengambil nilai `5`. Kode dalam arm match kemudian dieksekusi, sehingga kita menambahkan `1` ke nilai `val` dan membuat nilai `Option::Some` baru dengan total `6` di dalamnya. Karena arm pertama cocok, tidak ada arm lain yang dibandingkan.

Sekarang mari kita pertimbangkan panggilan kedua dari `plus_one` dalam fungsi utama kita, di mana `x` adalah `Option::None`. Kita masuk ke dalam match dan membandingkannya dengan arm pertama:

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/listing_05_05/src/lib.cairo:option_some}}
```

Nilai `Option::Some(val)` tidak cocok dengan pola `Option::None`, jadi kita melanjutkan ke arm berikutnya:

```rust
{{#include ../listings/ch06-enums-and-pattern-matching/listing_05_05/src/lib.cairo:option_none}}
```

Cocok! Tidak ada nilai untuk ditambahkan, sehingga program berhenti dan mengembalikan nilai `Option::None` pada sisi kanan dari `=>`.

Menggabungkan `match` dan enum berguna dalam banyak situasi. Anda akan melihat pola ini banyak digunakan dalam kode Cairo: `match` terhadap sebuah enum, mengikat sebuah variabel pada data di dalamnya, lalu menjalankan kode berdasarkan hal tersebut. Sedikit sulit pada awalnya, tetapi setelah Anda terbiasa, Anda akan berharap memiliki ini dalam semua bahasa. Ini secara konsisten menjadi favorit pengguna.

## Pencocokan Adalah Exhaustive

Ada satu aspek lain dari match yang perlu kita diskusikan: pola-pola di dalam arm harus mencakup semua kemungkinan. Pertimbangkan versi fungsi `plus_one` kita yang memiliki bug dan tidak akan dikompilasi:

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/no_listing_07_missing_match_arm/src/lib.cairo:here}}
```

```bash
$ scarb cairo-run --available-gas=200000000
    error: Unsupported match. Currently, matches require one arm per variant,
    in the order of variant definition.
    --> test.cairo:34:5
        match x {
        ^*******^
    Error: failed to compile: ./src/test.cairo
```

Cairo tahu bahwa kami tidak mencakup setiap kasus yang mungkin, dan bahkan tahu pola mana yang kami lupakan! Matches di Cairo adalah exhaustif: kami harus mencakup setiap kemungkinan terakhir agar kode menjadi valid. Terutama dalam kasus `Option<T>`, ketika Cairo mencegah kita untuk lupa menangani kasus `None` secara eksplisit, ia melindungi kita dari mengasumsikan bahwa kita memiliki nilai ketika kita mungkin memiliki null, sehingga membuat kesalahan miliaran dolar yang dibahas sebelumnya menjadi tidak mungkin.

## Pencocokan 0 dan Operator \_

Dengan menggunakan enum, kita juga dapat melakukan tindakan khusus untuk beberapa nilai tertentu, tetapi untuk semua nilai lainnya melakukan satu tindakan default. Saat ini hanya `0` dan operator `_` yang didukung.

Bayangkan kita sedang mengimplementasikan sebuah permainan di mana Anda mendapatkan nomor acak antara 0 dan 7. Jika Anda mendapatkan 0, Anda menang. Untuk semua nilai lainnya, Anda kalah. Berikut adalah `match` yang menerapkan logika tersebut, dengan nomor diinisialisasi secara langsung daripada menggunakan nilai acak.

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/no_listing_06_match_zero/src/lib.cairo:here}}
```

Pada arm pertama, pola yang digunakan adalah nilai literal 0. Pada arm terakhir yang mencakup semua nilai lainnya, polanya adalah karakter `_`. Kode ini dapat dikompilasi, meskipun kita tidak mencantumkan semua nilai yang mungkin dimiliki oleh `felt252`, karena pola terakhir akan cocok dengan semua nilai yang tidak tercantum secara spesifik. Pola tangkap-semuanya ini memenuhi persyaratan bahwa `match` harus eksaustif. Perlu diingat bahwa kita harus menempatkan arm tangkap-semuanya terakhir karena pola dievaluasi secara berurutan. Jika kita meletakkan arm tangkap-semuanya di awal, arm lainnya tidak akan pernah dijalankan, sehingga Cairo akan memberikan peringatan jika kita menambahkan arm setelah arm tangkap-semuanya!

<!-- TODO : might need to link the end of this chapter to patterns and matching chapter -->
