# Paket dan Crate

## Apa itu crate?

Sebuah crate merupakan jumlah kode terkecil yang dipertimbangkan oleh kompiler Cairo pada suatu waktu. Bahkan jika Anda menjalankan `cairo-compile` daripada `scarb build` dan menyertakan sebuah file kode sumber tunggal, kompiler akan mempertimbangkan file tersebut sebagai sebuah crate. Crate dapat berisi modul-modul, dan modul-modul tersebut mungkin didefinisikan dalam file-file lain yang dikompilasi bersama dengan crate tersebut, seperti yang akan dibahas pada bagian-bagian selanjutnya.

## Apa itu crate root?

Crate root merupakan file sumber `lib.cairo` yang menjadi awal dari kompilasi oleh kompiler Cairo dan membentuk modul root dari crate Anda (kami akan menjelaskan modul secara mendalam pada bagian [“Defining Modules to Control Scope”](./ch07-02-defining-modules-to-control-scope.md)).

## Apa itu sebuah paket?

Sebuah paket Cairo adalah kumpulan satu atau lebih crate dengan file Scarb.toml yang menjelaskan bagaimana cara membangun crate-crate tersebut. Ini memungkinkan pemisahan kode menjadi bagian-bagian yang lebih kecil dan dapat digunakan kembali, serta memfasilitasi manajemen dependensi yang lebih terstruktur.

## Membuat Paket dengan Scarb

Anda dapat membuat paket Cairo baru menggunakan perangkat baris perintah scarb. Untuk membuat paket baru, jalankan perintah berikut:

```bash
scarb new my_package
```

Perintah ini akan membuat direktori paket baru bernama `my_package` dengan struktur berikut:

```
my_package/
├── Scarb.toml
└── src
    └── lib.cairo
```

- `src/` adalah direktori utama di mana semua file sumber Cairo untuk paket akan disimpan.
- `lib.cairo` adalah modul root default dari crate, yang juga merupakan titik masuk utama dari paket.
- `Scarb.toml` adalah file manifest paket, yang berisi metadata dan opsi konfigurasi untuk paket, seperti dependensi, nama paket, versi, dan penulis. Anda dapat menemukan dokumentasi tentang ini pada [referensi scarb](https://docs.swmansion.com/scarb/docs/reference/manifest.html).

```toml
[package]
name = "my_package"
version = "0.1.0"

[dependencies]
# foo = { path = "vendor/foo" }
```

Saat Anda mengembangkan paket Anda, Anda mungkin ingin mengorganisir kode Anda ke dalam beberapa file sumber Cairo. Anda dapat melakukannya dengan membuat file-file `.cairo` tambahan di dalam direktori `src` atau subdirektorinya.