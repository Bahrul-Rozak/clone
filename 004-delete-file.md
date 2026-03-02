Halo teman-teman!

Setelah kita belajar membaca, menulis, dan menambahkan data ke file, sekarang saatnya membahas operasi yang sering diabaikan tapi sangat penting dalam pengembangan backend: **menghapus file**. Di dunia nyata, kamu tidak hanya menyimpan data—kadang kamu juga perlu membersihkannya. Misalnya: menghapus file cache yang sudah kadaluarsa, menghapus unggahan sementara setelah diproses, atau membersihkan log lama agar disk tidak penuh. Untungnya, Node.js menyediakan cara aman dan langsung untuk melakukannya lewat modul inti `fs` (File System), khususnya dengan fungsi `fs.unlink`.

Kenapa namanya `unlink` dan bukan `delete`? Karena di sistem operasi Unix/Linux (yang menjadi dasar banyak server modern), menghapus file sebenarnya berarti **melepaskan tautan (link)** antara nama file dan data fisik di disk. Jika tidak ada proses lain yang masih menggunakan file tersebut, sistem akan secara otomatis membebaskan ruang penyimpanannya. Jadi, `unlink` adalah istilah teknis yang akurat—tapi dalam praktik sehari-hari, kita sebut saja “hapus file”.

Berikut contoh dasar penggunaan `fs.unlink` dengan callback:
```js
const fs = require('fs');

fs.unlink('file-sampah.txt', (err) => {
  if (err) {
    // Misalnya file tidak ditemukan atau tidak punya izin hapus
    console.error('Gagal menghapus file:', err.message);
    return;
  }
  console.log('File berhasil dihapus!');
});
```

Perhatikan bahwa **tidak ada konfirmasi**—jika file ada dan kamu punya izin, ia akan langsung hilang. Tidak bisa dikembalikan. Jadi, pastikan logika hapusmu benar-benar aman, terutama jika nama file berasal dari input pengguna.

Untuk kode yang lebih modern dan mudah dibaca, gunakan versi Promise-nya lewat `fs.promises`:
```js
const fs = require('fs').promises;

async function bersihkanCache(namaFile) {
  try {
    await fs.unlink(namaFile);
    console.log(`Cache ${namaFile} berhasil dibersihkan.`);
  } catch (err) {
    if (err.code === 'ENOENT') {
      console.log(`File ${namaFile} tidak ditemukan — aman dilanjutkan.`);
    } else {
      console.error('Error saat menghapus file:', err.message);
    }
  }
}

// Contoh pemanggilan
bersihkanCache('temp/upload-123.tmp');
```

Di sini, kita juga menangani kasus khusus: error `ENOENT` (Error: No Entry) yang berarti file memang tidak ada. Dalam banyak skenario, ini bukan masalah—malah justru diharapkan. Jadi, alih-alih memperlakukannya sebagai kegagalan, kita anggap sebagai “sudah bersih”.

Namun, ada beberapa **perangkap umum** yang harus kamu waspadai:
1. **Jangan pernah hapus file berdasarkan input mentah dari pengguna** tanpa validasi path. Ini bisa menyebabkan *path traversal attack* (misalnya, pengguna mengirim `../../../etc/passwd`).
2. **Pastikan file benar-benar tidak digunakan** oleh proses lain. Di Windows, misalnya, file yang sedang dibuka tidak bisa dihapus.
3. **`unlink` hanya untuk file**, bukan folder. Untuk menghapus direktori, kamu perlu `fs.rmdir` (dan untuk folder berisi file, gunakan `fs.rm` dengan opsi `{ recursive: true }`).

Sebagai praktik terbaik, selalu:
- Validasi path file (gunakan `path.resolve()` dan batasi ke subdirektori tertentu),
- Tangani error spesifik seperti `ENOENT` atau `EACCES`,
- Gunakan logging untuk mencatat operasi hapus—terutama di lingkungan produksi.

**Kesimpulannya**, menghapus file di Node.js mungkin terlihat sederhana, tapi ia menyimpan risiko besar jika tidak dilakukan dengan hati-hati. Fungsi `fs.unlink` memberimu kekuatan untuk membersihkan sistem, tetapi kekuatan itu harus diimbangi dengan **validasi ketat, penanganan error yang cerdas, dan kesadaran keamanan**. Karena di dunia backend, satu baris kode `unlink` yang salah bisa menghapus data penting—dan tidak ada “Recycle Bin” di server produksi. Jadi, hapuslah dengan bijak, karena **membersihkan itu penting… tapi kehilangan data itu permanen**.
