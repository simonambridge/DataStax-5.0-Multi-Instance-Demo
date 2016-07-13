#DSE 5.0 Multi-Instance Demo#

The new Multi-Instance feature relased with DSE 5.0 allows the simple deployment of multiple DSE instances on a single machine.

This helps customer ensure that large hardware resources are effectively utilized for database applications.
Making more efficient use of hardware Helps reduce hardware cost.

##Build a DSE Node##
I thoroughly recommend you use Joel Jacobsone's latest vagrant build that provisions a single larger node designed to support multiple instances of DataStax Enterprise.

Joel's Multi-Instance host repo is here (along with a great multi-node vagrant build if you want to a three node cluster with opscentre)

After you've built the vagrant virtual box, if it isn't running, start it with:

```
vagrant up dse-node
```

##Configure The VM for DSE##

Now log into the DSE node:
```
vagrant ssh dse-node
```

You first need to update the ulimits so that your vargrant user can start DSE.
There is a separate conf file to set ulimits for the cassandra user
```
sudo vi /etc/security/limits.d/cassandra.conf
```
Add up update the entries in there to look like this:
```
vagrant - memlock unlimited
vagrant - nofile 100000
vagrant - nproc 32768
vagrant - as unlimited
```
Now log out and ssh back into the box as the vagrant user in order to pick up the new ulimits.

Back in the vagrant VM, we need to create some IP aliases for our new instances - use the ifconfig command:
```
ifconfig lo:0 127.0.0.2 netmask 255.0.0.0 up
ifconfig lo:1 127.0.0.3 netmask 255.0.0.0 up
ifconfig lo:2 127.0.0.4 netmask 255.0.0.0 up
```

##Add Node 1##

###Use ````add-node``` to add a multi-instance node

Use the add-node command. You can call your node what you like but it will be prefixed autoatically with ```dse-```. So ```node1``` will be ```dse-node1``` etc.

We don't have a valid seed address here as we're the first node.

> NB Do not attempt to add a node to the default dse node service that gets created when you install DSE. It wont let you do it even if you use it as the seed address with the same cluster name (you will end up with two clusters each called Cassandra but on different ports). Shut it down and forget it ever existed.

So, now we know that:
*1We only need the installation of DSE to give us the binaries, config and filiesystem layout. We don't need the service that is pre-configured for a stand-alone instance.
*2Multi-instance nodes seem to only want to talk to other multi-instance nodes.

Lets add the first node:
```
sudo dse add-node --node-id node1 --cluster Cassandra --listen-address 127.0.0.2 --rpc-address 127.0.0.2 --seeds 127.0.0.2
Installing and configuring from /usr/share/dse/templates/

+ Setting up node dse-node1...
  - Copying configs
      - Setting the cluster name.
      - Setting up JMX port
      - Setting up directories
Warning: Spark shuffle service port not set. Spark nodes will use the default binding options.
Done.
```

The template filesystem structure for this new node will now have been created. You'll find configuration files in:

```
sudo ls /etc/dse-node1
byoh	     cassandra	 dse-node1.init  dse.yaml  hadoop	   hive    pig	 spark	tomcat
byoh-env.sh  dse-env.sh  dserc-env.sh	 graph	   hadoop2-client  mahout  solr  sqoop
```

Here you set whether you want it to be a Cassandra, Spark, Solr node etc:
> ignore it if you just want Cassandra

```
sudo cat /etc/default/dse-node1
```
Data lives here:
```
sudo ls /var/lib/dse-node1
commitlog  data  hints	saved_caches  spark
```

Logs live here:
```
sudo ls /var/log/dse-node1
audit  debug.log  gremlin.log  output.log  spark  system.log
```
Although add-node told you that it was setting up the JMX port, I found that it uses 7199 each time it runs (e.g. it doesnt appear to check if the port is in use) so you need to set the port yourself. 

I set the port on the first node just so that it doesnt conflict with the port used by the default dse service that I might want to start one day.

###Set the Node 1 JMX port ```JMX_PORT="7299"```###
```
sudo vi /etc/dse-node1/cassandra/cassandra-env.sh
```
###Start The Node 1 Service###
```
sudo service dse-node1 start
 * Starting DSE daemons (dse-node1) dse-node1
DSE daemons (dse-node1) starting with only Cassandra enabled (edit /etc/default/dse-node1 to enable other features)
```
Check the log file to make sure everything's rosy:
```
sudo tail -100 /var/log/dse-node1/system.log
```
If all goes to plan you should see:
```
INFO  [main] 2016-07-13 05:46:18,137  ThriftServer.java:119 - Binding thrift service to /127.0.0.2:9160
INFO  [Thread-3] 2016-07-13 05:46:18,145  ThriftServer.java:136 - Listening for thrift clients...
INFO  [main] 2016-07-13 05:46:18,145  DseDaemon.java:827 - DSE startup complete.
```

We can check the cluster with the new multi-instance support - use the node name:
```
sudo dse *dse-node1* dsetool ring
Address          DC                   Rack         Workload             Graph  Status  State    Load             Owns                 Token                                        Health [0,1]
127.0.0.2        Cassandra            rack1        Cassandra            no     Up      Normal   133.53 KB        ?                    -9098225573757054999                         0.00
```

We can access via cqlsh:
```
$ cqlsh 127.0.0.2
Connected to Cassandra at 127.0.0.2:9042.
[cqlsh 5.0.1 | Cassandra 3.0.7.1159 | DSE 5.0.1 | CQL spec 3.4.0 | Native protocol v4]
Use HELP for help.
cqlsh> exit
```

##Add Node 2##

Same command as before, same cluster name, this time with a different virtual IP address.
> We  point to the first node that we created to use as a seed node

```
sudo dse add-node --node-id node2 --cluster Cassandra --listen-address 127.0.0.3 --rpc-address 127.0.0.3 --seeds 127.0.0.2
Installing and configuring from /usr/share/dse/templates/

+ Setting up node dse-node2...
  - Copying configs
      - Setting the cluster name.
      - Setting up JMX port
      - Setting up directories
Warning: Spark shuffle service port not set. Spark nodes will use the default binding options.
Done.
```

###Set the Node 2 JMX port ```JMX_PORT="7399"```###
```
sudo vi /etc/dse-node2/cassandra/cassandra-env.sh
```

###Start The Node 2 Service###

```
sudo service dse-node2 start
 * Starting DSE daemons (dse-node2) dse-node2
DSE daemons (dse-node2) starting with only Cassandra enabled (edit /etc/default/dse-node2 to enable other features)
```

Check it:
```
sudo tail -100 /var/log/dse-node2/system.log
```

How does my cluster look?
```
sudo dse dse-node1 dsetool ring
Server ID          Address          DC                   Rack         Workload             Graph  Status  State    Load             Owns                 Token                                        Health [0,1]
                                                                                                                                                         -9024954150119957189
08-00-27-88-0C-A6  127.0.0.2        Cassandra            rack1        Cassandra            no     Up      Normal   133.53 KB        ?                    -9098225573757054999                         0.10
08-00-27-88-0C-A6  127.0.0.3        Cassandra            rack1        Cassandra            no     Up      Normal   116.85 KB        ?                    -9024954150119957189                         0.10
```


##Add Node 3##

Now we're going to create a new multi-instance managed node, but this time in a different cluster.

Same command as before, different cluster name, different virtual IP address, and we don't have a seed address.
> We  point to the first node that we created to use as a seed node

```
sudo dse add-node --node-id node3 --cluster NewCluster --listen-address 127.0.0.4 --rpc-address 127.0.0.4 --seeds 127.0.0.4
Installing and configuring from /usr/share/dse/templates/

+ Setting up node dse-node3...
  - Copying configs
      - Setting the cluster name.
      - Setting up JMX port
      - Setting up directories
Warning: Spark shuffle service port not set. Spark nodes will use the default binding options.
Done.
```

Set the jmx port JMX_PORT="7499"
```
sudo vi /etc/dse-node3/cassandra/cassandra-env.sh
```

Check your different cluster name ```cluster_name: NewCluster```
```
sudo vi /etc/dse-node3/cassandra/cassandra.yaml
```

###Start The Node 3 Service###
```
sudo service dse-node3 start
 * Starting DSE daemons (dse-node3) dse-node3
DSE daemons (dse-node3) starting with only Cassandra enabled (edit /etc/default/dse-node3 to enable other features)
```

And again:
```
sudo tail -100 /var/log/dse-node3/system.log
```
We want to see:
```
INFO  [main] 2016-07-13 13:07:29,996  ThriftServer.java:119 - Binding thrift service to /127.0.0.4:9160
INFO  [Thread-3] 2016-07-13 13:07:30,005  ThriftServer.java:136 - Listening for thrift clients...
INFO  [main] 2016-07-13 13:07:30,005  DseDaemon.java:827 - DSE startup complete.
```

We can look at the first cluster we created:
```
sudo dse dse-node2 dsetool ring
Server ID          Address          DC                   Rack         Workload             Graph  Status  State    Load             Owns                 Token                                        Health [0,1]
                                                                                                                                                         -9024954150119957189
08-00-27-88-0C-A6  127.0.0.2        Cassandra            rack1        Cassandra            no     Up      Normal   133.53 KB        ?                    -9098225573757054999                         0.20
08-00-27-88-0C-A6  127.0.0.3        Cassandra            rack1        Cassandra            no     Up      Normal   116.99 KB        ?                    -9024954150119957189                         0.20
Note: you must specify a keyspace to get ownership information.
```

We can look at the second cluster:
```
sudo dse dse-node3 dsetool ring
Address          DC                   Rack         Workload             Graph  Status  State    Load             Owns                 Token                                        Health [0,1]
127.0.0.4        Cassandra            rack1        Cassandra            no     Up      Normal   88.15 KB         ?                    -2000014393878253047                         0.00
Note: you must specify a keyspace to get ownership information.
```




##Things go wrong sometimes - if you need to remove a node....##

Use the ```remove-node``` tool:
```
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
```
Manual commands:
```
sudo rm -rf /var/lib/dse-node3
sudo rm -rf /var/log/dse-node3
```
