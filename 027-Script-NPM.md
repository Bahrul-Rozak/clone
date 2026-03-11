Halo teman-teman!

Di setiap proyek Node.js, ada satu fitur yang **sangat powerful** tapi sering diabaikan oleh pemula: **NPM Scripts**. Ini bukan sekadar perintah untuk menjalankan aplikasi—tapi **otomatisasi lengkap** untuk seluruh workflow development-mu, dari testing sampai deployment!

Bayangkan kamu bisa:
- Jalankan server development dengan satu perintah: `npm run dev`
- Format semua kode otomatis: `npm run format`
- Jalankan test suite lengkap: `npm test`
- Build aplikasi untuk production: `npm run build`
- Deploy ke server: `npm run deploy`

Semua ini **tanpa instalasi tools tambahan**—cukup pakai NPM yang sudah ada di Node.js!

Mari kita pelajari **NPM Scripts secara mendalam**, lengkap dengan contoh praktis yang bisa kamu jalankan sendiri—step by step!

---

## 🔍 **Apa Itu NPM Scripts?**

**NPM Scripts** adalah perintah CLI (Command Line Interface) yang didefinisikan di field `scripts` dalam file `package.json`. Kamu bisa menjalankannya dengan perintah:

```bash
npm run <nama-script>
```

Atau untuk script built-in:

```bash
npm start
npm test
npm install
```

---

## 🔧 **Langkah 1: Buat Proyek Baru**

Buka terminal, lalu buat folder baru:

```bash
mkdir npm-scripts-demo
cd npm-scripts-demo
```

Inisialisasi package.json:

```bash
npm init -y
```

---

## 📄 **Langkah 2: Lihat Struktur package.json Default**

Cek isi `package.json`:

```bash
cat package.json
```

Output:

```json
{
  "name": "npm-scripts-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Perhatikan field `scripts`—ini adalah tempat kita mendefinisikan semua perintah NPM Scripts!

---

## 🎯 **Built-in Scripts (Script Bawaan NPM)**

NPM punya beberapa script **built-in** yang bisa dijalankan **tanpa** keyword `run`:

| **Script** | **Perintah** | **Kegunaan** |
|------------|--------------|--------------|
| `start` | `npm start` | Menjalankan aplikasi |
| `test` | `npm test` atau `npm t` | Menjalankan test |
| `install` | `npm install` | Install dependencies |
| `stop` | `npm stop` | Menghentikan aplikasi |
| `restart` | `npm restart` | Restart aplikasi |

Script lain harus pakai `npm run`:

```bash
npm run dev
npm run build
npm run lint
```

---

## 📝 **Langkah 3: Buat Script Sederhana**

Edit `package.json` dan tambahkan beberapa script:

```json
{
  "name": "npm-scripts-demo",
  "version": "1.0.0",
  "description": "Demo NPM Scripts",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "node index.js",
    "test": "echo \"Running tests...\" && exit 0",
    "hello": "echo \"Hello from NPM Script!\"",
    "list-files": "ls -la",
    "show-env": "env"
  },
  "keywords": ["npm", "scripts"],
  "author": "",
  "license": "ISC"
}
```

---

## 🧪 **Langkah 4: Jalankan Script-Script Dasar**

### **Test 1: Jalankan Script Custom**

```bash
npm run hello
```

Output:

```
> npm-scripts-demo@1.0.0 hello
> echo "Hello from NPM Script!"

Hello from NPM Script!
```

---

### **Test 2: Lihat File di Direktori**

```bash
npm run list-files
```

Output:

```
> npm-scripts-demo@1.0.0 list-files
> ls -la

total 16
drwxr-xr-x   4 user  staff   128 Mar 11 10:00 .
drwxr-xr-x  20 user  staff   640 Mar 11 10:00 ..
-rw-r--r--   1 user  staff   280 Mar 11 10:00 package.json
drwxr-xr-x   2 user  staff    64 Mar 11 10:00 node_modules
```

---

### **Test 3: Tampilkan Environment Variables**

```bash
npm run show-env
```

Output: (panjang, menampilkan semua env vars)

---

### **Test 4: Built-in Script (test)**

```bash
npm test
# atau
npm t
```

Output:

```
> npm-scripts-demo@1.0.0 test
> echo "Running tests..." && exit 0

Running tests...
```

---

## 📝 **Langkah 5: Buat Aplikasi Node.js Sederhana**

Buat file `index.js`:

```bash
touch index.js
```

Isi dengan:

```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from NPM Scripts Demo!\n');
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`🚀 Server running on http://localhost:${PORT}`);
});
```

---

## ▶️ **Langkah 6: Jalankan Aplikasi dengan NPM Script**

Jalankan dengan script `start`:

```bash
npm start
```

Output:

```
> npm-scripts-demo@1.0.0 start
> node index.js

🚀 Server running on http://localhost:3000
```

Buka browser atau gunakan curl:

```bash
curl http://localhost:3000
```

Output:

```
Hello from NPM Scripts Demo!
```

Tekan `Ctrl+C` untuk stop server.

---

## 🛠️ **Langkah 7: Tambahkan Development Tools**

Install beberapa package untuk development:

```bash
npm install --save-dev nodemon eslint prettier
```

---

## 📝 **Langkah 8: Buat Script Development yang Lebih Lengkap**

Edit `package.json`:

```json
{
  "name": "npm-scripts-demo",
  "version": "1.0.0",
  "description": "Demo NPM Scripts",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "echo \"Running tests...\" && exit 0",
    "test:watch": "npm test -- --watch",
    "build": "echo \"Building application...\" && exit 0",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write \"**/*.js\"",
    "format:check": "prettier --check \"**/*.js\"",
    "clean": "rm -rf dist",
    "prepare": "npm run build",
    "prepublishOnly": "npm test && npm run build",
    "postinstall": "echo \"Dependencies installed successfully!\""
  },
  "keywords": ["npm", "scripts"],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "nodemon": "^2.0.20",
    "eslint": "^8.30.0",
    "prettier": "^2.8.0"
  }
}
```

---

## 🧪 **Langkah 9: Jalankan Script Development**

### **Test 1: Development Mode dengan Nodemon**

```bash
npm run dev
```

Output:

```
> npm-scripts-demo@1.0.0 dev
> nodemon index.js

[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node index.js`
🚀 Server running on http://localhost:3000
```

**Keuntungan Nodemon**: Server otomatis restart saat kamu edit file `index.js`!

Coba edit `index.js` (ubah pesan), lalu simpan. Server akan auto-restart!

---

### **Test 2: Linting Kode**

```bash
npm run lint
```

Output:

```
> npm-scripts-demo@1.0.0 lint
> eslint .

# (Tidak ada error jika kode bersih)
```

---

### **Test 3: Format Kode Otomatis**

```bash
npm run format
```

Output:

```
> npm-scripts-demo@1.0.0 format
> prettier --write "**/*.js"

index.js 50ms
```

Kode-mu otomatis diformat sesuai standar Prettier!

---

### **Test 4: Clean Build Directory**

```bash
# Buat folder dist dummy dulu
mkdir dist
echo "dummy" > dist/test.txt

# Sekarang hapus
npm run clean
```

Output:

```
> npm-scripts-demo@1.0.0 clean
> rm -rf dist
```

Folder `dist` terhapus!

---

## 🔗 **Script Chaining (Menggabungkan Script)**

Kamu bisa **menggabungkan beberapa script** dalam satu perintah:

### **1. Sequential Execution (Berurutan)**

Gunakan `&&` untuk menjalankan script **satu per satu** (script berikutnya hanya jalan jika sebelumnya sukses):

```json
"scripts": {
  "build": "npm run clean && npm run lint && npm run compile"
}
```

### **2. Parallel Execution (Paralel)**

Gunakan `&` untuk menjalankan script **bersamaan** (Unix/Mac):

```json
"scripts": {
  "dev": "nodemon index.js & npm run watch-css"
}
```

Atau gunakan package `npm-run-all` untuk cross-platform:

```bash
npm install --save-dev npm-run-all
```

```json
"scripts": {
  "dev:server": "nodemon index.js",
  "dev:css": "watch 'npm run build:css' src/css",
  "dev": "run-p dev:server dev:css"
}
```

---

## 📝 **Langkah 10: Buat Script Chain Lengkap**

Edit `package.json`:

```json
{
  "name": "npm-scripts-demo",
  "version": "1.0.0",
  "description": "Demo NPM Scripts",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    
    "clean": "rm -rf dist",
    "compile": "echo \"Compiling code...\" && mkdir -p dist && cp index.js dist/",
    "build": "npm run clean && npm run compile",
    
    "lint": "eslint .",
    "format": "prettier --write \"**/*.js\"",
    "check": "npm run lint && npm run format:check",
    
    "test": "echo \"Running tests...\" && exit 0",
    "test:watch": "npm test -- --watch",
    
    "prepare": "npm run build",
    "prepublishOnly": "npm test && npm run build",
    "postinstall": "echo \"✅ Dependencies installed!\"",
    
    "validate": "npm run check && npm test",
    "deploy": "npm run validate && npm run build && echo \"🚀 Deploying...\""
  },
  "devDependencies": {
    "nodemon": "^2.0.20",
    "eslint": "^8.30.0",
    "prettier": "^2.8.0"
  }
}
```

---

## 🧪 **Langkah 11: Test Script Chain**

### **Test 1: Build Process**

```bash
npm run build
```

Output:

```
> npm-scripts-demo@1.0.0 build
> npm run clean && npm run compile

> npm-scripts-demo@1.0.0 clean
> rm -rf dist

> npm-scripts-demo@1.0.0 compile
> echo "Compiling code..." && mkdir -p dist && cp index.js dist/

Compiling code...
```

Cek folder `dist`:

```bash
ls dist
```

Output:

```
index.js
```

---

### **Test 2: Validation Script**

```bash
npm run validate
```

Output:

```
> npm-scripts-demo@1.0.0 validate
> npm run check && npm test

> npm-scripts-demo@1.0.0 check
> npm run lint && npm run format:check

> npm-scripts-demo@1.0.0 lint
> eslint .

> npm-scripts-demo@1.0.0 format:check
> prettier --check "**/*.js"

Checking formatting...
All matched files use Prettier code style!

> npm-scripts-demo@1.0.0 test
> echo "Running tests..." && exit 0

Running tests...
```

---

## 🎯 **Pre & Post Hooks**

NPM Scripts punya **hook system** yang otomatis jalan sebelum/after script tertentu:

```json
"scripts": {
  "prestart": "echo \"🚀 Starting server...\"",
  "start": "node index.js",
  "poststart": "echo \"✅ Server started!\"",
  
  "pretest": "echo \"🧪 Running tests...\"",
  "test": "jest",
  "posttest": "echo \"✅ Tests completed!\"",
  
  "prebuild": "npm run clean",
  "build": "webpack --mode production",
  "postbuild": "echo \"📦 Build completed!\""
}
```

### **Contoh:**

```bash
npm start
```

Output:

```
> npm-scripts-demo@1.0.0 prestart
> echo "🚀 Starting server..."

🚀 Starting server!

> npm-scripts-demo@1.0.0 start
> node index.js

🚀 Server running on http://localhost:3000

> npm-scripts-demo@1.0.0 poststart
> echo "✅ Server started!"

✅ Server started!
```

---

## 🌍 **Environment Variables di Scripts**

Kamu bisa **set environment variables** langsung di script:

### **Unix/Mac/Linux:**

```json
"scripts": {
  "dev": "NODE_ENV=development nodemon index.js",
  "prod": "NODE_ENV=production node index.js",
  "port": "PORT=8080 node index.js"
}
```

### **Windows (PowerShell):**

```json
"scripts": {
  "dev": "set NODE_ENV=development&& nodemon index.js",
  "prod": "set NODE_ENV=production&& node index.js"
}
```

### **Cross-Platform (Gunakan `cross-env`):**

```bash
npm install --save-dev cross-env
```

```json
"scripts": {
  "dev": "cross-env NODE_ENV=development nodemon index.js",
  "prod": "cross-env NODE_ENV=production node index.js",
  "start": "cross-env PORT=3000 node index.js"
}
```

---

## 📝 **Langkah 12: Tambahkan Env Variables ke Proyek**

Edit `package.json`:

```json
{
  "scripts": {
    "start": "node index.js",
    "dev": "cross-env NODE_ENV=development nodemon index.js",
    "prod": "cross-env NODE_ENV=production node index.js",
    "port": "cross-env PORT=8080 node index.js",
    "test": "cross-env NODE_ENV=test jest"
  },
  "devDependencies": {
    "nodemon": "^2.0.20",
    "cross-env": "^7.0.3"
  }
}
```

Install `cross-env`:

```bash
npm install --save-dev cross-env
```

---

## 🧪 **Langkah 13: Test Environment Variables**

Edit `index.js` untuk membaca env vars:

```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end(`Hello from ${process.env.NODE_ENV || 'development'} mode!\n`);
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`🚀 Server running on http://localhost:${PORT}`);
  console.log(`Environment: ${process.env.NODE_ENV || 'development'}`);
});
```

Jalankan dengan berbagai env:

### **Test 1: Development Mode**

```bash
npm run dev
```

Output:

```
🚀 Server running on http://localhost:3000
Environment: development
```

---

### **Test 2: Production Mode**

```bash
npm run prod
```

Output:

```
🚀 Server running on http://localhost:3000
Environment: production
```

---

### **Test 3: Custom Port**

```bash
npm run port
```

Output:

```
🚀 Server running on http://localhost:8080
Environment: undefined
```

---

## 📊 **Tabel Referensi Cepat**

| **Script** | **Perintah** | **Kegunaan** |
|------------|--------------|--------------|
| `start` | `npm start` | Jalankan aplikasi |
| `dev` | `npm run dev` | Development mode |
| `test` | `npm test` | Jalankan test |
| `build` | `npm run build` | Build untuk production |
| `lint` | `npm run lint` | Lint kode |
| `format` | `npm run format` | Format kode |
| `clean` | `npm run clean` | Hapus build artifacts |
| `validate` | `npm run validate` | Lint + test + build |

---

## 🎯 **Best Practices untuk NPM Scripts**

### 1. **Gunakan Script Built-in dengan Bijak**

```json
{
  "scripts": {
    "start": "node index.js",      // ✅ Wajib ada
    "test": "jest",                 // ✅ Wajib ada
    "build": "webpack --mode production"  // ✅ Untuk production
  }
}
```

---

### 2. **Pisahkan Development dan Production Scripts**

```json
{
  "scripts": {
    "dev": "nodemon index.js",
    "dev:debug": "nodemon --inspect index.js",
    "start": "node index.js",
    "start:prod": "pm2 start index.js"
  }
}
```

---

### 3. **Gunakan Pre/Post Hooks untuk Setup/Cleanup**

```json
{
  "scripts": {
    "prebuild": "npm run clean",
    "build": "webpack --mode production",
    "postbuild": "echo \"Build complete!\"",
    "pretest": "npm run lint",
    "test": "jest",
    "posttest": "npm run coverage"
  }
}
```

---

### 4. **Buat Script Composite untuk Workflow Lengkap**

```json
{
  "scripts": {
    "validate": "npm run lint && npm run test",
    "prepare": "npm run validate && npm run build",
    "deploy": "npm run prepare && ./deploy.sh"
  }
}
```

---

### 5. **Gunakan `--` untuk Pass Arguments ke Script**

```bash
# Pass arguments ke Jest
npm test -- --watch --coverage

# Pass arguments ke custom script
npm run build -- --prod --minify
```

---

### 6. **Dokumentasikan Script di README.md**

```markdown
## Available Scripts

### Development
- `npm run dev` - Start development server with hot reload
- `npm run lint` - Lint all files
- `npm run format` - Format all files

### Production
- `npm start` - Start production server
- `npm run build` - Build for production

### Testing
- `npm test` - Run all tests
- `npm run test:watch` - Run tests in watch mode

### Deployment
- `npm run deploy` - Deploy to production
```

---

## 🛠️ **Tips & Tricks**

### **1. Lihat Semua Script yang Tersedia**

```bash
npm run
```

Output: Daftar semua script di `package.json`

---

### **2. Jalankan Multiple Script dengan npm-run-all**

```bash
npm install --save-dev npm-run-all
```

```json
{
  "scripts": {
    "dev:server": "nodemon index.js",
    "dev:watch": "watch 'npm run build:css' src",
    "dev": "run-p dev:server dev:watch"
  }
}
```

---

### **3. Gunakan `concurrently` untuk Run Multiple Process**

```bash
npm install --save-dev concurrently
```

```json
{
  "scripts": {
    "dev": "concurrently \"nodemon index.js\" \"npm run watch-css\""
  }
}
```

---

### **4. Debug Script dengan `--verbose`**

```bash
npm run build --verbose
```

---

### **5. Skip Script Hooks**

```bash
# Skip pre/post hooks
npm start --ignore-scripts
```

---

## 📊 **Contoh Lengkap: package.json Production-Ready**

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "Production-ready Node.js application",
  "main": "dist/index.js",
  "scripts": {
    "prestart": "npm run build",
    "start": "node dist/index.js",
    "dev": "cross-env NODE_ENV=development nodemon src/index.js",
    "build": "npm run clean && npm run compile",
    "compile": "tsc",
    "clean": "rm -rf dist",
    "lint": "eslint src --ext .js,.ts",
    "lint:fix": "eslint src --ext .js,.ts --fix",
    "format": "prettier --write \"src/**/*.{js,ts,json}\"",
    "format:check": "prettier --check \"src/**/*.{js,ts,json}\"",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "check": "npm run lint && npm run format:check",
    "validate": "npm run check && npm test",
    "prepare": "npm run validate && npm run build",
    "prepublishOnly": "npm run validate && npm run build",
    "postinstall": "echo \"✅ Dependencies installed!\"",
    "deploy": "npm run prepare && ./scripts/deploy.sh"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "nodemon": "^2.0.20",
    "eslint": "^8.30.0",
    "prettier": "^2.8.0",
    "jest": "^29.0.0",
    "cross-env": "^7.0.3"
  }
}
```

---

## 🎯 **Kesimpulannya**

**NPM Scripts** adalah **senjata rahasia** setiap developer Node.js yang produktif. Dengan memanfaatkan fitur ini, kamu bisa:

1. ✅ **Otomatisasi seluruh workflow** development
2. ✅ **Standardisasi perintah** di seluruh tim
3. ✅ **Kurangi ketergantungan** pada tools eksternal
4. ✅ **Dokumentasikan proses** dengan jelas di `package.json`
5. ✅ **Tingkatkan produktivitas** dengan satu perintah untuk banyak tugas

### **Rule of Thumb:**

> **"Jika kamu melakukan sesuatu lebih dari sekali—buat NPM Script-nya!"**

Dari development, testing, linting, formatting, build, sampai deployment—semuanya bisa diotomatisasi dengan NPM Scripts. Dan yang terbaik? **Semua developer yang clone proyekmu langsung tahu cara menjalankannya!**

Dengan menguasai NPM Scripts, kamu tidak hanya jadi developer yang lebih efisien—tapi juga lebih profesional dan mudah berkolaborasi! 🚀

---

## 💡 **Quick Reference**

```bash
# Built-in scripts (no "run" needed)
npm start
npm test
npm install

# Custom scripts
npm run dev
npm run build
npm run lint

# Pass arguments to script
npm test -- --watch --coverage

# List all available scripts
npm run

# Skip pre/post hooks
npm start --ignore-scripts

# Debug script
npm run build --verbose
```

Sekarang kamu sudah paham **NPM Scripts** dari basic sampai advanced! Terapkan ini di setiap proyekmu, dan rasakan bedanya dalam produktivitas! 🎓
