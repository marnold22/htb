# Appointment

> IP: [10.129.123.171]

## NOTES / STEPS

1. Deploy Machine
2. OWASP Top 10 2021
   1. SQL INJECTION
      1. "A03:2021-Injection"
3. NMAP
   1. > nmap -sC -sV 10.129.123.171 -oN init_scan
      1. Port 80 -> http -> Apache httpd 2.4.38 ((Debian))
4. HTTP
   1. Nvaigate to website
      1. We are met with an input form (user login)
5. SQLi
   1. Estimated SQL statment
      1. SELECT * FROM users WHERE user='USERINPUT' AND password='USERINPUT'
   2. Injection
      1. USERNAME: admin' #
      2. PW: #
      3. SUCCESS!
         1. "Congratulations! Your flag is: `e3d0796d002a446c0e622226f42e9672`"

## QUESTIONS

1. What does SQL stand for?
   1. `structured query language`
2. What is one of the most common type of SQL vulnerabilities?
   1. `sql injection`
3. What is the 2021 OWASP Top 10 classification for this vulnerability?
   1. `A03:2021-Injection`
4. What does Nmap report as the service and version that are running on port 80 of the target?
   1. `Apache httpd 2.4.38 ((Debian))`
5. What is the standard port used for the HTTPS protocol?
   1. `443`
6. What is a folder called in web-application terminology?
   1. `directory`
7. What is the HTTP response code is given for 'Not Found' errors?
   1. `404`
8. Gobuster is one tool used to brute force directories on a webserver. What switch do we use with Gobuster to specify we're looking to discover directories, and not subdomains?
   1. `dir`
9. What single character can be used to comment out the rest of a line in MySQL?
   1. `#`
10. If user input is not handled carefully, it could be interpreted as a comment. Use a comment to login as admin without knowing the password. What is the first word on the webpage returned?
    1. `Congratulations`
11. Submit root flag.
    1. `e3d0796d002a446c0e622226f42e9672`
