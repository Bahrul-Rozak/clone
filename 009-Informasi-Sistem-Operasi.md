Halo teman-teman!

Pernahkah kamu ingin tahu di sistem operasi apa aplikasimu sedang berjalan? Apakah itu Windows, macOS, Linux, atau bahkan server cloud berbasis Alpine? Atau mungkin kamu butuh menyesuaikan perilaku aplikasi—misalnya, menyimpan file cache di lokasi yang sesuai dengan standar OS, atau menampilkan pesan error yang relevan dengan platform pengguna? Untungnya, Node.js menyediakan jawaban lewat **modul `os`**—sebuah jendela kecil ke dalam dunia sistem operasi tempat kode-mu dijalankan.

Modul `os` memberimu akses ke informasi penting tentang lingkungan runtime: jenis OS, arsitektur CPU, memori yang tersedia, direktori home pengguna, dan bahkan karakter pemisah baris (`\n` vs `\r\n`). Semua ini sangat berguna untuk membuat aplikasi yang **benar-benar lintas-platform**, bukan hanya secara teori, tapi juga dalam praktik nyata.

Mari kita lihat beberapa fungsi utama dan contoh penggunaannya:

```js
const os = require('os');

// 1. Jenis sistem operasi
console.log('Platform:', os.platform()); 
// Contoh output: 'win32', 'darwin' (macOS), 'linux'

// 2. Arsitektur CPU
console.log('Arsitektur:', os.arch()); 
// Contoh: 'x64', 'arm64'

// 3. Total dan memori bebas (dalam byte)
console.log('Total RAM:', os.totalmem() / (1024 ** 3), 'GB');
console.log('RAM bebas:', os.freemem() / (1024 ** 3), 'GB');

// 4. Direktori home pengguna
console.log('Home directory:', os.homedir());
// Contoh: '/home/andi' (Linux), 'C:\\Users\\Andi' (Windows)

// 5. Karakter pemisah baris — penting saat menulis file teks!
console.log('Baris baru:', JSON.stringify(os.EOL));
// Di Windows: "\r\n", di Unix/Linux/macOS: "\n"

// 6. Informasi CPU (array core)
const cpus = os.cpus();
console.log(`CPU: ${cpus.length} core`);
console.log('Model:', cpus[0].model);
```

Contoh nyata penggunaan: **menyimpan file konfigurasi di lokasi yang tepat**.  
Di Windows, aplikasi biasanya menyimpan data di `%APPDATA%`, sementara di Linux/macOS, di `~/.config`. Dengan modul `os`, kamu bisa otomatis menentukan lokasi ini:

```js
const path = require('path');
const os = require('os');

function getConfigPath(appName) {
  const platform = os.platform();
  if (platform === 'win32') {
    return path.join(os.homedir(), 'AppData', 'Roaming', appName);
  } else if (platform === 'darwin') {
    return path.join(os.homedir(), 'Library', 'Application Support', appName);
  } else {
    // Linux dan lainnya
    return path.join(os.homedir(), '.config', appName);
  }
}

const configDir = getConfigPath('MyFinanceApp');
console.log('Konfigurasi disimpan di:', configDir);
```

Atau, saat menulis log, kamu bisa pastikan format baris baru sesuai OS:
```js
const logEntry = `Error: Koneksi gagal${os.EOL}`;
// File log akan konsisten di semua platform
```

Modul `os` juga membantu dalam **diagnosa performa**. Misalnya, jika RAM bebas kurang dari 10%, kamu bisa menghentikan proses berat atau memberi peringatan:
```js
if (os.freemem() / os.totalmem() < 0.1) {
  console.warn('⚠️ Sistem kehabisan memori!');
}
```

**Kesimpulannya**, modul `os` mungkin terlihat kecil, tapi ia adalah jembatan penting antara aplikasimu dan dunia nyata tempat ia berjalan. Dengan informasi yang akurat tentang sistem operasi, kamu bisa membuat keputusan cerdas: menyimpan file di tempat yang benar, mengoptimalkan penggunaan sumber daya, menyesuaikan UX, dan memastikan kompatibilitas penuh di semua platform. Karena aplikasi yang hebat bukan hanya yang berjalan—tapi yang **paham di mana ia berada**. Dan dengan `os`, kamu tidak lagi buta terhadap lingkungan runtime-mu.
