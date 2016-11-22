
Contents
1.	Introduction	2
2.	Scope	2
3.	Intendant audience	2
4.	High Level diagram	2
5.	Used tools	3
6.	Configuration and setup	3
6.1 Percona XtraDB Cluster Configurations:	3
6.2 Load Balancing (LB)	6
6.2.1 HaProxy Configuration:	7
6.2.2 Keepalived configuration	8

 
1.	Introduction
This document is giving high level configuration about Percona 5.7, 3 node cluster with High Availability.
2.	Scope
This document covers about Percona db cluster setup and configuration in centos.
3.	Intendant audience 
All configuration and setup is being done on Centos  6, Linux platform. So Database Administrator or System Administrator or Developer should have basic knowledge of Linux command, MySQL database information to follow this document.
4.	High Level diagram
 
5.	Used tools  
•	Percona 5.7 cluster
•	HaProxy
•	KeepAlived
•	Yum
6.	Configuration and setup
6.1 Percona XtraDB Cluster Configurations:
Percona is an open source software company specializing in MySQL, MongoDB, and other open source database support, consulting, managed services, and training. 
Percona XtraDB Cluster is an active/active high availability and high scalability open source solution for MySQL clustering. It integrates Percona Server and Percona XtraBackup with the Codership Galera library of MySQL high availability solutions in a single package that enables you to create a cost-effective MySQL high availability cluster.
We can found more information 
https://www.percona.com/software/mysql-database/percona-xtradb-cluster
Installation Documentation: https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/LATEST/
This document only consist configuration and setup latest version 5.7. It needs to be setup in each node. Based on above diagram, we are doing in 3 nodes. 3 node clusters is best practice for clustering.
•	Node 1: 192.168.2.4, Node 2: 192.168.2.5, Node 3: 192.168.2.6, are the used node ip here for identification.  We will be installing Percona cluster in each node.
•	Percona XtraDB Cluster does not work with SELinux and AppArmor security modules. Make sure these modules are disabled in each node
•	Cent os 6 is being used as Linux Platform. Some command and process may be different base on platform.
•	If platform already have MySQL-server, then it will conflict. We will remove it. 
#yum remove mysql mysql-server
•	We can install Percona  by yum repository, rpm or manual download.  Here we are using Yum repository. We may need to add Percona repository in platform yum repository. For further you can follow Percona’s  official documentation.  
 
•	If any OS library missing. We need to install library in OS before start installing Percona cluster. Like my case. I need to install libev  required by Percona itself. I followed in each node.
## RHEL/CentOS 6 32-Bit ##
# wget http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
# rpm -ivh epel-release-6-8.noarch.rpm
## RHEL/CentOS 6 64-Bit ##
# wget http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
# rpm -ivh epel-release-6-8.noarch.rpm
yum install libev
•	Then we can start installation in any node. You can pick 1st node (192.168.2.4)
# yum install Percona-XtraDB-Cluster-57
•	After installation complete. Follow following step
o	Individual nodes should be configured to be able to bootstrap the cluster. For more information about bootstrapping the cluster, see Bootstrapping the cluster.
o	Configuration of First node:
1.	Make sure that the configuration file /etc/my.cnf on the first node (Node 1) contains the following:
[mysqld]
datadir=/var/lib/mysql
user=mysql
# Path to Galera library
wsrep_provider=/usr/lib64/libgalera_smm.so
# Cluster connection URL contains the IPs of node#1, node#2 and node#3
wsrep_cluster_address=gcomm:// 192.168.2.4, 192.168.2.5, 192.168.2.6
# In order for Galera to work correctly binlog format should be ROW
binlog_format=ROW
# MyISAM storage engine has only experimental support
default_storage_engine=InnoDB
# This InnoDB autoincrement locking mode is a requirement for Galera
innodb_autoinc_lock_mode=2
# Node 1 address
wsrep_node_address=192.168.2.4
# SST method
wsrep_sst_method=xtrabackup-v2
# Cluster name
wsrep_cluster_name=my_centos_cluster
# Authentication for SST method
wsrep_sst_auth="sstuser:s3cret"
2.	2. Start the first node with the following command:
[root@node1 ~]# /etc/init.d/mysql bootstrap-pxc
3.	After running step 2 command, default root password will be displayed at screen. That password should be used to loing mysql using root user. For more information 
https://www.percona.com/blog/2016/05/18/where-is-the-mysql-5-7-root-password/
 # mysql -u root –p
Enter password:<hit default password>

4.	after first time login. mysql 5.7 force to reset root password:
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'P@ssw0rd';
mysql> flush privileges;
5.	After the first node has been started, cluster status can be checked with the following command:
mysql> show status like ’wsrep%’;
6.	To perform State Snapshot Transfer using XtraBackup, set up a new user with proper privileges:
mysql@Node1> CREATE USER ’sstuser’@’localhost’ IDENTIFIED BY ’s3cret’;
mysql@Node1> GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO ’sstuser’@’localhost’;
mysql@Node1> FLUSH PRIVILEGES;
o	Configuration of Second node:
1.	Make sure that the configuration file /etc/my.cnf on the first node (Node 2) contains the following:
[mysqld]
datadir=/var/lib/mysql
user=mysql
# Path to Galera library
wsrep_provider=/usr/lib64/libgalera_smm.so
# Cluster connection URL contains the IPs of node#1, node#2 and node#3
wsrep_cluster_address=gcomm:// 192.168.2.4, 192.168.2.5, 192.168.2.6
# In order for Galera to work correctly binlog format should be ROW
binlog_format=ROW
# MyISAM storage engine has only experimental support
default_storage_engine=InnoDB
# This InnoDB autoincrement locking mode is a requirement for Galera
innodb_autoinc_lock_mode=2
# Node 1 address
wsrep_node_address=192.168.2.5
# SST method
wsrep_sst_method=xtrabackup-v2
# Cluster name
wsrep_cluster_name=my_centos_cluster
# Authentication for SST method
wsrep_sst_auth="sstuser:s3cret"
2.	Start the second node with the following command
[root@node2 ~]# /etc/init.d/mysql start
3.	Check node status
mysql> show status like ’wsrep%’;
 
o	Configuration of Third node:
1.	Make sure that the configuration file /etc/my.cnf on the first node (Node 2) contains the following:
[mysqld]
datadir=/var/lib/mysql
user=mysql
# Path to Galera library
wsrep_provider=/usr/lib64/libgalera_smm.so
# Cluster connection URL contains the IPs of node#1, node#2 and node#3
wsrep_cluster_address=gcomm:// 192.168.2.4, 192.168.2.5, 192.168.2.6
# In order for Galera to work correctly binlog format should be ROW
binlog_format=ROW
# MyISAM storage engine has only experimental support
default_storage_engine=InnoDB
# This InnoDB autoincrement locking mode is a requirement for Galera
innodb_autoinc_lock_mode=2
# Node 1 address
wsrep_node_address=192.168.2.6
# SST method
wsrep_sst_method=xtrabackup-v2
# Cluster name
wsrep_cluster_name=my_centos_cluster
# Authentication for SST method
wsrep_sst_auth="sstuser:s3cret"
2.	Start the third node with the following command
[root@node2 ~]# /etc/init.d/mysql start
3.	Check node status
mysql> show status like ’wsrep%’;
•	After configuring replication can be tested using any DML operation in one Node and checking in another node. If everything ok then all node synching together.

6.2 Load Balancing (LB)
For load balancing, here we are using 2 load balancing server with one virtual ip as Proxy server for Application to DB connection. For HA concept, Application will be connection DB through proxy IP. During connection time one of the LB will be chosen during connection time. This will be managed by Keepalived. If one LB is down then automatically another LB will be used. So we can make 2 LB server in two different type of network. As a result application can connect DB server if one side of network down.
Some programming language support itself supports load blancing during connection time. For example, java has its own connection pool mechanism where can directly give node ip in application configuration.
 
6.2.1 HaProxy Configuration:

HAProxy is a free, very fast and reliable solution offering high availability, load balancing, and proxying for TCP and HTTP-based applications. It is particularly suited for very high traffic web sites and powers quite a number of the world's most visited ones. Over the years it has become the de-facto standard opensource load balancer, is now shipped with most mainstream Linux distributions, and is often deployed by default in cloud platforms
•	We will be installing Haproxy in both LB server. #yum install haproxy
•	After installation we need to add entry in its configuration file(/etc/haproxy/haproxy.cfg) based on our requirement
•	Sample haproxy.cfg will be available in this document folder.
•	Mainly we need to add entry about request(connection request, http request) which to be load balanced and sever entries with their connectivity strategy.,
 for example:
server NODE01 192.168.2.4:3306 check
server NODE02 192.168.2.5:3306 check backup
server NODE03 192.168.2.6:3306 check backup
o	Here all first loads goes to NODE1 because it is configured as check and rest NODEs are configure as check with backup purpose. This parameter might need to be configured based on requirement.
•	We can monitor all request status and NODE status through browsing Proxy IP or each LB IP.  For this, We need to add following configuration in haproxy.cfg
listen statistics *:8080
   mode http
 stats enable
#    stats hide-version
    stats realm Haproxy\ Statistics    stats realm .
    stats uri /
    stats auth admin:P@ssw0rd
    stats refresh 5s
 
•	After this configuration ,we can monitor http:// 192.168.2.1:8080
 
6.2.2 Keepalived configuration
•	Keepalived tool uses to select one LB at a time. This tool will be installed in both LB server .i.e. 192.168.2.2 and 192.168.2.3 are LB servers based on above diagram. Virtual IP will be added in keepalived.conf configuration file. 
•	Installation of Keeplived in each LB server.
# yum install keepalived
•	Edit the Keepalived configuration file based on requirement.  /etc/keepalived/keepalived.conf	
•	For more information at https://docs.oracle.com/cd/E37670_01/E41138/html/section_uxg_lzh_nr.html
•	Sample keepalived.conf	 will be in this document folder.
To create auto startup during OS loading time :
#chkconfig keepalived on
#chkconfig haproxy on




