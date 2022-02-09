需要的一些 sql

* master 
```sql
show master status;
```

* slave
```sql
change master to master_host = '172.17.0.2',
    master_user = 'slave',
    master_password = '123123',
    master_log_file = 'mysql-bin.000010',
    master_log_pos = 5018;


stop slave;

reset master ;

reset slave ;

start slave ;

show slave status ;
```