# Guacamole Deployment with NGINX Proxy Manager and Google IDP SAML Integration

This guide provides step-by-step instructions for deploying Apache Guacamole with NGINX Proxy Manager and enabling SAML authentication.

---

## Prerequisites

- A Linux server (Ubuntu recommended)
- Docker and Docker Compose installed
- Access to a MySQL database

---

## Steps

### **Step 1: Install Docker and Docker Compose**

Follow the official Docker installation guide:
- [Install Docker](https://docs.docker.com/get-docker/)
- [Install Docker Compose](https://docs.docker.com/compose/install/)

Verify installation:
```sh
docker --version
docker-compose --version
```

---

### **Step 2: Create Docker Compose File and `.env` File**

Create a `docker-compose.yml` file with the necessary services for Guacamole, MySQL, and NGINX Proxy Manager. Additionally, configure the `.env` file with your environment-specific variables.

---

### **Step 3: Initialize the Guacamole Database**

#### 1. Download the Schema Files

```sh
wget "https://apache.org/dyn/closer.lua/guacamole/1.5.5/binary/guacamole-auth-jdbc-1.5.5.tar.gz?action=download" -O guacamole-auth-jdbc-1.5.5.tar.gz
```

#### 2. Extract the Package

```sh
tar -xzf guacamole-auth-jdbc-1.5.5.tar.gz
```

#### 3. Navigate to the Schema Directory

```sh
cd guacamole-auth-jdbc-1.5.5/mysql/schema
```

#### 4. Copy Initialization Scripts Into the MySQL Database Container

```sh
docker cp ./001-create-schema.sql guacamole_db:/tmp/
docker cp ./002-create-admin-user.sql guacamole_db:/tmp/
```

#### 5. Execute SQL Scripts Inside MySQL

1. Access the MySQL container:
   ```sh
   docker exec -it guacamole_db bash
   ```

2. Login to MySQL:
   ```sh
   mysql -u root -p
   ```

3. Execute the SQL scripts:
   ```sql
   USE guacamole_db;
   SOURCE /tmp/001-create-schema.sql;
   SOURCE /tmp/002-create-admin-user.sql;
   EXIT;
   ```

---

### **Step 4: Fix NGINX Proxy Manager Login Issue**

If you encounter issues logging into NGINX Proxy Manager, follow these steps:

#### 1. Access the MySQL Container

```sh
docker exec -it npm_db mysql -u root -p
```

#### 2. Update the Authentication Plugin

```sql
ALTER USER 'npm_user'@'%' IDENTIFIED WITH mysql_native_password BY 'npm_password';
FLUSH PRIVILEGES;

SELECT user, host, plugin FROM mysql.user;
```

---

### **Step 5: Enable SAML Authentication**

#### 1. Download and Place the SAML JAR File

```sh
wget https://apache.org/dyn/closer.lua/guacamole/1.5.5/binary/guacamole-auth-sso-1.5.5.tar.gz?action=download -O guacamole-auth-sso-1.5.5.tar.gz

tar -xzf guacamole-auth-sso-1.5.5.tar.gz

sudo mkdir -p /home/ubuntu/guacamole-setup/guacamole/extensions/

sudo cp ./guacamole-auth-sso-1.5.5/saml/guacamole-auth-sso-saml-1.5.5.jar /home/ubuntu/guacamole-setup/guacamole/extensions
```

---

### **Step 6: Configure NGINX and Guacamole**

1. Login to NGINX Proxy Manager and configure a reverse proxy for the Guacamole server.
2. Create an admin user for Google Identity Provider (IDP).

---

### **Step 7: Integrate with Google Admin Console**

1. Create an application in Google Admin Console.

   - **ACS URL**:
     ```
     https://guac.securityfocal.com/guacamole/api/ext/saml/callback
     ```

   - **Entity ID**:
     ```
     https://guac.securityfocal.com
     ```

2. Download the metadata file from Google Admin Console and upload it to the Guacamole server via the web interface.

---

## Additional Notes

- Default NGINX Proxy Manager credentials:
  - Email: `admin@example.com`
  - Password: `changeme`

- Default Guacamole credentials:
  - Username: `guacadmin`
  - Password: `guacadmin`

---

Feel free to customize this guide to suit your deployment requirements! 🚀
