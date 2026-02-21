# Android-tv-box-webserver (Apache, PHP-FPM, Tiny File Manager, phpMyAdmin, OQuiz)

This document explains how to set up a local web server on Termux for testing web apps like **OQuiz**, including **Apache**, **PHP-FPM**, **Tiny File Manager**, and **phpMyAdmin**, with security best practices.

---

## 1. Install Required Packages

Update Termux and install necessary packages:

```bash
pkg update -y && pkg upgrade -y
pkg install apache2 php php-fpm mariadb git wget unzip -y
Start Apache and PHP-FPM:

bash
Copy code
apachectl start
pgrep php-fpm >/dev/null || php-fpm &
2. Test PHP
Create a PHP test file:

bash
Copy code
echo "<?php phpinfo(); ?>" > $PREFIX/share/apache2/default-site/htdocs/test.php
Open in browser:

perl
Copy code
http://<your-local-ip>:8080/test.php
After verification, delete the test file:

bash
Copy code
rm $PREFIX/share/apache2/default-site/htdocs/test.php
3. MariaDB Setup (Secure)
Start MariaDB:

bash
Copy code
pgrep mysqld >/dev/null || mysqld_safe &
Login and create a secure user and database:

sql
Copy code
mariadb -u root
# CREATE DATABASE oquiz_db;
# CREATE USER 'db_user'@'localhost' IDENTIFIED BY 'your_strong_password';
# GRANT ALL PRIVILEGES ON oquiz_db.* TO 'db_user'@'localhost';
# FLUSH PRIVILEGES;
EXIT;
Check MariaDB socket if needed:

sql
Copy code
SHOW VARIABLES LIKE 'socket';
Use this socket in db.php if PHP cannot connect via localhost.

4. Apache + PHP-FPM Configuration
Edit Apache configuration:

bash
Copy code
nano $PREFIX/etc/apache2/httpd.conf
Ensure modules are loaded:

bash
Copy code
LoadModule proxy_module libexec/apache2/mod_proxy.so
LoadModule proxy_fcgi_module libexec/apache2/mod_proxy_fcgi.so
Add PHP-FPM handler:

apache
Copy code
<FilesMatch "\.php$">
    SetHandler "proxy:unix:/data/data/com.termux/files/usr/var/run/php-fpm.sock|fcgi://localhost/"
</FilesMatch>
5. Configure Root and App Directories
Set root to load your portfolio:

apache
Copy code
DirectoryIndex index.html index.php
To load TinyFileManager at /tinyfilemanager:

arduino
Copy code
http://yourdomain.com/tinyfilemanager
To load OQuiz at /oquiz:

arduino
Copy code
http://yourdomain.com/oquiz
To load phpMyAdmin at /phpmyadmin:

arduino
Copy code
http://yourdomain.com/phpmyadmin
6. Tiny File Manager Setup
bash
Copy code
cd $PREFIX/share/apache2/default-site/htdocs
wget -O tinyfm.zip https://github.com/prasathmani/tinyfilemanager/archive/refs/heads/master.zip
unzip tinyfm.zip
mv tinyfilemanager-master/* .
rm -rf tinyfilemanager-master tinyfm.zip

mkdir -p uploads
chmod 755 uploads
Security Notes:

Use strong login credentials.

Avoid 777 permissions.

7. phpMyAdmin Setup
bash
Copy code
cd $PREFIX/share/apache2/default-site/htdocs
git clone https://github.com/phpmyadmin/phpmyadmin.git
mv phpmyadmin/* .
rm -rf phpmyadmin
Protect phpMyAdmin with strong credentials or restrict via VPN/Cloudflare Access.

8. OQuiz Setup
Copy OQuiz folder into htdocs and configure db.php with placeholders:

php
Copy code
<?php
$host = "localhost";
$user = getenv('DB_USER');      // e.g., 'db_user'
$pass = getenv('DB_PASS');      // e.g., 'your_strong_password'
$db   = getenv('DB_NAME');      // e.g., 'oquiz_db'
$socket = "/data/data/com.termux/files/usr/var/run/mysqld.sock";

$conn = new mysqli($host, $user, $pass, $db, null, $socket);
if ($conn->connect_error) {
    die("DB Connection Failed: " . $conn->connect_error);
}
?>
9. Recommended File Structure
pgsql
Copy code
htdocs/
├─ Oquiz/
├─ phpmyadmin/
├─ tinyfilemanager.php
├─ uploads/   (permissions 755)
├─ index.html (portfolio)
└─ test.php   (delete after testing)
10. Security Recommendations
Keep uploads/ permissions at 755.

Remove test files after setup.

Use Cloudflare Access or VPN to protect /tinyfilemanager and /phpmyadmin.

Keep packages updated:

bash
Copy code
pkg update && pkg upgrade -y
11. Useful Commands
bash
Copy code
# Restart PHP-FPM
pkill php-fpm
php-fpm &

# Restart Apache
apachectl restart

# Login to MariaDB
mariadb -u root -p
12. Optional: Cloudflare Tunnel
Configure cloudflared to expose your server securely over HTTPS.

Map:

/ → portfolio

/tinyfilemanager → TinyFileManager

/oquiz → OQuiz

/phpmyadmin → phpMyAdmin
