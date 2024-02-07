# Responder

> IP: [10.129.82.38]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.129.82.38 -oN init_scan
      1. Port 80 -> http -> Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
3. HTTP
   1. Navigate to website
      1. Redirected to unika.htb
         1. Need to add this to our /etc/hosts file to be able to view
   2. SOURCE CODE
      1. Language -> index.php?page=LANGUAGE
4. RESPONDER
   1. > responder -h
      1. Interface = "-I"

## QUESTIONS

1. When visiting the web service using the IP address, what is the domain that we are being redirected to?
   1. `http://unika.htb/`
2. Which scripting language is being used on the server to generate webpages?
   1. `PHP`
3. What is the name of the URL parameter which is used to load different language versions of the webpage?
   1. `page`
4. Which of the following values for the `page` parameter would be an example of exploiting a Local File Include (LFI) vulnerability: "french.html", "//10.10.14.6/somefile", "../../../../../../../../windows/system32/drivers/etc/hosts", "minikatz.exe"
   1. `../../../../../../../../windows/system32/drivers/etc/hosts`
5. Which of the following values for the `page` parameter would be an example of exploiting a Remote File Include (RFI) vulnerability: "french.html", "//10.10.14.6/somefile", "../../../../../../../../windows/system32/drivers/etc/hosts", "minikatz.exe"
   1. `//10.10.14.6/somefile`
6. What does NTLM stand for?
   1. `new technology LAN manager`
7. Which flag do we use in the Responder utility to specify the network interface?
   1. `-I`
8. There are several tools that take a NetNTLMv2 challenge/response and try millions of passwords to see if any of them generate the same response. One such tool is often referred to as `john`, but the full name is what?.
   1. `john the ripper`
9. What is the password for the administrator user?
   1. ``
10. We'll use a Windows service (i.e. running on the box) to remotely access the Responder machine using the password we recovered. What port TCP does it listen on?
    1. ``
11. Submit root flag
    1. ``
