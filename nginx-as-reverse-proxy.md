## 🔁 1. Reverse Proxying
- default, Nginx is a web server—it grabs static files (like HTML, CSS, images) from a folder and hands them to the browser.
- Reverse Proxy means Nginx acts as a middleman or a "traffic cop." Instead of serving a file itself, it takes an incoming request from the internet and forwards it to a completely different application running on your server (like a Node.js, Python, or Go backend), then hands the response back to the user.
#### This require proxy block
```
 server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000; # Forwards traffic to Node.js
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

###  🔒 2. SSL Certificates (Certbot / Let's Encrypt)
SSL is what changes your site from insecure http:// to secure https://

**Let's Encrypt** is a free, automated certificate authority, and **Certbot** is the tool/bot that automatically talks to Nginx to set it up.

##### Why does it bloat your file?
To make a site secure, you have to add a second server block that listens on port 443 (HTTPS), point Nginx to where your security keys are stored, and add code to automatically redirect anyone typing http:// over to https://.

What the config looks like after Certbot touches it:
```
server {
    listen 443 ssl; # Managed by Certbot
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # Managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # Managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # Managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # Managed by Certbot

    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }
}

# Automatically redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri; 
}
```
---

#### how they get onto your server (Installation), how they stay valid (Management), and how Nginx actually reads them (Decryption)
### 💾 1. How Certificates are Installed
When you use a tool like Certbot (which acts as a client for the free certificate authority Let's Encrypt), the installation process relies on a security check called a Challenge.

1. The Request: Certbot contacts Let's Encrypt and says, "Hey, I own example.com. Please give me an SSL certificate."
2. The ACME Challenge: Let's Encrypt replies, "Prove it. Place this secret, random file inside your Nginx public directory at .well-known/acme-challenge/."
3. The Verification: Let's Encrypt tries to download that file via [http://example.com/.well-known/acme-challenge/](http://example.com/.well-known/acme-challenge/)<secret>.
   If it successfully reaches your Nginx container and reads the file, it proves you control the domain.
4. The Handover: Once proven, Let's Encrypt generates the certificates and Certbot downloads them onto your server (usually inside /etc/letsencrypt/live/[example.com/](https://example.com/)).

### ⏳ 2. How Certificates are Managed (The 90-Day Loop)
The Cron Job / Timer: When Certbot is installed, it sets up a background system timer (a Linux cron job or systemd timer) that runs quietly in the background twice a day.


### 🔑 3. How Certificates are Decrypted
This is where the magic of Asymmetric Cryptography (Public/Private Keys) comes in. Your Nginx configuration points to two crucial files:
```
ssl_certificate     /etc/letsencrypt/.../fullchain.pem; # The Public Key
ssl_certificate_key /etc/letsencrypt/.../privkey.pem;   # The Private Key
```
##### Step A: The Handshake (Asymmetric Encryption)
1. A user visits [https://example.com](https://example.com).

2. Nginx sends the user's browser the fullchain.pem (Public Key). As the name suggests, this is completely public; anyone can see it.

3. The browser uses this Public Key to encrypt a piece of random data (a temporary "session key") and sends it back to Nginx.

4. The Rule of Public Keys: Data encrypted with a Public Key can only be decrypted by its corresponding Private Key.
##### Step B: The Decryption (The Secret Sauce)
5. Nginx receives that encrypted session key.
6. Inside the secure confines of your server, Nginx uses privkey.pem (The Private Key) to decrypt the message and reveal the session key.

##### Step C: The Tunnel (Symmetric Encryption)
7. Now, both the browser and Nginx share the exact same temporary session key.
8. For the rest of the user's session, all data (passwords, credit cards, HTML) is encrypted and decrypted instantly using this shared key. When the user closes the tab, that key is thrown away.


