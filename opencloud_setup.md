# OpenCloud Setup Guide

A concise guide to setting up ownCloud — a self-hosted open-source cloud storage platform.

---

## Requirements

- A server or VPS (Linux recommended)
- PHP 8.0+
- MySQL/MariaDB or PostgreSQL
- Apache or Nginx web server
- Node.js (for some integrations)

---

## Installation Steps

### 1. Install Dependencies

```bash
sudo apt update
sudo apt install apache2 mariadb-server php php-mysql php-curl php-gd \
  php-mbstring php-xml php-zip php-intl php-bcmath -y
```

### 2. Download ownCloud

```bash
wget https://download.owncloud.com/server/stable/owncloud-complete-latest.tar.bz2
tar -xjf owncloud-complete-latest.tar.bz2
sudo mv owncloud /var/www/html/
sudo chown -R www-data:www-data /var/www/html/owncloud
```

### 3. Set Up Database

```sql
CREATE DATABASE owncloud;
CREATE USER 'ownclouduser'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL PRIVILEGES ON owncloud.* TO 'ownclouduser'@'localhost';
FLUSH PRIVILEGES;
```

### 4. Configure Apache

Create `/etc/apache2/sites-available/owncloud.conf`:

```apache
Alias /owncloud "/var/www/html/owncloud/"
<Directory "/var/www/html/owncloud/">
    Options +FollowSymlinks
    AllowOverride All
    Require all granted
</Directory>
```

```bash
sudo a2ensite owncloud
sudo a2enmod rewrite headers env dir mime
sudo systemctl restart apache2
```

### 5. Complete Setup via Browser

Visit `http://your-server-ip/owncloud` and enter:
- Admin username/password
- Database credentials from Step 3

---

## Node.js Integration (Optional)

```bash
npm install -g @owncloud/js-client
```

```js
const owncloud = require('@owncloud/js-client');
const oc = new owncloud('http://your-server/owncloud');
oc.login('admin', 'password').then(() => {
  oc.files.list('/').then(files => console.log(files));
});
```

---

## Quick Tips

- Enable HTTPS with Let's Encrypt: `sudo certbot --apache`
- Increase upload limit in `/etc/php/8.x/apache2/php.ini`: set `upload_max_filesize = 512M`
- Enable cron jobs for background tasks: `crontab -e` → `*/5 * * * * php /var/www/html/owncloud/occ system:cron`

---

*Last updated: 2026-05-06*
