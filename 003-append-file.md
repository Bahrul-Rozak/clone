Halo teman-teman!

Kalau sebelumnya kita sudah belajar cara **membaca** dan **menulis (menimpa)** file di Node.js, sekarang saatnya membahas operasi yang sering dipakai dalam skenario nyata: **append file**, atau menambahkan data ke akhir file yang sudah ada—tanpa menghapus isinya. Ini adalah teknik krusial terutama saat kamu membangun sistem logging, mencatat aktivitas pengguna, menyimpan riwayat transaksi, atau mengumpulkan data secara bertahap tanpa kehilangan informasi sebelumnya.

Di Node.js, operasi ini dilakukan lewat metode `fs.appendFile` dari modul inti `fs`. Berbeda dengan `fs.writeFile` yang akan **mengganti seluruh isi file**, `appendFile` justru **melekatkan data baru di bagian paling bawah**, sehingga data lama tetap utuh. Ini sangat efisien untuk kasus-kasus di mana kamu ingin “mencatat sesuatu” tanpa perlu membaca dulu, menggabungkan, lalu menulis ulang—yang bisa berisiko jika terjadi error di tengah proses.

Berikut contoh sederhana menggunakan callback:
```js
const fs = require('fs');

const pesanBaru = "Transaksi berhasil: Rp50.000\n";

fs.appendFile('riwayat.txt', pesanBaru, 'utf8', (err) => {
  if (err) {
    console.error('Gagal menambahkan data ke file:', err.message);
    return;
  }
  console.log('Data berhasil ditambahkan!');
});
```

Perhatikan bahwa kita menambahkan karakter `\n` di akhir pesan—ini penting agar setiap entri muncul di baris terpisah, sehingga file tetap mudah dibaca oleh manusia maupun mesin.

Namun, seperti biasa, di lingkungan backend modern yang mengandalkan kode yang rapi dan mudah di-debug, **pendekatan async/await jauh lebih disarankan**. Untungnya, `fs.promises` menyediakan versi Promise dari `appendFile`, sehingga kamu bisa mengintegrasikannya dengan mulus dalam fungsi async:

```js
const fs = require('fs').promises;

async function catatError(pesanError) {
  const timestamp = new Date().toISOString();
  const logEntry = `[${timestamp}] ERROR: ${pesanError}\n`;
  
  try {
    await fs.appendFile('error.log', logEntry, 'utf8');
    console.log('Error berhasil dicatat.');
  } catch (err) {
    console.error('Gagal mencatat error ke log:', err.message);
  }
}

// Contoh pemanggilan
catatError("Koneksi database timeout");
```

Dalam contoh di atas, setiap kali terjadi error, sistem akan menambahkan entri baru ke file `error.log`—tanpa mengganggu entri sebelumnya. File ini bisa dikirim ke tim DevOps, dianalisis otomatis, atau sekadar jadi jejak audit saat terjadi masalah.

Namun, ada beberapa hal penting yang harus kamu waspadai saat menggunakan `appendFile`:
1. **File akan dibuat otomatis** jika belum ada—ini fitur, bukan bug! Tapi pastikan direktori tujuan benar-benar ada, karena `appendFile` tidak akan membuat folder secara otomatis.
2. **Encoding harus konsisten**. Jika file awalnya disimpan dalam `'utf8'`, pastikan semua append juga pakai `'utf8'`—kalau tidak, bisa muncul karakter aneh atau corrupt.
3. **Race condition masih mungkin terjadi** jika banyak proses menulis ke file yang sama secara bersamaan. Untuk kasus high-concurrency, pertimbangkan penggunaan stream atau sistem logging terpusat seperti Winston atau Bunyan.

**Kesimpulannya**, `fs.appendFile` adalah alat sederhana namun sangat powerful untuk mencatat data secara bertahap tanpa kehilangan histori. Ia menjadi tulang punggung sistem logging, audit trail, dan pencatatan aktivitas di hampir semua aplikasi backend. Dengan memahami cara kerjanya—dan selalu menggabungkannya dengan error handling yang baik—kamu tidak hanya menambahkan teks ke file, tapi membangun **jejak digital yang andal, transparan, dan siap untuk di-debug kapan saja**. Karena di dunia backend, **apa yang tidak tercatat… sama saja tidak pernah terjadi**.
