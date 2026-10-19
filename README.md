<div align="center">

<img src="https://icon.icepanel.io/Technology/svg/Ubuntu.svg" width="100" alt="Ubuntu logo" />
<img src="https://upload.wikimedia.org/wikipedia/commons/b/b5/Termux.svg" width="100" alt="Termux logo" />

# Ubuntu 22.04 & PHP 8.5 on Termux

**A step-by-step guide to setting up an Ubuntu 22.04 environment (via `proot-distro`) on Termux, including PHP 8.5 and SSH server installation.**

[![Platform](https://img.shields.io/badge/platform-Termux-34A853?style=for-the-badge&logo=android&logoColor=white)](https://termux.dev)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://releases.ubuntu.com/22.04/)
[![PHP](https://img.shields.io/badge/PHP-8.5-777BB4?style=for-the-badge&logo=php&logoColor=white)](https://www.php.net/)
[![proot-distro](https://img.shields.io/badge/proot--distro-Container-blue?style=for-the-badge)](https://github.com/termux/proot-distro)
[![SSH](https://img.shields.io/badge/SSH-Enabled-black?style=for-the-badge&logo=openssh&logoColor=white)](https://www.openssh.com/)
[![Apache](https://img.shields.io/badge/Apache-2.4-D22128?style=for-the-badge&logo=apache&logoColor=white)](https://httpd.apache.org/)
[![MariaDB](https://img.shields.io/badge/MariaDB-Database-003545?style=for-the-badge&logo=mariadb&logoColor=white)](https://mariadb.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=for-the-badge)](http://makeapullrequest.com)

</div>

---

## Table of Contents

1. [Initial Termux Setup](#1-initial-termux-setup)
2. [Installing Ubuntu 22.04](#2-installing-ubuntu-2204)
3. [Accessing & Logging into Ubuntu](#3-accessing--logging-into-ubuntu)
4. [Installing PHP 8.5](#4-installing-php-85)
5. [Installing & Configuring SSH Server](#5-installing--configuring-ssh-server)
6. [Installing Apache, MySQL/MariaDB & Other Tools](#6-installing-apache-mysqlmariadb--other-tools)
7. [Useful Shortcut Commands](#7-useful-shortcut-commands)

---

## 1. Initial Termux Setup

Before you begin, make sure the `proot-distro` package in your Termux is up to date so it can properly pull images from the registry.

Open the Termux app and run:

```bash
pkg update && pkg upgrade -y
pkg install proot-distro -y
```

## 2. Installing Ubuntu 22.04

Since the default name `ubuntu` might already be taken or have a previous installation history, use a custom name (`ubuntu22`) to avoid conflicts:

```bash
proot-distro install ubuntu:22.04 --name ubuntu22
```

## 3. Accessing & Logging into Ubuntu

Once the installation is complete, log into the Ubuntu system with:

```bash
proot-distro login ubuntu22
```

> **Note:** After running this command, your terminal prompt will change as if you're inside the Ubuntu system.

## 4. Installing PHP 8.5

Run the following commands **inside** the Ubuntu terminal (after logging in).

### 4.1 Update & Install Initial Dependencies

```bash
apt update && apt upgrade -y
apt install software-properties-common curl -y
```

### 4.2 Install PHP 8.5 and Common Extensions

Refresh the package list again, then install PHP 8.5 along with commonly used extensions (MySQL, cURL, XML, etc.):

```bash
apt update
apt install php8.5 php8.5-cli php8.5-common php8.5-mysql php8.5-zip php8.5-gd php8.5-mbstring php8.5-curl php8.5-xml php8.5-bcmath -y
```

### 4.3 Verify Installation

Check that PHP 8.5 was installed correctly:

```bash
php -v
```

## 5. Installing & Configuring SSH Server

### 5.1 Install SSH Server & Client

Log into your Ubuntu environment first (`proot-distro login ubuntu22`), then install the OpenSSH packages:

```bash
apt update
apt install openssh-server openssh-client -y
```

### 5.2 Configure the SSH Port

The default SSH port (22) usually requires full root privileges that the proot environment doesn't have, so it's recommended to change the SSH port to a free one (e.g. `2222`).

Edit the SSH configuration file:

```bash
nano /etc/ssh/sshd_config
```

Find the line `#Port 22` or `Port 22`, and change it to:

```
Port 2222
```

Also find the line `#PermitRootLogin ...` or `PermitRootLogin prohibit-password`, and change it to:

```
PermitRootLogin yes
```

Save the file: press `Ctrl + O`, then `Enter`, then exit with `Ctrl + X`.

### 5.3 Set a Password for the User (Root)

Since you're logged in as root in proot-distro, make sure root has a password so SSH authentication can work:

```bash
passwd
```

Enter your new password twice.

### 5.4 Starting the SSH Server

Since background services don't start automatically in Termux, you'll need to start the SSH server manually each time you want to use it (every time after `proot-distro login ubuntu22`):

```bash
service ssh start
# or use:
/usr/sbin/sshd
```

### 5.5 Connecting via SSH

You can connect to this Ubuntu environment from another device (e.g. a laptop/PC using PuTTY/terminal, or another Termux app on the same phone) with:

```bash
ssh root@127.0.0.1 -p 2222
```

Enter the password you created in step [5.3](#53-set-a-password-for-the-user-root).

> **Extra tip:** If you want to connect from another device (like a laptop) on the same Wi-Fi network, check Termux's IP address on your phone by running `ifconfig` or `ip a` **in the main Termux terminal** (not inside Ubuntu), then use that IP when connecting via SSH from your laptop, for example:
>
> ```bash
> ssh root@192.168.x.x -p 2222
> ```

## 6. Installing Apache, MySQL/MariaDB & Other Tools

Run the following commands **inside** the Ubuntu terminal to build out a full local development stack (Apache + MySQL/MariaDB + phpMyAdmin + Composer).

### 6.1 Install Apache Web Server

```bash
apt update
apt install apache2 -y
```

Connect PHP to Apache:

```bash
apt install libapache2-mod-php8.5 -y
service apache2 restart
```

Start Apache manually (background services don't auto-start in Termux):

```bash
service apache2 start
```

By default, Apache serves files from `/var/www/html`. Open `http://127.0.0.1` in a browser on the same device to verify it's running, or from another device on the same network using Termux's IP address (see the [SSH remote tip](#55-connecting-via-ssh) above).

### 6.2 Install MySQL/MariaDB

Ubuntu 22.04's default repo ships **MariaDB** (a drop-in MySQL replacement), which installs cleanly inside proot-distro — the official `mysql-server` package can fail there due to missing systemd:

```bash
apt install mariadb-server mariadb-client -y
```

Start the database service:

```bash
service mariadb start
```

Secure the installation (set root password, remove anonymous users, etc.):

```bash
mysql_secure_installation
```

Log into the MySQL/MariaDB shell:

```bash
mysql -u root -p
```

### 6.3 (Optional) Install phpMyAdmin

A web UI for managing your databases:

```bash
apt install phpmyadmin -y
```

During setup, select **apache2** when prompted for the web server, and choose **yes** to configure the database with `dbconfig-common`. Once installed, access it at `http://127.0.0.1/phpmyadmin`.

### 6.4 (Optional) Install Composer

PHP's standard dependency manager, needed for most modern PHP frameworks (Laravel, Symfony, etc.):

```bash
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
composer --version
```

### 6.5 Starting Everything Together

Since Termux doesn't keep services running in the background automatically, start them each time after logging in:

```bash
service mariadb start
service apache2 start
service ssh start
```

## 7. Useful Shortcut Commands

| Command | Description |
|---|---|
| `proot-distro login ubuntu22` | Log back into Ubuntu |
| `exit` or `Ctrl + D` | Exit Ubuntu (back to Termux) |
| `proot-distro remove ubuntu22` | Completely remove the Ubuntu environment |