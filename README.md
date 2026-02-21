# 🌐 Android-tv-box-webserver
**Apache, PHP-FPM, Tiny File Manager, phpMyAdmin, OQuiz**  

This guide explains how to set up a **local web server on Termux** for testing web apps, with security best practices.  

---

## 🔹 1. Install Required Packages

Update Termux and install dependencies:

```bash
pkg update -y && pkg upgrade -y
pkg install apache2 php php-fpm mariadb git wget unzip -y
Start Apache and PHP-FPM:

bash
Copy code
apachectl start
pgrep php-fpm >/dev/null || php-fpm &
🔹 2. Test PHP
Create a test PHP file:

bash
Copy code
echo "<?php phpinfo(); ?>" > $PREFIX/share/apache2/default-site/htdocs/test.php
Open in your browser:

perl
Copy code
http://<your-local-ip>:8080/test.php
After testing, delete it:

bash
Copy code
rm $PREFIX/share/apache2/default-site/htdocs/test.php
🔹 3. MariaDB Setup (Secure)
Start MariaDB:

bash
Copy code
pgrep mysqld >/dev/null || mysqld_safe &
Create a database and secure user:

sql
Copy code
mariadb -u root
CREATE DATABASE oquiz_db;
CREATE USER 'db_user'@'localhost' IDENTIFIED BY 'your_strong_password';
GRANT ALL PRIVILEGES ON oquiz_db.* TO 'db_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
⚠️ Note: Check the MariaDB socket if PHP cannot connect via localhost:

sql
Copy code
SHOW VARIABLES LIKE 'socket';
🔹 4. Apache + PHP-FPM Configuration
Edit Apache configuration:

bash
Copy code
nano $PREFIX/etc/apache2/httpd.conf
Ensure required modules are loaded:

apache
Copy code
LoadModule proxy_module libexec/apache2/mod_proxy.so
LoadModule proxy_fcgi_module libexec/apache2/mod_proxy_fcgi.so
Add PHP-FPM handler:

apache
Copy code
<FilesMatch "\.php$">
    SetHandler "proxy:unix:/data/data/com.termux/files/usr/var/run/php-fpm.sock|fcgi://localhost/"
</FilesMatch>
🔹 5. Directory Setup
Set root to load your portfolio:

apache
Copy code
DirectoryIndex index.html index.php
Configure paths for apps:

Path	App
/	Portfolio
/tinyfilemanager	Tiny File Manager
/oquiz	OQuiz
/phpmyadmin	phpMyAdmin

🔹 6. Tiny File Manager Setup
bash
Copy code
cd $PREFIX/share/apache2/default-site/htdocs
wget -O tinyfm.zip https://github.com/prasathmani/tinyfilemanager/archive/refs/heads/master.zip
unzip tinyfm.zip
mv tinyfilemanager-master/* .
rm -rf tinyfilemanager-master tinyfm.zip

mkdir -p uploads
chmod 755 uploads
Security Tips

Use strong login credentials.

Avoid 777 permissions.

🔹 7. phpMyAdmin Setup
bash
Copy code
cd $PREFIX/share/apache2/default-site/htdocs
git clone https://github.com/phpmyadmin/phpmyadmin.git
mv phpmyadmin/* .
rm -rf phpmyadmin
Protect phpMyAdmin using Cloudflare Access or VPN.

🔹 8. OQuiz Setup
Copy OQuiz folder into htdocs and configure db.php:

php
Copy code
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
🔹 9. Recommended File Structure
pgsql
Copy code
htdocs/
├─ Oquiz/
├─ phpmyadmin/
├─ tinyfilemanager.php
├─ uploads/         (permissions 755)
├─ index.html       (portfolio)
└─ test.php         (delete after testing)
🔹 10. Security Recommendations
Keep uploads/ permissions at 755.

Remove test files after setup.

Use Cloudflare Access or VPN to protect /tinyfilemanager and /phpmyadmin.

Keep packages updated:

bash
Copy code
pkg update && pkg upgrade -y
🔹 11. Useful Commands
bash
Copy code
# Restart PHP-FPM
pkill php-fpm
php-fpm &

# Restart Apache
apachectl restart

# Login to MariaDB
mariadb -u root -p
🔹 12. Optional: Cloudflare Tunnel
Expose your server securely over HTTPS using cloudflared.

Map paths as in the table above.

Never commit your .json credentials.
