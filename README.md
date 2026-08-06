# Snipe-IT Server Installation

A step-by-step guide for deploying Snipe-IT on a Windows Server VM using IIS, PHP, MariaDB, and FastCGI.

## Environment

| Component | Value |
|------------|---------|
| OS | Windows Server 2022 Standard |
| Version | 21H2 |
| Build | 20348.5386 |
| Virtualization | VMware |
| Remote Access | RDCMan |
| Web Server | IIS |
| PHP Version | 8.5.9 (NTS) |
| Database | MariaDB |
| Composer | 2.x |
| Source Control | Git |

---

## Prerequisites

Install the following dependencies:

- PHP
- Composer
- IIS
- MariaDB/MySQL
- Git
- Chocolatey
- IIS URL Rewrite Module

---

## Installation Steps

### 1. Install PHP

Using Chocolatey:

```cmd
choco install php -y
```

Verify installation:

```cmd
php -v
```

Expected output:

```text
PHP 8.5.x
```

---

### 2. Configure PHP Extensions

Edit:

```text
C:\tools\php85\php.ini
```

Enable:

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

Verify:

```cmd
php -m
```

---

### 3. Install Composer

```cmd
choco install composer -y
```

Verify:

```cmd
composer diagnose
```

---

### 4. Install MariaDB

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

### 5. Download Snipe-IT

```cmd
cd C:\inetpub\wwwroot

git clone https://github.com/grokability/snipe-it.git
```

---

### 6. Install Dependencies

```cmd
cd C:\inetpub\wwwroot\snipe-it

composer install --no-dev --prefer-dist
```

---

### 7. Create Environment File

```cmd
copy .env.example .env
```

Generate application key:

```cmd
php artisan key:generate
```

---

### 8. Configure Database Connection

Edit `.env`:

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

### 9. Run Database Migrations

```cmd
php artisan migrate --force
```

---

### 10. Create Storage Link

```cmd
php artisan storage:link
```

---

## IIS Configuration

### Create Website

Site Name:

```text
Snipe-IT
```

Physical Path:

```text
C:\inetpub\wwwroot\snipe-it\public
```

Binding:

```text
http
Port: 8080
```

---

### Application Pool

Use:

```text
.NET CLR Version: No Managed Code
Managed Pipeline Mode: Integrated
```

---

## Enable PHP FastCGI

Register FastCGI:

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

## Folder Permissions

Grant write permissions:

```cmd
icacls "C:\inetpub\wwwroot\snipe-it\storage" /grant IIS_IUSRS:(OI)(CI)F /T
```

```cmd
icacls "C:\inetpub\wwwroot\snipe-it\storage" /grant IUSR:(OI)(CI)F /T
```

```cmd
icacls "C:\inetpub\wwwroot\snipe-it\bootstrap\cache" /grant IIS_IUSRS:(OI)(CI)F /T
```

```cmd
icacls "C:\inetpub\wwwroot\snipe-it\bootstrap\cache" /grant IUSR:(OI)(CI)F /T
```

---

## URL Rewrite

Install URL Rewrite Module:

```cmd
choco install urlrewrite -y
```

Restart IIS:

```cmd
iisreset
```

---

## Accessing Snipe-IT

Local access:

```text
http://localhost:8080
```

Remote access:

```text
http://<SERVER-IP>:8080
```

Example:

```text
http://192.168.1.10:8080
```

---

## Firewall Rule

Allow inbound TCP 8080:

```cmd
netsh advfirewall firewall add rule name="SnipeIT-8080" dir=in action=allow protocol=TCP localport=8080
```

---

## Troubleshooting

### PHP Startup: Unable to load dynamic library

Verify required extensions exist and are enabled in:

```text
C:\tools\php85\php.ini
```

---

### HTTP 500.19

Cause:

```text
Missing IIS URL Rewrite Module
```

Fix:

```cmd
choco install urlrewrite -y
```

---

### HTTP 403.14 Forbidden

Cause:

```text
PHP/FastCGI not configured
```

Fix:

Register FastCGI and PHP handler mappings.

---

### Permission denied: laravel.log

Cause:

```text
IIS user lacks write permission
```

Fix:

Apply `icacls` permissions to:

```text
storage
bootstrap\cache
```

---

## Final Result

Snipe-IT successfully deployed and accessible through:

```text
http://192.168.1.10:8080
```

with:

- IIS
- PHP 8.5
- MariaDB
- Composer
- FastCGI
- URL Rewrite
- Laravel Storage Links
