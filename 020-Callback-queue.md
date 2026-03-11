Halo teman-teman!

Kalau **Call Stack** adalah tempat fungsi dieksekusi, dan **Event Loop** adalah pengawas lalu lintas, maka **Callback Queue** adalah **antrian tempat callback menunggu giliran** untuk dipanggil. Ini adalah komponen krusial yang membuat JavaScript bisa menangani operasi asynchronous tanpa memblokir eksekusi utama.

Tanpa Callback Queue, JavaScript akan terjebak dalam paradigma "satu per satu"—dan aplikasi modern yang responsif tidak akan mungkin ada. Mari kita lihat **cara kerja Callback Queue secara nyata**, dengan kode yang bisa kamu jalankan sendiri!

---

### 🔍 **Apa Itu Callback Queue? (Penjelasan Sederhana)**

Bayangkan kamu di bank:
- Kamu ambil nomor antrian → itu seperti **mendaftarkan callback**.
- Kamu duduk menunggu → callback masuk ke **Callback Queue**.
- Teller memanggil nomor → **Event Loop** memindahkan callback dari queue ke **Call Stack**.
- Kamu dilayani → callback **dieksekusi**.

Jadi, urutannya:
```
Operasi Async → Selesai → Callback masuk Queue → Event Loop cek Call Stack kosong? → Ya → Callback ke Call Stack → Eksekusi!
```

---

### 🔧 **Langkah 1: Buat File Demo Callback Queue**

Buka terminal, lalu buat folder dan file:

```bash
mkdir callback-queue-demo
cd callback-queue-demo
```

Buat file:

```bash
# macOS/Linux:
touch demo.js

# Windows:
type nul > demo.js
```

---

### 📄 **Langkah 2: Tulis Kode untuk Mengamati Callback Queue**

Buka `demo.js`, lalu salin kode berikut:

```js
console.log('🚀 [1] Program dimulai');

// setTimeout 1 - masuk ke Callback Queue setelah 100ms
setTimeout(() => {
  console.log('⏰ [4] Timeout 1 selesai (100ms)');
}, 100);

// setTimeout 2 - masuk ke Callback Queue setelah 50ms
setTimeout(() => {
  console.log('⏰ [5] Timeout 2 selesai (50ms)');
}, 50);

// Kode sinkron - jalan langsung
console.log('✅ [2] Eksekusi kode sinkron');

// Loop berat (simulasi beban CPU)
for (let i = 0; i < 1e7; i++) {
  // Hanya untuk delay sinkron
}
console.log('✅ [3] Loop selesai');

// setTimeout 3 - masuk ke Callback Queue setelah 0ms
setTimeout(() => {
  console.log('⏰ [6] Timeout 3 selesai (0ms)');
}, 0);

console.log('✅ [3.5] Semua kode sinkron selesai');
```

---

### ▶️ **Langkah 3: Jalankan dan Amati Urutan Callback**

Di terminal, jalankan:

```bash
node demo.js
```

**Output yang muncul:**
```
🚀 [1] Program dimulai
✅ [2] Eksekusi kode sinkron
✅ [3] Loop selesai
✅ [3.5] Semua kode sinkron selesai
⏰ [5] Timeout 2 selesai (50ms)
⏰ [4] Timeout 1 selesai (100ms)
⏰ [6] Timeout 3 selesai (0ms)
```

> ❗ **Perhatikan hal menarik ini:**
> - Timeout 2 (50ms) selesai **sebelum** Timeout 1 (100ms) → sesuai waktu.
> - Tapi Timeout 3 (0ms) **tidak langsung jalan**! Ia tetap menunggu semua kode sinkron selesai.
> - Ini membuktikan: **Callback Queue harus menunggu Call Stack kosong**.

---

### 🧪 **Langkah 4: Callback Queue vs Microtask Queue**

JavaScript punya **dua jenis antrian**:
1. **Callback Queue** (Task Queue) → untuk `setTimeout`, `setInterval`, I/O callbacks
2. **Microtask Queue** → untuk `Promise.then()`, `queueMicrotask()`

Yang penting: **Microtask Queue diproses DULU sebelum Callback Queue!**

Buat file baru: `queue-priority.js`

```js
console.log('🚀 [1] Mulai');

// Microtask - masuk ke Microtask Queue
Promise.resolve().then(() => {
  console.log('⚡ [3] Microtask (Promise) selesai');
});

// Callback - masuk ke Callback Queue
setTimeout(() => {
  console.log('⏰ [4] Callback (setTimeout) selesai');
}, 0);

// Microtask kedua
Promise.resolve().then(() => {
  console.log('⚡ [3.5] Microtask kedua selesai');
});

console.log('✅ [2] Kode sinkron selesai');
```

Jalankan:

```bash
node queue-priority.js
```

**Output:**
```
🚀 [1] Mulai
✅ [2] Kode sinkron selesai
⚡ [3] Microtask (Promise) selesai
⚡ [3.5] Microtask kedua selesai
⏰ [4] Callback (setTimeout) selesai
```

> 🔥 **Ini bukti nyata:**
> - Semua microtask diproses **sebelum** callback dari setTimeout.
> - Meski setTimeout diatur 0ms, ia tetap **harus antri di belakang microtasks**.

---

### 📊 **Visualisasi Lengkap: Bagaimana Callback Queue Bekerja**

```
┌─────────────────────────────────────────────────────────┐
│                    JavaScript Runtime                    │
├─────────────────┬───────────────────┬───────────────────┤
│   Call Stack    │  Callback Queue   │  Microtask Queue  │
│                 │                   │                   │
│  [fungsiA()]    │  [setTimeout1]    │  [Promise1]       │
│  [fungsiB()]    │  [setTimeout2]    │  [Promise2]       │
│                 │                   │                   │
└─────────────────┴───────────────────┴───────────────────┘
         ↑                   ↑                   ↑
         │                   │                   │
         │                   │                   └─ Diproses DULU
         │                   │
         │                   └─ Diproses setelah Microtasks
         │
         └─ Dieksekusi SEKARANG
```

---

### 🧪 **Langkah 5: Contoh Nyata dengan File System (I/O Async)**

Buat file: `io-callback.js`

```js
const fs = require('fs');

console.log('[1] Program mulai');

// Operasi async - callback masuk ke Callback Queue saat selesai
fs.readFile('package.json', 'utf8', (err, data) => {
  console.log('[4] Callback readFile selesai');
});

// Kode sinkron lain
console.log('[2] Melanjutkan eksekusi...');

// Microtask
Promise.resolve().then(() => {
  console.log('[3] Microtask selesai');
});

console.log('[2.5] Selesai bagian sinkron');
```

Pastikan ada `package.json` di folder yang sama:

```bash
npm init -y
```

Jalankan:

```bash
node io-callback.js
```

**Output:**
```
[1] Program mulai
[2] Melanjutkan eksekusi...
[2.5] Selesai bagian sinkron
[3] Microtask selesai
[4] Callback readFile selesai
```

> 💡 **Pelajaran:**
> - `fs.readFile` tidak memblokir eksekusi.
> - Callback-nya menunggu di **Callback Queue**.
> - Microtask diproses dulu, baru callback I/O.

---

### ⚠️ **Peringatan Penting: Jangan Overload Callback Queue!**

Jika kamu mendaftarkan terlalu banyak callback sekaligus, antrian bisa memanjang dan performa menurun:

```js
// ❌ Contoh buruk: spam setTimeout
for (let i = 0; i < 10000; i++) {
  setTimeout(() => {
    console.log('Callback ke-', i);
  }, 0);
}
```

Solusinya:
- Gunakan **batching** (kelompokkan callback),
- Atau gunakan **Worker Threads** untuk beban berat.

---

### 🔗 **Hubungan Callback Queue dengan Event Loop**

Ini adalah siklus lengkapnya:

```
1. Kode sinkron masuk Call Stack → dieksekusi
2. Operasi async ditemukan → dikirim ke Web APIs / Thread Pool
3. Operasi async selesai → callback dikirim ke Callback Queue
4. Event Loop cek: Call Stack kosong?
   ├─ YA → Ambil callback dari queue → masukkan ke Call Stack
   └─ TIDAK → Tunggu sampai kosong
5. Ulangi dari langkah 1
```

---

### 🧪 **Langkah 6: Visualisasi Lengkap dengan Semua Konsep**

Buat file: `full-cycle.js`

```js
console.log('═══════════════════════════════════════════════');
console.log('🚀 [1] MULAI - Kode sinkron');

// setTimeout 1
setTimeout(() => {
  console.log('⏰ [6] CALLBACK 1 - dari setTimeout (100ms)');
}, 100);

// Promise 1 (Microtask)
Promise.resolve().then(() => {
  console.log('⚡ [4] MICROTASK 1 - dari Promise');
});

// setTimeout 2
setTimeout(() => {
  console.log('⏰ [7] CALLBACK 2 - dari setTimeout (50ms)');
}, 50);

// Kode sinkron
console.log('✅ [2] Kode sinkron lanjutan');

// Promise 2 (Microtask)
Promise.resolve().then(() => {
  console.log('⚡ [5] MICROTASK 2 - dari Promise');
});

// setTimeout 3 (0ms)
setTimeout(() => {
  console.log('⏰ [8] CALLBACK 3 - dari setTimeout (0ms)');
}, 0);

console.log('✅ [3] SEMUA KODE SINKRON SELESAI');
console.log('═══════════════════════════════════════════════');
```

Jalankan:

```bash
node full-cycle.js
```

**Output yang diharapkan:**
```
═══════════════════════════════════════════════
🚀 [1] MULAI - Kode sinkron
✅ [2] Kode sinkron lanjutan
✅ [3] SEMUA KODE SINKRON SELESAI
═══════════════════════════════════════════════
⚡ [4] MICROTASK 1 - dari Promise
⚡ [5] MICROTASK 2 - dari Promise
⏰ [7] CALLBACK 2 - dari setTimeout (50ms)
⏰ [6] CALLBACK 1 - dari setTimeout (100ms)
⏰ [8] CALLBACK 3 - dari setTimeout (0ms)
```

---

### **Kesimpulannya**

Callback Queue adalah **antrian tersembunyi** yang membuat JavaScript bisa menangani operasi asynchronous tanpa crash. Ia bekerja dengan aturan sederhana:

1. **Tunggu** sampai operasi async selesai,
2. **Antrikan** callback-nya,
3. **Tunggu lagi** sampai Call Stack kosong,
4. **Baru eksekusi**.

Dan yang paling penting: **Microtask Queue selalu didahulukan** sebelum Callback Queue.

Dengan memahami ini, kamu tidak lagi bingung kenapa:
- `setTimeout` tidak jalan tepat waktu,
- `Promise` selalu lebih cepat dari `setTimeout`,
- Atau kenapa kode async bisa "lompat" urutan eksekusinya.

Karena di dunia JavaScript, **tidak ada yang benar-benar "bersamaan"—semua ada gilirannya**. Dan Callback Queue adalah tempat di mana giliran itu ditentukan. 🎫
