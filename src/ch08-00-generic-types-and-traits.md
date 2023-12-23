# Tipe dan Trait Generik

Setiap bahasa pemrograman memiliki alat untuk mengatasi duplikasi konsep dengan efektif. Di Cairo, salah satu alat tersebut adalah generik: representasi abstrak untuk tipe konkret atau properti lain. Kita dapat menyatakan perilaku generik atau bagaimana mereka berhubungan dengan generik lain tanpa mengetahui apa yang akan menggantikan mereka saat mengompilasi dan menjalankan kode.

Fungsi, struktur, enumerasi, dan trait dapat menggunakan tipe generik sebagai bagian dari definisi mereka daripada tipe konkret seperti `u32` atau `ContractAddress`.

Generik memungkinkan kita untuk mengganti tipe tertentu dengan placeholder yang mewakili beberapa tipe untuk menghilangkan duplikasi kode.

Untuk setiap tipe konkret yang menggantikan tipe generik, kompiler membuat definisi baru, mengurangi waktu pengembangan bagi pemrogram, tetapi duplikasi kode pada tingkat kompilasi masih ada. Hal ini mungkin penting jika Anda menulis kontrak Starknet dan menggunakan generik untuk beberapa tipe yang akan menyebabkan peningkatan ukuran kontrak.

Selanjutnya, Anda akan belajar cara menggunakan trait untuk mendefinisikan perilaku secara generik. Anda dapat menggabungkan trait dengan tipe generik untuk membatasi tipe generik agar hanya menerima tipe-tipe yang memiliki perilaku tertentu, daripada hanya tipe apa pun.