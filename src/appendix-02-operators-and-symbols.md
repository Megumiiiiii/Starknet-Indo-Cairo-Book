# Lampiran B: Operator dan Simbol

Lampiran ini mencakup glosarium tentang sintaksis Cairo.

## Operator

Tabel B-1 berisi operator dalam bahasa Cairo, contoh penggunaan operator dalam konteks, penjelasan singkat, dan apakah operator tersebut dapat di-overload. Jika suatu operator dapat di-overload, maka dituliskan trait yang relevan untuk meng-overload operator tersebut.

<span class="caption">Tabel B-1: Operator</span>

| Operator | Contoh | Penjelasan | Dapat Di-Overload? |
|----------|--------|------------|---------------------|
| `!` | `!expr` | Komplemen bitwise atau logika | `Not` |
| `!=` | `expr != expr` | Perbandingan ketidaksetaraan | `PartialEq` |
| `%` | `expr % expr` | Sisa pembagian aritmatika | `Rem` |
| `%=` | `var %= expr` | Sisa pembagian aritmatika dan assignment | `RemEq` |
| `&` | `expr & expr` | AND bitwise | `BitAnd` |
| `&&` | `expr && expr` | AND logika dengan short-circuiting | |
| `*` | `expr * expr` | Perkalian aritmatika | `Mul` |
| `*=` | `var *= expr` | Perkalian aritmatika dan assignment | `MulEq` |
| `@` | `@var` | Snapshot | |
| `*` | `*var` | Desnap | |
| `+` | `expr + expr` | Penambahan aritmatika | `Add` |
| `+=` | `var += expr` | Penambahan aritmatika dan assignment | `AddEq` |
| `,` | `expr, expr` | Pemisah argumen dan elemen | |
| `-` | `-expr` | Penyangkalan aritmatika | `Neg` |
| `-` | `expr - expr` | Pengurangan aritmatika | `Sub` |
| `-=` | `var -= expr` | Pengurangan aritmatika dan assignment | `SubEq` |
| `->` | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Tipe kembalian fungsi dan closure | |
| `.` | `expr.ident` | Akses anggota | |
| `/` | `expr / expr` | Pembagian aritmatika | `Div` |
| `/=` | `var /= expr` | Pembagian aritmatika dan assignment | `DivEq` |
| `:` | `pat: type`, `ident: type` | Batasan | |
| `:` | `ident: expr` | Inisialisasi field struct | |
| `;` | `expr;` | Penutup pernyataan dan item | |
| `<` | `expr < expr` | Perbandingan kurang dari | `PartialOrd` |
| `<=` | `expr <= expr` | Perbandingan kurang dari atau sama dengan | `PartialOrd` |
| `=` | `var = expr` | Assignment | |
| `==` | `expr == expr` | Perbandingan kesetaraan | `PartialEq` |
| `=>` | `pat => expr` | Bagian dari sintaksis lengan match | |
| `>` | `expr > expr` | Perbandingan lebih dari | `PartialOrd` |
| `>=` | `expr >= expr` | Perbandingan lebih dari atau sama dengan | `PartialOrd` |
| `^` | `expr ^ expr` | XOR bitwise | `BitXor` |
| <code>&vert;</code> | <code>expr &vert; expr</code> | OR bitwise | `BitOr` |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code> | OR logika dengan short-circuiting | |

## Simbol Non Operator

Berikut ini adalah daftar semua simbol yang bukan digunakan sebagai operator; dengan kata lain, simbol-simbol ini tidak memiliki perilaku yang sama seperti pemanggilan fungsi atau metode.

Tabel B-2 menunjukkan simbol-simbol yang muncul sendiri dan valid dalam berbagai lokasi.

<span class="caption">Tabel B-2: Sintaksis Tunggal</span>

| Simbol | Penjelasan |
|--------|------------|
| `..._u8`, `..._usize`, dll. | Literal numerik dari tipe tertentu |
| `'...'` | String pendek |
| `_` | Pola ikut-aba; juga digunakan untuk membuat literal bilangan bulat mudah dibaca |

Tabel B-3 menunjukkan simbol-simbol yang digunakan dalam konteks jalur hierarki modul untuk mengakses suatu item.

<span class="caption">Tabel B-3: Sintaksis Terkait Jalur</span>

| Simbol | Penjelasan |
|--------|------------|
| `ident::ident` | Jalur namespace |
| `super::path` | Jalur relatif terhadap induk modul saat ini |
| `trait::method(...)` | Mendisambiguasi pemanggilan metode dengan menyebutkan trait yang mendefinisikannya |

Tabel B-4 menunjukkan simbol-simbol yang muncul dalam konteks penggunaan parameter tipe generik.

<span class="caption">Tabel B-4: Generik</span>

| Simbol | Penjelasan |
|--------|------------|
| `path<...>` | Menentukan parameter untuk tipe generik dalam suatu tipe (mis., `Vec<u8>`) |
| `path::<...>`, `method::<...>` | Menentukan parameter untuk tipe generik, fungsi, atau metode dalam suatu ekspresi; sering disebut sebagai turbofish |
| `fn ident<...> ...` | Mendefinisikan fungsi generik |
| `struct ident<...> ...` | Mendefinisikan struktur generik |
| `enum ident<...> ...` | Mendefinisikan enumerasi generik |
| `impl<...> ...` | Mendefinisikan implementasi generik |

Tabel B-5 menunjukkan simbol-simbol yang muncul dalam konteks pemanggilan atau definisi makro dan penentuan atribut pada suatu item.

<span class="caption">Tabel B-5: Makro dan Atribut</span>

| Simbol | Penjelasan |
|--------|------------|
| `#[meta]` | Atribut luar |

Tabel B-6 menunjukkan simbol-simbol yang membuat komentar.

<span class="caption">Tabel B-6: Komentar</span>

| Simbol | Penjelasan |
|--------|------------|
| `//` | Komentar baris |

Tabel B-7 menunjukkan simbol-simbol yang muncul dalam konteks penggunaan tuple.

<span class="caption">Tabel B-7: Tuple</span>


| Simbol | Penjelasan |
|--------|------------|
| `()` | Tuple kosong (juga disebut unit), baik literal maupun tipe |
| `(expr)` | Ekspresi dalam tanda kurung |
| `(expr,)` | Ekspresi tuple dengan satu elemen |
| `(type,)` | Tipe tuple dengan satu elemen |
| `(expr, ...)` | Ekspresi tuple |
| `(type, ...)` | Tipe tuple |
| `expr(expr, ...)` | Pemanggilan fungsi; juga digunakan untuk menginisialisasi `struct` tuple dan varian `enum` tuple |

Tabel B-8 menunjukkan konteks di mana kurung kurawal digunakan.

<span class="caption">Tabel B-8: Kurung Kurawal</span>

| Konteks | Penjelasan |
|---------|------------|
| `{...}` | Ekspresi blok |
| `Type {...}` | Literal `struct` |
