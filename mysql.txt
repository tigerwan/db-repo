


* On master
    shell> mysql -uroot -p$(sudo keydbgetkey y7mysqlroot) -e "stop slave;"
    shell>mysql -uroot -p$(sudo keydbgetkey y7mysqlroot) -e "SHOW MASTER STATUS\G"
    +-------------------+----------+--------------+------------------+-------------------+
    | File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
    +-------------------+----------+--------------+------------------+-------------------+
    | mysqld-bin.001483 | 87008169 |              |                  |                   |
    +-------------------+----------+--------------+------------------+-------------------+
    mysql> exit;
    shell> mysqldump --single-transaction --insert-ignore -u root -p$(sudo keydbgetkey y7mysqlroot) --all-databases  |  gzip  > dbdump.sql.gz
        might useful mysqldump arguments
            --single-transaction(innodb only):  Issue a BEGIN SQL statement before dumping data from server
                    use "SHOW ENGINES" to check, "https://dev.mysql.com/doc/refman/5.7/en/show-engines.html"
            --routines  : Dump stored routines (procedures and functions) from dumped databases
            --events: Dump events from dumped databases
            --trigers: Dump triggers for each dumped table
        watch out: you should NOT export/import mysql system database "mysql" to avoid the optential error
            how to dump all databases except mysql
                http://datacharmer.blogspot.com.au/2012/04/few-hacks-to-simulate-mysqldump-ignore.html
                    DATABASE_LIST=$(mysql -u root -p$(sudo keydbgetkey y7mysqlroot) -NBe 'show schemas' | grep -wv 'mysql\|performance_schema\|information_schema')                      
                    mysqldump --single-transaction --insert-ignore -u root -p$(sudo keydbgetkey y7mysqlroot) --databases $DATABASE_LIST | gzip > ./newdump.sql.gz
                        
                mysql:  -N, --skip-column-names
                        -B, --batch         Don't use history file. Disable interactive behavior.
                
                    
                    
                
                
    shell> mysql -uroot -p$(sudo keydbgetkey y7mysqlroot)
    mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'replica_host' IDENTIFIED BY 'm1rr0rIt'
    mysql> show grants for 'repl'@'replica_host';
    +----------------------------------------------------------------------------------------------------------------------------------------------+
    | Grants for repl@replica_host                                                                                               |
    +----------------------------------------------------------------------------------------------------------------------------------------------+
    | GRANT REPLICATION SLAVE ON *.* TO 'repl'@'replica_host' IDENTIFIED BY PASSWORD '*378661BDA9D330EEE167A372E17F20245D5565D3' |
    +----------------------------------------------------------------------------------------------------------------------------------------------+
    1 row in set (0.00 sec)
    mysql> FLUSH PRIVILEGES;

On slave
    shell> screen
    shell> date; mysql -uroot -p$(sudo keydbgetkey mysqlroot) < dbdump_final.sql 2>&1 ;date 
    mysql> CHANGE MASTER TO MASTER_HOST='127.0.0.1',MASTER_PORT=3308, MASTER_USER='repl', MASTER_PASSWORD='m1rr0rIt', MASTER_LOG_FILE='mysqld-bin.001483', MASTER_LOG_POS=  87008169;
            3308 is listend by ystunnel ( yinst start aunz_ystunnel, yinst start ystunnel)
    mysql> start slave;
    mysql> show slave status\G

Appendix1
    /home/y/etc/my.cnf
    /home/y/libexec64/mysqld --basedir=/home/y --datadir=/home/y/var/mysql/data --plugin-dir=/home/y/lib64/mysql/plugin/ --user=mysql --log-error=/home/y/logs/mysql/mysqld.err --open-files-limit=16384 --pid-file=/home/y/var/mysql/mysqld.pid --socket=/tmp/mysql.sock --port=3306
    /bin/sh /home/y/bin/mysqld_safe --datadir=/home/y/var/mysql/data --pid-file=/home/y/var/mysql/mysqld.pid --pid-file=/home/y/var/mysql/mysqld.pid --ledir=/home/y/libexec64 --nice=-10
    
    
Appendix2
    [jwan@worsehearse ~]$ mysql -u root -p$(sudo keydbgetkey mysqlroot) -e "show slave status\G"
    Warning: Using a password on the command line interface can be insecure.
    *************************** 1. row ***************************
                   Slave_IO_State: Waiting for master to send event
                      Master_Host: 127.0.0.1
                      Master_User: repl
                      Master_Port: 3308
                    Connect_Retry: 60
                  Master_Log_File: mysqld-bin.001485
              Read_Master_Log_Pos: 874709170
                   Relay_Log_File: mysqld-relay-bin.000002
                    Relay_Log_Pos: 1148598
            Relay_Master_Log_File: mysqld-bin.001483
                 Slave_IO_Running: Yes
                Slave_SQL_Running: Yes
                  Replicate_Do_DB:
              Replicate_Ignore_DB:
               Replicate_Do_Table:
           Replicate_Ignore_Table:
          Replicate_Wild_Do_Table:
      Replicate_Wild_Ignore_Table:
                       Last_Errno: 0
                       Last_Error:
                     Skip_Counter: 0
              Exec_Master_Log_Pos: 88156483
                  Relay_Log_Space: 1681676413
                  Until_Condition: None
                   Until_Log_File:
                    Until_Log_Pos: 0
               Master_SSL_Allowed: No
               Master_SSL_CA_File:
               Master_SSL_CA_Path:
                  Master_SSL_Cert:
                Master_SSL_Cipher:
                   Master_SSL_Key:
            Seconds_Behind_Master: 162360
    Master_SSL_Verify_Server_Cert: No
                    Last_IO_Errno: 0
                    Last_IO_Error:
                   Last_SQL_Errno: 0
                   Last_SQL_Error:
      Replicate_Ignore_Server_Ids:
                 Master_Server_Id: 461054276
                      Master_UUID: eb95456f-3da7-11e6-a4e8-d8d385db446a
                 Master_Info_File: /home/y/var/mysql/data/master.info
                        SQL_Delay: 0
              SQL_Remaining_Delay: NULL
          Slave_SQL_Running_State: Reading event from the relay log
               Master_Retry_Count: 86400
                      Master_Bind:
          Last_IO_Error_Timestamp:
         Last_SQL_Error_Timestamp:
                   Master_SSL_Crl:
               Master_SSL_Crlpath:
               Retrieved_Gtid_Set:
                Executed_Gtid_Set:
                    Auto_Position: 0
                    
                    
Appendix 3: how to setup multiple threads replica
       stop  slave;
       select @@slave_parallel_workers;         ===== or how global variables like 'slave_parallel_workers';
       set global slave_parallel_workers=8;    =====并发处理线程数
       start slave;
       
Appendix 4: delete user database
        mydbs="cobra feeds_rssdb genie movies_2009 narc navigation notification_helper notifications nz_weather nztv platforms rampv2 scheduler sport sport_v2 sport_v3 test tnz tv"
        mypwd=$(sudo keydbgetkey mysqlroot)
        for i in $(echo $mydbs)
        do
            echo "DROP DATABASE IF EXISTS $i;"
            date
            mysql -uroot -p"$mypwd" -e "DROP DATABASE IF EXISTS $i;"
        done
        mysql -uroot -p"$mypwd" -e "show databases;"
        

Appendix 5: revoke permssion
    revoke replication slave  on *.* from 'repl'@'test-vlan145.ops.aue.yahoo.com';
    drop user 'repl'@'test-vlan145.ops.aue.yahoo.com'

Appendix 6: update passowrd
    UPDATE mysql.user SET Password=PASSWORD('xxxxxxxx') WHERE User='mos_dev'
    FLUSH PRIVILEGES;
      
Appendix 7: show users
    select user,host from mysql.user where user='marcelo'
    
Appendix 8: show variables
    mysql -u root -p$(sudo keydbgetkey mysqlroot) -e "show global variables like 'binlog_format'"
    in /etc/my.cnf binlog_format=STATEMENT|ROW|MIXED
    
Appendix 10: show transaction from transaction log
    statement based log
        sudo mysqlbinlog --base64-output=AUTO --start-position=609113224 --stop-position=609113224  /home/y/var/mysql/data/mysqld-relay-bin.000024 > /tmp/1.txt

        sudo mysqlbinlog --base64-output=AUTO --start-position=$(mysql -u root -p$(sudo keydbgetkey mysqlroot) -e "show slave status\G"|egrep "Relay_Log_Pos" | awk -F":" '{print $2}')  /home/y/var/mysql/data/mysqld-relay-bin.000009   > /tmp/x.txt
        
    row based log
        sudo mysqlbinlog --base64-output=decode-rows -vv --start-datetime="2017-07-17 19:50:00" --stop-datetime="2017-07-17 20:00:00"  /home/y/var/mysql/data/mysqld-bin.002137  > /tmp/2.txt
        
        sudo mysqlbinlog --base64-output=decode-rows -vv --start-datetime="2017-07-17 23:57:00" --stop-datetime="2017-07-18 00:00:00"    /home/y/var/mysql/data/mysqld-relay-bin.000024 > /tmp/2.txt
    
    counting affect rows in each transaction(https://dzone.com/articles/identifying-useful-info-mysql)
        cat /tmp/1.txt | awk \
        'BEGIN {s_type=""; s_count=0;count=0;insert_count=0;update_count=0;delete_count=0;flag=0;} \
        {if(match($0, /Table_map: .* mapped to number/)) {printf "Timestamp : " $1 " " $2 " Table : " $(NF-4); flag=1} \
        else if (match($0, /(INSERT INTO .*..*)/)) {count=count+1;insert_count=insert_count+1;s_type="INSERT"; s_count=s_count+1;}  \
        else if (match($0, /(UPDATE .*..*)/)) {count=count+1;update_count=update_count+1;s_type="UPDATE"; s_count=s_count+1;} \
        else if (match($0, /(DELETE FROM .*..*)/)) {count=count+1;delete_count=delete_count+1;s_type="DELETE"; s_count=s_count+1;}  \
        else if (match($0, /^(# at) /) && flag==1 && s_count>0) {print " Query Type : "s_type " " s_count " row(s) affected" ;s_type=""; s_count=0; }  \
        else if (match($0, /^(COMMIT)/)) {print "[Transaction total : " count " Insert(s) : " insert_count " Update(s) : " update_count " Delete(s) : " \
        delete_count "] \n+----------------------+----------------------+----------------------+----------------------+"; \
        count=0;insert_count=0;update_count=0; delete_count=0;s_type=""; s_count=0; flag=0} } '

            
        cat /tmp/1.txt | awk \
        'BEGIN {s_type=""; s_count=0;count=0;insert_count=0;update_count=0;delete_count=0;flag=0;} \
        {if(match($0, /Table_map: .* mapped to number/)) {flag=1} \
        else if (match($0, /(INSERT INTO .*..*)/)) {count=count+1;insert_count=insert_count+1;s_type="INSERT"; s_count=s_count+1;}  \
        else if (match($0, /(UPDATE .*..*)/)) {count=count+1;update_count=update_count+1;s_type="UPDATE"; s_count=s_count+1;} \
        else if (match($0, /(DELETE FROM .*..*)/)) {count=count+1;delete_count=delete_count+1;s_type="DELETE"; s_count=s_count+1;}  \
        else if (match($0, /^(COMMIT)/)) {print "[Transaction total : " count " Insert(s) : " insert_count " Update(s) : " update_count " Delete(s) : " \
        delete_count "] \n+----------------------+----------------------+----------------------+----------------------+"; \
        count=0;insert_count=0;update_count=0; delete_count=0;s_type=""; s_count=0; flag=0} } '
        
        
   roughly show the count of updated of table and time stamp    
        egrep -i "Table_map" /tmp/1.txt | awk '{print "timestamp:"$1" "$2", table:"$11}'

Appendix 11: update user's password
    UPDATE mysql.user SET Password=PASSWORD('mos_dev_platform2') WHERE User='mos_dev'; 
    UPDATE mysql.user SET Password=PASSWORD('db_dev_platform2') WHERE User='db_dev'; 
    FLUSH PRIVILEGES;

Appendix 12: monitor status via http
    http://hub4.alib.aunz.gq1.yahoo.com:4080/mysql_slave
    http://hub1.alib.aunz.gq1.yahoo.com:4080/mysql_master/single_writer
    
Appendix 13: /home/y/conf/keydb/mysqlroot.keydb is part of mysql_grants, it's created when pkg is started
    
    
Appendix 14: show the db's size 
SELECT table_schema "Data Base Name", sum( data_length + index_length ) / 1024 / 1024 / 1024 "Data Base Size in GB"  FROM information_schema.TABLES GROUP BY table_schema;    

