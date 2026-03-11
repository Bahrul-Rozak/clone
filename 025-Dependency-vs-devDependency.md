Halo teman-teman!

Di dunia Node.js, kamu pasti sering melihat dua field di `package.json`: **`dependencies`** dan **`devDependencies`**. Keduanya berisi package yang diinstal, tapi **ada perbedaan mendasar** yang sangat penting untuk dipahami—terutama saat kamu mulai bekerja di proyek production atau berkolaborasi dengan tim.

Mari kita kupas tuntas **apa perbedaan antara dependencies dan devDependencies**, kapan harus menggunakan masing-masing, dan bagaimana ini mempengaruhi aplikasimu—lengkap dengan contoh praktis yang bisa kamu jalankan sendiri!

---

## 🔍 **Apa Itu Dependencies vs DevDependencies?**

### **Dependencies (Production Dependencies)**
```json
"dependencies": {
  "express": "^4.18.2",
  "mongoose": "^7.0.0",
  "bcrypt": "^5.1.0"
}
```

**Dependencies** adalah package yang **dibutuhkan untuk menjalankan aplikasi di production**. Tanpa package ini, aplikasimu **tidak akan berfungsi** saat di-deploy ke server.

Contoh:
- `express` - Web framework
- `mongoose` - MongoDB ODM
- `bcrypt` - Password hashing
- `dotenv` - Environment variables
- `cors` - CORS handling
- `jsonwebtoken` - Authentication

---

### **DevDependencies (Development Dependencies)**
```json
"devDependencies": {
  "nodemon": "^2.0.20",
  "eslint": "^8.30.0",
  "prettier": "^2.8.0",
  "jest": "^29.0.0"
}
```

**DevDependencies** adalah package yang **hanya dibutuhkan saat development**, tapi **tidak diperlukan di production**. Package ini biasanya untuk:

- **Testing** (Jest, Mocha, Chai)
- **Linting** (ESLint, Prettier)
- **Build tools** (Webpack, Babel, TypeScript compiler)
- **Development server** (Nodemon, ts-node)
- **Code quality** (Husky, lint-staged)

---

## 🎯 **Perbedaan Utama**

| **Aspek** | **dependencies** | **devDependencies** |
|-----------|------------------|---------------------|
| **Kapan dibutuhkan?** | Production (server live) | Development (saat coding) |
| **Diinstal di production?** | ✅ Ya | ❌ Tidak (default) |
| **Ukuran bundle** | Mempengaruhi ukuran final | Tidak mempengaruhi |
| **Contoh** | Express, Mongoose, bcrypt | Nodemon, ESLint, Jest |
| **Install command** | `npm install express` | `npm install --save-dev nodemon` |

---

## 🔧 **Langkah 1: Buat Folder Proyek Baru**

Buka terminal, lalu buat folder baru:

```bash
mkdir dependency-vs-devdep
cd dependency-vs-devdep
```

---

## 📄 **Langkah 2: Inisialisasi package.json**

Jalankan perintah:

```bash
npm init -y
```

File `package.json` akan dibuat dengan konfigurasi default.

---

## 📦 **Langkah 3: Install Dependencies (Production)**

Sekarang, mari kita instal beberapa package yang **dibutuhkan di production**:

```bash
npm install express dotenv cors
```

Cek file `package.json`:

```bash
cat package.json
```

Output:

```json
{
  "name": "dependency-vs-devdep",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.0.3",
    "express": "^4.18.2"
  }
}
```

Perhatikan: ketiga package masuk ke field **`dependencies`**.

---

## 🛠️ **Langkah 4: Install DevDependencies (Development Only)**

Sekarang, instal package yang **hanya dibutuhkan saat development**:

```bash
npm install --save-dev nodemon eslint prettier jest
```

Flag `--save-dev` atau `-D` memberi tahu NPM bahwa package ini adalah **dev dependency**.

Cek lagi `package.json`:

```bash
cat package.json
```

Output:

```json
{
  "name": "dependency-vs-devdep",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.0.3",
    "express": "^4.18.2"
  },
  "devDependencies": {
    "eslint": "^8.30.0",
    "jest": "^29.0.0",
    "nodemon": "^2.0.20",
    "prettier": "^2.8.0"
  }
}
```

Sekarang ada **dua field terpisah**:
- `dependencies` → untuk production
- `devDependencies` → untuk development

---

## 📝 **Langkah 5: Buat Aplikasi Sederhana**

Buat file `index.js`:

```bash
touch index.js
```

Isi dengan:

```js
require('dotenv').config();
const express = require('express');
const cors = require('cors');

const app = express();
const port = process.env.PORT || 3000;

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Express!' });
});

app.get('/health', (req, res) => {
  res.json({ status: 'OK', uptime: process.uptime() });
});

app.listen(port, () => {
  console.log(`🚀 Server berjalan di http://localhost:${port}`);
});
```

Buat file `.env`:

```bash
touch .env
```

Isi dengan:

```env
PORT=3000
NODE_ENV=development
```

---

## 🧪 **Langkah 6: Buat Script di package.json**

Edit `package.json` dan tambahkan script:

```json
{
  "name": "dependency-vs-devdep",
  "version": "1.0.0",
  "description": "Contoh perbedaan dependencies vs devDependencies",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest",
    "lint": "eslint .",
    "format": "prettier --write \"**/*.js\""
  },
  "keywords": ["nodejs", "npm", "dependencies"],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.0.3",
    "express": "^4.18.2"
  },
  "devDependencies": {
    "eslint": "^8.30.0",
    "jest": "^29.0.0",
    "nodemon": "^2.0.20",
    "prettier": "^2.8.0"
  }
}
```

---

## ▶️ **Langkah 7: Jalankan Aplikasi di Development**

Jalankan dengan Nodemon (dev tool):

```bash
npm run dev
```

Output:

```
🚀 Server berjalan di http://localhost:3000
[nodemon] clean exit - waiting for changes before restart
```

Buka browser atau gunakan curl:

```bash
curl http://localhost:3000/
```

Output:

```json
{"message":"Hello from Express!"}
```

**Perhatikan**: Nodemon adalah **devDependency**, jadi hanya bisa digunakan saat development.

---

## 🚀 **Langkah 8: Simulasi Production Deployment**

Sekarang, mari kita simulasi bagaimana aplikasi di-deploy ke production.

### **Cara 1: Install Hanya Dependencies (Recommended untuk Production)**

Buat folder baru untuk simulasi production:

```bash
cd ..
mkdir production-simulasi
cd production-simulasi
```

Copy file-file penting dari proyek sebelumnya:

```bash
cp ../dependency-vs-devdep/package.json .
cp ../dependency-vs-devdep/index.js .
cp ../dependency-vs-devdep/.env .
```

Sekarang, instal **hanya dependencies production** (tanpa devDependencies):

```bash
npm install --production
```

Atau:

```bash
npm ci --production
```

Perintah ini akan:
- ✅ Install package di `dependencies`
- ❌ **TIDAK** install package di `devDependencies`

Cek folder `node_modules`:

```bash
ls node_modules | grep -E "express|nodemon|eslint"
```

Output:

```
express
eslint        # ❌ Seharusnya tidak ada!
jest          # ❌ Seharusnya tidak ada!
nodemon       # ❌ Seharusnya tidak ada!
prettier      # ❌ Seharusnya tidak ada!
```

Wait... masih ada! Kenapa?

Karena NPM versi lama masih install semua. Untuk NPM 7+, gunakan:

```bash
npm install --omit=dev
```

Atau set environment variable:

```bash
export NODE_ENV=production
npm install
```

Sekarang coba lagi:

```bash
rm -rf node_modules package-lock.json
export NODE_ENV=production
npm install
```

Cek lagi:

```bash
ls node_modules | grep -E "express|nodemon|eslint"
```

Output:

```
express  # ✅ Hanya dependencies production
```

Sempurna! Sekarang **hanya package production yang terinstal**.

---

### **Cara 2: Gunakan Docker (Production Real-World)**

Buat file `Dockerfile`:

```bash
touch Dockerfile
```

Isi dengan:

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ONLY production dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Start application
CMD ["node", "index.js"]
```

Build Docker image:

```bash
docker build -t myapp-production .
```

Jalankan container:

```bash
docker run -p 3000:3000 myapp-production
```

Output:

```
🚀 Server berjalan di http://localhost:3000
```

**Keuntungan menggunakan Docker**:
- ✅ Hanya production dependencies yang masuk ke image
- ✅ Ukuran image lebih kecil
- ✅ Lebih aman (tidak ada dev tools di production)
- ✅ Konsisten di semua environment

---

## 📊 **Perbandingan Ukuran: Development vs Production**

Mari kita lihat perbedaan ukuran `node_modules`:

### **Development (dengan devDependencies)**

```bash
cd ../dependency-vs-devdep
du -sh node_modules
```

Output:

```
150M    node_modules
```

### **Production (tanpa devDependencies)**

```bash
cd ../production-simulasi
du -sh node_modules
```

Output:

```
45M    node_modules
```

**Perbedaan**: 150MB vs 45MB = **105MB lebih ringan!**

Ini sangat penting untuk:
- 🚀 **Faster deployment** (upload lebih cepat)
- 💰 **Lower hosting costs** (storage lebih kecil)
- 🔒 **Better security** (attack surface lebih kecil)
- ⚡ **Faster cold starts** (di serverless/FaaS)

---

## 🧪 **Langkah 9: Uji Perbedaan Behavior**

### **Test 1: Jalankan di Production Mode**

Dari folder `production-simulasi`:

```bash
npm start
```

Output:

```
🚀 Server berjalan di http://localhost:3000
```

✅ Berhasil! Karena `node` adalah command bawaan Node.js, bukan dari package.

---

### **Test 2: Coba Jalankan Dev Script di Production**

```bash
npm run dev
```

Output:

```
> dependency-vs-devdep@1.0.0 dev
> nodemon index.js

sh: nodemon: command not found
```

❌ **Error!** Karena `nodemon` adalah devDependency dan tidak terinstal di production.

Ini **sengaja**! Di production, kamu seharusnya:
- Tidak pakai Nodemon (auto-restart tidak diperlukan)
- Pakai process manager seperti PM2, systemd, atau Docker

---

### **Test 3: Coba Lint di Production**

```bash
npm run lint
```

Output:

```
> dependency-vs-devdep@1.0.0 lint
> eslint .

sh: eslint: command not found
```

❌ **Error lagi!** Karena ESLint tidak terinstal di production.

Dan ini **benar**! Linting seharusnya:
- Dilakukan di development
- Dilakukan di CI/CD pipeline
- **Tidak** dijalankan di production server

---

## 📋 **Kapan Menggunakan Dependencies vs DevDependencies?**

### ✅ **Gunakan `dependencies` untuk:**

Package yang **dibutuhkan saat runtime** (saat aplikasi berjalan):

```bash
# Web framework
npm install express

# Database
npm install mongoose mongodb pg

# Authentication
npm install jsonwebtoken bcrypt passport

# Environment variables
npm install dotenv

# CORS
npm install cors

# Body parser
npm install body-parser

# Logging
npm install winston morgan

# Validation
npm install joi yup

# Caching
npm install redis

# File upload
npm install multer
```

---

### ✅ **Gunakan `devDependencies` untuk:**

Package yang **hanya dibutuhkan saat development/testing**:

```bash
# Development server (auto-restart)
npm install --save-dev nodemon

# Testing
npm install --save-dev jest mocha chai supertest

# Linting
npm install --save-dev eslint eslint-config-prettier

# Formatting
npm install --save-dev prettier

# Build tools
npm install --save-dev webpack webpack-cli babel-loader

# TypeScript
npm install --save-dev typescript @types/node ts-node

# Git hooks
npm install --save-dev husky lint-staged

# Documentation
npm install --save-dev jsdoc

# Coverage
npm install --save-dev nyc
```

---

## 🎯 **Best Practices**

### 1. **Selalu Gunakan `--save-dev` untuk Dev Tools**

```bash
# ❌ Jangan
npm install nodemon

# ✅ Gunakan
npm install --save-dev nodemon
```

---

### 2. **Audit Package Sebelum Install**

Sebelum install package, cek dulu:
- Apakah ini dibutuhkan di production?
- Apakah ada alternatif yang lebih ringan?
- Apakah package ini masih aktif dikembangkan?

```bash
npm view <package-name> dependencies
npm view <package-name> devDependencies
```

---

### 3. **Gunakan `npm prune` untuk Bersihkan DevDependencies di Production**

Setelah deploy, bersihkan devDependencies yang tidak sengaja terinstal:

```bash
npm prune --production
```

Ini akan:
- Hapus semua package di `devDependencies`
- Hapus package yang tidak ada di `package.json`
- Menjaga `node_modules` tetap bersih

---

### 4. **Gunakan `.npmrc` untuk Otomatisasi**

Buat file `.npmrc` di root project:

```bash
touch .npmrc
```

Isi dengan:

```ini
# Selalu install production-only di CI/CD
production=true

# Jangan install package optional
optional=false

# Cache directory
cache=./.npm-cache
```

---

### 5. **Lock File Harus Di-commit**

Selalu commit `package-lock.json` ke Git:

```bash
git add package.json package-lock.json
git commit -m "Add dependencies"
```

Ini memastikan:
- Semua developer dapat versi yang sama persis
- Production deployment konsisten
- Tidak ada "works on my machine" problem

---

## 🛠️ **Tips Tambahan**

### **Cek Apa yang Terinstal**

```bash
# List semua package
npm list

# List hanya production dependencies
npm list --production

# List hanya devDependencies
npm list --dev

# List depth 0 (hanya top-level)
npm list --depth=0
```

---

### **Pindahkan Package dari Dependencies ke DevDependencies**

Jika kamu salah install:

```bash
# Uninstall dari dependencies
npm uninstall <package-name>

# Install sebagai devDependency
npm install --save-dev <package-name>
```

Atau gunakan tools seperti `npm-check`:

```bash
npx npm-check --save-dev
```

---

### **Cek Ukuran Setiap Package**

```bash
npx npm-check --skip-unused
```

Atau:

```bash
npm list --depth=0 --json | jq '.dependencies'
```

---

## 📊 **Visualisasi Struktur**

```
dependency-vs-devdep/
├── node_modules/
│   ├── express/           ← dependencies (production)
│   ├── cors/              ← dependencies (production)
│   ├── dotenv/            ← dependencies (production)
│   ├── nodemon/           ← devDependencies
│   ├── eslint/            ← devDependencies
│   ├── jest/              ← devDependencies
│   └── prettier/          ← devDependencies
├── index.js
├── .env
├── package.json
└── package-lock.json
```

---

## 🎯 **Kesimpulannya**

Memahami perbedaan antara **`dependencies`** dan **`devDependencies`** adalah **fundamental** dalam pengembangan Node.js yang baik. Ini bukan sekadar konvensi—tapi **best practice** yang mempengaruhi:

1. **Ukuran aplikasi** → Production lebih ringan tanpa dev tools
2. **Keamanan** → Attack surface lebih kecil di production
3. **Biaya hosting** → Storage dan bandwidth lebih hemat
4. **Performance** → Cold start lebih cepat (terutama di serverless)
5. **Konsistensi** → Semua environment pakai package yang sama

### **Rule of Thumb:**

> **"Jika package ini tidak dibutuhkan saat aplikasi BERJALAN di production → masukkan ke `devDependencies`."**

Dengan memisahkan keduanya dengan benar, kamu:
- ✅ Membuat aplikasi lebih efisien
- ✅ Mengurangi risiko security vulnerabilities
- ✅ Mempermudah kolaborasi dengan tim
- ✅ Mengikuti standar industri

Dan yang paling penting: **kamu menunjukkan bahwa kamu adalah developer yang peduli pada kualitas dan best practices**! 🚀

---

## 💡 **Quick Reference**

```bash
# Install production dependency
npm install express

# Install dev dependency
npm install --save-dev nodemon

# Install ONLY production (for deployment)
npm install --production

# Clean up devDependencies in production
npm prune --production

# List production dependencies only
npm list --production

# List devDependencies only
npm list --dev
```

Sekarang kamu sudah paham **kapan dan mengapa** menggunakan `dependencies` vs `devDependencies`. Terapkan ini di setiap proyekmu, dan kamu akan menjadi developer Node.js yang lebih profesional! 🎓
