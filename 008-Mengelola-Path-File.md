Halo teman-teman!

Saat kamu bekerja dengan file di Node.js—entah itu membaca konfigurasi, menyimpan log, atau mengunggah gambar—ada satu masalah kecil yang sering bikin frustasi: **perbedaan format path antar sistem operasi**. Di Windows, path ditulis dengan backslash (`C:\Users\Nama\file.txt`), sementara di macOS dan Linux, pakai forward slash (`/home/nama/file.txt`). Kalau kamu menulis path secara manual dengan string biasa, aplikasimu bisa jalan sempurna di laptopmu… tapi **error total saat di-deploy ke server Linux**.

Untungnya, Node.js menyediakan solusi elegan lewat dua modul inti: **`path`** dan **`os`**. Kali ini, kita fokus dulu pada **modul `path`**, yang dirancang khusus untuk **mengelola path file secara lintas-platform (cross-platform)**—tanpa perlu tahu sistem operasi apa yang sedang dijalankan.

Modul `path` memberimu fungsi-fungsi cerdas yang otomatis menyesuaikan dengan OS tempat kode dijalankan. Misalnya, `path.join()` akan menggabungkan segmen path menggunakan **separator yang benar** untuk sistem tersebut.

Contoh:
```js
const path = require('path');

// Gabungkan direktori dan nama file
const filePath = path.join('data', 'users', 'profil.json');
console.log(filePath);
// Di Windows: data\users\profil.json
// Di Linux/macOS: data/users/profil.json
```

Ini jauh lebih aman daripada menulis `'data/users/profil.json'` secara manual—karena di Windows, forward slash kadang ditoleransi, tapi tidak selalu konsisten, terutama saat berinteraksi dengan API native.

Fungsi lain yang sangat berguna adalah `path.resolve()`. Ia mengubah path relatif menjadi **path absolut** berdasarkan direktori kerja saat ini (`__dirname` atau `process.cwd()`).

Contoh:
```js
const configPath = path.resolve(__dirname, 'config', 'app.json');
console.log(configPath);
// Misal di /home/app → /home/app/config/app.json
// Di C:\project → C:\project\config\app.json
```

Ini penting saat kamu ingin memastikan file selalu diakses dari lokasi yang tepat—terutama di aplikasi yang mungkin dijalankan dari direktori berbeda.

Kamu juga bisa memecah path menjadi komponennya:
```js
const fullPath = '/home/user/docs/laporan.pdf';
console.log(path.dirname(fullPath));    // '/home/user/docs'
console.log(path.basename(fullPath));   // 'laporan.pdf'
console.log(path.extname(fullPath));    // '.pdf'
```

Dan jika kamu butuh ekstrak nama file tanpa ekstensi:
```js
const nameOnly = path.basename(fullPath, path.extname(fullPath));
console.log(nameOnly); // 'laporan'
```

Sekarang, bagaimana jika kamu ingin memastikan path yang kamu buat **selalu kompatibel**, bahkan saat menerima input dari pengguna? Selalu gunakan `path.join()` atau `path.resolve()`—jangan gabung string manual!

Contoh praktis dalam aplikasi backend:
```js
const fs = require('fs').promises;
const path = require('path');

async function bacaProfil(username) {
  // Aman: tidak rentan path traversal jika divalidasi dengan benar
  const safePath = path.join(__dirname, 'data', 'users', `${username}.json`);
  
  // (Catatan: tetap validasi `username` agar tidak mengandung ../ !)
  try {
    const data = await fs.readFile(safePath, 'utf8');
    return JSON.parse(data);
  } catch (err) {
    throw new Error(`Profil ${username} tidak ditemukan`);
  }
}
```

> ⚠️ **Peringatan keamanan**: Meski `path.join()` membantu format, ia **tidak mencegah path traversal attack** sendiri. Jika `username` berisi `'../../../etc/passwd'`, path-nya tetap bisa keluar dari folder yang diizinkan. Jadi, selalu **validasi dan sanitasi input pengguna** sebelum digabung ke path.

**Kesimpulannya**, modul `path` adalah alat wajib bagi setiap developer Node.js yang bekerja dengan file. Ia menghilangkan kompleksitas perbedaan sistem operasi, membuat kode-mu portabel, rapi, dan siap produksi di mana saja—dari laptop Windows hingga server cloud berbasis Linux. Dengan `path.join()`, `path.resolve()`, dan kawan-kawannya, kamu tidak hanya mengelola path… tapi membangun fondasi yang **tahan terhadap lingkungan yang tak terduga**. Karena di dunia backend, **lokasi file harus pasti—bukan kira-kira**.
