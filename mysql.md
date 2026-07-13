## MySQL Cheat Sheet & Architecture Guide

```
========================================================================
=                      MySQL Architecture Guide                      =
========================================================================

+------------------------------------------------------------------+
| Client Applications (Web, BI Tools, Drivers)                     |
+-------+----------------------------------------------------------+
        |  Network Protocol (e.g., TCP/IP on 3306)
        v
+------------------------------------------------------------------+
|              MySQL Server Instance (Logical Server)              |
|                                                                  |
|  +---------------------+                                         |
|  | Connection Manager  |                                         |
|  | (Handles threads)   |                                         |
|  +---------+-----------+                                         |
|            |                                                     |
|  +---------v-----------+  [my.cnf]                               |
|  | Parser &            |  `max_connections`                      |
|  | Preprocessor        |  `bind-address`                         |
|  +---------+-----------+                                         |
|            |                                                     |
|  +---------v-----------+  [EXPLAIN]                              |
|  | Query Optimizer &  |  Checks index availability               |
|  | Execution Plan      |  Selects lowest-cost plan                |
|  +---------+-----------+                                         |
|            |                                                     |
|  +---------v-----------+                                         |
|  | Query Executor      |-----------------+                       |
|  | (Calls Storage API) |                 |                       |
|  +---------+-----------+                 v                       |
|            |                      +------+------+                |
|            |                      | Binary Log  |                |
|            |                      | (binlog)    |                |
|            |                      +------+------+                |
|            |                             |                       |
|            |                             |                       |
+------------+-----------------------------+-----------------------+
             | Storage Engine API (Calls & Callbacks)
             v
+-----------------------------------------+------------------------+
|          Storage Engines (InnoDB)       |  +------------------+  |
|                                         |  | (MyISAM / Others)|  |
|  +-------------------+                  |  +------------------+  |
|  | InnoDB Buffer Pool|--+               |                        |
|  | (RAM Cache)       |  |               |                        |
|  | Data & Indexes    |<-+ Read/Write    |                        |
|  +---------+---------+   Data Pages     |                        |
|            |                            |                        |
|  +---------v---------+   Redo Logs      |                        |
|  | Redo Log Buffer  |---+ flushed       |                        |
|  | (Memory)          |                  |                        |
|  +-------------------+   sequentially   |                        |
+------------+----------------------------+------------------------+
             | File System (Flushing & Sync)
             v
+------------------------------------------------------------------+
|                      File System & Disks                         |
|                                                                  |
|  +-------------------+  +---------------+  +------------------+  |
|  | Data Files (.ibd) |  | Undo Logs     |  | Redo Logs        |  |
|  | Table Data/Index  |  | (ROLLBACK)    |  | (ib_logfile0, 1) |  |
|  +-------------------+  +---------------+  +------------------+  |
|                                            | (ACID Recovery)  |  |
+--------------------------------------------+------------------+--+
```
####  1. Installation Process (Ubuntu/Debian)
```bash
# 1. Update the local package index
sudo apt update

# 2. Install the MySQL Server package
sudo apt install -y mysql-server

# 3. Run the security script to configure authentication, validate passwords, and remove insecure defaults
sudo mysql_secure_installation

# 4. Verify the service status
sudo systemctl status mysql
```

#### 2. The Core Configuration Files
MySQL's behavior is primarily controlled by the main configuration file located at `/etc/mysql/mysql.conf.d/mysqld.cnf` (or `/etc/mysql/my.cnf`)
- `bind-address`: By default, this is set to `127.0.0.1 (localhost)`. To allow remote connections, change this to `0.0.0.0` or a specific network IP interface.
- `max_connections`: Controls the maximum number of concurrent client connections (Default is usually 151).
- `innodb_buffer_pool_size:` The most critical performance setting for InnoDB. It dictates how much RAM MySQL uses to cache data and indexes (typically set to 50%–80% of total system memory on a dedicated DB server).
- `innodb_log_file_size`: Sets the size of the redo log files, crucial for write performance and recovery

#### 3. The Call Flow (Under the Hood)
When a client application executes a query, MySQL processes it through the following architectural layers:
1. `Connection Handling `(The Connection Manager): MySQL uses a thread-per-connection model. When a client connects, the manager assigns or spawns a dedicated thread to handle that session's requests and authenticates credentials.
2. `Parser & Preprocessor:` The SQL text string is converted into a parse tree. The preprocessor verifies that tables and columns exist, and validates user privileges.
3. `The Optimizer:` MySQL evaluates multiple execution strategies (e.g., determining index choice, join order, and subquery rewrites) and selects the lowest-cost execution plan.
4. `Storage Engine Interface` (The Executor): Unlike Postgres, MySQL separates its query processor from data storage. The Executor calls the unified Storage Engine API (typically InnoDB) to read or write rows.
5.` Buffer Pool & Disk I/O`: The engine looks for pages in the InnoDB Buffer Pool (RAM). If missing, it fetches them from the data files on disk into RAM.
6.` Redo Log (Write-Ahead Logging) & Binlog`: For write operations, modifications are written sequentially to the Redo Log (for crash recovery / ACID compliance) and the Binary Log (Binlog) (used for replication and point-in-time recovery) before the data pages are dirtied and eventually flushed to disk.


#### 4. Troubleshooting (Tshoot) Guide
**Issue A: "Connection Refused" (App cannot reach the DB)**
- Check the service: Run `sudo systemctl status mysql` to ensure it is running.

- Check network binding: Run `ss -nltp | grep 3306`. If it lists `127.0.0.1:3306`, it refuses remote endpoints. Open `/etc/mysql/mysql.conf.d/mysqld.cnf`, update `bind-address = 0.0.0.0`, and restart via sudo `systemctl restart mysql`.

**Issue B: "Access denied for user..."**
- Root Cause: MySQL restricts users not just by password, but by the host they connect from (e.g., 'user'@'localhost' vs 'user'@'%').
- Fix: Log into the console and verify user scopes:
```SQL
SELECT user, host, plugin FROM mysql.user;
-- If needed, create or alter the user to allow remote connections:
CREATE USER 'myuser'@'%' IDENTIFIED BY 'mypassword';
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

**Issue C: High CPU or Slow Queries**
Check active threads: Run SHOW FULL PROCESSLIST; or query the information schema:
```SQL
SELECT id, user, host, db, command, time, state, info 
FROM information_schema.processlist 
WHERE command != 'Sleep' ORDER BY time DESC;
```
- Analyze Performance: Prepend EXPLAIN or EXPLAIN ANALYZE (MySQL 8.0+) to your query to see index utilization, join types, and actual execution times.
  
  
#### 5. Creating Databases & Users
Log into the interactive terminal client as root:
```
 sudo mysql -u root -p
```
**Create a User and Grant Privileges**
```SQL
-- Create a user that can connect from any host ('%')
CREATE USER 'devops_user'@'%' IDENTIFIED BY 'SecurePassword123!';

-- Grant specific privileges (or ALL PRIVILEGES) on a specific database
GRANT ALL PRIVILEGES ON corporate_db.* TO 'devops_user'@'%';

-- Reload the privileges to ensure everything applies immediately
FLUSH PRIVILEGES;
```

**Creating a Database**
```SQL
-- Create a new database with standard UTF-8 character encoding
CREATE DATABASE corporate_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- List all databases to verify
SHOW DATABASES;

-- Switch context to your new database
USE corporate_db;
```
---

###Excercise : rollout a deployment of mysql with pv , so the data remains accessible after even if the deploy dies
###### Note You need to specify one of the following as an environment variable for mysql to work,otherwise it will fail
- MYSQL_ROOT_PASSWORD
- MYSQL_ALLOW_EMPTY_PASSWORD
- MYSQL_RANDOM_ROOT_PASSWORD

Detiled of Excercise
- 1.) Create a PersistentVolume mysql-pv, its capacity should be 250Mi, set other parameters as per your preference.
- 2.) Create a PersistentVolumeClaim to request this PersistentVolume storage. Name it as mysql-pv-claim and request a 250Mi of storage. Set other parameters as per your preference.
- 3.) Create a deployment named mysql-deployment, use any mysql image as per your preference. Mount the PersistentVolume at mount path /var/lib/mysql.
- 4.) Create a NodePort type service named mysql and set nodePort to 30007.
- 5.) Create a secret named mysql-root-pass having a key pair value, where key is password and its value is YUIidhb667, create another secret named mysql-user-pass having some key pair values, where first key is username and its value is kodekloud_pop, second key is password and value is LQfKeWWxWD, create one more secret named mysql-db-url, key name is database and value is kodekloud_db7
- 6.) Define some environment variables within the container:
    * a.) name: MYSQL_ROOT_PASSWORD, should pick value from secretKeyRef name: mysql-root-pass and key: password
    * b.) name: MYSQL_DATABASE, should pick value from secretKeyRef name: mysql-db-url and key: database
    * c.) name: MYSQL_USER, should pick value from secretKeyRef name: mysql-user-pass key key: username
    * d.) name: MYSQL_PASSWORD, should pick value from secretKeyRef name: mysql-user-pass and key: password  this is the problem
 
1. Create Secrets
Create the necessary secrets to store sensitive database credentials.
```
# Root password secret
kubectl create secret generic mysql-root-pass --from-literal=password=YUIidhb667

# User credentials secret
kubectl create secret generic mysql-user-pass --from-literal=username=kodekloud_pop --from-literal=password=LQfKeWWxWD

# Database name secret
kubectl create secret generic mysql-db-url --from-literal=database=kodekloud_db7
```
2. Create PersistentVolume and PersistentVolumeClaim
Save the following as mysql-storage.yaml and apply it to provision 250Mi of storage.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/mnt/data/mysql"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
```
3. Create MySQL Deployment
   his configuration mounts the persistent volume at /var/lib/mysql and injects credentials from the secrets created in Step 1
```yaml
   apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-pass
              key: password
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-db-url
              key: database
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: password
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim   
```

4. Create NodePort Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
    nodePort: 30007
```
It allows you to connect to the MySQL database from outside the Kubernetes cluster
