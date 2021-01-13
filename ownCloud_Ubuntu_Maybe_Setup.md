Notes taken from https://doc.owncloud.com/server/admin_manual/installation/ubuntu_18_04.html#create-the-occ-helper-script \
Initital Setup
```
sudo apt update && apt upgrade -y
sudo apt install -y apache2 libapache2-mod-php mariadb-server openssl php-imagick php-common php-curl php-gd php-imap php-intl php-json php-mbstring php-mysql php-ssh2 php-xml php-zip php-apcu php-redis redis-server wget
sed -i "s#html#owncloud#" /etc/apache2/sites-available/000-default.conf
service apache2 restart
```
Edit file "/etc/apache2/sites-available/owncloud.conf" \
File=
```
Alias /owncloud "/var/www/owncloud/"
<Directory /var/www/owncloud/>
  Options +FollowSymlinks
  AllowOverride All

 <IfModule mod_dav.c>
  Dav off
 </IfModule>

 SetEnv HOME /var/www/owncloud
 SetEnv HTTP_HOME /var/www/owncloud
</Directory>
```
Outside of file, continue
```
a2ensite owncloud.conf
systemctl reloud apache2
mysql -u root -e "CREATE DATABASE IF NOT EXISTS owncloud; GRANT ALL PRIVILEGES ON owncloud.* TO owncloud@localhost IDENTIFIED BY 'BETTER_PASS_HERE'";
a2enmod dir env headers mime rewrite setenvif
service apache2 reload
```
Download ownCloud
```
cd /var/www/
wget https://download.owncloud.org/community/owncloud-10.6.0.tar.bz2 && tar -xjf owncloud-10.6.0.tar.bz2 && chown -R www-data. owncloud
```
Install ownCloud
```
occ maintenance:install --database "mysql" --database-name "owncloud" --database-user "owncloud" --database-pass "BETTER_PASS_HERE" --admin-user "admin" --admin-pass "BETTER_PASS_HERE"
```
Configure ownCloud’s Trusted Domains
```
myip=$(hostname -I|cut -f1 -d ' ')
occ config:system:set trusted_domains 1 --value="$myip"
```
Caching and File Locking
```
occ config:system:set memcache.local --value '\OC\Memcache\APCu'
occ config:system:set memcache.locking --value '\OC\Memcache\Redis'
occ config:system:set redis --value '{"host": "127.0.0.1", "port": "6379"}' --type json
```
Log Rotation \
Edit file "/etc/logrotate.d/owncloud"
```
/var/www/owncloud/data/owncloud.log {
  size 10M
  rotate 12
  copytruncate
  missingok
  compress
  compresscmd /bin/gzip
}
```
Finish Up
```
cd /var/www/
chown -R www-data. owncloud
```
