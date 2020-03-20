# Real World Project 
# install all the nessery software 
sudo apt install docker.io
sudo apt install docker-compose 
sudo apt mariadb-client 
# git clone the maxscale repo 
git clone https://github.com/gustanik/CNA350


# git the right directory 
cd CNA350 
cd maxscale 


docker-compose up -d --build 

#output
Starting maxscale_master2_1 ... 
Starting maxscale_master_1 ... 
Starting maxscale_master2_1
Starting maxscale_master2_1 ... done
Starting maxscale_slave2_1 ... 
Starting maxscale_master_1 ... done
Starting maxscale_slave1_1 ... 
Starting maxscale_slave2_1 ... done
Starting maxscale_maxscale_1 ... 
Starting maxscale_maxscale_1 ... done
phpmyadmin is up-to-date

docker-compose ps 


       Name                   Command            State            Ports         
--------------------------------------------------------------------------------
maxscale_master2_1    docker-entrypoint.sh       Up      0.0.0.0:4003->3306/tcp 
                      mysql ...                                                 
maxscale_master_1     docker-entrypoint.sh       Up      0.0.0.0:4001->3306/tcp 
                      mysql ...                                                 
maxscale_maxscale_1   docker-entrypoint.sh       Up      0.0.0.0:3306->3306/tcp,
                      maxsc ...                          0.0.0.0:4006->4006/tcp,
                                                         0.0.0.0:4007->4007/tcp,
                                                         0.0.0.0:8989->8989/tcp 
maxscale_slave1_1     docker-entrypoint.sh       Up      0.0.0.0:4002->3306/tcp 
                      mysql ...                                                 
maxscale_slave2_1     docker-entrypoint.sh       Up      0.0.0.0:4004->3306/tcp 
                      mysql ...                                                 
phpmyadmin            /docker-entrypoint.sh      Up      0.0.0.0:8080->80/tcp   
                      apac ...   
                      
 # making sure all the server runing properly 
 docker-compose exec maxscale maxctrl list servers
 ─────────┬───────────┬──────┬─────────────┬─────────────────┬───────────┐
│ Server  │ Address   │ Port │ Connections │ State           │ GTID      │
├─────────┼───────────┼──────┼─────────────┼─────────────────┼───────────┤
│ server1 │ master    │ 3306 │ 0           │ Master, Running │ 0-3000-32 │
├─────────┼───────────┼──────┼─────────────┼─────────────────┼───────────┤
│ server2 │ slave1    │ 3306 │ 0           │ Slave, Running  │ 0-3000-32 │
├─────────┼───────────┼──────┼─────────────┼─────────────────┼───────────┤
│ server3 │ master2   │ 3306 │ 0           │ Master, Running │ 0-3002-31 │
├─────────┼───────────┼──────┼─────────────┼─────────────────┼───────────┤
│ server4 │ slave2    │ 3306 │ 0           │ Slave, Running  │ 0-3002-31 │
├─────────┼───────────┼──────┼─────────────┼─────────────────┼───────────┤
│ Shard-A │ 127.0.0.1 │ 4006 │ 0           │ Running         │           │
├─────────┼───────────┼──────┼─────────────┼─────────────────┼───────────┤
│ Shard-B │ 127.0.0.1 │ 4007 │ 0           │ Running         │           │
└─────────┴───────────┴──────┴─────────────┴─────────────────┴───────────┘
# we have to make sure all our databases exis on our subdirectory  
 /CNA350/maxscale/sql/shard-A/master$ mysql -umaxuser -pmaxpwd -h 127.0.0.1 -P 3306 -e "show databases"
+--------------------+
| Database           |
+--------------------+
| mysql              |
| information_schema |
| performance_schema |
| zipcodes_one       |
| zipcodes_two       |
+--------------------+

# access the master shard database zipcode two
:~/CNA350/maxscale/sql/shard-A/master$ mysql -umaxuser -pmaxpwd -h 127.0.0.1 -P 3306 -e "SELECT * FROM zipcodes_two.zipcodes_two LIMIT 5;"
+---------+-------------+-------------+-------+--------------+-----------+------------+----------------------+---------------+-----------------+---------------------+------------+
| Zipcode | ZipCodeType | City        | State | LocationType | Coord_Lat | Coord_Long | Location             | Decommisioned | TaxReturnsFiled | EstimatedPopulation | TotalWages |
+---------+-------------+-------------+-------+--------------+-----------+------------+----------------------+---------------+-----------------+---------------------+------------+
|   42040 | STANDARD    | FARMINGTON  | KY    | PRIMARY      | 36.67     | -88.53     | NA-US-KY-FARMINGTON  | FALSE         | 465             | 896                 | 11562973   |
|   41524 | STANDARD    | FEDSCREEK   | KY    | PRIMARY      | 37.4      | -82.24     | NA-US-KY-FEDSCREEK   | FALSE         |                 |                     |            |
|   42533 | STANDARD    | FERGUSON    | KY    | PRIMARY      | 37.06     | -84.59     | NA-US-KY-FERGUSON    | FALSE         | 429             | 761                 | 9555412    |
|   40022 | STANDARD    | FINCHVILLE  | KY    | PRIMARY      | 38.15     | -85.31     | NA-US-KY-FINCHVILLE  | FALSE         | 437             | 839                 | 19909942   |
|   40023 | STANDARD    | FISHERVILLE | KY    | PRIMARY      | 38.16     | -85.42     | NA-US-KY-FISHERVILLE | FALSE         | 1884            | 3733                | 113020684  |
+---------+-------------+-------------+-------+--------------+-----------+------------+----------------------+---------------+-----------------+---------------------+------------+
# access the slave shard database zipcode one 
~/CNA350/maxscale/sql/shard-A/slave$ mysql -umaxuser -pmaxpwd -h 127.0.0.1 -P 3306 -e "SELECT * FROM zipcodes_one.zipcodes_one LIMIT 5;"
+---------+-------------+----------+-------+--------------+-----------+------------+-------------------+---------------+-----------------+---------------------+------------+
| Zipcode | ZipCodeType | City     | State | LocationType | Coord_Lat | Coord_Long | Location          | Decommisioned | TaxReturnsFiled | EstimatedPopulation | TotalWages |
+---------+-------------+----------+-------+--------------+-----------+------------+-------------------+---------------+-----------------+---------------------+------------+
|     705 | STANDARD    | AIBONITO | PR    | PRIMARY      | 18.14     | -66.26     | NA-US-PR-AIBONITO | FALSE         |                 |                     |            |
|     610 | STANDARD    | ANASCO   | PR    | PRIMARY      | 18.28     | -67.14     | NA-US-PR-ANASCO   | FALSE         |                 |                     |            |
|     611 | PO BOX      | ANGELES  | PR    | PRIMARY      | 18.28     | -66.79     | NA-US-PR-ANGELES  | FALSE         |                 |                     |            |
|     612 | STANDARD    | ARECIBO  | PR    | PRIMARY      | 18.45     | -66.73     | NA-US-PR-ARECIBO  | FALSE         |                 |                     |            |
|     601 | STANDARD    | ADJUNTAS | PR    | PRIMARY      | 18.16     | -66.72     | NA-US-PR-ADJUNTAS | FALSE         |                 |                     |            |
+---------+-------------+----------+-------+--------------+-----------+------------+-------------------+---------------+-----------------+---------------------+
