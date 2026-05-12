================================================================================
        MariaDB - COMPLETE STUDY NOTES (MIT450)
================================================================================

YOUR INFO (replace with your own values):
  Username 1  : developer1
  Username 2  : developer2
  Username 3  : developer3
  wwwuser     : wwwuser
  Password    : YourPassword123
  Student No  : w01234567

================================================================================
PART 0 - CHANGE HOSTNAME
================================================================================

-- Change hostname to your student number:
   hostname w{yourstudentno}
   example: hostname w01234567

-- Verify hostname changed:
   hostname

================================================================================
PART 1 - INSTALLATION
================================================================================

-- Install MariaDB client and server:
   sudo apt install mariadb-client mariadb-server -y

-- Enable MariaDB to start at boot and start it now:
   sudo systemctl enable --now mariadb

-- Verify it is running:
   sudo systemctl status mariadb

================================================================================
PART 2 - CONFIGURING MariaDB SERVER
================================================================================

--------------------------------------------------------------------------------
STEP 1: SECURE THE INSTALLATION
--------------------------------------------------------------------------------

-- Run the security script:
   sudo mysql_secure_installation

   ANSWER THE PROMPTS AS FOLLOWS:
   - Set root password?          -> YES (set a strong password)
   - Remove anonymous users?     -> YES
   - Disallow root login remotely? -> YES (no remote root login)
   - Remove test database?       -> YES
   - Reload privilege tables?    -> YES

--------------------------------------------------------------------------------
STEP 2: LOGIN TO MariaDB SHELL
--------------------------------------------------------------------------------

-- Login as root:
   mysql -u root -p
   (enter the password you set in step 1)

   YOU ARE NOW INSIDE THE MariaDB SHELL
   All commands inside here END WITH A SEMICOLON ;

--------------------------------------------------------------------------------
STEP 3: CREATE DEVELOPER USERS
--------------------------------------------------------------------------------

-- Create a developer user:
   CREATE USER 'developer1'@'localhost' IDENTIFIED BY 'Password123';

-- Create wwwuser:
   CREATE USER 'wwwuser'@'localhost' IDENTIFIED BY 'Password123';

-- Create 2 more developer users:
   CREATE USER 'developer2'@'localhost' IDENTIFIED BY 'Password123';
   CREATE USER 'developer3'@'localhost' IDENTIFIED BY 'Password123';

--------------------------------------------------------------------------------
STEP 4: CREATE DATABASES
--------------------------------------------------------------------------------

-- Create the production database:
   CREATE DATABASE wwwprod;

-- Create test database for developer1:
   CREATE DATABASE testwwwdeveloper1;

-- Create test databases for developer2 and developer3:
   CREATE DATABASE testwwwdeveloper2;
   CREATE DATABASE testwwwdeveloper3;

-- Verify databases were created:
   SHOW DATABASES;

--------------------------------------------------------------------------------
STEP 5: GRANT PRIVILEGES
--------------------------------------------------------------------------------

-- Give developer1 ALL privileges on wwwprod (production):
   GRANT ALL PRIVILEGES ON wwwprod.* TO 'developer1'@'localhost';

-- Give wwwuser ALL privileges on wwwprod:
   GRANT ALL PRIVILEGES ON wwwprod.* TO 'wwwuser'@'localhost';

-- Give developer1 ALL privileges on their OWN test database:
   GRANT ALL PRIVILEGES ON testwwwdeveloper1.* TO 'developer1'@'localhost';

-- Give developer2 privileges on wwwprod AND their test database:
   GRANT ALL PRIVILEGES ON wwwprod.* TO 'developer2'@'localhost';
   GRANT ALL PRIVILEGES ON testwwwdeveloper2.* TO 'developer2'@'localhost';

-- Give developer3 privileges on wwwprod AND their test database:
   GRANT ALL PRIVILEGES ON wwwprod.* TO 'developer3'@'localhost';
   GRANT ALL PRIVILEGES ON testwwwdeveloper3.* TO 'developer3'@'localhost';

-- IMPORTANT: Apply privilege changes:
   FLUSH PRIVILEGES;

--------------------------------------------------------------------------------
STEP 6: VERIFY CONFIGURATION
--------------------------------------------------------------------------------

-- View all users in MariaDB:
   SELECT user,host,password FROM mysql.user;

-- Check grants for a specific user:
   SHOW GRANTS FOR 'developer1'@'localhost';
   SHOW GRANTS FOR 'wwwuser'@'localhost';

--------------------------------------------------------------------------------
STEP 7: CREATE A CUSTOM DATABASE WITH TABLES
--------------------------------------------------------------------------------

-- Create a database of your choice (example: a library database):
   CREATE DATABASE librarydb;

-- Switch to that database:
   USE librarydb;

-- Create first table (example: books):
   CREATE TABLE books (
       id          INT AUTO_INCREMENT PRIMARY KEY,
       title       VARCHAR(100),
       author      VARCHAR(100),
       genre       VARCHAR(50),
       year        INT,
       available   BOOLEAN
   );

-- Create second table (example: members):
   CREATE TABLE members (
       id          INT AUTO_INCREMENT PRIMARY KEY,
       first_name  VARCHAR(50),
       last_name   VARCHAR(50),
       email       VARCHAR(100),
       phone       VARCHAR(20),
       join_date   DATE
   );

-- Verify tables were created:
   SHOW TABLES;

-- View structure of a table:
   DESCRIBE books;
   DESCRIBE members;

--------------------------------------------------------------------------------
STEP 8: INSERT RECORDS INTO TABLES
--------------------------------------------------------------------------------

-- Insert 2 records into books table:
   INSERT INTO books (title, author, genre, year, available)
   VALUES ('The Great Gatsby', 'F. Scott Fitzgerald', 'Fiction', 1925, TRUE);

   INSERT INTO books (title, author, genre, year, available)
   VALUES ('1984', 'George Orwell', 'Dystopian', 1949, TRUE);

-- Insert 2 records into members table:
   INSERT INTO members (first_name, last_name, email, phone, join_date)
   VALUES ('Garry', 'Kasparov', 'garry@email.com', '555-1234', '2026-01-15');

   INSERT INTO members (first_name, last_name, email, phone, join_date)
   VALUES ('John', 'Smith', 'john@email.com', '555-5678', '2026-02-20');

-- Verify records were inserted:
   SELECT * FROM books;
   SELECT * FROM members;

--------------------------------------------------------------------------------
STEP 9: CREATE USER FOR YOUR CUSTOM DATABASE
--------------------------------------------------------------------------------

-- Create a user with your name and grant permissions to your database:
   CREATE USER 'garry'@'localhost' IDENTIFIED BY 'Password123';
   GRANT ALL PRIVILEGES ON librarydb.* TO 'garry'@'localhost';
   FLUSH PRIVILEGES;

-- Verify:
   SHOW GRANTS FOR 'garry'@'localhost';

--------------------------------------------------------------------------------
STEP 10: EXIT MariaDB SHELL
--------------------------------------------------------------------------------

-- Exit the MariaDB shell:
   EXIT;
   or
   QUIT;

--------------------------------------------------------------------------------
STEP 11: EXPORT DATABASE (run from BASH terminal, NOT MariaDB shell)
--------------------------------------------------------------------------------

-- Export/dump a database to a .sql file:
   mysqldump -u garry -pPassword123 librarydb > librarydb.sql

   NOTE: There is NO SPACE between -p and the password!
   Format: mysqldump -u {username} -p{password} {databasename} > {filename}.sql

-- Verify the dump file was created:
   ls -lh librarydb.sql
   cat librarydb.sql

================================================================================
QUICK REFERENCE - MariaDB SHELL COMMANDS
================================================================================

DATABASE COMMANDS:
   SHOW DATABASES;                          : list all databases
   CREATE DATABASE {name};                  : create a database
   USE {name};                              : switch to a database
   DROP DATABASE {name};                    : delete a database
   SELECT DATABASE();                       : show current database

TABLE COMMANDS:
   SHOW TABLES;                             : list tables in current database
   DESCRIBE {tablename};                    : show table structure
   DROP TABLE {tablename};                  : delete a table

USER COMMANDS:
   SELECT user,host,password FROM mysql.user;    : list all users
   CREATE USER '{user}'@'localhost'
     IDENTIFIED BY '{password}';           : create a user
   DROP USER '{user}'@'localhost';          : delete a user
   SHOW GRANTS FOR '{user}'@'localhost';    : show user permissions

PRIVILEGE COMMANDS:
   GRANT ALL PRIVILEGES ON {db}.* TO '{user}'@'localhost';   : grant all
   GRANT SELECT ON {db}.* TO '{user}'@'localhost';           : read only
   REVOKE ALL PRIVILEGES ON {db}.* FROM '{user}'@'localhost';: remove privs
   FLUSH PRIVILEGES;                        : apply privilege changes

DATA COMMANDS:
   SELECT * FROM {table};                   : view all records
   SELECT {col1},{col2} FROM {table};       : view specific columns
   INSERT INTO {table} ({col1},{col2})
     VALUES ({val1},{val2});                : insert a record
   UPDATE {table} SET {col}={val}
     WHERE {condition};                     : update a record
   DELETE FROM {table} WHERE {condition};   : delete a record

================================================================================
PRIVILEGE LEVELS EXPLAINED
================================================================================

   ALL PRIVILEGES  : full access (SELECT, INSERT, UPDATE, DELETE, DROP etc)
   SELECT          : read only
   INSERT          : add new records
   UPDATE          : modify existing records
   DELETE          : remove records
   CREATE          : create tables/databases
   DROP            : delete tables/databases

   Syntax:
   GRANT {privilege} ON {database}.{table} TO '{user}'@'{host}';
   Use * as wildcard: wwwprod.* means ALL tables in wwwprod

================================================================================
IMPORTANT FILE LOCATIONS
================================================================================

   /etc/mysql/                   : MariaDB config directory
   /etc/mysql/mariadb.conf.d/    : MariaDB config files
   /var/lib/mysql/               : database data files
   /var/log/mysql/               : MariaDB log files

================================================================================
SYSTEMCTL COMMANDS FOR MariaDB
================================================================================

   sudo systemctl start mariadb      : start MariaDB
   sudo systemctl stop mariadb       : stop MariaDB
   sudo systemctl restart mariadb    : restart MariaDB
   sudo systemctl enable mariadb     : enable at boot
   sudo systemctl status mariadb     : check status

================================================================================
COMMON MISTAKES TO AVOID
================================================================================

   1. ALWAYS end MariaDB commands with a semicolon ;
   2. NO SPACE between -p and password in mysqldump
      CORRECT:   mysqldump -u user -pPassword db > db.sql
      INCORRECT: mysqldump -u user -p Password db > db.sql
   3. FLUSH PRIVILEGES after granting permissions
   4. Use USE {database}; before creating tables
   5. Hostnames in quotes: 'username'@'localhost' not username@localhost
   6. Database names are case sensitive in Linux

================================================================================
SUMMARY OF LAB STRUCTURE
================================================================================

   Users created:
   - developer1  -> wwwprod (all) + testwwwdeveloper1 (all)
   - developer2  -> wwwprod (all) + testwwwdeveloper2 (all)
   - developer3  -> wwwprod (all) + testwwwdeveloper3 (all)
   - wwwuser     -> wwwprod (all)
   - garry       -> librarydb (all)

   Databases created:
   - wwwprod              : production database
   - testwwwdeveloper1    : test db for developer1
   - testwwwdeveloper2    : test db for developer2
   - testwwwdeveloper3    : test db for developer3
   - librarydb            : custom database with tables

================================================================================
END OF NOTES
================================================================================
