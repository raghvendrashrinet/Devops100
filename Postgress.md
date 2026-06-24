## PostgreSQL

#### 1. Installation Process
```
# 1. Add the PostgreSQL signing key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/keyrings/postgresql.gpg

# 2. Add the repository to your system sources
echo "deb [signed-by=/etc/apt/keyrings/postgresql.gpg] http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list

# 3. Update packages and install PostgreSQL
sudo apt update
sudo apt install -y postgresql postgresql-contrib

# 4. Verify it's running
sudo systemctl status postgresql
```

#### 2. The Core Configuration Files
PostgreSQL is primarily controlled by three critical files located in the data directory
`/etc/postgresql/<version>/main/`

- 1. `postgresql.conf` (The engine Settings): Controls core data behavior
     * `listen_addresses`:y default, it is set to 'localhost'. If you need external apps to connect, change this to `'*'`.
     * `max_connections:` Maximum number of concurrent client connections.
     * `shared_buffers`:Dictates how much dedicated memory Postgres can use for caching data (usually set to 25% of system RAM).
- 2.  `pg_hba.conf`: (Host-Based Authentication): The firewall of Postgres.  
       It decides who can log in, from where, and how.

      Format: TYPE  DATABASE  USER  ADDRESS  METHOD
     Example: host  all       all  10.0.0.0/24  scram-sha-256
    (Allows any user from the 10.0.0.x network to connect using a password).
- 3. `pg_ident.conf`:(User Mapping): Maps operating system usernames to PostgreSQL database usernames (used less frequently)
---

### 3. The Call Flow (Under the Hood)
When a client application sends a query, what actually happens inside Postgres?
1. `Connection (The Postmaster)`: A primary supervisor process (called postmaster) listens on port 5432. When your app connects, the postmaster forks a dedicated Backend Worker Process just for your connection.
2. `Parsing & Rewriting`: The worker process takes your SQL string, parses it to check for syntax errors, and applies views or rules. 
3. `The Optimizer/Planner:`he smartest part of Postgres. It looks at table statistics and calculates the fastest execution plan (e.g., Should I use an index scan or scan the whole table?).
4. `Execution & Shared Buffers:` The executor runs the plan. It checks the Shared Buffers (RAM cache). If the data is there, it reads it instantly. If not, it pulls it from the disk into RAM.
5. `WAL (Write-Ahead Logging)`:If you are writing data (INSERT/UPDATE), Postgres writes the transaction to the WAL file on disk before altering the actual table data. This ensures that if the server crashes or loses power unexpectedly, no committed data is ever lost.


---
## 4. Troubleshooting (Tshoot) Guide
#### Issue A: "Connection Refused" (App cannot reach the DB)   
* Check the service status: Run sudo systemctl status postgresql. Is it active?
* Check the port binding: Run ss -nltp | grep 5432. If it says `127.0.0.1:5432`, it is blocking remote connections. Go to `postgresql.conf`, change `listen_addresses = '*'`, and restart the service.

#### Issue B: "pg_hba.conf rejects connection"  
* Look at the exact error message: It usually tells you which line/IP failed.
* Fix: Open /etc/postgresql/.../main/pg_hba.conf, add a line granting access to your client IP or subnet, and reload configuration without breaking active connections using:
`sudo systemctl reload postgresql`
#### Issue C: High CPU or Slow Queries  
Check running queries: Log into the database and view active processes:
```sql
SELECT pid, age(clock_timestamp(), query_start), usename, query 
FROM pg_stat_activity 
WHERE state != 'idle';
```
- Analyze the performance: Prepend EXPLAIN ANALYZE to your slow query. It tells you exactly where the execution plan spent its time and if it is missing a critical table index.


---

### Creating Database  
Once postgres running , you first need to log into the PostgreSQL interactive terminal `(psql)` as the default `admin` user:
```
sudo -i -u postgres psql
```

#### Create the user with a password:
```
CREATE USER myuser WITH PASSWORD 'mypassword';

or by command line

createuser --interactive --username=postgres

```

#### Grant Permissions (Optional):
```
ALTER USER myuser WITH SUPERUSER;
```


#### 1. Creating a Database
```sql
-- Create a new database named 'company_db'
CREATE DATABASE kodekloud_db6;

-- List all databases to verify it was created
\l

-- Connect (switch) to your new database
\c company_db;
```
