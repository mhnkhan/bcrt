---
### Tracking learning Redis
---

## Redis
###### Mahdi Khan

<details>
<summery>Problem — Redis on Windows is not officially suppoerted.</summery>

  Redis does not provide distro for Windows. :-(

  Port of old version is available but that is too old. WSL2 too is now “the way”! But for convenience Memurai which is compatible with Redis v5 will be used temporarily.

  উইন্ডোজের জন্য রেডিস চালানো যায় তিন ভাবে—
  - পুরাতন পোর্ট (কম্পাটিবল, দ্রুত, অনেক ফিচার নাই)
    - নতুন নামে পোর্ট—মেমুরাই
  - ডাব্লিউএসএল২ (ধীর, ইউনিক্স পরিবেশ)
  - ভার্চুয়ালাইজেশন
    - ভার্চুয়াল মেশিন (আরো ধীর, ইউনিক্স)
	- ডকার কন্টেইনার (ক্রস প্লাটফর্ম, ধীর, ইউনিক্স পরিবেশ)
</details>

Redis is an in memory, non-reletional database structure server. Kind of like cache. Redis is a full service cache which is used as a database.

It works using Key stores system. It store Key value pair. In a syntax like 'Key:Value'. It can also works with JSON with help of a module.

Redis can work in distributed environment. Either as Sentinel or as Cluster. Now Cluster will be used for learning purpose.

#### Installation

Install Memurai. Create shortcuts named redis, redis-cli, redis-server with Memurai equivalent executable in the windows directory.

The company behind memurai contribute in MSOpenTech's Redis-for-Windows port. Now they made their own port named it Memurai and sell it. It's free for developers.



Redis has an official Python client. To connect existing database localhost:6379 to Redis Python client-

>>> import redis
>>> r = redis.Redis(host='127.0.0.1', port=6379, db=0, decode_responses=True)
We can run Redis cli command in this python script

>>> r.get("pet")
'cat'
>>> r.set("Forrest", "tree")
True
>>> r.get("Forrest")
'Gump'

> If it shows 'tree' don't worry!

To access another redis server in the local network (lets say 192.168.0.101:6379), open redis.conf file of the server and change bind 127.0.0.1 to bind 0.0.0.0, and restart the server with /etc/init.d/redis-server restart. From another device, run redis-cli -h 192.168.0.101 -p 6379 -a <password_if_any> (set up a password from requirepass <password> in redis.conf file).


### Understanding redis

Redis is not a NoSQL store but has similarity. It works with many languages. Here Python will be used.

#### What is Redis Cluster?
Redis Cluster is a set of Redis instances, designed for scaling a database by partitioning it, thus making it more resilient. Each member in the cluster, whether a primary or a secondary replica, manages a subset of the hash slot. If a master becomes unreachable, then its replica (named slave before) is promoted to master. In a minimal Redis Cluster made up of three master nodes, each with a single replica node (to allow minimal failover), each master node is assigned a hash slot range between 0 and 16,383. Node A contains hash slots from 0 to 5000, node B from 5001 to 10000, node C from 10001 to 16383. Communication inside the cluster is made via an internal bus, using a gossip protocol to propagate information about the cluster or to discover new nodes.

Every Redis Cluster node requires two TCP connections open. The normal Redis TCP port used to serve clients, for example 6379, plus the port obtained by adding 10000 to the data port, so 16379. Make sure you open both ports in your firewall, otherwise Redis cluster nodes will be not able to communicate.

Redis Cluster does not use consistent hashing, but a different form of sharding where every key is conceptually part of what we call an hash slot. There are 16384 hash slots in Redis Cluster.

Every node in a Redis Cluster is responsible for a subset of the hash slots, so for example you may have a cluster with 3 nodes, where:

- Node A contains hash slots from 0 to 5500.
- Node B contains hash slots from 5501 to 11000.
- Node C contains hash slots from 11001 to 16383.

### Redis command

In redis installation directory,

Running Redis server: `redis-server` or with .exe
Running Redis command line interface: `redis-cli` or with .exe
Quitting both with saving: `shutdown save` `quit`
Quitting both without saving: `shutdown nosave` `quit`

### Creating and using a Redis Cluster

A redis cluster uses master-replica configuration to support distributed environment. In this example we will create 3 master nodes and 3 replica nodes. Each master node having at least 1 replica. We need atleast 3 master nodes to create a cluster.

#### Why Do you need a minimum of 3 masters?

During the failure detection, the majority of the master nodes are required to come to an agreement. If there are only 2 masters, say A and B and B failed, then the A master node cannot reach to a decision according to the protocol. The A node needs another third node, say C, to tell A that it also cannot reach B.

If you want to get a cluster up and running in no time with minimal configuration then use the `create-cluster` script shipped by default in redis package. There are many tutorials on how to create cluster using `create-cluster` (ruby) script (Requires Ruby to be installed) so I am not going to cover that. Rather i will showcase how to do it in a manually which gives you great degree of freedom to tweak with cluster configuration parameters.

#### Installation

We can just use 3 nodes. We want to use replicas so for each master we will use one replica node.

Since we are simulating a 6 node cluster (3 masters and 3 replicas) we will create 6 folders: 6001, 6002, 6003, 6004, 6005, 6006. Here name of the folder represents the port number on which each instance will run. We can name folders as we wish like master-1, master-2, master-3, node-1, node-2, node-3. We can also work without folders.

For we can use different computers for each node as node works in server following method no. 1 below. Just placing each folder in each computer. Or we can just use one PC. We will do it like this for convenience.

1. On each node i.e. inside each folder, download and make the redis package by executing following commands for UNIX-

> You will need gcc to have make command available.

```bash
$ wget http://download.redis.io/releases/redis-6.0.8.tar.gz
$ tar xzf redis-6.0.8.tar.gz
$ cd redis-6.0.8
$ make
```

> This is the command for UNIX to install Redis: `sudo make install`.

For windows just copying Memurai installaion will do.

You can also simply download the package in a folder and copy it to other folders. This will save you some download time and bandwidth.
The binaries that are now compiled are available in the src directory.

2. We can also use one redis package for all folders. If redis is installed.

3. It will also work without folders. In tha case everything needs to be done inside redis package directory.

> Using virtualizaion, we can do it using Vagrant and Vmware.

#### Running Redis in cluster mode

Now that you have redis, you can see a `redis.conf` file inside src directory. This is the configuration file in which you should define all cluster configuration parameters. Following is an example of minimal configuration you should have in `redis.conf` to start with-

```
port 6001
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

As you can see what enables the cluster mode is simply the `cluster-enabled` directive. Every instance also contains the path of a file where the configuration for this node is stored, which by default is nodes.conf. This file is never touched by humans; it is simply generated at startup by the Redis Cluster instances, and updated every time it is needed. Note that the minimal cluster that works as expected requires to contain at least three master nodes.

Create a redis.conf file inside each of the directories, from 6001 to 6006. As a template for your configuration file just use the small example above, but make sure to replace the port number 6001 with the right port number according to the directory name.
Finally open 6 terminal tabs in your favorite terminal application. Start every instance like that, one every tab:

> Running Redis server with a configuration file.
In windows redis-server is redis-server.exe. First name should work too.

```
./src/redis-server ./redis.conf
```

This ID will be used forever by this specific instance in order for the instance to have a unique name in the context of the cluster. Every node remembers every other node using this IDs, and not by IP or port. IP addresses and ports may change, but the unique node identifier will never change for all the life of the node. We call this identifier simply Node ID.

#### Creating the cluster
Now bind these servers in redis-cli so that it knows how to hash and store data according to hash by typing & executing this command in a new terminal window.

```
./redis-cli --cluster create 127.0.0.1:6001 127.0.0.1:6002 127.0.0.1:6003 127.0.0.1:6004 127.0.0.1:6005 127.0.0.1:6006 --cluster-replicas 1
```

> `./redis-cli --cluster create 127.0.0.1:6001 127.0.0.1:6002 127.0.0.1:6003` This would only bind master nodes.

The option `--cluster-replicas 1` means that we want a replica for every master created. The other arguments are the list of addresses of the instances you want to use to create the new cluster.
Obviously the only setup with our requirements is to create a cluster with 3 masters and 3 replicas. Redis has 16384 slots for hashing, we need to distribute these slots to different  master nodes. You can see in following output it says `> [OK] All 16384 slots covered.`


```
> Adding replica 127.0.0.1:6005 to 127.0.0.1:6001
> Adding replica 127.0.0.1:6006 to 127.0.0.1:6002
> Adding replica 127.0.0.1:6004 to 127.0.0.1:6003
```

In above screenshot, following lines means exactly what they say.

Nodes running on ports 6001, 6002, and 6003 are masters and nodes running on ports 6005, 6006 and 6004 are replicas of those masters respectively.
The cluster is now up and running. You can test it by executing commands—

```
$ set key value
…
$ set car	black
…
$ get car
> black
```

Following command indicates that i am using redis-cli in cluster mode and connecting to node running on port 6001.

```
./src/redis-cli -c -p 6001
```

After that you can see key value pairs are getting stored in nodes depending on which hash slots they are getting saved into. It also shows i can access any key from any node. In above screenshot key foo was saved in node 6003 but i was able to retrieve it even when I was on node 6002.

Test some more now.

Redis cluster is now up and fully functioning.

#### Install redis-py-cluster python client

`pip install redis-py-cluster`

Execute rcp.py to access redis-cluster using python.

In python file, when the code gets connected with one server of the cluster, then it automatically finds the rest servers and using hashing algorithm, it inserts data in a round robin fashion.

To connect python with our existing redis cluster in port 6001-6006:

>>> from rediscluster import RedisCluster
>>> nodes = [{"host":"127.0.0.1", "port":"6001","host":"127.0.0.1", "port":"6002","host":"127.0.0.3", "port":"6003","host":"127.0.0.1", "port":"6004","host":"127.0.0.1", "port":"6005","host":"127.0.0.1", "port":"6006"}]
>>> r = RedisCluster(startup_nodes = nodes, decode_responses = True)
Then run redis cli command in the cluster:

>>> r.set("BD","Dhaka")
True
>>> r.get("BD")
'Dhaka'

#### How Redis manage its storage

Redis Cluster is the native sharding implementation available within Redis that allows you to automatically distribute your data across multiple nodes without having to rely on external tools and utilities.

The entire keyspace in Redis Clusters is divided into 16384 slots (called hash slots) and these slots are assigned to multiple Redis nodes. A given key is mapped to one of these slots, and the hash slot for a key is computed as:

`HASH_SLOT = CRC16(key) mod 16384`

### Testing server/node failure recovery

Simulation

Incomplete. Later…


#### Credit
- Andrey Redko https://www.javacodegeeks.com/2015/09/redis-clustering.html#InstallingRedis
- Vishal Khare https://medium.com/@iamvishalkhare/create-a-redis-cluster-faa89c5a6bb4
- Dhammika Saman Kumara https://medium.com/swlh/getting-started-with-redis-cluster-on-windows-6435d0ffd87
- https://dingyuliang.me/redis-3-2-create-cluster-windows
- rob https://stackoverflow.com/a/46648136
- Shafin Hasnat https://github.com/shafinhasnat/REDIS-made-easy/README.md