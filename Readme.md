## Uninstall Existing Database ##

Note: make sure you backup your data bafore doing following steps.

##Stop mysql and check

sudo service  mysqld stop

sudo ps aux | grep mysql



##Remove or rename existing db folder and files.

sudo mv /vol/data/mysql  /vol/data/mysql__old

sudo mv /etc/my.cnf /etc/my.old__cnf

sudo mv /etc/my.cnf.d /etc/my.cnf.d__old

##Remove or rename existing db folder and files.

sudo mv /var/lib/mysql  /var/lib/mysql__old

sudo mv /etc/my.cnf /etc/my.old__cnf

sudo mv /etc/my.cnf.d /etc/my.cnf.d__old

## Remove Existing Mariadb-Galera from all 3 servers

sudo yum remove MariaDB-client MariaDB-common MariaDB-compat MariaDB-connect-engine MariaDB-server galera-4

## Install New one 

sudo yum -y install MariaDB-server MariaDB-client MariaDB-connect-engine

## Check my.cnf exists and /etc/my.cnf.d/galera.cnf (should not exists)

ll /etc/my.cnf

ll /etc/my.cnf.d/galera.cnf

## Start DB

sudo systemctl start mariadb ; sudo systemctl enable mariadb

## Set password

sudo mysql -uroot

set password = password("WelComE.01!");

CREATE USER 'cluster-user'@'%' IDENTIFIED BY 'clusterpass';

GRANT ALL PRIVILEGES ON *.* TO 'cluster-user'@'%' IDENTIFIED BY 'clusterpass' WITH GRANT OPTION;



CREATE USER 'clustercheck'@'localhost' IDENTIFIED BY 'clustercheck';

GRANT PROCESS ON *.* TO 'clustercheck'@'localhost' IDENTIFIED BY 'clustercheck';



SET session auto_increment_increment = 1;

SET global auto_increment_increment = 1;



SET session auto_increment_offset = 1;


SET global auto_increment_offset = 1;



quit;



----------------------------------

sudo yum -y install  rsync policycoreutils-python

----------------------------------



----------------------------------

Step 3 — Configuring the First Node



sudo vi /etc/my.cnf.d/galera.cnf
```


[mysqld]

binlog_format=ROW

default-storage-engine=innodb

innodb_autoinc_lock_mode=2

bind-address=0.0.0.0



# Galera Provider Configuration

wsrep_on=ON

wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so



# Galera Cluster Configuration

wsrep_cluster_name="web_cluster"

wsrep_cluster_address="gcomm://172.16.1.173,172.17.1.173,172.18.1.173"



# Galera Synchronization Configuration

wsrep_sst_method=rsync

wsrep_sst_auth=cluster-user:clusterpass





# Galera Node Configuration

wsrep_node_address="172.16.1.173"

wsrep_node_name="web-db-01"

```

----------------------------------

sudo service firewalld stop

sudo systemctl stop mariadb

-------------------



#On the first node, bootstrap the cluster by executing

sudo galera_new_cluster

sudo ps aux | grep mysql





#start serverss one by one 2nd and 3rd

sudo systemctl start  mariadb



#check status

sudo systemctl status  mariadb





------

Stop all 3

sudo systemctl stop mariadb



sudo rm -rf /vol/data/mysql



sudo mv /var/lib/mysql /vol/data/



sudo mkdir -p 700 /vol/data/mysql/mysqltmp

sudo chown -R mysql:mysql /vol/data/mysql/mysqltmp



----------



## edit my.cnf

sudo vi /etc/my.cnf



```

#

# This group is read both both by the client and the server

# use it for options that affect everything

#

[client-server]



#

# include all files from the config directory

#

!includedir /etc/my.cnf.d





[mysqld]



datadir=/vol/data/mysql

socket=/vol/data/mysql/mysql.sock





# 80% RAM innodb_buffer_pool_size 52G = 55834574848

innodb_buffer_pool_size=55834574848

#innodb_flush_method=normal





#sort_buffer_size=8000M

#join_buffer_size = 3000M

#read_rnd_buffer_size=3000M



#1G= 1073741824 500M = 524288000 50M= 52428800

sort_buffer_size=52428800

join_buffer_size=52428800

read_rnd_buffer_size=52428800

max_sort_length=8388608

max_length_for_sort_data=1048





# 1.5G = 1610612736 500M = 524288000

innodb_lock_wait_timeout=90

query_cache_type=1

query_cache_size=52428800

query_cache_limit=52428800

thread_cache_size=90

max_connections=500





#9G = 9663676416

max_heap_table_size=9663676416

tmp_table_size=9663676416

max_allowed_packet=52428800

table_open_cache=300

#Support Large Txn

#innodb_log_buffer_size=9663676416





#Log

log_output=FILE



log_warnings=2

log_error=mariadb.err



slow_query_log

slow_query_log = 1

slow_query_log_file = mariadb-slow.log

long_query_time=600

log_queries_not_using_indexes=ON





tmpdir=/vol/data/mysql/mysqltmp



character-set-server=utf8

collation-server=utf8_unicode_ci

skip-character-set-client-handshake

#character-set-connection=utf8



innodb_buffer_pool_instances=4

innodb_page_cleaners=4



group_concat_max_len=1048576







# 2GB for last txn

#wsrep_max_ws_size=2147483647

#binlog_row_image='MINIMAL'


auto_increment_increment = 1


```


## sudo cat /vol/data/mysql/grastate.dat

```[neon@neon-web-db-01 ~]$ sudo cat /vol/data/mysql/grastate.dat

# GALERA saved state

version: 2.1

uuid:    1422a148-0f6c-11ea-bdad-5b132c2a2ccc

seqno:   6

safe_to_bootstrap: 1

[neon@neon-web-db-01 ~]$
 

[neon@neon-web-db-02 ~]$ sudo cat /vol/data/mysql/grastate.dat

# GALERA saved state

version: 2.1

uuid:    1422a148-0f6c-11ea-bdad-5b132c2a2ccc

seqno:   4

safe_to_bootstrap: 0

[neon@neon-web-db-02 ~]$



 [neon@neon-web-db-03 ~]$ sudo cat /vol/data/mysql/grastate.dat

# GALERA saved state

version: 2.1

uuid:    1422a148-0f6c-11ea-bdad-5b132c2a2ccc

seqno:   5

safe_to_bootstrap: 0

[neon@neon-web-db-03 ~]$

```
 
 


## start again 
 
## start 1 server

sudo galera_new_cluster

##start 2 server
 
 sudo systemctl start mariadb  
 
## start 3 server
 
 sudo systemctl start mariadb  
 
 
