<p><strong>&nbsp;</strong></p>
<p>Contents</p>
<ol>
<li><a href="#_Toc467586650"> Introduction. 2</a></li>
<li><a href="#_Toc467586651"> Scope. 2</a></li>
<li><a href="#_Toc467586652"> Intendant audience. 2</a></li>
<li><a href="#_Toc467586653"> High Level diagram.. 2</a></li>
<li><a href="#_Toc467586654"> Used tools. 3</a></li>
<li><a href="#_Toc467586655"> Configuration and setup. 3</a></li>
</ol>
<p><a href="#_Toc467586656">6.1 Percona XtraDB Cluster Configurations: 3</a></p>
<p><a href="#_Toc467586657">6.2 Load Balancing (LB). 6</a></p>
<p><a href="#_Toc467586658">6.2.1 HaProxy Configuration: 7</a></p>
<p><a href="#_Toc467586659">6.2.2 Keepalived configuration. 8</a></p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p><strong>&nbsp;</strong></p>
<h1><a name="_Toc467586650"></a>1.&nbsp;&nbsp;&nbsp;&nbsp; Introduction</h1>
<p>This document is giving high level configuration about Percona 5.7, 3 node cluster with High Availability.</p>
<h1><a name="_Toc467586651"></a>2.&nbsp;&nbsp;&nbsp;&nbsp; Scope</h1>
<p>This document covers about Percona db cluster setup and configuration in centos.</p>
<h1><a name="_Toc467586652"></a>3.&nbsp;&nbsp;&nbsp;&nbsp; Intendant audience</h1>
<p>All configuration and setup is being done on Centos&nbsp; 6, Linux platform. So Database Administrator or System Administrator or Developer should have basic knowledge of Linux command, MySQL database information to follow this document.</p>
<h1><a name="_Toc467586653"></a>4.&nbsp;&nbsp;&nbsp;&nbsp; High Level diagram</h1>
 <img src="High Level Diagram.png" alt="High Level Diagram" height="761px" width="848px"> 
  
<h1><a name="_Toc467586654"></a>5.&nbsp;&nbsp;&nbsp;&nbsp; Used tools &nbsp;</h1>
<ul>
<li>Percona 5.7 cluster</li>
<li>HaProxy</li>
<li>KeepAlived</li>
<li>Yum</li>
</ul>
<h1><a name="_Toc467586655"></a>6.&nbsp;&nbsp;&nbsp;&nbsp; Configuration and setup</h1>
<h2><a name="_Toc467586656"></a>6.1 Percona XtraDB Cluster Configurations:</h2>
<p>Percona is an open source software company specializing in MySQL, MongoDB, and other open source database support, consulting, managed services, and training.</p>
<p>Percona XtraDB Cluster is an active/active high availability and high scalability open source solution for MySQL clustering. It integrates Percona Server and Percona XtraBackup with the Codership Galera library of MySQL high availability solutions in a single package that enables you to create a cost-effective MySQL high availability cluster.</p>
<p>We can found more information</p>
<p><a href="https://www.percona.com/software/mysql-database/percona-xtradb-cluster">https://www.percona.com/software/mysql-database/percona-xtradb-cluster</a></p>
<p><strong>Installation Documentation</strong>: <a href="https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/LATEST/">https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/LATEST/</a></p>
<p>This document only consist configuration and setup latest version 5.7. It needs to be setup in each node. Based on above diagram, we are doing in 3 nodes. 3 node clusters is best practice for clustering.</p>
<ul>
<li><strong>Node 1: 192.168.2.4, Node 2: 192.168.2.5, Node 3: 192.168.2.6, </strong>are the used node ip here for identification<strong>. </strong>We will be installing Percona cluster in each node.</li>
<li>Percona XtraDB Cluster does not work with <strong><em>SELinux</em></strong> and <strong><em>AppArmor</em></strong> security modules. Make sure these modules are disabled in each node</li>
<li>Cent os 6 is being used as Linux Platform. Some command and process may be different base on platform.</li>
<li>If platform already have MySQL-server, then it will conflict. We will remove it. <br /> <strong>#yum remove mysql mysql-server</strong></li>
<li>We can install Percona by yum repository, rpm or manual download.&nbsp; Here we are using Yum repository. We may need to add Percona repository in platform yum repository. For further you can follow Percona&rsquo;s&nbsp; official <a href="https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/LATEST/">documentation</a>. &nbsp;</li>
</ul>
<p>&nbsp;</p>
<p>&nbsp;</p>
<ul>
<li>If any OS library missing. We need to install library in OS before start installing Percona cluster. Like my case. I need to install <strong>libev </strong>required by Percona itself. I followed in each node.</li>
</ul>
<p>## RHEL/CentOS 6 32-Bit ##</p>
<p># wget http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm</p>
<p># rpm -ivh epel-release-6-8.noarch.rpm</p>
<p>## RHEL/CentOS 6 64-Bit ##</p>
<p># wget http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm</p>
<p># rpm -ivh epel-release-6-8.noarch.rpm</p>
<p>yum install libev</p>
<ul>
<li>Then we can start installation in any node. You can pick 1<sup>st</sup> node (<strong>168.2.4</strong>)</li>
</ul>
<p># yum install Percona-XtraDB-Cluster-57</p>
<ul>
<li>After installation complete. Follow following step
<ul>
<li>Individual nodes should be configured to be able to bootstrap the cluster. For more information about bootstrapping the cluster, see Bootstrapping the cluster.</li>
<li><strong>Configuration of First node:</strong></li>
</ul>
</li>
</ul>
<ol>
<li>Make sure that the configuration file /etc/my.cnf on the first node (Node 1) contains the following:<br /> <em>[mysqld]</em></li>
</ol>
<p><em>datadir=/var/lib/mysql</em></p>
<p><em>user=mysql</em></p>
<p><em># Path to Galera library</em></p>
<p><em>wsrep_provider=/usr/lib64/libgalera_smm.so</em></p>
<p><em># Cluster connection URL contains the IPs of node#1, node#2 and node#3</em></p>
<p><em>wsrep_cluster_address=gcomm://</em><strong> 192.168.2.4, 192.168.2.5, 192.168.2.6<br /> </strong><em># In order for Galera to work correctly binlog format should be ROW</em></p>
<p><em>binlog_format=ROW</em></p>
<p><em># MyISAM storage engine has only experimental support</em></p>
<p><em>default_storage_engine=InnoDB</em></p>
<p><em># This InnoDB autoincrement locking mode is a requirement for Galera</em></p>
<p><em>innodb_autoinc_lock_mode=2</em></p>
<p><em># Node 1 address</em></p>
<p><em>wsrep_node_address=</em><strong>192.168.2.4</strong></p>
<p><em># SST method</em></p>
<p><em>wsrep_sst_method=xtrabackup-v2</em></p>
<p><em># Cluster name</em></p>
<p><em>wsrep_cluster_name=my_centos_cluster</em></p>
<p><em># Authentication for SST method</em></p>
<p><em>wsrep_sst_auth="sstuser:s3cret"</em></p>
<ol start="2">
<li><em> Start the first node with the following command:<br /> [root@node1 ~]# /etc/init.d/mysql bootstrap-pxc</em></li>
<li><em>After running step 2 command, default root password will be displayed at screen. That password should be used to loing mysql using root user. For more information <br /> </em><a href="https://www.percona.com/blog/2016/05/18/where-is-the-mysql-5-7-root-password/"><em>https://www.percona.com/blog/2016/05/18/where-is-the-mysql-5-7-root-password/</em></a><em><br /> # mysql -u root &ndash;p<br /> Enter password:&lt;hit default password&gt;<br /> <br /> </em></li>
<li><em>after first time login. mysql 5.7 force to reset root password:<br /> mysql&gt; ALTER USER 'root'@'localhost' IDENTIFIED BY 'P@ssw0rd';<br /> mysql&gt; flush privileges;</em></li>
<li><em>After the first node has been started, cluster status can be checked with the following command:<br /> mysql&gt; show status like &rsquo;wsrep%&rsquo;;</em></li>
<li>To perform State Snapshot Transfer using XtraBackup, set up a new user with proper privileges:<br /> <em>mysql@Node1&gt; CREATE USER &rsquo;sstuser&rsquo;@&rsquo;localhost&rsquo; IDENTIFIED BY &rsquo;s3cret&rsquo;;<br /> mysql@Node1&gt; GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO &rsquo;sstuser&rsquo;@&rsquo;localhost&rsquo;;<br /> mysql@Node1&gt; FLUSH PRIVILEGES;</em></li>
</ol>
<ul>
<li><strong>Configuration of Second node:</strong></li>
</ul>
<ol>
<li>Make sure that the configuration file /etc/my.cnf on the first node (Node 2) contains the following:</li>
</ol>
<p><em>[mysqld]</em></p>
<p><em>datadir=/var/lib/mysql</em></p>
<p><em>user=mysql</em></p>
<p><em># Path to Galera library</em></p>
<p><em>wsrep_provider=/usr/lib64/libgalera_smm.so</em></p>
<p><em># Cluster connection URL contains the IPs of node#1, node#2 and node#3</em></p>
<p><em>wsrep_cluster_address=gcomm://</em><strong> 192.168.2.4, 192.168.2.5, 192.168.2.6<br /> </strong><em># In order for Galera to work correctly binlog format should be ROW</em></p>
<p><em>binlog_format=ROW</em></p>
<p><em># MyISAM storage engine has only experimental support</em></p>
<p><em>default_storage_engine=InnoDB</em></p>
<p><em># This InnoDB autoincrement locking mode is a requirement for Galera</em></p>
<p><em>innodb_autoinc_lock_mode=2</em></p>
<p><em># Node 1 address</em></p>
<p><em>wsrep_node_address=</em><strong>192.168.2.5</strong></p>
<p><em># SST method</em></p>
<p><em>wsrep_sst_method=xtrabackup-v2</em></p>
<p><em># Cluster name</em></p>
<p><em>wsrep_cluster_name=my_centos_cluster</em></p>
<p><em># Authentication for SST method</em></p>
<p><em>wsrep_sst_auth="sstuser:s3cret"</em></p>
<ol start="2">
<li><em>Start the second node with the following command</em></li>
</ol>
<p>[root@node2 ~]# /etc/init.d/mysql start</p>
<ol start="3">
<li><em>Check node status<br /> </em>mysql&gt; <strong>show </strong>status <strong>like </strong>&rsquo;wsrep%&rsquo;;</li>
</ol>
<p><strong><br /> </strong></p>
<ul>
<li><strong>Configuration of Third node:</strong></li>
</ul>
<ol>
<li>Make sure that the configuration file /etc/my.cnf on the first node (Node 2) contains the following:</li>
</ol>
<p><em>[mysqld]</em></p>
<p><em>datadir=/var/lib/mysql</em></p>
<p><em>user=mysql</em></p>
<p><em># Path to Galera library</em></p>
<p><em>wsrep_provider=/usr/lib64/libgalera_smm.so</em></p>
<p><em># Cluster connection URL contains the IPs of node#1, node#2 and node#3</em></p>
<p><em>wsrep_cluster_address=gcomm://</em><strong> 192.168.2.4, 192.168.2.5, 192.168.2.6<br /> </strong><em># In order for Galera to work correctly binlog format should be ROW</em></p>
<p><em>binlog_format=ROW</em></p>
<p><em># MyISAM storage engine has only experimental support</em></p>
<p><em>default_storage_engine=InnoDB</em></p>
<p><em># This InnoDB autoincrement locking mode is a requirement for Galera</em></p>
<p><em>innodb_autoinc_lock_mode=2</em></p>
<p><em># Node 1 address</em></p>
<p><em>wsrep_node_address=</em><strong>192.168.2.6</strong></p>
<p><em># SST method</em></p>
<p><em>wsrep_sst_method=xtrabackup-v2</em></p>
<p><em># Cluster name</em></p>
<p><em>wsrep_cluster_name=my_centos_cluster</em></p>
<p><em># Authentication for SST method</em></p>
<p><em>wsrep_sst_auth="sstuser:s3cret"</em></p>
<ol start="2">
<li><em>Start the third node with the following command</em></li>
</ol>
<p>[root@node2 ~]# /etc/init.d/mysql start</p>
<ol start="3">
<li><em>Check node status<br /> </em>mysql&gt; <strong>show </strong>status <strong>like </strong>&rsquo;wsrep%&rsquo;;</li>
</ol>
<ul>
<li>After configuring replication can be tested using any DML operation in one Node and checking in another node. If everything ok then all node synching together.</li>
</ul>
<p><em>&nbsp;</em></p>
<h2><a name="_Toc467586657"></a>6.2 Load Balancing (LB)</h2>
<p>For load balancing, here we are using 2 load balancing server with one virtual ip as Proxy server for Application to DB connection. For HA concept, Application will be connection DB through proxy IP. During connection time one of the LB will be chosen during connection time. This will be managed by Keepalived. If one LB is down then automatically another LB will be used. So we can make 2 LB server in two different type of network. As a result application can connect DB server if one side of network down.</p>
<p>Some programming language support itself supports load blancing during connection time. For example, java has its own connection pool mechanism where can directly give node ip in application configuration.</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<h3><a name="_Toc467586658"></a>6.2.1 HaProxy Configuration:</h3>
<p><br /> HAProxy is a free, very fast and reliable solution offering high availability, load balancing, and proxying for TCP and HTTP-based applications. It is particularly suited for very high traffic web sites and powers quite a number of the world's most visited ones. Over the years it has become the de-facto standard opensource load balancer, is now shipped with most mainstream Linux distributions, and is often deployed by default in cloud platforms</p>
<ul>
<li>We will be installing Haproxy in both LB server. <strong><em>#yum install haproxy</em></strong></li>
<li>After installation we need to add entry in its configuration file(<strong>/etc/haproxy/haproxy.cfg</strong>) based on our requirement</li>
<li>Sample haproxy.cfg will be available in this document folder.</li>
<li>Mainly we need to add entry about request(connection request, http request) which to be load balanced and sever entries with their connectivity strategy.<strong>,<br /> for example:</strong><br /> server NODE01 <strong>168.2.4</strong>:3306 check<br /> server NODE02 <strong>192.168.2.5</strong>:3306 check backup<br /> server NODE03 <strong>192.168.2.6</strong>:3306 check backup
<ul>
<li>Here all first loads goes to NODE1 because it is configured as check and rest NODEs are configure as check with backup purpose. This parameter might need to be configured based on requirement.</li>
</ul>
</li>
<li>We can monitor all request status and NODE status through browsing Proxy IP or each LB IP. For this, We need to add following configuration in <strong>cfg<br /> </strong>listen statistics *:8080</li>
</ul>
<p>&nbsp;&nbsp; mode http<br /> &nbsp;stats enable<br /> #&nbsp;&nbsp;&nbsp; stats hide-version<br /> &nbsp;&nbsp;&nbsp; stats realm Haproxy\ Statistics&nbsp;&nbsp;&nbsp; stats realm .<br /> &nbsp;&nbsp;&nbsp; stats uri /<br /> &nbsp;&nbsp;&nbsp; stats auth admin:P@ssw0rd<br /> &nbsp;&nbsp;&nbsp; stats refresh 5s</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<ul>
<li>After this configuration ,we can monitor http:// 192.168.2.1:8080</li>
</ul>
 <img src="Haproxy.png" alt="haproxy" height="761px" width="848px"> 
<h3><a name="_Toc467586659"></a>6.2.2 Keepalived configuration</h3>
<ul>
<li>Keepalived tool uses to select one LB at a time. This tool will be installed in both LB server .i.e. 192.168.2.2 and 192.168.2.3 are LB servers based on above diagram. Virtual IP will be added in keepalived.conf configuration file.</li>
<li>Installation of Keeplived in each LB server.<br /> # yum install keepalived</li>
<li>Edit the Keepalived configuration file based on requirement. /etc/keepalived/keepalived.conf&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</li>
<li>For more information at <a href="https://docs.oracle.com/cd/E37670_01/E41138/html/section_uxg_lzh_nr.html">https://docs.oracle.com/cd/E37670_01/E41138/html/section_uxg_lzh_nr.html</a></li>
<li>Sample keepalived.conf will be in this document folder.</li>
</ul>
<p>To create auto startup during OS loading time :<br /> #chkconfig keepalived on<br /> #chkconfig haproxy on</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
