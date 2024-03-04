# Perfection [SEASONAL]

> IP: [10.10.11.253]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.10.11.253 -oN init_scan
      1. Port 22 -> ssh
      2. Port 80 -> http
      3. Port 8000 -> http-alt
3. HTTP
   1. Navigate to website and inspect source code
      1. "Powered by WEBrick 1.7.0"
      2. It looks like it is a weighted grade calculator
         1. HOME, ABOUT, WEIGHTED-GRADE
      3. WEIGHTED-GRADE
         1. On this page we see an input form (that looks like a table)
            1. Maybe we can use XSS
         2. FORM
            1. Looking at the form we see:

                ```html
                    <form method="POST" action="/weighted-grade-calc">
                        ...
                    </form>
                ```

                1. The action is /weighted-grade-calc
    2. /WEIGHTED-CALC
       1. Navigate to this page and we see:

            ```text
                Sinatra doesn’t know this ditty.
                Try this:
                
                get '/weighted-grade-calc' do
                    "Hello World"
                end
            ```

   2. GOBUSTER
      1. > gobuster dir -x php,txt,html -u http://10.10.11.253/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
         1. /about

## QUESTIONS

1. Submit flag
