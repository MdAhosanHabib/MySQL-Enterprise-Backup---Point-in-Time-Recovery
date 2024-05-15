# MySQL Enterprise Backup & Point in Time Recovery

![MySQL Enterprise Backup Point in Time Recovery](https://github.com/MdAhosanHabib/MySQL-Enterprise-Backup---Point-in-Time-Recovery/assets/43145662/04f8a538-0657-4811-8b92-be9a4b3eba29)

This guide provides step-by-step instructions for creating backups using MySQL Enterprise Backup (MEB) and performing point-in-time recovery (PITR) in MySQL Enterprise Edition.

## Create Backup User for MySQL Enterprise Backup

```bash
mysql -u root -p

CREATE USER 'mysqlbackup'@'localhost' IDENTIFIED BY 'baCKup#y2233';

GRANT SELECT, BACKUP_ADMIN, RELOAD, PROCESS, SUPER, REPLICATION CLIENT ON *.*
TO 'mysqlbackup'@'localhost';

GRANT CREATE, INSERT, DROP, UPDATE ON mysql.backup_progress TO
'mysqlbackup'@'localhost';

GRANT CREATE, INSERT, DROP, UPDATE, SELECT, ALTER ON mysql.backup_history
TO 'mysqlbackup'@'localhost';
FLUSH PRIVILEGES;

# create directory for backup
mkdir -p /db_backup/MEB/
```

## Take The Backup & Validate
```bash
mysqlbackup --user=mysqlbackup --password --host=localhost --backup-image=/db_backup/MEB/my2.mbi \
--backup-dir=/db_backup/MEB backup-to-image

# validate backup
mysqlbackup --backup-image=/db_backup/MEB/my.mbi validate

# backup status kept here
mysql> use mysql;
mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| backup_history            |
| backup_progress           |
+---------------------------+
```

## Database Creation and Data Insertion
```bash
SHOW DATABASES;

CREATE DATABASE Test;

USE Test;

-- Create the table Test1
CREATE TABLE Test1 (
    ID INT AUTO_INCREMENT PRIMARY KEY,
    Name VARCHAR(255),
    Address VARCHAR(255),
    Work VARCHAR(255)
);

-- Insert 10 rows of data into the Test1 table
INSERT INTO Test1 (Name, Address, Work) VALUES
('John Doe', '123 Main St', 'Engineer'),
('Jane Smith', '456 Oak Ave', 'Teacher'),
('Michael Johnson', '789 Elm St', 'Doctor'),
('Emily Davis', '101 Pine St', 'Artist'),
('Chris Wilson', '202 Maple Ave', 'Accountant'),
('Sarah Brown', '303 Cedar St', 'Lawyer'),
('David Lee', '404 Birch Ave', 'Developer'),
('Jennifer Taylor', '505 Walnut St', 'Designer'),
('Daniel Martinez', '606 Pine Ave', 'Chef'),
('Michelle Rodriguez', '707 Oak St', 'Writer');

commit;
```

## Restore Database
```bash
systemctl stop mysqld.service

cp /mysql/mysql/binlog /mysql/mysql/binlog-bkp

cd /mysql/mysql/data
rm -rf *

cd /mysql/mysql/binlog
rm -rf *

mysqlbackup --datadir=/mysql/mysql/data --backup-image=/db_backup/MEB/my2.mbi --backup-dir=/db_backup/TMP copy-back-and-apply-log

systemctl start mysqld.service
```

## Recover to a specific time

Full Backup: 11:03 AM
Table Create: 11:05 AM
Table Drop: 11:09 AM

Here, we found the MySQL Binlog position from the restore Log.

![Start Recover from This time](https://github.com/MdAhosanHabib/MySQL-Enterprise-Backup---Point-in-Time-Recovery/assets/43145662/b0e8eb5e-1542-418b-b939-9b1612f05111)

Here the position is 553.

Now, we need to find the Binlog position number from the expected Binlog file.
```bash
# find Binlog Position by DateTime
mysqlbinlog --start-datetime="2024-05-15 11:01:00" --stop-datetime="2024-05-15 11:08:00" --verbose /mysql/mysql/binlog-bkp/mysql-bin.000005 > /db_backup/TMP/recover2.sql
```

The converted file:
![To Recover at This time](https://github.com/MdAhosanHabib/MySQL-Enterprise-Backup---Point-in-Time-Recovery/assets/43145662/97107dff-d6db-40c3-a8af-2b38a0ebc29b)

After finding the Log position number, we can recover
```bash
# Now Recover by Binlog Pos
mysqlbinlog --start-position="553" --stop-position="1757" /mysql/mysql/binlog-bkp/mysql-bin.000005 | mysql -u root -p
```

That's all.

Regards,
Ahosan

Medium: 
