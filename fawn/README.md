# Fawn

> IP: [10.129.177.202]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.129.177.202 -oN init_scan

        ```text
            PORT   STATE SERVICE VERSION
            21/tcp open  ftp     vsftpd 3.0.3
            | ftp-syst: 
            |   STAT: 
            | FTP server status:
            |      Connected to ::ffff:10.10.14.49
            |      Logged in as ftp
            |      TYPE: ASCII
            |      No session bandwidth limit
            |      Session timeout in seconds is 300
            |      Control connection is plain text
            |      Data connections will be plain text
            |      At session startup, client count was 3
            |      vsFTPd 3.0.3 - secure, fast, stable
            |_End of status
            | ftp-anon: Anonymous FTP login allowed (FTP code 230)
            |_-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
            Service Info: OS: Unix
        ```

        1. We see ftp on port 21 running version `vsftpd 3.0.3`
        2. It also looks like the flag is accessible by the anonymous user
3. FTP
   1. > ftp anonymous@10.129.177.202
      1. > get flag.txt

## QUESTIONS

1. What does the 3-letter acronym FTP stand for?
   1. `file transfer protocol`
2. Which port does the FTP service listen on usually?
   1. `21`
3. What acronym is used for the secure version of FTP?
   1. `sftp`
4. What is the command we can use to send an ICMP echo request to test our connection to the target?
   1. `ping`
5. From your scans, what version is FTP running on the target?
   1. `3.0.3`
6. From your scans, what OS type is running on the target?
   1. `unix`
7. What is the command we need to run in order to display the 'ftp' client help menu?
   1. `ftp -h`
8. What is username that is used over FTP when you want to log in without having an account?
   1. `anonymous`
9. What is the response code we get for the FTP message 'Login successful'?
   1. `230`
10. There are a couple of commands we can use to list the files and directories available on the FTP server. One is dir. What is the other that is a common way to list files on a Linux system.
    1. `ls`
11. What is the command used to download the file we found on the FTP server?
    1. `get`
12. Submit root flag
    1. `035db21c881520061c53e0536e44f815`
