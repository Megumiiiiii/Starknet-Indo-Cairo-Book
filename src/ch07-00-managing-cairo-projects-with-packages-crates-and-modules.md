# Mengelola Proyek Cairo dengan Package, Crate, dan Modul

Ketika Anda menulis program-program besar, mengorganisir kode Anda akan menjadi semakin penting. Dengan mengelompokkan fungsionalitas terkait dan memisahkan kode dengan fitur-fitur yang berbeda, Anda akan menjelaskan di mana menemukan kode yang mengimplementasikan fitur tertentu dan ke mana harus pergi untuk mengubah cara kerja suatu fitur.

Program-program yang telah kita tulis sebelumnya berada dalam satu modul di satu file. Seiring dengan pertumbuhan proyek, Anda sebaiknya mengorganisir kode dengan membaginya ke dalam beberapa modul dan beberapa file. Seiring dengan pertumbuhan sebuah Package, Anda dapat mengekstrak bagian-bagian ke dalam crate terpisah yang menjadi dependensi eksternal. Bab ini mencakup semua teknik tersebut.

Kami juga akan membahas tentang mengenkapsulasi detail implementasi, yang memungkinkan Anda untuk menggunakan kembali kode pada level yang lebih tinggi: setelah Anda mengimplementasikan sebuah operasi, kode lain dapat memanggil kode Anda tanpa harus mengetahui bagaimana cara implementasinya bekerja.

Konsep terkait adalah cakupan (scope): konteks bertingkat di mana kode ditulis memiliki serangkaian nama yang didefinisikan sebagai "dalam cakupan." Saat membaca, menulis, dan mengompilasi kode, para pemrogram dan kompilator perlu tahu apakah suatu nama tertentu pada suatu tempat tertentu merujuk pada variabel, fungsi, struktur, enumerasi, modul, konstanta, atau item lainnya, dan apa arti dari item tersebut. Anda dapat membuat cakupan dan mengubah nama-nama yang berada dalam atau di luar cakupan. Anda tidak dapat memiliki dua item dengan nama yang sama dalam cakupan yang sama.

Cairo memiliki beberapa fitur yang memungkinkan Anda untuk mengelola organisasi kode Anda. Fitur-fitur ini, kadang-kadang secara kolektif disebut sebagai _sistem modul_, meliputi:

- **Package:** Fitur Scarb yang memungkinkan Anda membangun, menguji, dan berbagi crate.
- **Crate:** Sebuah pohon modul yang sesuai dengan satu unit kompilasi. Ini memiliki direktori root, dan modul root yang didefinisikan dalam file `lib.cairo` di bawah direktori ini.
- **Modul** dan **use:** Memungkinkan Anda mengendalikan organisasi dan cakupan dari item-item.
- **Path:** Cara untuk menamai sebuah item, seperti struct, fungsi, atau modul.

Dalam bab ini, kami akan membahas semua fitur ini, mendiskusikan bagaimana mereka saling berinteraksi, dan menjelaskan cara menggunakan mereka untuk mengelola cakupan. Pada akhirnya, Anda seharusnya memiliki pemahaman yang kuat tentang sistem modul dan mampu bekerja dengan cakupan seperti seorang profesional!