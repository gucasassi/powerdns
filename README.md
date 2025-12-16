# PowerDNS: Ubuntu 24.04 (Proxmox LXC)

This repository contains the configuration files and step-by-step instructions for installing and configuring a PowerDNS environment (both Authoritative and Recursor servers) within an **Ubuntu 24.04 LTS** container running on **Proxmox LXC**.

## 🚀 Environment Setup

The following components are required for this deployment:

- **Host Environment:** Proxmox VE
- **Container OS:** Ubuntu 24.04 LTS (LXC Container)

## 🛠️ Proxmox LXC Container Creation

Before proceeding with the PowerDNS installation, let's create the Ubuntu 24.04 LXC container in Proxmox. Here are some recommended settings to ensure a smooth deployment:

- **CPU:** 1 Core
- **Memory:** 512MiB
- **Storage:** 8 GB or more
- **Template:** ubuntu-24.04-standard_24.04-2_amd64.tar.zst
- **Unprivileged container:** Recommended for better security

You can create the container using the Proxmox web interface or via CLI. Adjust resources as needed for your environment.

---

## 🛠️ Installation Steps

Once your container is up and running, follow these steps inside your Ubuntu 24.04 LXC container to set up PowerDNS:

### 1. Update and Install Dependencies

First, access your LXC container via the Proxmox shell or SSH:

```sh
login: root
Password:
```

Once inside, update your package lists and upgrade all installed software to ensure your system is up to date:

```sh
apt update && apt upgrade -y
```

### 2. Install MariaDB Server

PowerDNS requires a database to store zone and record information. **MariaDB** is a popular, reliable choice. Install it with the following command:

```bash
apt install mariadb-server -y
```

After installation, make sure the MariaDB service is running:

```bash
systemctl status mariadb
```

### 3. Secure MariaDB Installation

Once MariaDB is running, it's a good idea to secure your installation:

```bash
mysql_secure_installation
```

You will be prompted with several questions. Here is a recommended set of answers for a typical PowerDNS deployment:

- Switch to unix_socket authentication? [Y/n] n
- Change the root password? [Y/n] y
- New password: <choose-a-strong-password>
- Re-enter new password: <repeat-password>
- Remove anonymous users? [Y/n] y
- Disallow root login remotely? [Y/n] y
- Remove test database and access to it? [Y/n] y
- Reload privilege tables now? [Y/n] y

Adjust these answers as needed for your environment and security requirements.

### 4. Create PowerDNS Database and User

Now, let's create the database and user for PowerDNS. This will keep your DNS data organized and secure.

1. Access the MariaDB shell:
   ```bash
   mysql -u root -p
   ```
2. Create the database and user (replace `your_strong_password` with a secure password):
   ```sql
   CREATE DATABASE powerdns;
   CREATE USER 'powerdns'@'localhost' IDENTIFIED BY 'your_strong_password';
   GRANT ALL PRIVILEGES ON powerdns.* TO 'powerdns'@'localhost';
   FLUSH PRIVILEGES;
   ```
3. Do not exit the MariaDB client yet! The next step is done from here.

### 5. Create PowerDNS Tables (Schema)

With the MariaDB shell open and the database/user created, select the powerdns database:

```sql
USE powerdns;
```

Now, open the [powerdns-schema.sql](./mariadb/powerdns-schema.sql) file from this repository, copy all its contents, and paste them into the MariaDB client. This will create all the required tables.

To verify the tables were created, run:

```sql
SHOW TABLES;
```

You should see a list of tables like `comments`, `cryptokeys`, `domainmetadata`, `domains`, `records`, `supermasters` and `tsigkeys`.
