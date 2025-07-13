# SafeLine WAF

<div align="center">
  <h2>Self Hosted Web Application Firewall</h2>
  <img src="SS/safeline.png" alt="SafeLine WAF">
</div>

---

## About SafeLine

SafeLine is an advanced Web Application Firewall (WAF) designed to protect web applications from a wide range of cyber threats with robust security measures. Developed by Chaitin Tech, it offers an open-source solution that combines ease of deployment with powerful protection capabilities. 

Ideal for both enterprise and individual use, SafeLine safeguards applications like the Damn Vulnerable Web Application (DVWA) in this cybersecurity homelab project. It integrates seamlessly with platforms such as Ubuntu Server, providing real-time threat detection and mitigation. This project demonstrates SafeLine's effectiveness in securing web applications against attacks like SQL injection.

## Key Features of SafeLine

- **Comprehensive Threat Protection**: Detects and blocks various attacks, including SQL injection, XSS, and brute force attempts, ensuring robust application security.
- **HTTP Flood Defense**: Implements rate-limiting to mitigate denial-of-service (DoS) attacks, protecting server resources from excessive requests.
- **Customizable Rules**: Allows users to create tailored rules, such as blocking specific IPs (e.g., 10.0.0.41), for precise security control.
- **SSL/TLS Support**: Supports secure communication by integrating self-signed or custom SSL certificates for encrypted connections.
- **User-Friendly Dashboard**: Provides an intuitive interface for monitoring traffic, analyzing attack logs, and configuring security settings in real time.

---

## Implementation Guide

<details id="step1">
<summary><strong>Step 1: Installing Both Machines on VMware</strong></summary>

### Machine Setup

**Kali Linux (IP: 10.0.0.41)**:
- Download from [kali.org](https://www.kali.org/get-kali)
- Install in VMware with 2 GB RAM, 20 GB disk, and bridged networking

**Ubuntu Server (IP: 10.0.0.147)**:
- Download from [ubuntu.com](https://ubuntu.com/download/server)
- Install with 2 GB RAM, 20 GB disk, and bridged networking

### Connectivity Testing

Check IPs and connectivity between machines:

```bash
ping 10.0.0.147  # From Kali
ping 10.0.0.41   # From Ubuntu
```

<img src="SS/ping from Kali.png" alt="Ping from Kali">

<img src="SS/ping from ubuntu.png" alt="Ping from Ubuntu">

</details>

<details id="step2">
<summary><strong>Step 2: Prerequisites</strong></summary>

### 2.1 Clone DVWA from Git

Clone DVWA (or download):
```bash
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git
```

If git is not installed, install it first:
```bash
sudo apt-get install -y git
```

### 2.2 Set File Permissions

```bash
sudo chown -R www-data:www-data DVWA
sudo chmod -R 755 DVWA
```

### 2.3 DNS Resolution Setup

Edit `/etc/hosts` on both Kali and Ubuntu:
```bash
sudo nano /etc/hosts
```

Add: `10.0.0.147 dvwa.local`

This will allow access to DVWA at `http://dvwa.local:8080/DVWA/` from Kali.

<img src="SS/dns res ubuntu.png" alt="DNS Resolution Ubuntu">

<img src="SS/dsn res kali.png" alt="DNS Resolution Kali">

### 2.4 Ubuntu Configurations

**Installing OpenSSL**
```bash
sudo apt-get install -y openssl
```

<img src="SS/installing openssl.png" alt="Installing OpenSSL">

**Installing and Configuring LAMP Stack**

This installs Apache2, PHP and MySQL:
```bash
sudo apt-get install -y apache2 php php-mysql mysql-server
sudo mysql_secure_installation
```

Set MySQL root password: `ubuntu` (for testing purpose)

**DVWA Configuration**

DVWA has a config file at `DVWA/config/config.inc.php`. Update it if necessary:
```php
$DBMS = 'MySQL';
$db = 'dvwa';
$user = 'dvwa_user';
$pass = 'p@ssw0rd';
$host = 'localhost';
```

> **Note**: The config.php file may be showing as `DVWA/config/config.inc.php.dist`. Rename to `DVWA/config/config.inc.php`.

<img src="SS/config.php.png" alt="Config PHP">

**Create DVWA Database**
```bash
sudo mysql -u root -p
CREATE DATABASE dvwa;
CREATE USER 'dvwa_user'@'localhost' IDENTIFIED BY 'p@ssw0rd';
GRANT ALL ON dvwa.* TO 'dvwa_user'@'localhost';
FLUSH PRIVILEGES;
exit;
```

**Initialize DVWA**
- Navigate to `http://dvwa.local/setup.php` in your browser
- Click **`[Create/ResetDatabase]`**
- This will automatically create a random database

### 2.5 Changing the DVWA Listening Port to 8080

Edit Apache configuration:
```bash
sudo nano /etc/apache2/ports.conf
```

Change:
```
Listen 80
```
to:
```
Listen 8080
```

### 2.6 Changing the Virtual Host to Port

Edit the apache Virtual host:
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Change:
```
<VirtualHost *:80>
```
to:
```
<VirtualHost *:8080>
```

<img src="SS/default.conf.png" alt="Default Configuration">

**Restart Apache**
```bash
sudo systemctl restart apache2
```

</details>

<details id="step3">
<summary><strong>Step 3: Installing SafeLine in Ubuntu</strong></summary>

### SafeLine Installation

Install SafeLine WAF:
```bash
bash -c "$(curl -fsSLk https://waf.chaitin.com/release/latest/manager.sh)" -- --en
```

**Reference**: [https://safepoint.cloud/landing/safeline](https://safepoint.cloud/landing/safeline)

<img src="SS/install safeline.png" alt="Install SafeLine">

Access the dashboard at `https://10.0.0.147:9443` with provided credentials.

<img src="SS/install cred.png" alt="Installation Credentials">

</details>

<details id="step4">
<summary><strong>Step 4: Using SafeLine</strong></summary>

### 4.1 SafeLine WAF Dashboard

**Application Tab**: Add DVWA (www.dvwa.local, port 443, reverse proxy to http://10.0.0.147:8080)

**HTTP Flood**: Protects against DoS with rate limiting

**Auth**: Provides username/password authentication

> Used a 7-day PRO license trial

#### Dashboard Components

- **Application tab**: Link applications that need protection (DVWA in our case)
- **HTTP flood**: Protects against DOS attacks using rate limiting features
- **Auth**: Provides username and password authentication for applications and websites

<img src="SS/dashboard.png" alt="SafeLine Dashboard">

### 4.2 Setting up Application Rules

**Configuration**:
- Domain: `www.dvwa.local`
- Port: `443` (HTTPS)
- Reverse Proxy: `http://10.0.0.147:8080`
- Requires SSL certificate

Set the Reverse proxy to domain (ubuntu) IP: `10.0.0.147:8080`

> **Note**: Any request to the service comes into the SafeLine firewall and forwards the request to port 8080 on the server in the background.

### 4.3 Creating SSL Certificate

Generate private key:
```bash
openssl genrsa -out private.key 4096
```

Generate private.csr:
```bash
openssl req -new -key private.key -out private.csr
```

<img src="SS/private.png" alt="Private Key Generation">

Generate SSL certificate:
```bash
openssl x509 -req -days 365 -in private.csr -signkey private.key -out private.crt
```

<img src="SS/ssl key.png" alt="SSL Key Generation">

Import into SafeLine:

<img src="SS/import.png" alt="SSL Import">

### 4.4 Testing the Application Rule from Kali Browser

Access `http://dvwa.local` from Kali; it redirects to `https://dvwa.local`.

This rule allows incoming traffic only through port 443 (HTTPS).

<img src="SS/https 1.png" alt="HTTPS Test 1">

<img src="SS/https 2.png" alt="HTTPS Test 2">

<img src="SS/https 3.png" alt="HTTPS Test 3">

</details>

<details id="step5">
<summary><strong>Step 5: Setting up HTTP Flood Rules</strong></summary>

### Rate Limiting Configuration

Set rate limiting to block IPs after 3 requests in 10 seconds for 5 minutes.

**Testing Process**:
- Access DVWA multiple times from Kali
- Check SafeLine dashboard for blocked IPs

For testing, if more than 3 access requests are made from an IP within 10 seconds, the rule blocks the IP for 5 minutes.

<img src="SS/flood 1.png" alt="Flood Rule 1">

<img src="SS/flood 2.png" alt="Flood Rule 2">

Testing by accessing the site from Kali and clicking on multiple options. Access is denied suddenly.

<img src="SS/flood 3.png" alt="Flood Rule 3">

Looking at the HTTP flood request in the SafeLine dashboard, the Kali IP was blocked. There's also an option to unblock the IP from the dashboard.

<img src="SS/flood 4.png" alt="Flood Rule 4">

</details>

<details id="step6">
<summary><strong>Step 6: Setting Authentication Rule</strong></summary>

### Authentication Setup

Enable authentication in SafeLine with test credentials:
- Username: `admin`
- Password: `password`

<img src="SS/auth 1.png" alt="Authentication Setup">

Test from Kali; an authentication page appears before DVWA.

<img src="SS/auth 2.png" alt="Authentication Test">

The firewall captures the request, waiting for approval.

> **Note**: Rules can also be set to auto-approve after successful authentication.

<img src="SS/auth 3.png" alt="Authentication Approval">

</details>

<details id="step7">
<summary><strong>Step 7: Creating Custom Rules</strong></summary>

### Custom IP Blocking

Create a rule to deny any request from Kali IP (10.0.0.41):

Add deny rule in SafeLine:

<img src="SS/custom 1.png" alt="Custom Rule Setup">

Test from Kali; access is blocked:

<img src="SS/custom 2.png" alt="Custom Rule Test">

</details>

<details id="step8">
<summary><strong>Step 8: Preventing Attacks</strong></summary>

### 8.1 SQL Injection with Balanced Rules

**Testing Process**:
1. In DVWA, set security to low
2. Try SQL injection (e.g., `admin' OR '1'='1`)
3. SafeLine blocks it
4. Check dashboard logs for SQL injection blocked by SafeLine

<img src="SS/balanced 1.png" alt="Balanced Rules 1">

<img src="SS/balanced 2.png" alt="Balanced Rules 2">

### 8.2 Disabling Attack Rules

**Demonstration**:
- Disable SafeLine attack rules
- SQL injection succeeds, revealing usernames/passwords

<img src="SS/disable 1.png" alt="Disabled Rules 1">

<img src="SS/disable 2.png" alt="Disabled Rules 2">

> **Note**: Other attacks such as hping, HTTP floods from CLI, sqlmap etc. can also be performed and monitored using the SafeLine dashboard.

</details>

<details id="step9">
<summary><strong>Step 9: Statistics Dashboard</strong></summary>

### Monitoring and Analytics

View SafeLine dashboard for:
- Request counts
- Blocked IPs
- Attack logs
- Real-time statistics

<img src="SS/stat.png" alt="Statistics Dashboard">

</details>

---

## Conclusion

SafeLine WAF provides comprehensive protection for web applications through its intuitive dashboard and powerful security features. This implementation demonstrates effective mitigation of common web application vulnerabilities including SQL injection, DoS attacks, and unauthorized access attempts.

<!-- The system's flexibility in creating custom rules and real-time monitoring capabilities make it an excellent choice for both educational purposes and production environments.

<script>
document.addEventListener('DOMContentLoaded', function() {
    const details = document.querySelectorAll('details');
    
    details.forEach(detail => {
        detail.addEventListener('toggle', function() {
            if (this.open) {
                // Close all other details
                details.forEach(otherDetail => {
                    if (otherDetail !== this && otherDetail.open) {
                        otherDetail.open = false;
                    }
                });
            }
        });
    });
});
</script>

<style>
details {
    margin: 20px 0;
    border: 1px solid #ddd;
    border-radius: 8px;
    background: #f9f9f9;
    transition: all 0.3s ease;
}

details:hover {
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

details[open] {
    background: #fff;
    border-color: #2a7ae2;
}

summary {
    padding: 15px 20px;
    cursor: pointer;
    background: #f0f0f0;
    border-radius: 8px;
    font-size: 1.1em;
    color: #2a7ae2;
    transition: background 0.3s ease;
}

summary:hover {
    background: #e8f4f8;
}

details[open] summary {
    background: #2a7ae2;
    color: white;
    margin-bottom: 15px;
}

details div, details > *:not(summary) {
    padding: 0 20px 20px;
}

code {
    background: #f4f4f4;
    padding: 2px 6px;
    border-radius: 3px;
    font-family: 'Consolas', 'Monaco', monospace;
}

pre {
    background: #2d3748;
    color: #e2e8f0;
    padding: 15px;
    border-radius: 5px;
    overflow-x: auto;
    margin: 10px 0;
}

pre code {
    background: none;
    padding: 0;
    color: inherit;
}

img {
    max-width: 100%;
    height: auto;
    border-radius: 5px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    margin: 10px 0;
}

blockquote {
    border-left: 4px solid #2a7ae2;
    margin: 15px 0;
    padding: 10px 20px;
    background: #f8f9fa;
    border-radius: 0 5px 5px 0;
}

h1, h2, h3 {
    color: #2c3e50;
}

h1 {
    text-align: center;
    border-bottom: 3px solid #2a7ae2;
    padding-bottom: 10px;
}

ul, ol {
    padding-left: 20px;
}

li {
    margin: 8px 0;
}

hr {
    border: none;
    height: 2px;
    background: linear-gradient(to right, #2a7ae2, #4a90e2, #2a7ae2);
    margin: 30px 0;
}
</style> */
