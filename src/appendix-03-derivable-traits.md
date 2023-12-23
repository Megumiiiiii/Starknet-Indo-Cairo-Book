# Lampiran C: Trait yang Dapat Di-Derive

Di berbagai bagian dalam buku ini, kami telah membahas atribut `derive`, yang dapat Anda aplikasikan pada definisi struktur atau enumerasi. Atribut `derive` menghasilkan kode untuk menerapkan sebuah trait bawaan pada tipe yang Anda anotasi dengan sintaks `derive`.

Pada lampiran ini, kami menyediakan referensi komprehensif yang mendetail tentang semua trait dalam pustaka standar yang kompatibel dengan atribut `derive`.

Trait-trait yang terdaftar di sini adalah satu-satunya trait yang didefinisikan oleh pustaka inti yang dapat diimplementasikan pada tipe-tipe Anda menggunakan `derive`. Trait lain yang didefinisikan dalam pustaka standar tidak memiliki perilaku default yang masuk akal, sehingga terserah pada Anda untuk mengimplementasikannya sesuai dengan logika yang sesuai dengan tujuan yang ingin Anda capai.

Daftar trait yang dapat di-derive yang disediakan dalam lampiran ini tidak mencakup semua kemungkinan: pustaka eksternal dapat mengimplementasikan `derive` untuk trait mereka sendiri, memperluas daftar trait yang kompatibel dengan `derive`.

## PartialEq untuk Perbandingan Kesetaraan

Trait `PartialEq` memungkinkan perbandingan antara instance dari suatu tipe untuk kesetaraan, dengan demikian memungkinkan operator == dan !=.

Ketika `PartialEq` di-derive pada struktur, dua instance adalah sama hanya jika semua field sama, dan instance tersebut tidak sama jika ada field yang tidak sama. Ketika di-derive pada enumerasi, setiap varian adalah sama dengan dirinya sendiri dan tidak sama dengan varian lainnya.

Contoh:

```rust
#[derive(PartialEq, Drop)]
struct A {
    item: felt252
}

fn main() {
    let first_struct = A {
        item: 2
    };
    let second_struct = A {
        item: 2
    };
    assert(first_struct == second_struct, 'Struktur berbeda');
}
```

## Clone dan Copy untuk Menduplikasi Nilai

Trait `Clone` menyediakan fungsionalitas untuk membuat salinan mendalam (deep copy) dari suatu nilai.

Menerapkan `Clone` mengimplementasikan metode `clone`, yang pada gilirannya memanggil clone pada setiap komponen tipe tersebut. Ini berarti semua field atau nilai dalam tipe harus juga mengimplementasikan `Clone` untuk mendapatkan turunan `Clone`.

Contoh:

```rust
use clone::Clone;

#[derive(Clone, Drop)]
struct A {
    item: felt252
}

fn main() {
    let first_struct = A {
        item: 2
    };
    let second_struct = first_struct.clone();
    assert(second_struct.item == 2, 'Tidak sama');
}
```

`Copy` trait memungkinkan untuk menduplikasi nilai-nilai. Anda dapat menurunkan `Copy` pada jenis apa pun yang bagian-bagiannya semuanya menerapkan `Copy`.

Contoh:

```rust
#[derive(Copy, Drop)]
struct A {
    item: felt252
}

fn main() {
    let first_struct = A {
        item: 2
    };
    let second_struct = first_struct;
    assert(second_struct.item == 2, 'Not equal');
    assert(first_struct.item == 2, 'Not Equal'); // Sifat Copy mencegah first_struct pindah ke second_struct
}
```

## Serialisasi dengan Serde

`Serde` menyediakan implementasi trait untuk fungsi `serialize` dan `deserialize` untuk struktur data yang didefinisikan dalam kreatif Anda. Ini memungkinkan Anda untuk mengubah struktur Anda menjadi larik (atau sebaliknya).

Contoh:

```rust
use serde::Serde;
use array::ArrayTrait;

#[derive(Serde, Drop)]
struct A {
    item_one: felt252,
    item_two: felt252,
}

fn main() {
    let first_struct = A {
        item_one: 2,
        item_two: 99,
    };
    let mut output_array = ArrayTrait::new();
    let serialized = first_struct.serialize(ref output_array);
    panic(output_array);
}
```

Output:

```Bash
Run panicked with [2 (''), 99 ('c'), ].
```

Kita bisa melihat di sini bahwa struktur A kita telah di-serialize menjadi array output.

Juga, kita bisa menggunakan fungsi `deserialize` untuk mengonversi array yang telah di-serialize kembali ke struktur A kita.

Contoh:

```rust
use serde::Serde;
use array::ArrayTrait;
use option::OptionTrait;

#[derive(Serde, Drop)]
struct A {
    item_one: felt252,
    item_two: felt252,
}

fn main() {
    let first_struct = A {
        item_one: 2,
        item_two: 99,
    };
    let mut output_array = ArrayTrait::new();
    let mut serialized = first_struct.serialize(ref output_array);
    let mut span_array = output_array.span();
    let deserialized_struct: A = Serde::<A>::deserialize(ref span_array).unwrap();
}
```

Di sini kita mengonversi kembali array span yang telah di-serialize ke struktur A. `deserialize` mengembalikan `Option` sehingga kita perlu melakukan unwrap. Ketika menggunakan deserialize, kita juga perlu menentukan tipe yang ingin kita deserialkan ke dalamnya.

## Drop dan Destruct

Ketika keluar dari lingkup, variabel perlu dipindahkan terlebih dahulu. Di sinilah sifat `Drop` turun. Anda dapat menemukan lebih banyak detail tentang penggunaannya [di sini](ch04-01-what-is-ownership.md#the-drop-trait).

Selain itu, Dictionary perlu disusun sebelum keluar dari lingkup. Memanggil secara manual metode `squash` pada masing-masing dapat dengan cepat menjadi redundan. Sifat `Destruct` memungkinkan Dictionary untuk secara otomatis disusun ketika mereka keluar dari lingkup. Anda juga dapat menemukan informasi lebih lanjut tentang `Destruct` [di sini](ch04-01-what-is-ownership.md#the-destruct-trait).

## Simpan

Menyimpan struktur yang ditentukan pengguna dalam variabel penyimpanan dalam kontrak Starknet memerlukan sifat `Store` untuk diimplementasikan untuk jenis ini. Anda dapat secara otomatis mendapatkan sifat `store` untuk semua struktur yang tidak mengandung jenis kompleks seperti Dictionary atau Array.

Contoh:

```rust, noplayground
#[starknet::contract]
mod contract {
    #[derive(Drop, starknet::Store)]
    struct A {
        item_one: felt252,
        item_two: felt252,
    }

    #[storage]
    struct Storage {
        my_storage: A,
    }
}

```

Di sini kami menunjukkan implementasi `struct A` yang mendapat sifat Store. `struct A` ini kemudian digunakan sebagai variabel penyimpanan dalam kontrak.

## PartialOrd dan Ord untuk Perbandingan Penyusunan

Selain sifat `PartialEq`, pustaka standar juga menyediakan sifat `PartialOrd` dan `Ord` untuk membandingkan nilai-nilai untuk penyusunan.

Sifat `PartialOrd` memungkinkan perbandingan antara instance dari suatu jenis untuk penyusunan, sehingga mengaktifkan operator <, <=, >, dan >=.

Ketika `PartialOrd` diturunkan pada struktur, dua instance diurutkan dengan membandingkan setiap bidang secara bergantian.
