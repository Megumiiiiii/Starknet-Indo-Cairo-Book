# Overloading Operator

Overloading operator adalah fitur dalam beberapa bahasa pemrograman yang memungkinkan pengulangan penggunaan operator standar, seperti penambahan (+), pengurangan (-), perkalian (\*), dan pembagian (/), untuk bekerja dengan tipe yang ditentukan pengguna. Ini dapat membuat sintaks kode lebih intuitif, dengan memungkinkan operasi pada tipe yang ditentukan pengguna diekspresikan dengan cara yang sama seperti operasi pada tipe primitif.

Dalam Cairo, overloading operator dicapai melalui implementasi trait tertentu. Setiap operator memiliki trait terkait, dan pengulangan operator melibatkan penyediaan implementasi trait tersebut untuk tipe kustom.
Namun, penting untuk menggunakan overloading operator secara bijak. Penyalahgunaan dapat menyebabkan kebingungan, membuat kode lebih sulit dipelihara, misalnya ketika tidak ada makna semantik pada operator yang di-overload.

Pertimbangkan contoh di mana dua `Potion` perlu digabungkan. `Potion` memiliki dua bidang data, yaitu mana dan kesehatan. Menggabungkan dua `Potion` seharusnya menambahkan bidang-bidang mereka masing-masing.

```rust
{{#include ../listings/ch11-advanced-features/no_listing_01_potions/src/lib.cairo}}
```

Dalam kode di atas, kita mengimplementasikan trait `Add` untuk tipe `Potion`. Fungsi add mengambil dua argumen: `lhs` dan `rhs` (kiri dan kanan). Isi fungsi mengembalikan instance `Potion` baru, dengan nilai bidangnya merupakan kombinasi dari `lhs` dan `rhs`.

Seperti yang diilustrasikan dalam contoh tersebut, pengulangan operator memerlukan spesifikasi tipe konkret yang di-overload. Trait generik yang di-overload adalah `Add<T>`, dan kita mendefinisikan implementasi konkret untuk tipe `Potion` dengan `Add<Potion>`.