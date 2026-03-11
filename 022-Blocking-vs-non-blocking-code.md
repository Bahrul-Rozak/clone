Halo teman-teman!

Bayangkan kamu di kasir supermarket:
- **Blocking**: Kasir melayani satu pelanggan, dan **semua pelanggan lain harus menunggu** sampai selesai—meski hanya untuk hitung kembalian.
- **Non-blocking**: Kasir catat pesanan pelanggan pertama, lalu **langsung layani pelanggan berikutnya** sambil menunggu pesanan pertama diproses di dapur.

Di dunia JavaScript dan Node.js, konsep ini disebut **Blocking vs Non-blocking Code**—dan memahami perbedaannya adalah **kunci untuk membuat aplikasi yang cepat dan responsif**.

Mari kita lihat **perbedaan nyata** antara keduanya, dengan contoh kode yang bisa kamu jalankan sendiri!

---

### 🔍 **Apa Itu Blocking vs Non-blocking?**

#### **Blocking Code (Sinkron)**
- Kode dieksekusi **berurutan, satu per satu**.
- Operasi berikutnya **harus menunggu** operasi sebelumnya selesai.
- Jika ada operasi berat (baca file besar, query database lambat), **semua terhenti**.
- Contoh: `fs.readFileSync()`, `for` loop berat, `while` loop tanpa break.

#### **Non-blocking Code (Asinkron)**
- Kode dieksekusi **tanpa menunggu** operasi sebelumnya selesai.
- Operasi berat dijalankan di latar belakang, sementara program **lanjut ke tugas berikutnya**.
- Saat operasi selesai, callback-nya dipanggil.
- Contoh: `fs.readFile()`, `setTimeout()`, `fetch()`, `Promise`.

---

### 🔧 **Langkah 1: Buat Folder & File Demo**

Buka terminal, lalu buat folder:

```bash
mkdir blocking-nonblocking-demo
cd blocking-nonblocking-demo
```

Buat file:

```bash
# macOS/Linux:
touch blocking.js nonblocking.js compare.js server-blocking.js server-nonblocking.js

# Windows:
type nul > blocking.js
type nul > nonblocking.js
type nul > compare.js
type nul > server-blocking.js
type nul > server-nonblocking.js
```

---

### 📄 **Langkah 2: Contoh Blocking Code**

Buka `blocking.js`, lalu salin kode berikut:

```js
const fs = require('fs');

console.log('🚀 [1] Program mulai');

// Operasi blocking: baca file secara sinkron
console.log('⏳ [2] Mulai baca file (BLOCKING)...');
const data = fs.readFileSync('package.json', 'utf8');
console.log('✅ [3] File selesai dibaca:', data.substring(0, 50) + '...');

// Kode berikutnya TIDAK JALAN sampai file selesai dibaca
console.log('✅ [4] Kode setelah baca file');

// Simulasi operasi berat lainnya
console.log('⏳ [5] Mulai loop berat (BLOCKING)...');
for (let i = 0; i < 1e8; i++) {
  // Loop berat untuk simulasi
}
console.log('✅ [6] Loop selesai');

console.log('✅ [7] Program selesai');
```

> 💡 Pastikan ada file `package.json` di folder yang sama:
> ```bash
> npm init -y
> ```

---

### 📄 **Langkah 3: Contoh Non-blocking Code**

Buka `nonblocking.js`, lalu salin kode berikut:

```js
const fs = require('fs');

console.log('🚀 [1] Program mulai');

// Operasi non-blocking: baca file secara asinkron
console.log('⏳ [2] Mulai baca file (NON-BLOCKING)...');
fs.readFile('package.json', 'utf8', (err, data) => {
  if (err) return console.error('❌ Error:', err);
  console.log('✅ [5] File selesai dibaca:', data.substring(0, 50) + '...');
});

// Kode berikutnya LANGSUNG JALAN tanpa menunggu
console.log('✅ [3] Kode setelah baca file (jalan duluan!)');

// Simulasi operasi lain
console.log('⏳ [4] Mulai operasi lain...');
setTimeout(() => {
  console.log('✅ [6] Operasi lain selesai');
}, 100);

console.log('✅ [7] Program selesai (tapi file belum selesai dibaca!)');
```

---

### ▶️ **Langkah 4: Jalankan dan Bandingkan**

Jalankan file blocking:

```bash
node blocking.js
```

**Output:**
```
🚀 [1] Program mulai
⏳ [2] Mulai baca file (BLOCKING)...
✅ [3] File selesai dibaca: {...}
✅ [4] Kode setelah baca file
⏳ [5] Mulai loop berat (BLOCKING)...
✅ [6] Loop selesai
✅ [7] Program selesai
```

> ⏱️ **Perhatikan**: Semua kode berjalan **berurutan**. Tidak ada yang "lompat".

---

Jalankan file non-blocking:

```bash
node nonblocking.js
```

**Output:**
```
🚀 [1] Program mulai
⏳ [2] Mulai baca file (NON-BLOCKING)...
✅ [3] Kode setelah baca file (jalan duluan!)
⏳ [4] Mulai operasi lain...
✅ [7] Program selesai (tapi file belum selesai dibaca!)
✅ [6] Operasi lain selesai
✅ [5] File selesai dibaca: {...}
```

> ⚡ **Perhatikan**:  
> - Kode `[3]` dan `[4]` jalan **sebelum** file selesai dibaca!  
> - Callback `[5]` jalan **terakhir**, meski dipanggil duluan!  
> - Ini adalah **non-blocking** dalam aksi!

---

### 📄 **Langkah 5: Perbandingan Langsung dalam Satu File**

Buka `compare.js`, lalu salin kode berikut:

```js
const fs = require('fs');

console.log('╔════════════════════════════════════════════════════════╗');
console.log('║           PERBANDINGAN BLOCKING vs NON-BLOCKING        ║');
console.log('╚════════════════════════════════════════════════════════╝\n');

// ========== BLOCKING ==========
console.log('🔴 BLOCKING CODE:');
console.log('─────────────────────────────────────────────────────────');

const startBlocking = Date.now();
console.log('[1] Mulai operasi blocking...');

// Baca file 1 (blocking)
const file1 = fs.readFileSync('package.json', 'utf8');
console.log('[2] File 1 selesai dibaca');

// Baca file 2 (blocking)
const file2 = fs.readFileSync('package.json', 'utf8');
console.log('[3] File 2 selesai dibaca');

// Baca file 3 (blocking)
const file3 = fs.readFileSync('package.json', 'utf8');
console.log('[4] File 3 selesai dibaca');

const endBlocking = Date.now();
console.log(`⏱️  Total waktu blocking: ${endBlocking - startBlocking}ms\n`);

// ========== NON-BLOCKING ==========
console.log('🟢 NON-BLOCKING CODE:');
console.log('─────────────────────────────────────────────────────────');

const startNonBlocking = Date.now();
console.log('[1] Mulai operasi non-blocking...');

let completed = 0;
const total = 3;

function checkDone() {
  completed++;
  if (completed === total) {
    const endNonBlocking = Date.now();
    console.log(`⏱️  Total waktu non-blocking: ${endNonBlocking - startNonBlocking}ms`);
    console.log('\n╔════════════════════════════════════════════════════════╗');
    console.log('║                   KESIMPULAN:                          ║');
    console.log('║  Non-blocking lebih cepat karena operasi berjalan      ║');
    console.log('║  secara paralel, bukan berurutan!                     ║');
    console.log('╚════════════════════════════════════════════════════════╝');
  }
}

// Baca file 1 (non-blocking)
fs.readFile('package.json', 'utf8', (err, data) => {
  console.log('[2] File 1 selesai dibaca');
  checkDone();
});

// Baca file 2 (non-blocking)
fs.readFile('package.json', 'utf8', (err, data) => {
  console.log('[3] File 2 selesai dibaca');
  checkDone();
});

// Baca file 3 (non-blocking)
fs.readFile('package.json', 'utf8', (err, data) => {
  console.log('[4] File 3 selesai dibaca');
  checkDone();
});
```

Jalankan:

```bash
node compare.js
```

**Output kira-kira:**
```
╔════════════════════════════════════════════════════════╗
║           PERBANDINGAN BLOCKING vs NON-BLOCKING        ║
╚════════════════════════════════════════════════════════╝

🔴 BLOCKING CODE:
─────────────────────────────────────────────────────────
[1] Mulai operasi blocking...
[2] File 1 selesai dibaca
[3] File 2 selesai dibaca
[4] File 3 selesai dibaca
⏱️  Total waktu blocking: 15ms

🟢 NON-BLOCKING CODE:
─────────────────────────────────────────────────────────
[1] Mulai operasi non-blocking...
[2] File 1 selesai dibaca
[3] File 2 selesai dibaca
[4] File 3 selesai dibaca
⏱️  Total waktu non-blocking: 8ms

╔════════════════════════════════════════════════════════╗
║                   KESIMPULAN:                          ║
║  Non-blocking lebih cepat karena operasi berjalan      ║
║  secara paralel, bukan berurutan!                     ║
╚════════════════════════════════════════════════════════╝
```

> 🔥 **Ini bukti nyata**:  
> - Blocking: 15ms (3 file × 5ms = 15ms, berurutan)  
> - Non-blocking: 8ms (3 file dibaca bersamaan, selesai hampir barengan)

---

### 📄 **Langkah 6: Dampak di Server Production**

Sekarang, mari lihat **dampak nyata** di server HTTP.

#### **Server dengan Blocking Code**

Buka `server-blocking.js`:

```js
const http = require('http');
const fs = require('fs');

const server = http.createServer((req, res) => {
  console.log(`📨 Request masuk: ${req.url}`);

  if (req.url === '/blocking') {
    // ❌ BLOCKING: baca file secara sinkron
    console.log('⏳ Mulai baca file (BLOCKING)...');
    const data = fs.readFileSync('package.json', 'utf8');
    
    // Simulasi proses berat
    for (let i = 0; i < 1e9; i++) {} // Loop berat!
    
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ message: 'Blocking request selesai', data: data }));
    console.log('✅ Blocking request selesai');
  }
  else if (req.url === '/fast') {
    // Request cepat
    res.writeHead(200);
    res.end('Fast response!');
    console.log('✅ Fast request selesai');
  }
});

server.listen(3000, () => {
  console.log('🔴 Server BLOCKING berjalan di http://localhost:3000');
  console.log('Coba akses:');
  console.log('- http://localhost:3000/blocking (akan memblokir)');
  console.log('- http://localhost:3000/fast (akan tertahan jika /blocking sedang diproses)');
});
```

---

#### **Server dengan Non-blocking Code**

Buka `server-nonblocking.js`:

```js
const http = require('http');
const fs = require('fs');

const server = http.createServer((req, res) => {
  console.log(`📨 Request masuk: ${req.url}`);

  if (req.url === '/nonblocking') {
    // ✅ NON-BLOCKING: baca file secara asinkron
    console.log('⏳ Mulai baca file (NON-BLOCKING)...');
    
    fs.readFile('package.json', 'utf8', (err, data) => {
      if (err) {
        res.writeHead(500);
        return res.end('Error membaca file');
      }

      // Simulasi proses berat (tapi tidak memblokir request lain)
      setTimeout(() => {
        res.writeHead(200, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ message: 'Non-blocking request selesai', data: data }));
        console.log('✅ Non-blocking request selesai');
      }, 1000); // Delay 1 detik
    });
  }
  else if (req.url === '/fast') {
    // Request cepat - TIDAK TERTAHAN oleh /nonblocking
    res.writeHead(200);
    res.end('Fast response!');
    console.log('✅ Fast request selesai');
  }
});

server.listen(3000, () => {
  console.log('🟢 Server NON-BLOCKING berjalan di http://localhost:3000');
  console.log('Coba akses:');
  console.log('- http://localhost:3000/nonblocking (tidak memblokir)');
  console.log('- http://localhost:3000/fast (tetap cepat, meski /nonblocking sedang diproses)');
});
```

---

### ▶️ **Langkah 7: Uji Dampak di Server**

#### **Uji Server Blocking**

1. Jalankan server blocking di terminal pertama:
   ```bash
   node server-blocking.js
   ```

2. Buka **dua tab browser**:
   - Tab 1: `http://localhost:3000/blocking`
   - Tab 2: `http://localhost:3000/fast`

3. **Hasil**: Tab 2 akan **menunggu** sampai Tab 1 selesai! Karena server single-threaded terblokir oleh operasi sinkron.

---

#### **Uji Server Non-blocking**

1. Hentikan server blocking (Ctrl+C), lalu jalankan server non-blocking:
   ```bash
   node server-nonblocking.js
   ```

2. Buka **dua tab browser**:
   - Tab 1: `http://localhost:3000/nonblocking`
   - Tab 2: `http://localhost:3000/fast`

3. **Hasil**: Tab 2 langsung responsif! Meski Tab 1 masih memproses, server tetap bisa melayani request lain.

---

### 📊 **Tabel Perbandingan Lengkap**

| **Aspek** | **Blocking Code** | **Non-blocking Code** |
|-----------|-------------------|-----------------------|
| **Eksekusi** | Berurutan, satu per satu | Paralel, tidak menunggu |
| **Responsivitas** | Rendah (UI freeze) | Tinggi (UI tetap responsif) |
| **Performa** | Lambat untuk I/O berat | Cepat, efisien |
| **Penggunaan CPU** | 100% saat operasi berat | Rendah, CPU idle saat menunggu I/O |
| **Scalability** | Buruk (satu request blokir semua) | Baik (bisa handle banyak request) |
| **Debugging** | Mudah (urutan jelas) | Lebih sulit (callback hell) |
| **Contoh** | `fs.readFileSync()`, loop berat | `fs.readFile()`, `Promise`, `async/await` |

---

### ⚠️ **Common Pitfalls (Jebakan Umum)**

#### ❌ **Pitfall 1: Loop Berat di Event Loop**

```js
// ❌ JANGAN LAKUKAN INI DI SERVER!
app.get('/calculate', (req, res) => {
  let result = 0;
  for (let i = 0; i < 1e12; i++) { // Loop sangat berat!
    result += i;
  }
  res.send(`Result: ${result}`);
});
// Semua request lain akan tertahan!
```

**Solusi**: Gunakan Worker Threads atau pindahkan ke background job.

---

#### ❌ **Pitfall 2: Mengira Semua Async = Non-blocking**

```js
// ❌ Salah: crypto.pbkdf2Sync itu SYNC!
const crypto = require('crypto');
const hash = crypto.pbkdf2Sync('password', 'salt', 100000, 64, 'sha512');
// Ini akan memblokir Event Loop!
```

**Solusi**: Gunakan versi async:
```js
crypto.pbkdf2('password', 'salt', 100000, 64, 'sha512', (err, hash) => {
  // Callback dipanggil saat selesai
});
```

---

#### ❌ **Pitfall 3: JSON.parse() pada Data Besar**

```js
// ❌ JSON.parse() itu SYNC dan bisa lambat!
app.post('/data', (req, res) => {
  const data = JSON.parse(hugeJsonString); // Bisa blokir!
  res.send('OK');
});
```

**Solusi**: Gunakan stream parser atau batasi ukuran payload.

---

### ✅ **Best Practices**

1. **Gunakan async/await** untuk kode yang lebih bersih:
   ```js
   async function handleRequest(req, res) {
     const data = await fs.promises.readFile('file.txt', 'utf8');
     res.send(data);
   }
   ```

2. **Hindari operasi sinkron di route handler**:
   ```js
   // ❌ Jangan
   const data = fs.readFileSync('config.json');
   
   // ✅ Gunakan
   const data = await fs.promises.readFile('config.json');
   ```

3. **Gunakan Worker Threads untuk CPU-bound tasks**:
   ```js
   const { Worker } = require('worker_threads');
   const worker = new Worker('./heavy-task.js');
   ```

4. **Batasi ukuran payload**:
   ```js
   app.use(express.json({ limit: '10mb' }));
   ```

5. **Gunakan streaming untuk file besar**:
   ```js
   fs.createReadStream('large-file.zip').pipe(res);
   ```

---

### 🧪 **Bonus: Visualisasi dengan Timestamp**

Buat file `visualize.js`:

```js
const fs = require('fs');

console.log('\n╔════════════════════════════════════════════════════════╗');
console.log('║              VISUALISASI WAKTU EKSEKUSI                ║');
console.log('╚════════════════════════════════════════════════════════╝\n');

function log(msg) {
  const time = new Date().toISOString().split('T')[1];
  console.log(`[${time}] ${msg}`);
}

log('🚀 Program mulai');

// Blocking operation
log('🔴 Mulai operasi BLOCKING...');
const startBlocking = Date.now();
fs.readFileSync('package.json');
const endBlocking = Date.now();
log(`🔴 Operasi BLOCKING selesai (${endBlocking - startBlocking}ms)`);

// Non-blocking operation
log('🟢 Mulai operasi NON-BLOCKING...');
const startNonBlocking = Date.now();
fs.readFile('package.json', () => {
  const endNonBlocking = Date.now();
  log(`🟢 Operasi NON-BLOCKING selesai (${endNonBlocking - startNonBlocking}ms)`);
});

log('✅ Kode setelah non-blocking (jalan duluan!)');

// Simulasi kode lain
setTimeout(() => {
  log('⏳ Operasi lain selesai');
}, 50);
```

Jalankan:

```bash
node visualize.js
```

---

### **Kesimpulannya**

Perbedaan antara **Blocking vs Non-blocking Code** adalah fondasi utama dalam pemrograman JavaScript, terutama di Node.js:

- **Blocking Code** = Berurutan, mudah dipahami, tapi **membunuh performa** di aplikasi yang menangani banyak request.
- **Non-blocking Code** = Paralel, lebih efisien, tapi **butuh pemahaman Event Loop** untuk menghindari bug.

Di server production:
- **Jangan pernah gunakan operasi sinkron** di route handler.
- **Gunakan async/await** atau Promise untuk operasi I/O.
- **Pindahkan CPU-bound tasks** ke Worker Threads.

Karena di dunia backend, **responsivitas = uang**. Semakin cepat server merespons, semakin banyak pengguna yang puas—dan semakin sedikit yang kabur ke kompetitor.

Dan dengan memahami konsep Blocking vs Non-blocking ini, kamu sekarang punya **senjata untuk membangun aplikasi yang cepat, scalable, dan tidak mudah crash**! 🚀
