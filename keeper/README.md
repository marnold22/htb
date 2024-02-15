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

## QUESTIONS

1. User flag
   1. ``
2. Root flag
   1. ``
