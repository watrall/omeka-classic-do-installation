# Omeka Classic Digital Ocean Manual Installation Tutorial

This repository contains a step-by-step guide for installing [Omeka Classic](https://omeka.org/classic/) on a [DigitalOcean Droplet](https://www.digitalocean.com/).  
It is written with **students, scholars, and cultural heritage professionals** in mind — people who are experts in their own fields but may not have much technical background.  

The tutorial is designed to do two things at once:
1. Help you successfully install and run Omeka Classic.  
2. Teach you the basics of how servers work, how to connect to them, and how software like Omeka Classic fits together with web servers, databases, and the command line.  

By the end, you’ll have a working Omeka Classic site and a clearer sense of how server-based web applications are set up and managed.  

---

This guide walks you through a **manual install of Omeka Classic** on a fresh [DigitalOcean Droplet](https://www.digitalocean.com/).  
It is written for **students and scholars with little to no server experience**. Every step explains **what you are doing** and **why** you are doing it.

---

## Table of Contents
1. [What is Omeka Classic?](#what-is-omeka-classic)
2. [What You’ll Need](#what-youll-need)
3. [Step 1: Create the Droplet](#step-1-create-the-droplet-your-small-cloud-server)
4. [Step 2: Connect to the Server with SSH](#step-2-connect-to-the-server-with-ssh)
5. [Step 3: Create a Safe Admin User](#step-3-create-a-safe-admin-user-recommended)
6. [Step 4: Turn on a Simple Firewall](#step-4-turn-on-a-simple-firewall)
7. [Step 5: Update the System](#step-5-update-the-system)
8. [Step 6: Install Apache (the Web Server)](#step-6-install-apache-the-web-server)
9. [Step 7: Install PHP (the Language Omeka Classic Runs On)](#step-7-install-php-the-language-omeka-classic-runs-on)
10. [Step 8: Install MySQL (the Database)](#step-8-install-mysql-the-database)
11. [Step 9: Download Omeka Classic](#step-9-download-omeka-classic-into-your-web-folder)
12. [Step 10: Configure Omeka Classic Database Connection](#step-10-tell-omeka-classic-how-to-reach-the-database)
13. [Step 11: Permissions for Files](#step-11-allow-apache-to-write-where-it-needs-to)
14. [Step 12: Configure Apache for Omeka Classic](#step-12-make-apache-serve-omeka-classic-from-your-ip-or-domain)
15. [Step 13: Run the Web Installer](#step-13-run-the-omeka-classic-web-installer)
16. [Step 14: Optional Domain and HTTPS](#step-14-optional-use-a-real-domain-and-https)
17. [Troubleshooting](#troubleshooting-cheatsheet)
18. [Quick Command Summary](#quick-command-summary-copy-paste-block)
19. [Official References](#reference-links-official)

---

## What is Omeka Classic?

[Omeka Classic](https://omeka.org/classic/) is a free, open-source web publishing platform for cultural heritage collections.  
Key features include:

- Designed for **single-site installations** (unlike Omeka S).  
- Flexible metadata management with Dublin Core at its core.  
- Extensible with plugins and themes.  
- Widely used in libraries, museums, and classrooms for collection-based projects.  

---

## What You’ll Need

- A [DigitalOcean account](https://www.digitalocean.com/).  
- A basic Droplet (Ubuntu 24.04 LTS, 1–2 GB RAM is fine).  
- Optional: a domain name (for easier access and HTTPS).  
- No prior server knowledge is required — this tutorial explains everything step by step.  

Omeka Classic requirements:  
- PHP 7.4–8.0 (newer PHP 8.1+ may not work).  
- MySQL 5.7+ or MariaDB equivalent.  
- Apache with `mod_rewrite`.  
See: [Omeka Classic Installation Docs](https://omeka.org/classic/docs/Installation/).

---

## Step 1: Create the Droplet (your small cloud server)

*(same as Omeka S tutorial)*  

---

## Step 2: Connect to the Server with SSH

*(same as Omeka S tutorial, with terminal explanation)*  

---

## Step 3: Create a Safe Admin User

*(same as Omeka S tutorial)*  

---

## Step 4: Turn on a Simple Firewall

*(same as Omeka S tutorial)*  

---

## Step 5: Update the System

*(same as Omeka S tutorial)*  

---

## Step 6: Install Apache (the Web Server)

*(same as Omeka S tutorial)*  

---

## Step 7: Install PHP (the Language Omeka Classic Runs On)

Omeka Classic works best with PHP 7.4–8.0. On Ubuntu 24.04, you may need to install PHP 8.0 explicitly:

```bash
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install -y php8.0 libapache2-mod-php8.0 php8.0-mysql php8.0-xml php8.0-mbstring php8.0-gd php8.0-zip
```

Enable Apache rewrite:

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

**Why.** Omeka Classic does not yet fully support PHP 8.1+, so using PHP 8.0 ensures compatibility.  
The `add-apt-repository` command adds a trusted software source so Ubuntu can install older PHP versions that Omeka Classic still relies on.

---

## Step 8: Install MySQL (the Database)

*(same as Omeka S tutorial, but DB can be named `omeka` instead of `omeka_s`)*  

```sql
CREATE DATABASE omeka;
CREATE USER 'omekauser'@'localhost' IDENTIFIED BY 'STRONG_PASSWORD';
GRANT ALL PRIVILEGES ON omeka.* TO 'omekauser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**Why.** These commands create a new database named `omeka`, set up a user with a password, and give that user full access to the database.

---

## Step 9: Download Omeka Classic into Your Web Folder

```bash
cd /var/www
sudo mkdir -p /var/www/omeka
sudo chown $USER:$USER /var/www/omeka
cd /var/www/omeka

wget https://github.com/omeka/Omeka/releases/latest/download/omeka.zip
unzip omeka.zip
shopt -s dotglob
mv omeka-*/* .
rm -rf omeka-* omeka.zip
```

**Why.** This fetches the official Omeka Classic release.  
The `shopt -s dotglob` command makes sure hidden files (like `.htaccess`) are also moved into place.

---

## Step 10: Tell Omeka Classic How to Reach the Database

Edit `db.ini`:

```bash
sudo nano /var/www/omeka/db.ini
```

Fill in:

```
host     = "localhost"
username = "omekauser"
password = "STRONG_PASSWORD"
dbname   = "omeka"
```

---

## Step 11: Allow Apache to Write Where It Needs To

```bash
sudo chown -R www-data:www-data /var/www/omeka
```

**Why.** Omeka Classic needs broader write access than Omeka S — the web server must be able to update files in themes, plugins, and uploads. That’s why we give ownership of the whole Omeka folder to Apache.

---

## Step 12: Make Apache Serve Omeka Classic from Your IP or Domain

*(same as Omeka S tutorial, but point DocumentRoot to `/var/www/omeka`)*  

---

## Step 13: Run the Omeka Classic Web Installer

In your browser:

```
http://YOUR_DOMAIN_OR_IP/install/install.php
```

Fill in:
- Admin username/password  
- Email  
- Site title  

---

## Step 14: Optional: Use a Real Domain and HTTPS

*(same as Omeka S tutorial)*  

---

## Troubleshooting Cheatsheet

- **PHP version errors**: Use PHP 7.4–8.0, not 8.1+.  
- **.htaccess errors**: Ensure Apache’s rewrite module is enabled and `AllowOverride All` is set.  
- **Permissions issues**: Omeka Classic may need writable access to `themes/` and `plugins/` as well as `files/`.  
- **Database errors**: If you see a database connection error, double-check that the username, password, and database name in `db.ini` match exactly what you created in MySQL.

---

## Quick Command Summary (Copy-Paste Block)

```bash
# Install Apache, MySQL, PHP 8.0
sudo apt update && sudo apt upgrade -y
sudo apt install -y apache2 mysql-server
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install -y php8.0 libapache2-mod-php8.0 php8.0-mysql php8.0-xml php8.0-mbstring php8.0-gd php8.0-zip
sudo a2enmod rewrite && sudo systemctl restart apache2

# Database
sudo mysql -e "CREATE DATABASE omeka; CREATE USER 'omekauser'@'localhost' IDENTIFIED BY 'STRONG_PASSWORD'; GRANT ALL PRIVILEGES ON omeka.* TO 'omekauser'@'localhost'; FLUSH PRIVILEGES;"

# Download Omeka Classic
cd /var/www && sudo mkdir omeka && sudo chown $USER:$USER omeka && cd omeka
wget https://github.com/omeka/Omeka/releases/latest/download/omeka.zip
unzip omeka.zip && shopt -s dotglob && mv omeka-*/* . && rm -rf omeka-* omeka.zip
```

---

## Reference Links (Official)

- **Omeka Classic**
  - [Home](https://omeka.org/classic/)  
  - [Docs](https://omeka.org/classic/docs/Installation/)  
  - [GitHub Repo](https://github.com/omeka/Omeka)  

- **DigitalOcean**
  - [Create a Droplet](https://docs.digitalocean.com/products/droplets/how-to/create/)  
  - [Initial Ubuntu Setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu)  
  - [Apache on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-22-04)  
  - [MySQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04)  
  - [Firewall with UFW](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu)  
  - [Let’s Encrypt SSL](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu)  
