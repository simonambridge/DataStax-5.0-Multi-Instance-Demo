vagrant ssh dse-node

set ulimits for the cassandra user
sudo vi /etc/security/limits.d/cassandra.conf

vagrant - memlock unlimited
vagrant - nofile 100000
vagrant - nproc 32768
vagrant - as unlimited

set IP aliases
ifconfig lo:0 127.0.0.2 netmask 255.0.0.0 up
ifconfig lo:1 127.0.0.3 netmask 255.0.0.0 up
ifconfig lo:2 127.0.0.4 netmask 255.0.0.0 up


sudo dse add-node --node-id node1 --cluster Cassandra --listen-address 127.0.0.2 --rpc-address 127.0.0.2 --seeds 127.0.0.2

Set the jmx port JMX_PORT="7299"
sudo vi /etc/dse-node1/cassandra/cassandra-env.sh

sudo service dse-node1 start
 * Starting DSE daemons (dse-node1) dse-node1
DSE daemons (dse-node1) starting with only Cassandra enabled (edit /etc/default/dse-node1 to enable other features)

sudo tail -100 /var/log/dse-node1/system.log


nodetool status
Datacenter: Cassandra
=====================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Owns    Host ID                               Token                                    Rack
UN  127.0.0.1  137.68 KB  ?       44c0bffe-9654-49fb-9790-b1c854b4cbb2  -8369723004072756799                     rack1


sudo dse dse-node1 dsetool ring
Address          DC                   Rack         Workload             Graph  Status  State    Load             Owns                 Token                                        Health [0,1]
127.0.0.2        Cassandra            rack1        Cassandra            no     Up      Normal   133.53 KB        ?                    -9098225573757054999                         0.00


$ cqlsh 127.0.0.2
Connected to Cassandra at 127.0.0.2:9042.
[cqlsh 5.0.1 | Cassandra 3.0.7.1159 | DSE 5.0.1 | CQL spec 3.4.0 | Native protocol v4]
Use HELP for help.
cqlsh> exit



sudo dse add-node --node-id node2 --cluster Cassandra --listen-address 127.0.0.3 --rpc-address 127.0.0.3 --seeds 127.0.0.2
Installing and configuring from /usr/share/dse/templates/

+ Setting up node dse-node2...
  - Copying configs
      - Setting the cluster name.
      - Setting up JMX port
      - Setting up directories
Warning: Spark shuffle service port not set. Spark nodes will use the default binding options.
Done.

set the jmx port JMX_PORT="7399"
sudo vi /etc/dse-node2/cassandra/cassandra-env.sh

sudo service dse-node2 start
 * Starting DSE daemons (dse-node2) dse-node2
DSE daemons (dse-node2) starting with only Cassandra enabled (edit /etc/default/dse-node2 to enable other features)

sudo tail -100 /var/log/dse-node2/system.log

sudo dse dse-node1 dsetool ring
Server ID          Address          DC                   Rack         Workload             Graph  Status  State    Load             Owns                 Token                                        Health [0,1]
                                                                                                                                                         -9024954150119957189
08-00-27-88-0C-A6  127.0.0.2        Cassandra            rack1        Cassandra            no     Up      Normal   133.53 KB        ?                    -9098225573757054999                         0.10
08-00-27-88-0C-A6  127.0.0.3        Cassandra            rack1        Cassandra            no     Up      Normal   116.85 KB        ?                    -9024954150119957189                         0.10



sudo dse add-node --node-id node3 --cluster Cassandra2 --listen-address 127.0.0.4 --rpc-address 127.0.0.4 --seeds 127.0.0.4
Installing and configuring from /usr/share/dse/templates/

+ Setting up node dse-node3...
  - Copying configs
      - Setting the cluster name.
      - Setting up JMX port
      - Setting up directories
Warning: Spark shuffle service port not set. Spark nodes will use the default binding options.
Done.

set the jmx port JMX_PORT="7499"
$ sudo vi /etc/dse-node3/cassandra/cassandra-env.sh


sudo service dse-node2 start
 * Starting DSE daemons (dse-node3) dse-node3
DSE daemons (dse-node3) starting with only Cassandra enabled (edit /etc/default/dse-node3 to enable other features)

sudo tail -100 /var/log/dse-node3/system.log


sudo dse dse-node2 dsetool ring
Server ID          Address          DC                   Rack         Workload             Graph  Status  State    Load             Owns                 Token                                        Health [0,1]
                                                                                                                                                         -9024954150119957189
08-00-27-88-0C-A6  127.0.0.2        Cassandra            rack1        Cassandra            no     Up      Normal   133.53 KB        ?                    -9098225573757054999                         0.20
08-00-27-88-0C-A6  127.0.0.3        Cassandra            rack1        Cassandra            no     Up      Normal   116.99 KB        ?                    -9024954150119957189                         0.20
Note: you must specify a keyspace to get ownership information.

sudo dse dse-node3 dsetool ring
Address          DC                   Rack         Workload             Graph  Status  State    Load             Owns                 Token                                        Health [0,1]
127.0.0.4        Cassandra            rack1        Cassandra            no     Up      Normal   88.15 KB         ?                    -6577610989629133512                         0.00
Note: you must specify a keyspace to get ownership information.






$ sudo dse remove-node node1
##############################
#
# WARNING
# You're trying to remove node dse-node1
# This means that all configuration files for dse-node1 will be deleted
#
##############################

Do you wish to continue?
1) Yes
2) No
#? Yes
#? 1
 * Stopping DSE daemons (dse-node1) dse-node1                                                                                                                                               [ OK ]
Deleting /etc/dse/serverconfig/dse-node1
Deleting /etc/dse-node1
Deleting /etc/init.d/dse-node1
Deleting /etc/default/dse-node1
Deleting /var/run/dse-node1

