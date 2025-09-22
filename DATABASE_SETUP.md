# Database Setup Guide

## Issue: Database Connection Failed

The QR validation system requires a running MySQL database. Here's how to set it up:

## 1. Install MySQL (if not already installed)
- Download MySQL from: https://dev.mysql.com/downloads/installer/
- Install MySQL Server and MySQL Workbench
- During installation, set root password (remember this!)

## 2. Start MySQL Service
- Open Services (services.msc)
- Find "MySQL" or "MySQL80" service
- Right-click → Start

Or use command line:
```cmd
net start mysql
```

## 3. Create Database and Tables
Open MySQL Workbench or command line and run:

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS barangay_db;
USE barangay_db;

-- Create users table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    password VARCHAR(255) NOT NULL
);

-- Create certificate_of_indigency table
CREATE TABLE certificate_of_indigency (
    clearance_id INT AUTO_INCREMENT PRIMARY KEY,
    LastName VARCHAR(255),
    FirstName VARCHAR(255),
    MiddleName VARCHAR(255),
    Address TEXT,
    Age INT,
    Birthdate DATE,
    ContactNumber VARCHAR(20),
    Gender VARCHAR(10),
    Purpose TEXT,
    IssuedOn DATE,
    hash_code VARCHAR(64),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create barangay_clearance table
CREATE TABLE barangay_clearance (
    clearance_id INT AUTO_INCREMENT PRIMARY KEY,
    LastName VARCHAR(255),
    FirstName VARCHAR(255),
    MiddleName VARCHAR(255),
    Address TEXT,
    Age INT,
    Birthdate DATE,
    ContactNumber VARCHAR(20),
    Gender VARCHAR(10),
    Purpose TEXT,
    IssuedOn DATE,
    hash_code VARCHAR(64),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create business_permit table
CREATE TABLE business_permit (
    id INT AUTO_INCREMENT PRIMARY KEY,
    last_name VARCHAR(255),
    first_name VARCHAR(255),
    middle_name VARCHAR(255),
    address TEXT,
    business_name VARCHAR(255),
    business_nature VARCHAR(255),
    business_address TEXT,
    issued_on DATE,
    hash_code VARCHAR(64),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert test user
INSERT INTO users (name, password) VALUES ('admin', 'password');

-- Insert your existing record (adjust values as needed)
INSERT INTO barangay_clearance (
    LastName, FirstName, MiddleName, Address, Age, 
    Purpose, IssuedOn, hash_code
) VALUES (
    'Doe', 'John', 'M', 'McArthur Highway corner Aguas Street, Balibago', 20,
    'School Requirement', '2025-09-22', 'db8237d31e7ce91969940bd1e3967001'
);
```

## 4. Update Server Configuration
In `server.js`, update the database connection settings:

```javascript
const db = mysql.createConnection({
  host: "localhost",
  user: "root",                    // Your MySQL username
  password: "YOUR_MYSQL_PASSWORD", // Your actual MySQL password
  database: "barangay_db",
});
```

## 5. Test Connection
1. Start the server: `node server.js`
2. Should see: "Connected to MySQL Database"
3. Test QR validation in the web app

## Common Issues:
- **Error 1045**: Wrong password → Update password in server.js
- **Error 1049**: Database doesn't exist → Create barangay_db database
- **ECONNREFUSED**: MySQL not running → Start MySQL service
- **Port 3306**: MySQL using different port → Check MySQL configuration

## Quick Test Commands:
```cmd
# Check if MySQL is running
netstat -an | findstr :3306

# Start MySQL service
net start mysql

# Test server connection
curl -X POST http://localhost:5000/api/validate-qr -H "Content-Type: application/json" -d "{\"hash\":\"db8237d31e7ce91969940bd1e3967001\"}"
```