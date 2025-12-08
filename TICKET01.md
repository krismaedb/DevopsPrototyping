
# 🎫 TICKET01 Complete Setup Guide – Healthcare Clinic Project

## Project Overview

The TICKET01 VM hosts **UVDesk**, a modern open-source helpdesk solution for the clinic's IT support team, featuring:

  * Email-to-ticket integration with the Mailcow server.
  * Agent & end-user portals.
  * SLA, queues, knowledge base, and ticket automation.

### Final Network Details

| Item | Value |
| :--- | :--- |
| **VM Name** | **TICKET01** |
| **IP Address** | **10.10.40.55/24** (corrected) |
| **Gateway** | **10.10.40.1** (HSRP VIP) |
| **DNS Servers** | 10.10.40.10 (DC01), 10.10.40.11 (DC02) |
| **VLAN** | 40 (IT/Servers) |
| **Hostname** | **ticket01.healthclinic.local** |
| **Helpdesk URL** | `http://10.10.40.55/helpdesk/public` (or `http://ticket.healthclinic.local`) |

### VM Specifications (Proxmox)

| Setting | Value |
| :--- | :--- |
| VM ID | **105** |
| Name | TICKET01 |
| Node | pve-main |
| CPU | 2–4 cores (host) |
| RAM | **4096 MB (4 GB)** |
| Disk | 40 GB (local-lvm) |
| Network | VirtIO → vmbr0 (VLAN 40) |
| ISO | `debian-12.8.0-amd64-netinst.iso` |

-----

## Part 1 – Create VM in Proxmox

Follow the standard VM creation steps, ensuring you use:

  * **VM ID:** **105**
  * **Name:** **TICKET01**

-----

## Part 2 – Debian 12 Installation (Headless Server)

1.  Boot from the ISO and choose **Install** (not Graphical install).
2.  Configure Language, Location, and Keyboard: **English** $\rightarrow$ **Canada** $\rightarrow$ **American English**.
3.  Set **Hostname:** `ticket01`
4.  Set **Domain name:** `healthclinic.local`
5.  Set **Root password:** `Clinic@2025!`
6.  Create normal user: `ticketadmin` / password: `Clinic@2025!`
7.  **Partitioning:** Guided $\rightarrow$ entire disk $\rightarrow$ all files in one partition.
8.  **Software selection:**
      * Select `[X] SSH server`
      * Select `[X] standard system utilities`
      * (Leave everything else unchecked)
9.  Install **GRUB** to `/dev/sda` $\rightarrow$ Finish $\rightarrow$ Reboot.

-----

## Part 3 – Static Network Configuration (IP 10.10.40.55)

1.  Edit the network configuration file:

    ```bash
    sudo nano /etc/network/interfaces
    ```

2.  Replace everything after "source ..." with the static configuration:

    ```text
    auto ens18
    iface ens18 inet static
        address 10.10.40.55/24
        gateway 10.10.40.1
        dns-nameservers 10.10.40.10 10.10.40.11
        dns-search healthclinic.local
    ```

    Save ($\text{Ctrl}+\text{O}$) $\rightarrow$ Enter $\rightarrow$ Exit ($\text{Ctrl}+\text{X}$).

3.  Verify the hosts file is correct for FQDN resolution:

    ```bash
    sudo nano /etc/hosts
    ```

    Ensure it contains:

    ```text
    127.0.0.1       localhost
    10.10.40.55     ticket01.healthclinic.local ticket01
    ::1             localhost ip6-localhost ip6-loopback
    ```

4.  Apply the network changes:

    ```bash
    sudo systemctl restart networking   # or reboot
    ```

5.  **Verify connectivity:**

    ```bash
    ip addr show ens18 | grep 10.10.40.55
    ping -c 3 10.10.40.1
    ping -c 3 10.10.40.10
    ping -c 3 8.8.8.8
    ```

-----

## Part 4 – System Preparation

Update the system and install essential utilities:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git vim nano net-tools htop dnsutils ca-certificates gnupg lsb-release unzip
sudo hostnamectl set-hostname ticket01.healthclinic.local
```

-----

## Part 5 – PostgreSQL 15 Installation

1.  Add the PostgreSQL repository and install the server and client:

    ```bash
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/keyrings/postgresql.gpg
    sudo chmod a+r /etc/apt/keyrings/postgresql.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/postgresql.gpg] http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list
    sudo apt update
    sudo apt install -y postgresql-15 postgresql-client-15 libpq-dev
    ```

2.  Create the UVDesk database and user:

    ```bash
    sudo -u postgres psql
    ```

    Inside `psql`, run the following commands:

    ```sql
    ALTER USER postgres PASSWORD 'Clinic@2025!';
    CREATE USER uvdesk WITH PASSWORD 'Clinic@2025!';
    CREATE DATABASE uvdesk OWNER uvdesk ENCODING 'UTF8';
    GRANT ALL PRIVILEGES ON DATABASE uvdesk TO uvdesk;
    \q
    ```

-----

## Part 6 – Apache + PHP + Extensions

1.  Install **Apache**, **PHP 8.2**, and the required extensions (including PostgreSQL/`pgsql` and IMAP/`imap`):

    ```bash
    sudo apt install -y apache2 \
        php8.2 php8.2-pgsql php8.2-imap php8.2-curl php8.2-gd \
        php8.2-mbstring php8.2-xml php8.2-intl php8.2-bcmath php8.2-zip \
        php8.2-mailparse php-pear php-dev
    ```

2.  Enable the `mailparse` extension (required for email processing):

    ```bash
    sudo pecl install mailparse
    echo "extension=mailparse.so" | sudo tee /etc/php/8.2/mods-available/mailparse.ini
    sudo phpenmod mailparse
    sudo systemctl restart apache2
    ```

3.  Install **Composer** (PHP dependency manager):

    ```bash
    curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/local/bin/composer
    ```

-----

## Part 7 – Install UVDesk

1.  Use Composer to create the UVDesk project in the web root:

    ```bash
    cd /var/www/html
    sudo composer create-project uvdesk/community-skeleton helpdesk
    ```

2.  Set the correct permissions for the web server:

    ```bash
    sudo chown -R www-data:www-data helpdesk
    sudo chmod -R 755 helpdesk
    ```

3.  Create the **Apache virtual host** configuration:

    ```bash
    sudo nano /etc/apache2/sites-available/helpdesk.conf
    ```

    Paste the configuration below:

    ```apache
    <VirtualHost *:80>
        ServerName ticket.healthclinic.local
        ServerAlias 10.10.40.55
        DocumentRoot /var/www/html/helpdesk/public
        <Directory /var/www/html/helpdesk/public>
            AllowOverride All
            Require all granted
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/helpdesk_error.log
        CustomLog ${APACHE_LOG_DIR}/helpdesk_access.log combined
    </VirtualHost>
    ```

4.  Enable the new site, enable the `rewrite` module, and restart Apache:

    ```bash
    sudo a2ensite helpdesk.conf
    sudo a2enmod rewrite
    sudo systemctl restart apache2
    ```

-----

## Part 8 – Web Installer (Browser)

1.  Open the URL in your browser: `http://10.10.40.55/helpdesk/public`

2.  Follow the **UVDesk setup wizard**:

      * **System Check:** Ensure everything is green.
      * **2 Database Configuration:**
          * Driver: **PostgreSQL**
          * Host: `localhost`
          * Port: `5432`
          * Database: `uvdesk`
          * Username: `uvdesk`
          * Password: `Clinic@2025!`
      * **3 Admin Account:**
          * Email: `admin@healthclinic.local`
          * Password: `Clinic@2025!`
      * **4 Finish:** You'll be redirected to the login page.

3.  Log in with `admin@healthclinic.local` / `Clinic@2025!`.

-----

## Part 9 – Quick Polish (Optional but Looks Great)

Add a **DNS A record** on your DC01 server to allow access via the friendly name:

  * **Name:** `ticket`
  * **IP:** `10.10.40.55`

The helpdesk can now be reached at `http://ticket.healthclinic.local`.

-----

## ✅ Completion Checklist

  * VM created (ID 105).
  * Debian 12 installed.
  * Static IP **10.10.40.55** configured.
  * PostgreSQL running & DB created.
  * Apache + PHP installed.
  * UVDesk installed and reachable.
  * Admin account created.
  * Can log in at `http://10.10.40.55/helpdesk/public`.

You now have a beautiful, modern helpdesk ready for your demo\! Take a screenshot of the **UVDesk dashboard** and the **ticket creation page**.

-----
