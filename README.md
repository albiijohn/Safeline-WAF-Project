# Safeline-WAF-Project
Cybersecurity homelab with SafeLine WAF, DVWA, Kali Linux and Ubuntu on VMware

Create Your Own Web Application Firewall using SafeLine WAF
# ########################################################################
<details>
<summary style="font-weight: bold; color: #2a7ae2; font-size: 1.2em;">Step 1: Installing Both Machines on VMware</summary>

- **Kali Linux (IP: 10.0.0.41)**:
  - Download from [kali.org](https://www.kali.org/get-kali).
  - Install in VMware with 2 GB RAM, 20 GB disk, and bridged networking.
- **Ubuntu Server (IP: 10.0.0.147)**:
  - Download from [ubuntu.com](https://ubuntu.com/download/server).
  - Install with 2 GB RAM, 20 GB disk, and bridged networking.
- **Check IPs and Connectivity**:
  ```bash
  ping 10.0.0.147  # From Kali
  ping 10.0.0.41   # From Ubuntu
</details>

# ########################################################################

<details>

<summary style="font-weight: bold; color: #2a7ae2; font-size: 1.2em;">Step 2: Prerequisites</summary>

2.1 DNS Resolution Setup
Edit /etc/hosts on both Kali and Ubuntu:
sudo nano /etc/hosts

Add:
10.0.0.147 dvwa.local

Access DVWA at http://dvwa.local:8080/DVWA from Kali.
2.2 Ubuntu Configurations
2.2.1 Installing OpenSSL
sudo apt-get install -y openssl

2.2.2 Installing and Configuring LAMP Stack
Install Apache2, PHP, and MySQL:
sudo apt-get install -y apache2 php php-mysql mysql-server
sudo mysql_secure_installation

Set MySQL root password: ubuntu.
Configure DVWA (/var/www/html/DVWA/config/config.inc.php):
sudo cp /var/www/html/DVWA/config/config.inc.php.dist /var/www/html/DVWA/config/config.inc.php
sudo sed -i "s/'db_user' => '.*'/'db_user' => 'dvwa_user'/" /var/www/html/DVWA/config/config.inc.php
sudo sed -i "s/'db_password' => '.*'/'db_password' => 'p@ssw0rd'/" /var/www/html/DVWA/config/config.inc.php
sudo sed -i "s/'db_database' => '.*'/'db_database' => 'dvwa'/" /var/www/html/DVWA/config/config.inc.php

Create DVWA database:
sudo mysql -u root -p
CREATE DATABASE dvwa;
CREATE USER 'dvwa_user'@'localhost' IDENTIFIED BY 'p@ssw0rd';
GRANT ALL ON dvwa.* TO 'dvwa_user'@'localhost';
FLUSH PRIVILEGES;
exit;

2.2.3 Changing the DVWA Listening Port to 8080
Edit Apache configuration:
sudo nano /etc/apache2/ports.conf

Change:
Listen 80

to:
Listen 8080

Update virtual host:
sudo nano /etc/apache2/sites-available/000-default.conf

Change:
<VirtualHost *:80>

to:
<VirtualHost *:8080>

Restart Apache:
sudo systemctl restart apache2


Screenshot: LAMP stack configured and DVWA accessible.

</details>

# ########################################################################

<details>

<summary style="font-weight: bold; color: #2a7ae2; font-size: 1.2em;">Step 3: Installing SafeLine in Ubuntu</summary>

Install SafeLine WAF:
bash -c "$(curl -fsSLk https://waf.chaitin.com/release/latest/manager.sh)" -- --en

Access the dashboard at https://10.0.0.147:9443 with provided credentials.Reference: safepoint.cloud  

Screenshot: SafeLine WAF installation complete.

</details>

# ########################################################################

<details>
<summary style="font-weight: bold; color: #2a7ae2; font-size: 1.2em;">Step 4: Using SafeLine</summary>


4.1 SafeLine WAF Dashboard

Application Tab: Add DVWA (www.dvwa.local, port 443, reverse proxy to http://10.0.0.147:8080).
HTTP Flood: Protects against DoS with rate limiting.
Auth: Provides username/password authentication.
Use a 7-day PRO license trial (code: ZFGYUXVXABSUH7KTMQG4FG4B).
Screenshot: Dashboard with DVWA added.

4.2 Setting up Application Rules

Domain: www.dvwa.local
Port: 443 (HTTPS)
Reverse Proxy: http://10.0.0.147:8080
Requires SSL certificate.

4.3 Creating SSL Certificate
Generate SSL certificate:
sudo mkdir /etc/ssl/dvwa
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/dvwa/dvwa.key \
  -out /etc/ssl/dvwa/dvwa.crt

Import into SafeLine:

Certificate: /etc/ssl/dvwa/dvwa.crt
Key: /etc/ssl/dvwa/dvwa.key

4.4 Testing the Application Rule from Kali Browser
Access http://dvwa.local from Kali; it redirects to https://dvwa.local.  

Screenshot: Successful HTTPS redirection.

</details>

# ########################################################################

<details>
  


<summary style="font-weight: bold; color: #2a7ae2; font-size: 1.2em;">Step 5: Setting up HTTP Flood Rules</summary>

Set rate limiting (block IPs after 3 requests in 10 seconds for 5 minutes):

Test by accessing DVWA multiple times from Kali.
Check SafeLine dashboard for blocked IPs.
Screenshot: HTTP flood protection in action.

</details>

# ########################################################################

<details>
  


<summary style="font-weight: bold; color: #2a7ae2; font-size: 1.2em;">Step 6: Setting Authentication Rule</summary>

Enable authentication in SafeLine:

Credentials: admin / password
Test from Kali; an authentication page appears before DVWA.
Screenshot: Authentication prompt before DVWA.

</details>

# ########################################################################

<details>
  

<summary style="font-weight: bold; color: #2a7ae2; font-size: 1.2em;">Step 7: Creating Custom Rules</summary>

Block Kali IP (10.0.0.41):

Add deny rule in SafeLine.
Test from Kali; access is blocked.
Screenshot: Custom rule blocking Kali IP.

</details>

# ########################################################################

<details>



<summary style="font-weight: bold; color: #2a7ae2; font-size: 1.2em;">Step 8: Preventing Attacks</summary>

8.1 Trying SQL Injection with Balanced Rules

In DVWA, set security to low, try SQL injection (e.g., admin' OR '1'='1).
SafeLine blocks it; check dashboard logs.
Screenshot: SQL injection blocked by SafeLine.

8.2 Disabling Attack Rules

Disable SafeLine attack rules; SQL injection succeeds, revealing usernames/passwords.
Screenshot: SQL injection succeeds without rules.

</details>

# ########################################################################

<details>
  


<summary style="font-weight: bold; color: #2a7ae2; font-size: 1.2em;">Step 9: Statistics Dashboard</summary>

View SafeLine dashboard for request counts, blocked IPs, and attack logs.  

Screenshot: Statistics overview in SafeLine.

</details>

Notes

Isolate VMs from production networks.
Update Kali, Ubuntu, and SafeLine WAF regularly.
Screenshots unavailable? Follow steps to recreate visuals.
