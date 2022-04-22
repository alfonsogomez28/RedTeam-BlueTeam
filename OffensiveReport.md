# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services
Nmap scan results for each machine reveal the below services and OS details:

`nmap -Pn -sV -O 192.168.1.110`<br>
![nmap scan](/images/nmapscan.png)<br>
This scan identifies the services below as potential points of entry:
- Target 1
  - Port 22 - SSH
  - Port 80 - HTTP
  - Port 139 - NetBIOS
  - Port 445 - NetBIOS

The following vulnerabilities were identified on each target:
- Target 1
  1. Wordpress Enumeration
  2. Weak, Short Passwords
  3. Password Hash readable in MySQL (no salt)
  4. Privilege Escalation Misconfiguration (python pty vulnerable)

### Exploitation

The Red Team was able to penetrate `Target 1` and retrieve the following confidential data:
- Target 1
  - `flag1.txt`: b9bbcb33ellb80be759c4e844862482d
    - **Exploit Used**
      - command: `wpscan --url http://192.168.1.110/wordpress -u`
      - Wordpress enumeration found usernames `michael` and `steven`
      - Guessing password to both using obvious possible guess found password for user which was `michael`
      - By searching through directories and files, `flag1` was found in `/var/www/html/service.html`
      
  - `flag2.txt`: fc3fd58dcdad9ab23faca6e9a3e581c
    - **Exploit Used**
      - Using the same method of searching through the directories and files, `flag2` was found in `/var/www/flag2.txt`
      - 
    - `flag3.txt`: afc01ab56b50591e7dccf93122770cd2
    - **Exploit Used**
      - One of the important files to search for is the `wp-config.php` file which gives details of how worpress is configured on the server
      - `wp-config.php` was found in `/var/www/html/wordpress` using `cat wp-config.php` show credentials to login to MySQL database: root:R@v3nSecurity
      - Using the command `mysql -u root -p'R@v3nSecurity' -h localhost` we can now login to MySQL
      - Once in MySQL, you can see all databases by using the command `show databases;`, then running the command to select the database in question `use wordpress;`
      - Once in the wordpress database, tables can viewed by using command `show tables;`
      - We see the wp_posts table we can run `select * from wp_posts;` and now have `flag3`
    
      - `flag4.txt`: 715dea6c055b9fe3337544932f2941ce
    - **Exploit Used**
      - Using the same database, we can now get view entries in the wp_users table by running `select * from wp_users;`
      - We can see the password hashes for both users, we can save those hashes in a txt file using the format: `user1:$Passwordhash1` and save it as `wp_hashes.txt`
      - The hashes can be cracked using John the ripper with the command `john wp_hashes.txt`, doing sowe now have user steven's password `pink84`
      - SSH can be used to get into the target machine, command: `ssh steven@192.168.1.110` and entering the password when prompted
      - Once logged in we can see what privileges the user has (command: `ls -l`), which tells us that steven can run python scripts
      - One vulnerability we can use is spawning a root terminal with command: `sudo python -c ‘import pty;pty.spawn(“/bin/bash”)’`
      - Success! now we can see `flag4.txt` in the root folder