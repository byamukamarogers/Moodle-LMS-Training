# Lesson 1: Installing Moodle LMS on Ubuntu 24.04

## Objective
By the end of this lesson, participants will have a fully functional Moodle installation accessible from a web browser, using Apache, PHP, and MariaDB on Ubuntu Server 24.04.

---

## 1. Prerequisites
- Ubuntu 24.04 LTS server (fresh installation)
- Sudo privileges
- Domain name or subdomain (e.g., lms.example.com)
- Basic Linux command-line knowledge
- Internet connection

---

## 2. Step 1: Update the Server
```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

---

## 3. Step 2: Install Apache, PHP, and Extensions
```bash
sudo add-apt-repository ppa:ondrej/php # Press enter when prompted.
sudo apt update

sudo apt install apache2 -y

sudo apt install php8.2 php8.2-cli php8.2-fpm libapache2-mod-php8.2 \
php8.2-mysql php8.2-intl php8.2-xmlrpc php8.2-soap php8.2-curl \
php8.2-gd php8.2-mbstring php8.2-zip php8.2-xml php8.2-bcmath -y

sudo a2enmod rewrite
sudo systemctl restart apache2
sudo systemctl enable apache2
php -v
```

### Adjust PHP Configuration
```bash
sudo nano /etc/php/8.2/apache2/php.ini
```
Set or adjust:
```
max_input_vars = 5000
memory_limit = 512M
upload_max_filesize = 100M
post_max_size = 100M
```
Restart Apache:
```bash
sudo systemctl restart apache2
```

---

## 4. Step 3: Install and Configure Database
```bash
sudo apt install mariadb-server mariadb-client -y
sudo mysql_secure_installation
```

### Create Database and User
```bash
sudo mysql -u root -p
```
Then inside the MySQL shell:
```sql
CREATE DATABASE moodledb DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'StrongPassw0rd!';
GRANT ALL PRIVILEGES ON moodledb.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 5. Step 4: Configure Domain Mapping
Create an **A record** in your DNS provider pointing your domain to the server IP.

Test:
```bash
ping lms.example.com
```

---

## 6. Step 5: Download and Setup Moodle
```bash
For more details follow the official documentation on the link below
**https://docs.moodle.org/501/en/Step-by-step_Installation_Guide_for_Ubuntu**
cd /var/www/html
sudo wget https://download.moodle.org/download.php/direct/stable500/moodle-latest-500.tgz
sudo tar -zxvf moodle-latest-500.tgz
sudo mv moodle /var/www/html/moodle
```

Create the data directory:
```bash
sudo mkdir /var/www/moodledata
sudo chown -R www-data:www-data /var/www/moodledata
sudo chmod -R 755 /var/www/moodledata
sudo chown -R www-data:www-data /var/www/html/moodle
sudo chmod -R 755 /var/www/html/moodle
```

---

## 7. Step 6: Configure Apache Virtual Host
```bash
sudo nano /etc/apache2/sites-available/moodle.conf
```
Add the following configuration:
```apache
<VirtualHost *:80>
    ServerName lms.example.com
    DocumentRoot /var/www/html/moodle

    <Directory /var/www/html/moodle>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/moodle_error.log
    CustomLog ${APACHE_LOG_DIR}/moodle_access.log combined
</VirtualHost>
```
Enable the site:
```bash
sudo a2ensite moodle.conf
sudo a2dissite 000-default.conf
sudo systemctl reload apache2
```

---

## 8. Step 7: Complete Moodle Installation
Open your browser and go to:
```
http://lms.example.com
```
Follow the installation wizard and provide:
- Database: `moodledb`
- User: `moodleuser`
- Password: `StrongPassw0rd!`
- Data directory: `/var/www/moodledata`

Once installation finishes, you can log in as the Moodle admin.

---

## 9. Step 8: Lesson Wrap-up
### Recap
- Installed and configured Apache, PHP, and MariaDB
- Created database and user
- Downloaded and configured Moodle
- Set up domain mapping and Apache virtual host

### Next Lesson Preview
- Adding SSL with Let's Encrypt
- Setting up cron jobs
- Configuring backups and mail
- Installing themes and plugins

---

## Tips
- Ensure PHP version >= 8.2 for Moodle 5.0
- Keep `moodledata` outside webroot for security
- Set correct timezone:
```bash
sudo timedatectl set-timezone Africa/Kampala
```
