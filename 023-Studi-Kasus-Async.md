Halo teman-teman!

Sekarang saatnya kita **gabungkan semua konsep asynchronous** yang sudah kita pelajari—Event Loop, Call Stack, Callback Queue, Microtask vs Macrotask, dan Blocking vs Non-blocking—dalam **satu studi kasus nyata** yang bisa kamu jalankan sendiri!

Kita akan buat **simulasi sistem restoran sederhana** yang menangani pesanan pelanggan. Ini adalah contoh nyata bagaimana aplikasi backend bekerja dengan berbagai operasi async sekaligus.

---

## 🍽️ **Studi Kasus: Sistem Restoran Sederhana**

Bayangkan kamu punya restoran dengan alur kerja seperti ini:

1. **Pelanggan memesan** → data masuk ke sistem
2. **Validasi pesanan** → cek apakah item ada di menu
3. **Proses pembayaran** → tunggu konfirmasi (simulasi async)
4. **Kirim ke dapur** → masak (simulasi delay)
5. **Kirim notifikasi** → SMS/Email ke pelanggan
6. **Update database** → simpan riwayat pesanan

Semua ini harus berjalan **tanpa memblokir** request lain—karena restoran bisa punya banyak pelanggan sekaligus!

---

### 🔧 **Langkah 1: Buat Folder & File Proyek**

Buka terminal, lalu buat folder:

```bash
mkdir restoran-async-demo
cd restoran-async-demo
```

Buat file-file yang diperlukan:

```bash
# macOS/Linux:
touch server.js menu.json orders.json

# Windows:
type nul > server.js
type nul > menu.json
type nul > orders.json
```

---

### 📄 **Langkah 2: Siapkan Data Menu (menu.json)**

Buka `menu.json`, lalu isi dengan:

```json
{
  "items": [
    { "id": 1, "nama": "Nasi Goreng", "harga": 15000 },
    { "id": 2, "nama": "Mie Ayam", "harga": 12000 },
    { "id": 3, "nama": "Bakso", "harga": 18000 },
    { "id": 4, "nama": "Soto", "harga": 14000 },
    { "id": 5, "nama": "Ayam Goreng", "harga": 20000 }
  ]
}
```

---

### 📄 **Langkah 3: Inisialisasi Database Orders (orders.json)**

Buka `orders.json`, lalu isi dengan array kosong:

```json
[]
```

---

### 📄 **Langkah 4: Tulis Kode Server Lengkap**

Buka `server.js`, lalu salin kode berikut (ini panjang, tapi saya jelaskan bagian per bagian):

```js
const http = require('http');
const fs = require('fs');
const url = require('url');

// ========================================
// 1. HELPER FUNCTIONS
// ========================================

// Baca file async (non-blocking)
function readFileAsync(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, 'utf8', (err, data) => {
      if (err) reject(err);
      else resolve(JSON.parse(data));
    });
  });
}

// Tulis file async (non-blocking)
function writeFileAsync(path, data) {
  return new Promise((resolve, reject) => {
    fs.writeFile(path, JSON.stringify(data, null, 2), 'utf8', (err) => {
      if (err) reject(err);
      else resolve();
    });
  });
}

// Simulasi delay async (misal: proses pembayaran, masak)
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Log dengan timestamp
function log(message) {
  const timestamp = new Date().toISOString().split('T')[1].split('.')[0];
  console.log(`[${timestamp}] ${message}`);
}

// ========================================
// 2. CORE BUSINESS LOGIC
// ========================================

// Validasi pesanan (sync - cepat, tidak perlu async)
function validateOrder(order, menu) {
  const errors = [];
  
  if (!order.customerName) errors.push('Nama pelanggan wajib diisi');
  if (!order.items || order.items.length === 0) errors.push('Pesanan tidak boleh kosong');
  
  order.items.forEach(item => {
    const menuItem = menu.items.find(m => m.id === item.id);
    if (!menuItem) errors.push(`Item ID ${item.id} tidak ditemukan di menu`);
    if (item.quantity <= 0) errors.push(`Jumlah item ${item.id} harus lebih dari 0`);
  });
  
  return errors;
}

// Hitung total harga
function calculateTotal(order, menu) {
  return order.items.reduce((total, item) => {
    const menuItem = menu.items.find(m => m.id === item.id);
    return total + (menuItem.harga * item.quantity);
  }, 0);
}

// Simulasi proses pembayaran (async - butuh waktu)
async function processPayment(orderId, total) {
  log(`💳 Memproses pembayaran untuk pesanan #${orderId} (Rp${total.toLocaleString()})...`);
  
  // Simulasi delay 1 detik (seperti menunggu bank)
  await delay(1000);
  
  // Simulasi 90% sukses, 10% gagal
  const success = Math.random() > 0.1;
  
  if (success) {
    log(`✅ Pembayaran pesanan #${orderId} berhasil!`);
    return { success: true, transactionId: `TRX-${Date.now()}` };
  } else {
    log(`❌ Pembayaran pesanan #${orderId} gagal!`);
    return { success: false, error: 'Pembayaran ditolak oleh bank' };
  }
}

// Simulasi masak di dapur (async - butuh waktu)
async function cookOrder(orderId, items) {
  log(`👨‍🍳 Pesanan #${orderId} masuk ke dapur...`);
  
  // Setiap item butuh waktu masak berbeda
  const cookTimes = {
    1: 2000, // Nasi Goreng: 2 detik
    2: 1500, // Mie Ayam: 1.5 detik
    3: 2500, // Bakso: 2.5 detik
    4: 1800, // Soto: 1.8 detik
    5: 2200  // Ayam Goreng: 2.2 detik
  };
  
  // Cari item dengan waktu masak terlama
  const maxTime = items.reduce((max, item) => {
    const time = cookTimes[item.id] || 2000;
    return Math.max(max, time);
  }, 0);
  
  // Simulasi masak
  await delay(maxTime);
  
  log(`✅ Pesanan #${orderId} selesai dimasak!`);
  return { ready: true, cookTime: maxTime };
}

// Simulasi kirim notifikasi (async)
async function sendNotification(customerName, orderId, message) {
  log(`📱 Mengirim notifikasi ke ${customerName}...`);
  
  // Simulasi delay 300ms
  await delay(300);
  
  log(`✅ Notifikasi terkirim: "${message}"`);
  return { sent: true };
}

// Simpan pesanan ke database (async)
async function saveOrder(order) {
  log(`💾 Menyimpan pesanan #${order.id} ke database...`);
  
  const orders = await readFileAsync('./orders.json');
  orders.push(order);
  await writeFileAsync('./orders.json', orders);
  
  log(`✅ Pesanan #${order.id} tersimpan!`);
  return { saved: true };
}

// ========================================
// 3. MAIN ORDER PROCESSING FUNCTION
// ========================================

async function processOrder(orderData) {
  const startTime = Date.now();
  log(`\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━`);
  log(`🚀 Memulai pemrosesan pesanan...`);
  
  try {
    // STEP 1: Baca menu (async)
    log(`📋 Membaca menu...`);
    const menu = await readFileAsync('./menu.json');
    log(`✅ Menu berhasil dimuat (${menu.items.length} item)`);
    
    // STEP 2: Validasi pesanan (sync)
    log(`🔍 Memvalidasi pesanan...`);
    const validationErrors = validateOrder(orderData, menu);
    if (validationErrors.length > 0) {
      throw new Error(`Validasi gagal: ${validationErrors.join(', ')}`);
    }
    log(`✅ Pesanan valid!`);
    
    // STEP 3: Hitung total (sync)
    const total = calculateTotal(orderData, menu);
    log(`💰 Total harga: Rp${total.toLocaleString()}`);
    
    // STEP 4: Generate ID pesanan
    const orderId = Date.now();
    log(`🏷️  ID Pesanan: #${orderId}`);
    
    // STEP 5: Proses pembayaran (async)
    const paymentResult = await processPayment(orderId, total);
    if (!paymentResult.success) {
      throw new Error(paymentResult.error);
    }
    
    // STEP 6: Masak pesanan (async)
    const cookResult = await cookOrder(orderId, orderData.items);
    
    // STEP 7: Kirim notifikasi (async) - BISA DIJALANKAN PARALEL!
    const notificationPromise = sendNotification(
      orderData.customerName,
      orderId,
      `Pesanan #${orderId} Anda sudah siap! Total: Rp${total.toLocaleString()}`
    );
    
    // STEP 8: Simpan ke database (async) - BISA DIJALANKAN PARALEL!
    const savePromise = saveOrder({
      id: orderId,
      customerName: orderData.customerName,
      items: orderData.items,
      total: total,
      timestamp: new Date().toISOString()
    });
    
    // Tunggu kedua operasi paralel selesai
    await Promise.all([notificationPromise, savePromise]);
    
    const endTime = Date.now();
    const duration = endTime - startTime;
    
    log(`🎉 Pesanan #${orderId} berhasil diproses dalam ${duration}ms!`);
    log(`━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\n`);
    
    return {
      success: true,
      orderId: orderId,
      total: total,
      duration: duration,
      message: 'Pesanan berhasil diproses!'
    };
    
  } catch (error) {
    log(`❌ Gagal memproses pesanan: ${error.message}`);
    log(`━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\n`);
    throw error;
  }
}

// ========================================
// 4. HTTP SERVER
// ========================================

const server = http.createServer(async (req, res) => {
  // Set headers
  res.setHeader('Content-Type', 'application/json');
  res.setHeader('Access-Control-Allow-Origin', '*');
  
  // Handle CORS preflight
  if (req.method === 'OPTIONS') {
    res.writeHead(204);
    return res.end();
  }
  
  const parsedUrl = url.parse(req.url, true);
  const path = parsedUrl.pathname;
  const method = req.method;
  
  // Route: POST /order - Buat pesanan baru
  if (path === '/order' && method === 'POST') {
    let body = '';
    req.on('data', chunk => body += chunk.toString());
    req.on('end', async () => {
      try {
        const orderData = JSON.parse(body);
        
        // Proses pesanan (async)
        const result = await processOrder(orderData);
        
        res.writeHead(200);
        res.end(JSON.stringify(result));
      } catch (error) {
        res.writeHead(400);
        res.end(JSON.stringify({ 
          success: false, 
          error: error.message 
        }));
      }
    });
  }
  
  // Route: GET /menu - Lihat menu
  else if (path === '/menu' && method === 'GET') {
    try {
      const menu = await readFileAsync('./menu.json');
      res.writeHead(200);
      res.end(JSON.stringify(menu));
    } catch (error) {
      res.writeHead(500);
      res.end(JSON.stringify({ error: 'Gagal membaca menu' }));
    }
  }
  
  // Route: GET /orders - Lihat semua pesanan
  else if (path === '/orders' && method === 'GET') {
    try {
      const orders = await readFileAsync('./orders.json');
      res.writeHead(200);
      res.end(JSON.stringify(orders));
    } catch (error) {
      res.writeHead(500);
      res.end(JSON.stringify({ error: 'Gagal membaca pesanan' }));
    }
  }
  
  // Route: GET /status - Status server
  else if (path === '/status' && method === 'GET') {
    res.writeHead(200);
    res.end(JSON.stringify({ 
      status: 'OK', 
      message: 'Server restoran berjalan dengan baik' 
    }));
  }
  
  // Route tidak ditemukan
  else {
    res.writeHead(404);
    res.end(JSON.stringify({ error: 'Route tidak ditemukan' }));
  }
});

// ========================================
// 5. START SERVER
// ========================================

const PORT = 3000;
server.listen(PORT, () => {
  log(`\n╔════════════════════════════════════════════════════════╗`);
  log(`║         🍽️  SISTEM RESTORAN - SERVER BERJALAN         ║`);
  log(`║              http://localhost:${PORT}                    ║`);
  log(`╚════════════════════════════════════════════════════════╝\n`);
  
  console.log('Endpoint yang tersedia:');
  console.log(`  GET  http://localhost:${PORT}/status    - Status server`);
  console.log(`  GET  http://localhost:${PORT}/menu      - Lihat menu`);
  console.log(`  GET  http://localhost:${PORT}/orders    - Lihat semua pesanan`);
  console.log(`  POST http://localhost:${PORT}/order     - Buat pesanan baru\n`);
  
  console.log('Contoh request POST /order:');
  console.log(JSON.stringify({
    customerName: "Andi",
    items: [
      { id: 1, quantity: 2 },  // 2 Nasi Goreng
      { id: 3, quantity: 1 }   // 1 Bakso
    ]
  }, null, 2));
  console.log('');
});
```

---

### ▶️ **Langkah 5: Jalankan Server**

Di terminal, jalankan server:

```bash
node server.js
```

**Output yang muncul:**
```
╔════════════════════════════════════════════════════════╗
║         🍽️  SISTEM RESTORAN - SERVER BERJALAN         ║
║              http://localhost:3000                     ║
╚════════════════════════════════════════════════════════╝

Endpoint yang tersedia:
  GET  http://localhost:3000/status    - Status server
  GET  http://localhost:3000/menu      - Lihat menu
  GET  http://localhost:3000/orders    - Lihat semua pesanan
  POST http://localhost:3000/order     - Buat pesanan baru

Contoh request POST /order:
{
  "customerName": "Andi",
  "items": [
    { "id": 1, "quantity": 2 },
    { "id": 3, "quantity": 1 }
  ]
}
```

---

### 🧪 **Langkah 6: Uji Sistem dengan Berbagai Request**

Buka **terminal baru** (jangan tutup server!), lalu uji berbagai endpoint:

#### ✅ **Uji 1: Cek Status Server**

```bash
curl http://localhost:3000/status
```

**Output:**
```json
{"status":"OK","message":"Server restoran berjalan dengan baik"}
```

---

#### ✅ **Uji 2: Lihat Menu**

```bash
curl http://localhost:3000/menu
```

**Output:**
```json
{
  "items": [
    {"id":1,"nama":"Nasi Goreng","harga":15000},
    {"id":2,"nama":"Mie Ayam","harga":12000},
    {"id":3,"nama":"Bakso","harga":18000},
    {"id":4,"nama":"Soto","harga":14000},
    {"id":5,"nama":"Ayam Goreng","harga":20000}
  ]
}
```

---

#### ✅ **Uji 3: Buat Pesanan Baru (Kasus Sukses)**

```bash
curl -X POST http://localhost:3000/order \
  -H "Content-Type: application/json" \
  -d '{
    "customerName": "Andi",
    "items": [
      { "id": 1, "quantity": 2 },
      { "id": 3, "quantity": 1 }
    ]
  }'
```

**Output di terminal server (lihat urutan async!):**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚀 Memulai pemrosesan pesanan...
📋 Membaca menu...
✅ Menu berhasil dimuat (5 item)
🔍 Memvalidasi pesanan...
✅ Pesanan valid!
💰 Total harga: Rp48,000
🏷️  ID Pesanan: #1709123456789
💳 Memproses pembayaran untuk pesanan #1709123456789 (Rp48,000)...
✅ Pembayaran pesanan #1709123456789 berhasil!
👨‍🍳 Pesanan #1709123456789 masuk ke dapur...
✅ Pesanan #1709123456789 selesai dimasak!
📱 Mengirim notifikasi ke Andi...
💾 Menyimpan pesanan #1709123456789 ke database...
✅ Notifikasi terkirim: "Pesanan #1709123456789 Anda sudah siap! Total: Rp48,000"
✅ Pesanan #1709123456789 tersimpan!
🎉 Pesanan #1709123456789 berhasil diproses dalam 4523ms!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Output di terminal curl:**
```json
{
  "success": true,
  "orderId": 1709123456789,
  "total": 48000,
  "duration": 4523,
  "message": "Pesanan berhasil diproses!"
}
```

> 🔥 **Perhatikan di log server:**
> - Notifikasi dan save database **dijalankan paralel** (lihat timestamp-nya hampir bersamaan)
> - Ini adalah **non-blocking code** dalam aksi!

---

#### ✅ **Uji 4: Buat Pesanan dengan Error (Item Tidak Ada)**

```bash
curl -X POST http://localhost:3000/order \
  -H "Content-Type: application/json" \
  -d '{
    "customerName": "Budi",
    "items": [
      { "id": 99, "quantity": 1 }
    ]
  }'
```

**Output:**
```json
{
  "success": false,
  "error": "Validasi gagal: Item ID 99 tidak ditemukan di menu"
}
```

---

#### ✅ **Uji 5: Lihat Semua Pesanan yang Sudah Tersimpan**

```bash
curl http://localhost:3000/orders
```

**Output:**
```json
[
  {
    "id": 1709123456789,
    "customerName": "Andi",
    "items": [
      { "id": 1, "quantity": 2 },
      { "id": 3, "quantity": 1 }
    ],
    "total": 48000,
    "timestamp": "2026-03-11T10:15:30.123Z"
  }
]
```

---

### 🧪 **Langkah 7: Uji Paralel Request (Buktikan Non-blocking!)**

Buka **3 terminal baru**, lalu jalankan 3 request **bersamaan**:

**Terminal 1:**
```bash
curl -X POST http://localhost:3000/order \
  -H "Content-Type: application/json" \
  -d '{"customerName":"Pelanggan 1","items":[{"id":1,"quantity":1}]}'
```

**Terminal 2:**
```bash
curl -X POST http://localhost:3000/order \
  -H "Content-Type: application/json" \
  -d '{"customerName":"Pelanggan 2","items":[{"id":2,"quantity":1}]}'
```

**Terminal 3:**
```bash
curl http://localhost:3000/status
```

**Hasil:**
- Terminal 3 akan langsung dapat respons **meski** Terminal 1 dan 2 sedang memproses pesanan!
- Ini membuktikan **server tidak terblokir** oleh operasi async yang sedang berjalan!

---

### 📊 **Penjelasan Konsep Async dalam Studi Kasus Ini**

#### 1. **Event Loop**
- Server menerima request → masuk ke Call Stack
- Operasi async (readFile, delay, dll) → dikirim ke Web APIs
- Callback → masuk Callback Queue
- Event Loop cek Call Stack kosong → pindahkan callback ke Call Stack

#### 2. **Call Stack**
- Setiap request masuk → fungsi `processOrder()` masuk stack
- Fungsi helper dipanggil → masuk ke atas stack
- Saat async → fungsi keluar dari stack, tunggu di queue

#### 3. **Callback Queue vs Microtask Queue**
- `setTimeout` (delay) → masuk Callback Queue
- `Promise.then()` → masuk Microtask Queue (lebih prioritas)

#### 4. **Microtask vs Macrotask**
- `readFileAsync()` → Promise → Microtask
- `delay()` → setTimeout → Macrotask
- Microtask diproses **sebelum** Macrotask!

#### 5. **Blocking vs Non-blocking**
- ✅ **Non-blocking**: `readFile`, `writeFile`, `setTimeout`
- ❌ **Blocking**: Tidak ada! Semua operasi I/O pakai async

#### 6. **Paralelisasi dengan Promise.all()**
```js
// Kedua operasi ini jalan BERSAMAAN, bukan berurutan!
await Promise.all([notificationPromise, savePromise]);
```

---

### 🎯 **Kesimpulan Studi Kasus**

Dalam studi kasus sistem restoran ini, kamu melihat **semua konsep async bekerja bersama**:

1. **Event Loop** mengatur alur request dan callback
2. **Call Stack** mengeksekusi fungsi satu per satu
3. **Callback Queue** menampung callback dari operasi async
4. **Microtask Queue** memproses Promise lebih cepat dari setTimeout
5. **Non-blocking code** memungkinkan server handle banyak request sekaligus
6. **Promise.all()** memungkinkan operasi paralel untuk efisiensi

Dan yang paling penting: **semua ini adalah pola yang dipakai di aplikasi production nyata!**

Dari e-commerce, food delivery, sampai fintech—semua menggunakan pola yang sama:
- Terima request
- Validasi (sync)
- Proses async (pembayaran, notifikasi, database)
- Return response

Sekarang kamu tidak hanya **paham teori** async JavaScript—tapi juga **bisa implementasinya** dalam aplikasi nyata! 🚀

---

### 💡 **Bonus: Visualisasi Timeline Eksekusi**

```
Request masuk
    ↓
[Call Stack] processOrder()
    ↓
[Call Stack] readFileAsync('./menu.json') → Async!
    ↓ (tunggu di Callback Queue)
[Call Stack] validateOrder() → Sync, cepat
    ↓
[Call Stack] calculateTotal() → Sync, cepat
    ↓
[Call Stack] processPayment() → Async!
    ↓ (tunggu di Callback Queue)
[Call Stack] cookOrder() → Async!
    ↓ (tunggu di Callback Queue)
[Call Stack] Promise.all([sendNotification(), saveOrder()]) → PARALEL!
    ↓
Event Loop cek queue → jalankan callback sesuai prioritas
    ↓
Response dikirim ke client
```

Inilah keindahan **asynchronous programming** di Node.js! 🎨
