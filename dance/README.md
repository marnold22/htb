# Dance

> IP: [10.129.33.34]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.129.33.34 -oN init_scan

        ```text
            PORT    STATE SERVICE       VERSION
            135/tcp open  msrpc         Microsoft Windows RPC
            139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
            445/tcp open  microsoft-ds?
            Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

            Host script results:
            |_clock-skew: 4h00m00s
            | smb2-time: 
            |   date: 2024-01-30T04:21:13
            |_  start_date: N/A
            | smb2-security-mode: 
            |   3:1:1: 
            |_    Message signing enabled but not required
        ```

3. SMB
   1. > smbclient -L 10.129.33.34

        ```text
            Sharename       Type      Comment
            ---------       ----      -------
            ADMIN$          Disk      Remote Admin
            C$              Disk      Default share
            IPC$            IPC       Remote IPC
            WorkShares      Disk 
        ```

   2. > smbclient \\\\10.129.33.34\\ADMIN$
      1. NT_STATUS_ACCESS_DENIED
   3. > smbclient \\\\10.129.33.34\\C$
      1. NT_STATUS_ACCESS_DENIED
   4. > smbclient \\\\10.129.33.34\\IPC$
      1. NT_STATUS_ACCESS_DENIED
   5. > smbclient \\\\10.129.33.34\\WorkShares
      1. > ls
         1. Amy.J, James.p
      2. > cd Amy.j
         1. > get worknotes.txt
      3. > cd James.p
         1. > get flag.txt

## QUESTIONS

1. What does the 3-letter acronym SMB stand for?
   1. `server message block`
2. What port does SMB use to operate at?
   1. `445`
3. What is the service name for port 445 that came up in our Nmap scan?
   1. `microsoft-ds`
4. What is the 'flag' or 'switch' that we can use with the smbclient utility to 'list' the available shares on Dancing?
   1. `-l`
5. How many shares are there on Dancing?
   1. `4`
6. What is the name of the share we are able to access in the end with a blank password?
   1. `workshares`
7. What is the command we can use within the SMB shell to download the files we find?
   1. `get`
8. Submit root flag
   1. `5f61c10dffbc77a704d76016a22f1664`
