# Enumerasi

Enum, singkatan dari "enumerations," adalah cara untuk mendefinisikan tipe data kustom yang terdiri dari satu set nilai bernama yang tetap, disebut _variant_. Enum berguna untuk mewakili kumpulan nilai terkait di mana setiap nilai bersifat berbeda dan memiliki makna tertentu.

## Variant dan Nilai Enum

Berikut adalah contoh sederhana dari sebuah enum:

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/no_listing_01_enum_example/src/lib.cairo:enum_example}}
```

Pada contoh ini, kami telah mendefinisikan sebuah enum yang disebut `Direction` dengan empat variant: `North`, `East`, `South`, dan `West`. Konvensi penamaan yang digunakan adalah PascalCase untuk variant enum. Setiap variant mewakili nilai yang berbeda dari tipe Direction. Pada contoh ini, variant-variant tidak memiliki nilai terkait. Salah satu variant dapat diinstansiasi menggunakan sintaks ini:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no_listing_01_enum_example/src/lib.cairo:here}}
```

Mudah untuk menulis kode yang berbeda tergantung pada variant dari sebuah instance enum, seperti pada contoh ini untuk menjalankan kode spesifik sesuai dengan sebuah Direction. Anda dapat mempelajari lebih lanjut tentang hal ini pada halaman [Konstruksi Aliran Kontrol Match](ch06-02-the-match-control-flow-construct.md).

## Enum Kombinasi dengan Tipe Kustom

Enum juga dapat digunakan untuk menyimpan data yang lebih menarik yang terkait dengan setiap variant. Sebagai contoh:

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/no_listing_02_enum_message/src/lib.cairo:message}}
```

Pada contoh ini, enum `Message` memiliki tiga variant: `Quit`, `Echo`, dan `Move`, masing-masing dengan tipe yang berbeda:

- `Quit` tidak memiliki nilai terkait.
- `Echo` adalah sebuah bilangan bulat tunggal.
- `Move` adalah sebuah tuple dari dua nilai u128.

Anda bahkan dapat menggunakan Struct atau Enum lain yang Anda definisikan di dalam salah satu variant Enum Anda.

## Implementasi Trait untuk Enum

Di Cairo, Anda dapat mendefinisikan trait dan mengimplementasikannya untuk enum kustom Anda. Ini memungkinkan Anda untuk mendefinisikan metode dan perilaku yang terkait dengan enum. Berikut contoh dari pendefinisian sebuah trait dan implementasinya untuk enum `Message` sebelumnya:

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/no_listing_02_enum_message/src/lib.cairo:trait_impl}}
```

Pada contoh ini, kami mengimplementasikan trait `Processing` untuk `Message`. Berikut adalah cara penggunaannya untuk memproses pesan `Quit`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no_listing_02_enum_message/src/lib.cairo:main}}
```

Menjalankan kode ini akan mencetak `quitting`.

## Enum Option dan Keuntungannya

Enum Option adalah enum standar dalam Cairo yang mewakili konsep dari nilai opsional. Enum ini memiliki dua variant: `Some: T` dan `None: ()`. `Some: T ` menunjukkan bahwa ada sebuah nilai dengan tipe `T`, sedangkan `None` mewakili ketiadaan nilai.

```rust,noplayground
enum Option<T> {
    Some: T,
    None: (),
}
```

Enum `Option` berguna karena memungkinkan Anda untuk secara eksplisit mewakili kemungkinan dari sebuah nilai yang tidak ada, membuat kode Anda lebih ekspresif dan lebih mudah untuk dipahami. Menggunakan `Option` juga dapat membantu mencegah bug yang disebabkan oleh penggunaan nilai `null` yang tidak terinisialisasi atau tidak terduga.

Sebagai contoh, berikut adalah sebuah fungsi yang mengembalikan indeks dari elemen pertama dalam sebuah array dengan nilai tertentu, atau None jika elemen tersebut tidak ada.

Kami menunjukkan dua pendekatan untuk fungsi di atas:

- Pendekatan Rekursif `find_value_recursive`
- Pendekatan Iteratif `find_value_iterative`

> Catatan: di masa depan akan lebih baik jika digantikan contoh yang lebih sederhana menggunakan perulangan dan tanpa kode terkait gas.

```rust,noplayground
{{#include ../listings/ch06-enums-and-pattern-matching/no_listing_03_enum_option/src/lib.cairo}}
```

Menjalankan kode ini akan mencetak `it worked`.
