# 🌐 Android TV Box Webserver (Termux)

**Stack:** Apache, PHP-FPM, MariaDB, Tiny File Manager, phpMyAdmin, OQuiz  

This guide explains how to set up a **local web server on Termux** (Android) for testing web applications, complete with security best practices.

---

## 🔹 1. Install Required Packages

Update Termux and install the necessary dependencies:

```bash
pkg update -y && pkg upgrade -y
pkg install apache2 php php-fpm mariadb git wget unzip -y
```

Start Apache and PHP-FPM:

```bash
apachectl start
pgrep php-fpm >/dev/null || php-fpm &
```

## 🔹 2. Test PHP

Create a test PHP file to verify the installation:

```bash
echo "<?php phpinfo(); ?>" > $PREFIX/share/apache2/default-site/htdocs/test.php
```

Open your browser and navigate to:
`http://<your-local-ip>:8080/test.php`

**Note:** After verifying PHP works, delete the test file for security:

```bash
rm $PREFIX/share/apache2/default-site/htdocs/test.php
```

## 🔹 3. MariaDB Setup (Secure)

Start the MariaDB server:

```bash
pgrep mysqld >/dev/null || mysqld_safe &
```

Log in to MariaDB to create a database and a secure user:

```sql
mariadb -u root
```

Execute the following SQL commands:

```sql
CREATE DATABASE oquiz_db;
CREATE USER 'db_user'@'localhost' IDENTIFIED BY 'your_strong_password';
GRANT ALL PRIVILEGES ON oquiz_db.* TO 'db_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

⚠️ **Note:** If PHP cannot connect via localhost, check the MariaDB socket path:

```sql
SHOW VARIABLES LIKE 'socket';
```

## 🔹 4. Apache + PHP-FPM Configuration

Edit the Apache configuration file:

```bash
nano $PREFIX/etc/apache2/httpd.conf
```

Ensure the required modules are uncommented and loaded:

```apache
LoadModule proxy_module libexec/apache2/mod_proxy.so
LoadModule proxy_fcgi_module libexec/apache2/mod_proxy_fcgi.so
```

Add the PHP-FPM handler to process `.php` files:

```apache
<FilesMatch "\.php$">
    SetHandler "proxy:unix:/data/data/com.termux/files/usr/var/run/php-fpm.sock|fcgi://localhost/"
</FilesMatch>
```

## 🔹 5. Directory Setup

Set the default index files to load your portfolio first:

```apache
DirectoryIndex index.html index.php
```

### App Routing Map

| Path | Application |
| --- | --- |
| `/` | Portfolio |
| `/tinyfilemanager` | Tiny File Manager |
| `/oquiz` | OQuiz |
| `/phpmyadmin` | phpMyAdmin |

## 🔹 6. Tiny File Manager Setup

Download and extract Tiny File Manager to your web root:

```bash
cd $PREFIX/share/apache2/default-site/htdocs
wget -O tinyfm.zip [https://github.com/prasathmani/tinyfilemanager/archive/refs/heads/master.zip](https://github.com/prasathmani/tinyfilemanager/archive/refs/heads/master.zip)
unzip tinyfm.zip
mv tinyfilemanager-master/* .
rm -rf tinyfilemanager-master tinyfm.zip

mkdir -p uploads
chmod 755 uploads
```

**Security Tips:**

* Use strong login credentials.
* Avoid `777` permissions. Keep directories like `uploads` at `755`.

## 🔹 7. phpMyAdmin Setup

Clone phpMyAdmin directly into your `htdocs` directory:

```bash
cd $PREFIX/share/apache2/default-site/htdocs
git clone [https://github.com/phpmyadmin/phpmyadmin.git](https://github.com/phpmyadmin/phpmyadmin.git)
mv phpmyadmin/* .
rm -rf phpmyadmin
```

*Tip: Protect phpMyAdmin using Cloudflare Access or a VPN.*

## 🔹 8. OQuiz Setup

Copy your OQuiz folder into `htdocs` and configure your database connection (e.g., `db.php`):

```php
<?php
$host = "localhost";
$user = getenv('DB_USER');
$pass = getenv('DB_PASS');
$db   = getenv('DB_NAME');
$socket = "/data/data/com.termux/files/usr/var/run/mysqld.sock";

$conn = new mysqli($host, $user, $pass, $db, null, $socket);
if ($conn->connect_error) {
    die("DB Connection Failed: " . $conn->connect_error);
}
?>
```

## 🔹 9. Recommended File Structure

```text
htdocs/
├── Oquiz/
├── phpmyadmin/
├── uploads/             # (permissions 755)
├── tinyfilemanager.php
└── index.html           # (portfolio)
```

*(Remember to delete `test.php` after testing!)*

## 🔹 10. Security Recommendations

* Keep `uploads/` permissions strictly at `755`.
* Remove test files immediately after setup.
* Use **Cloudflare Access** or a **VPN** to restrict access to `/tinyfilemanager` and `/phpmyadmin`.
* Keep packages updated regularly:
```bash
pkg update && pkg upgrade -y
```

## 🔹 11. Useful Commands

```bash
# Restart PHP-FPM
pkill php-fpm
php-fpm &

# Restart Apache
apachectl restart

# Login to MariaDB
mariadb -u root -p
```

## 🔹 12. Optional: Cloudflare Tunnel

Expose your local server securely over HTTPS using `cloudflared`.

* Map your application paths exactly as shown in the App Routing Map.
* ⚠️ **Never commit your Cloudflare `.json` credentials to GitHub.**

### Cloudflare Tunnel Setup for Android TV Box Web Server

This section explains how to configure and run a Cloudflare Tunnel to expose your local web server to a domain.

---

#### 1. Authenticate Cloudflare Tunnel

```bash
cloudflared login
```

This will open a browser to log in to your Cloudflare account.

---

#### 2. Create a Tunnel

```bash
cloudflared tunnel create <TUNNEL_NAME>
```

This will create a tunnel and generate a credentials file.

---

#### 3. Configure Tunnel

Create or edit the configuration file `~/.cloudflared/config.yml`:

```yaml
tunnel: <TUNNEL_NAME>
credentials-file: /path/to/credentials.json

ingress:
  - hostname: your-domain.com
    service: http://localhost:8080
  - service: http_status:404
```

Replace placeholders with your own tunnel name and domain. **Do not share credentials files publicly.**

---

#### 4. Configure Online in Cloudflare Dashboard

1. Log in to your [Cloudflare Dashboard](https://dash.cloudflare.com/).
2. Go to **Zero Trust → Access → Tunnels**.
3. Click your tunnel (`<TUNNEL_NAME>`).
4. Ensure the **Hostname** matches your domain (`your-domain.com`) and points to your tunnel.
5. Confirm DNS records are set to **CNAME** pointing to `*.cfargotunnel.com` as instructed.
6. Save changes.

This ensures your domain routes through Cloudflare to your local server.

---

#### 5. Run the Tunnel

To run the tunnel manually:

```bash
cloudflared tunnel run <TUNNEL_NAME>
```

---

#### 6. Auto-Start Tunnel and Services on Boot

You can safely auto-start Cloudflare Tunnel along with other services using Termux boot scripts. **Avoid exposing sensitive info** like real IPs or credentials.

Example sanitized boot script (`~/.termux/boot/start-sshd`):

```bash
#!/data/data/com.termux/files/usr/bin/sh

# Start SSH
sshd

# Start PHP-FPM if not running
pgrep php-fpm >/dev/null || php-fpm &

# Start Apache
apachectl start

# Start MariaDB if not running
pgrep mariadbd >/dev/null || mariadbd-safe &

# Start Cloudflare Tunnel in background
cloudflared tunnel run <TUNNEL_NAME> &
```

* Paths, tunnel names, and credentials are placeholders for security.
* Do not upload real credentials or IPs to public repositories.
