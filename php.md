## PHP & PHP-FPM (FastCGI Process Manager)
#### Introduction & Architecture
PHP is a server-side scripting language primarily used for web development. In a modern DevOps infrastructure, PHP is rarely run as an Apache module (mod_php). Instead, it uses PHP-FPM (FastCGI Process Manager), an alternative PHP FastCGI implementation with features highly useful for heavily loaded sites.
- Nginx acts as the reverse proxy/web server handling static files (.html, .css, images) and SSL termination.
- PHP-FPM acts as the application server processing dynamic backend script files (.php).
- They communicate via a Unix Socket or a TCP Loopback Socket.

#### Installation & Process Management
Ubuntu/Debian
```
sudo apt update
sudo apt install php-fpm php-cli php-mysql php-curl php-xml -y
```
RHEL/Rocky Linux
```
sudo dnf install epel-release -y
sudo dnf module enable php:8.2 -y
sudo dnf install php-fpm php-cli php-mysqlnd -y
```
#### Managing the Service
```
# Check status
sudo systemctl status php8.2-fpm

# Restart after changes
sudo systemctl restart php8.2-fpm
```
#### Core Configuration File (php.ini)
The global behavior of PHP is governed by the `php.ini` file. In a PHP-FPM setup, this is typically located at 
- Ubuntu: `/etc/php/8.2/fpm/php.ini.`
- CentOS/RHEL: `/etc/php-fpm.conf`
```ini,TOML
# Resource Limits
memory_limit = 256M      ; Maximum amount of memory a script may consume
upload_max_filesize = 50M ; Maximum allowed size for uploaded files
post_max_size = 50M       ; Must be equal to or larger than upload_max_filesize
max_execution_time = 60   ; Maximum execution time of each script, in seconds

# Security Settings
expose_php = Off          ; Prevents PHP from sending its version in headers
display_errors = Off     ; Disables showing errors to users (Crucial for production)
log_errors = On          ; Ensures errors are recorded in logs instead
```
#### PHP-FPM Pool Configuration (www.conf)
PHP-FPM uses pools to isolate environments. The default pool configuration file is usually located at 
`/etc/php/8.2/fpm/pool.d/www.conf.`

 ##### Connection Configuration (The "Where" part)
 This directive tells PHP-FPM where to listen for incoming FastCGI requests from Nginx:
 ```Ini,TOML
; Option A: Unix Socket (Faster, recommended for single-server setups)
listen = /run/php/php8.2-fpm.sock

; Option B: TCP Socket (Required if Nginx and PHP are on different servers)
; listen = 127.0.0.1:9000
```
##### Performance Tuning (Process Managers)
```ini.TOML
pm = dynamic              ; Controls how process managers control child processes
pm.max_children = 50      ; Max active worker processes allowed
pm.start_servers = 5      ; Number of child processes created on startup
pm.min_spare_servers = 5  ; Minimum number of idle child processes
pm.max_spare_servers = 10 ; Maximum number of idle child processes
```
#### Integrating PHP with Nginx
o make Nginx pass `.php` requests to your PHP application server, update your Nginx configuration file (`nginx.conf` or site block):
```Nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Pass PHP scripts to PHP-FPM
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        
        # This MUST match the 'listen' directive in your www.conf
        fastcgi_pass unix:/run/php/php8.2-fpm.sock; 
        
        # Security parameter to prevent script execution issues
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```
#### Troubleshooting & Logs
When a website returns a 502 Bad Gateway or 504 Gateway Timeout, it usually indicates a breakdown in communication between Nginx and PHP-FPM.


*Essential Log Locations*
- PHP-FPM System Error Log: `/var/log/php8.2-fpm.log `(Catches master process startup/shutdown and crashed workers).
- PHP Application Error Log: Look at the path specified in your `php.ini` under error_log, or check `/var/log/nginx/error.log` if Nginx captures it.

  ---

  ### NGINX & PHP :
   Whats the relation ?
   #####  🏢 The Restaurant Analogy
**Imagine a busy restaurant:**
- **Nginx is the Waiter (The Web Server)**: The waiter stands at the front, greets customers, takes orders, and quickly hands out things that don't need cooking (like a glass of water or a menu).
- **PHP-FPM is the Chef (The Application Server):** The chef stays in the kitchen. They don't talk to customers. They only take raw ingredients (PHP code) and cook them into a meal (HTML page) when the waiter brings them an order.

1. Why Nginx comes into play
Nginx is a master at handling static files (images, CSS files, plain .html pages). It is incredibly fast, lightweight  
However, Nginx has one major limitation: It doesn't speak `PHP`. If a user requests `profile.php`, Nginx doesn't know what to do with that code. It cannot execute backend logic, read database data, or log users in.  
2. What is PHP-FPM?
*PHP-FPM stands for FastCGI Process Manager.*
It is a dedicated, background service whose entire job is to sit and wait for Nginx to pass it .php files. When it receives a file, it executes the PHP code, connects to your database if needed, generates the final HTML output, and hands it back to Nginx to send to the user.

3. How they work together (The Workflow)
```
  [User's Browser] 
       │
       ▼ (Requests: /index.php)
┌──────────────┐
│    Nginx     │ ◄─── (Hey, this is a PHP file! I can't read this.)
└──────┬───────┘
       │
       ▼ (Passes request via Unix Socket / TCP Network)
┌──────────────┐
│   PHP-FPM    │ ◄─── (Executes the PHP code, talks to Database)
└──────┬───────┘
       │
       ▼ (Returns clean HTML back to Nginx)
┌──────────────┐
│    Nginx     │ ◄─── (Sends the clean HTML back to the User)
└──────┬───────┘
       │
       ▼
[User's Browser renders the page]

```
##### Summary: Why this setup is so popular
Instead of having one program try to do everything, separating them gives you the best of both worlds:

- Speed: Nginx handles the easy static stuff instantly.

- Stability: If a heavy PHP script crashes, PHP-FPM restarts its background processes automatically, but Nginx stays online and keeps serving the rest of your website.
