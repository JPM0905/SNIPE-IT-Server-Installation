# Snipe-IT Server Installation

A step-by-step guide for deploying and configuring Snipe-IT on Windows Server using IIS, PHP, Composer, MariaDB, and FastCGI.

![image_alt](https://github.com/JPM0905/SNIPE-IT-Server-Installation/blob/1a8a1c0294b0dc78c1b5ed2f37aa04ccf06ed9da/image/Windows-Specification.png)
---

# Project Overview

This repository documents the complete deployment process for a self-hosted Snipe-IT Asset Management System running on a Windows Server virtual machine.

## Objectives

- Deploy Snipe-IT using IIS
- Configure PHP FastCGI
- Configure MariaDB database
- Create a production-ready asset management environment
- Allow network access through a custom IIS port

---

# Environment

## Windows Server Specification

![Windows Specification](images/Windows Host Environment

- Windows Server 2022 Standard
- Version 21H2
- Build 20348.5386
- Hosted on VMware
- Managed through RDCMan

---

# Dependencies

The following components were installed before deploying Snipe-IT:

- IIS (Internet Information Services)
- PHP 8.5.9 (NTS)
- Composer
- MariaDB
- Git
- Chocolatey
- IIS URL Rewrite Module

---

# PHP Installation & Configuration

## PHP Version Verification
![image_alt](https://github.com/JPM0905/SNIPE-IT-Server-Installation/blob/9724d88e288a268179723f8864d4ddbbfd3aaa41/image/php-version.png)

Verify installation:

```cmd
php -v
```

---

## PHP Configuration

![image_alt](https://github.com/JPM0905/SNIPE-IT-Server-Installation/blob/9724d88e288a268179723f8864d4ddbbfd3aaa41/image/php-configuration.png)

Verify configuration file:

```cmd
php --ini
```

Enabled extensions:

```ini
extension=bcmath
extension=curl
extension=exif
extension=fileinfo
extension=gd
extension=mbstring
extension=mysqli
extension=openssl
extension=pdo_mysql
extension=sodium
extension=zip
```

---

## PHP Modules

![image_alt](https://github.com/JPM0905/SNIPE-IT-Server-Installation/blob/9724d88e288a268179723f8864d4ddbbfd3aaa41/image/php-modules.png)

Verify PHP modules:

```cmd
php -m
```

---

# Composer

## Composer Verification

![image_alt](https://github.com/JPM0905/SNIPE-IT-Server-Installation/blob/9724d88e288a268179723f8864d4ddbbfd3aaa41/image/composer.png)

Verify installation:

```cmd
composer -V
```

Validate environment:

```cmd
composer diagnose
```

---

# MariaDB Configuration

## Database Verification

![image_alt](https://github.com/JPM0905/SNIPE-IT-Server-Installation/blob/9724d88e288a268179723f8864d4ddbbfd3aaa41/image/mysql.png)

Login to MariaDB:

```cmd
mysql -u root
```

Create database:

```sql
CREATE DATABASE snipeit CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Create user:

```sql
CREATE USER 'snipeit'@'localhost' IDENTIFIED BY '';
```

Grant permissions:

```sql
GRANT ALL PRIVILEGES ON snipeit.* TO 'snipeit'@'localhost';
FLUSH PRIVILEGES;
```

---

# Download Snipe-IT

Clone repository:

```cmd
cd C:\inetpub\wwwroot

git clone https://github.com/grokability/snipe-it.git
```

---

# Install Dependencies

Navigate to installation folder:

```cmd
cd C:\inetpub\wwwroot\snipe-it
```

Install dependencies:

```cmd
composer install --no-dev --prefer-dist
```

---

# Application Configuration

Create environment file:

```cmd
copy .env.example .env
```

Generate application key:

```cmd
php artisan key:generate
```

Update:

```env
APP_URL=http://192.168.1.10:8080

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=snipeit
DB_USERNAME=snipeit
DB_PASSWORD=
```

---

# Database Migration

Execute migrations:

```cmd
php artisan migrate --force
```

Create storage symlink:

```cmd
php artisan storage:link
```

Clear Laravel cache:

```cmd
php artisan optimize:clear
```

---

# IIS Configuration

## Website Settings

```text
Site Name: Snipe-IT

Physical Path:
C:\inetpub\wwwroot\snipe-it\public

Binding:
http://*:8080
```

---

## Application Pool

Recommended settings:

```text
.NET CLR Version:
No Managed Code

Managed Pipeline Mode:
Integrated
```

---

# FastCGI Configuration

Register PHP FastCGI:

```cmd
%windir%\System32\inetsrv\appcmd.exe set config /section:system.webServer/fastCgi /+"[fullPath='C:\tools\php85\php-cgi.exe']"
```

Register PHP Handler:

```cmd
%windir%\System32\inetsrv\appcmd.exe set config /section:system.webServer/handlers /+"[name='PHP-FastCGI',path='*.php',verb='*',modules='FastCgiModule',scriptProcessor='C:\tools\php85\php-cgi.exe',resourceType='Either']"
```

Restart IIS:

```cmd
iisreset
```

---

# URL Rewrite

Install URL Rewrite:

```cmd
choco install urlrewrite -y
```

Restart IIS:

```cmd
iisreset
```

---

# Preflight Validation

Before completing setup, verify all checks pass.

![image_alt](https://github.com/JPM0905/SNIPE-IT-Server-Installation/blob/9724d88e288a268179723f8864d4ddbbfd3aaa41/image/snipeit-preflight-check.png)
Expected result:

```text
All checks passed
```

---

# Final Deployment

After successful configuration and administrator account creation, Snipe-IT becomes accessible through:

```text
http://192.168.1.10:8080
```

---

# Troubleshooting

## HTTP 500.19

Cause:

```text
Missing IIS URL Rewrite Module
```

Solution:

```cmd
choco install urlrewrite -y
```

---

## HTTP 403.14

Cause:

```text
PHP/FastCGI Handler Not Configured
```

Solution:

Register FastCGI and PHP handlers.

---

## Permission Denied (laravel.log)

Solution:

```cmd
icacls "C:\inetpub\wwwroot\snipe-it\storage" /grant IIS_IUSRS:(OI)(CI)F /T
```

```cmd
icacls "C:\inetpub\wwwroot\snipe-it\bootstrap\cache" /grant IIS_IUSRS:(OI)(CI)F /T
```

---

# Result

✅ PHP Installed

✅ Composer Installed

✅ MariaDB Configured

✅ IIS Configured

✅ FastCGI Registered

✅ URL Rewrite Installed

✅ Laravel Migrations Completed

✅ Snipe-IT Successfully Deployed

✅ Accessible via HTTP Port 8080
