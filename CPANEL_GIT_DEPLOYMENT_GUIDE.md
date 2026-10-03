# 🚀 Complete cPanel Git Deployment Guide for Elephant House AR Game
## 🌐 Target URL: `https://ehwonderonline.com/activation`
## 📦 GitHub Repository: `https://github.com/dilmith-loops/elephant-house-kiosk.git`

This guide provides step-by-step instructions to host and deploy the Elephant House AR Game on your cPanel account using **Git™ Version Control** (Git Connect).

---

## 🏗️ Architecture Overview

The repository is built to run **both the Next.js Frontend and Laravel Backend together** on Apache/LiteSpeed cPanel hosting without needing Node.js or a separate PM2 process on the server:

* **🎮 Frontend (Next.js 16)**: Pre-compiled static export assets with `basePath: '/activation'` (`index.html`, `404.html`, `_next/`, `eh-portal/`, `privacy/`, `terms/`, 3D models, and MediaPipe WASM face tracking). Specially adapted with dynamic responsive scaling for **55" Floor Standing Android Touch Kiosks (Portrait 9:16)** as well as mobile devices.
* **🔌 Backend (Laravel 11)**: Located in [`backend/`](backend/), handling scores, high-frequency kiosk player sessions, multi-player device concurrency, leaderboards, and admin controls.
* **🔀 Root Router ([`.htaccess`](.htaccess) & [`index.php`](index.php))**:
  - Automatically routes `/activation/api/*` and `/activation/uploads/*` to the Laravel backend.
  - Automatically serves `/activation/`, `/activation/eh-portal/`, `/activation/terms/`, and static assets without collision.

---

## 📋 Prerequisites in cPanel

Before connecting Git, ensure the following are configured in your cPanel dashboard:

### 1. PHP Version (8.2 or 8.3)
1. Go to **cPanel ➔ MultiPHP Manager** (or **Select PHP Version**).
2. Set `ehwonderonline.com` to use **PHP 8.2** or **PHP 8.3**.
3. Verify that standard extensions are active:
   `pdo_mysql`, `mbstring`, `openssl`, `bcmath`, `curl`, `fileinfo`, `tokenizer`, `xml`, `zip`.

### 2. MySQL Database Setup
In **cPanel ➔ MySQL® Databases**:
- Create your database, user, and secure password.
- Assign all privileges to the user for that database.

If not already imported:
1. Go to **cPanel ➔ phpMyAdmin**.
2. Select your created database.
3. Click the **Import** tab.
4. Choose [`elephanthouse_game.sql`](elephanthouse_game.sql) from the repository root.
5. Click **Import** to load initial schema, settings, and default admin.

---

## 🚀 Step-by-Step Git Connect in cPanel

### Method 1: Using cPanel Git™ Version Control (Recommended UI Method)

1. Log in to your **cPanel** dashboard for `ehwonderonline.com`.
2. In the **Files** section, click **Git™ Version Control**.
3. Click the blue **Create** button (top right).
4. Fill in the fields:
   * **Clone URL**: `https://github.com/dilmith-loops/elephant-house-kiosk.git`
   * **Repository Path**: `public_html/activation`
     *(cPanel will automatically create the `activation` folder inside `public_html`)*
   * **Repository Name**: `activation`
5. Click **Create**.
   * cPanel will clone the repository into `/home/YOUR_USER/public_html/activation`.

---

### Method 2: Using cPanel Terminal / SSH (Fastest)

If your hosting provides the **Terminal** feature:

```bash
# 1. Go to public_html
cd ~/public_html

# 2. Clone repository into the 'activation' folder:
git clone https://github.com/dilmith-loops/elephant-house-kiosk.git activation
```

---

## ⚙️ Backend Configuration

### 1. Configure `backend/.env`
In cPanel **File Manager** (or Terminal):
1. Navigate into `public_html/activation/backend/`.
2. Copy `.env.cpanel.example` to `.env`:
   ```bash
   cp .env.cpanel.example .env
   ```
3. Edit `backend/.env` and insert your production database credentials:

```env
APP_NAME="Elephant House AR Game"
APP_ENV=production
APP_KEY=base64:REPLACE_WITH_YOUR_LARAVEL_APP_KEY
APP_DEBUG=false
APP_URL=https://ehwonderonline.com/activation

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_cpanel_db_name
DB_USERNAME=your_cpanel_db_user
DB_PASSWORD=your_cpanel_db_password

SESSION_DRIVER=database
CACHE_STORE=database
CORS_ALLOWED_ORIGINS=*
```

---

### 2. Verify PHP Dependencies (`backend/vendor/`)

#### If you have cPanel Terminal:
```bash
cd ~/public_html/activation/backend
composer install --no-dev --optimize-autoloader
php artisan config:cache
php artisan route:cache
```

#### If you do NOT have Terminal / Composer:
1. In your local repository, run:
   ```bash
   cd backend
   composer install --no-dev
   zip -r vendor.zip vendor
   ```
2. Upload `vendor.zip` to `public_html/activation/backend/` using cPanel File Manager and extract it.

---

### 3. File Permissions
Ensure Laravel storage and bootstrap cache directories are writable:
* `public_html/activation/backend/storage/` -> `775` (or `755`)
* `public_html/activation/backend/bootstrap/cache/` -> `775` (or `755`)

In cPanel Terminal:
```bash
chmod -R 775 ~/public_html/activation/backend/storage
chmod -R 775 ~/public_html/activation/backend/bootstrap/cache
```

---

## 🔄 How to Pull Future Updates (1-Click Deployment)

Whenever you push commits to GitHub `main`:

1. Open cPanel ➔ **Git™ Version Control**.
2. Find the repository (`activation`) and click **Manage**.
3. Go to the **Pull or Deploy** tab.
4. Click **Update from Remote**.
5. Your live game at `https://ehwonderonline.com/activation` is instantly updated!

---

## 🛠️ Verification & Troubleshooting Checklist

| URL to Test | Expected Result | Solution if Failing |
| :--- | :--- | :--- |
| **`https://ehwonderonline.com/activation`** | 3D Ice cream game interface with camera prompt (supports 55" Portrait Kiosk & mobile). | Check that `.htaccess` exists in `public_html/activation/` (Enable "Show Hidden Files" in cPanel File Manager). |
| **`https://ehwonderonline.com/activation/api/settings`** | Returns JSON `{"status":true,"settings":{...}}`. | Check `backend/.env` database credentials and verify PHP version is 8.2+. |
| **`https://ehwonderonline.com/activation/eh-portal/`** | Elephant House Admin Login screen. | Ensure permissions allow reading HTML files (`644`). |
| **500 Server Error** | HTTP 500 error page. | Check `backend/storage/logs/laravel.log` and verify `storage/` directory permissions (`chmod -R 775`). |
| **404 on API requests** | API endpoint returns 404. | Confirm `mod_rewrite` is enabled on Apache and `.htaccess` is present in `public_html/activation/`. |
