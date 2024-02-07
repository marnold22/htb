# Crocodile

> IP: [10.129.1.15]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.129.1.15 -oN init_scan
      1. Port 21 -> ftp -> vsftpd 3.0.3
      2. Port 80 -> http -> Apache httpd 2.4.41 ((Ubuntu))
3. FTP
   1. Connect using the nonymous user
      1. > ftp anonymous@10.129.1.15
         1. > ls
         2. > get allowed.userlist
         3. > get allowed.userlist.passwd
4. FILES
   1. Looking through the userlist we see
      1. ADMIN
5. HTTP
   1. GOBUSTER
      1. > gobuster dir -x php,txt,html -u http://10.129.1.15/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
         1. login.php
         2. logout.php
         3. config.php
   2. LOGIN
      1. Using our userslist and userslist.passwd lets try the admin combo to signin
         1. UN: admin
         2. PW: rKXM59ESxesUFHAd
         3. REDIRECTED: to a admin dashboard with the flag `Here is your flag: c7110277ac44d78b6a9fff2232434d16`

## QUESTIONS

1. What Nmap scanning switch employs the use of default scripts during a scan?
   1. `-sC`
2. What service version is found to be running on port 21?
   1. `vsftpd 3.0.3`
3. What FTP code is returned to us for the "Anonymous FTP login allowed" message?
   1. `230`
4. After connecting to the FTP server using the ftp client, what username do we provide when prompted to log in anonymously?
   1. `anonymous`
5. After connecting to the FTP server anonymously, what command can we use to download the files we find on the FTP server?
   1. `get`
6. What is one of the higher-privilege sounding usernames in 'allowed.userlist' that we download from the FTP server?
   1. `admin`
7. What version of Apache HTTP Server is running on the target host?
   1. `Apache httpd 2.4.41`
8. What switch can we use with Gobuster to specify we are looking for specific filetypes?
   1. `-x`
9. Which PHP file can we identify with directory brute force that will provide the opportunity to authenticate to the web service?
   1. `login.php`
10. Submit root flag
    1. `c7110277ac44d78b6a9fff2232434d16`
