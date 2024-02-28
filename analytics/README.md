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

## QUESTIONS

1. Submit user flag
   1. ``
2. Submit root flag
   1. ``
