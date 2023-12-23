## Lampiran A: Kata Kunci

Daftar berikut berisi kata kunci yang diperuntukkan bagi penggunaan bahasa Kairo saat ini atau di masa mendatang.

Ada dua kategori kata kunci:

- ketat
- disimpan

Ada kategori ketiga, yaitu fungsi dari perpustakaan inti. Meskipun namanya tidak dicadangkan, namun tidak disarankan untuk digunakan sebagai nama item apa pun untuk mengikuti praktik yang baik.

---

### Kata kunci yang ketat

Kata kunci ini hanya dapat digunakan dalam konteks yang benar. Mereka tidak dapat digunakan sebagai nama item apa pun.

- `as` - Ganti nama impor
- `break` - segera keluar dari loop
- `const` - Tentukan item konstan
- `continue` - Lanjutkan ke iterasi loop berikutnya
- `else` - Penggantian untuk konstruksi aliran kontrol `if` dan `if let`
- `enum` - Tentukan enumerasi
- `extern` - Fungsi yang ditentukan pada tingkat kompiler menggunakan petunjuk yang tersedia di tingkat Kairo1 dengan deklarasi ini
- `false` - Boolean salah secara literal
- `fn` - Mendefinisikan suatu fungsi
- `if` - Cabang berdasarkan hasil ekspresi kondisional
- `impl` - Menerapkan fungsionalitas bawaan atau sifat
- `implicits` - Jenis parameter fungsi khusus yang diperlukan untuk melakukan tindakan tertentu
- `let` - Ikat variabel
- `loop` - Ulangi tanpa syarat
- `match` - Cocokkan nilai dengan pola
- `mod` - Tentukan modul
- `mut` - Menunjukkan mutabilitas variabel
- `nopanic` - Fungsi yang ditandai dengan notasi ini berarti fungsi tersebut tidak akan pernah panik.
- `of` - Implementasi suatu sifat
- `ref` - Parameter yang diteruskan secara implisit dikembalikan pada akhir suatu fungsi
- `return` - Kembali dari fungsi
- `struct` - Tentukan struktur
- `trait` - Tentukan suatu sifat
- `true` - Boolean benar secara literal
- `type` - Tentukan alias tipe
- `use` - Membawa simbol ke dalam ruang lingkup

---

### Kata kunci yang dipesan

Kata kunci ini belum digunakan, namun dicadangkan untuk penggunaan di masa mendatang. Kata kunci tersebut memiliki batasan yang sama dengan kata kunci ketat. Alasan dibalik hal ini adalah untuk membuat program yang ada saat ini kompatibel dengan versi Kairo yang akan datang dengan melarang mereka menggunakan kata kunci tersebut.

- `Self`
- `assert`
- `do`
- `dyn`
- `for`
- `hint`
- `in`
- `macro`
- `move`
- `pub`
- `static_assert`
- `self`
- `static`
- `super`
- `try`
- `typeof`
- `unsafe`
- `where`
- `while`
- `with`
- `yield`

---

### Fungsi bawaan

Bahasa pemrograman Kairo menyediakan beberapa fungsi spesifik yang memiliki tujuan khusus. Kami tidak akan membahas semuanya dalam buku ini, tetapi tidak disarankan menggunakan nama fungsi ini sebagai nama item lainnya.

\- `assert` - Fungsi ini memeriksa ekspresi boolean, dan jika bernilai salah, fungsi panik akan dipicu. - `panic` - Fungsi ini menghentikan program.
