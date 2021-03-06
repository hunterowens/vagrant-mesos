Setting up Mesos using Vagrant
===

This contains two vagrant sandbox environment.

* Standalone environment on VirtualBox or EC2
* Multi nodes cluster environment on VirtualBox or EC2 (VPC)

The mesos installation is provided by mesos chef cookbook.  Please see [everpeace/cookbook-mesos](http://github.com/everpeace/cookbook-mesos).

Prerequisites
----
* VirtualBox: <https://www.virtualbox.org/> (not required if you use ec2.)
* vagrant 1.2+: <http://www.vagrantup.com/>
* vagrant plugins
    * [vagrant-omnibus](https://github.com/schisamo/vagrant-omnibus)
          `$ vagrant plugin install vagrant-omnibus`
    * [vagrant-berkshelf](https://github.com/RiotGames/vagrant-berkshelf)
          `$ vagrant plugin install vagrant-berkshelf`
    * [vagrant-hosts](https://github.com/adrienthebo/vagrant-hosts)
          `$ vagrant plugin install vagrant-hosts`
    * [vagrant-cachier](https://github.com/fgrehm/vagrant-cachier)
          `$ vagrant plugin install vagrant-cachier`
    * [vagrant-aws](https://github.com/mitchellh/vagrant-aws) (only if you use ec2.)
    	   `$ vagrant plugin install vagrant-aws`

Standalone Environment with VirtualBox
----
It's so simple! 

    $ cd standalone
    $ vagrant up

After box is up, you can see mesos web UI on: <http://localhost:5050> if everything went well. 

Standalone Environment with EC2
----
1. configure ec2 credentials and some settings defined in `standalone/aws_config.yml`. You have to fill up `EDIT_HERE` parts.  Security group you'll set must accept at least tcp port 22(SSH) and 5050(mesos-master web ui) from outside of ec2.
	
		# Please set AWS credentials
		access_key_id:  EDIT_HERE
		secret_access_key: EDIT_HERE
		
		# please choose one from 
		# ["ap-northeast-1", "ap-southeast-1", "eu-west-1", "sa-east-1", "us-east-1", 
		#  "us-west-1", "ap-southeast-2", "us-west-2"]
		region: us-east-1
		
		# array of security groups. e.g. ['sg*** ']
		security_groups: EDIT_HERE
		
		# see http://aws.amazon.com/ec2/instance-types/#selecting-instance-types
		# for other instance types and its specs.
		instance_type: m1.small
		
		keypair_name: EDIT_HERE
		
		ssh_private_key_path: EDIT_HERE

2. you can spin up mesos box on ec2 by the same way with the case of virtual box

        cd standalone
        vagrant up --provider aws
   
   After box is up, you can see mesos web UI on: `http://#_public_dns_of_the_VM_#:5050` if everything went well. 


Multinode environment with VirtualBox
----
### Cluster Configuration
Cluster configuration is defined at `cluster.yml`.  You can edit the file to congigure cluster configurations.

```
# Mesos cluster configurations

# The numbers of servers
##############################
zk_n: 3          # hostname will be zk1, zk2, …      
master_n: 2      # hostname will be master1,master2,…
slave_n : 3      # hostname will be slave1,slave2,… 

# Memory setting(only for virtualbox)
##########################################
zk_mem    : 256
master_mem: 256
slave_mem : 256

# private ip bases
# When ec2, this should be matched with 
# private addresses defined by subnet_id below.
################################################
zk_ipbase    : "172.31.0."
master_ipbase: "172.31.1."
slave_ipbase : "172.31.2."
```

### Launch a Cluster
This takes several minutes(10 to 20 min.).  It's time to go grabbing some coffee.

```
$ cd multinodes
$ ./vagrant up
```

mesos master listen on port 5050 on (virtual private IP). So, by normal virtual box NAT setting, host os can't connect to mesos master. To connect mesos master, you can use ssh port forwarding like this.

```
$ vagrant ssh master1 -- -L15050:master1:5050
```
Then, you can connect mesos web ui on <http://localhost:15050>

#### Destroy a Cluster
this operations all VM instances forming the cluster.

```
$ cd multinodes
$ ./vagrant destroy
```

Multinode environment with EC2(VPC)
----
Because we require to assign private IP addreses to VM instances, this Vagrantfile use Amazon VPC (you'll have to set subnet_id and security grooups both associated to the same VPC instance).

### Cluster Configuration
You have to configure some additional stuff on `cluster.yml` related to EC2.  Please note that 

* subnet_id should be a VPC subnet
* security groups should be ones associated to the VPC instance.

```
(cont.)
# EC2 Configurations
# please choose one region from  
# ["ap-northeast-1", "ap-southeast-1", "eu-west-1", "sa-east-1",
#  "us-east-1", "us-west-1", "ap-southeast-2", "us-west-2"]
########################
access_key_id:  EDIT_HERE
secret_access_key: EDIT_HERE
subnet_id: EDIT_HERE               # VPC subnet id
security_groups: ["EDIT_HERE"]     # array of VPN security groups. e.g. ['sg*** ']
keypair_name: EDIT_HERE
ssh_private_key_path: EDIT_HERE
region: EDIT_HERE            

# see http://aws.amazon.com/ec2/instance-types/#selecting-instance-types
zk_instance_type: m1.small
master_instance_type: m1.small
slave_instance_type: m1.small
```

### Launch a Cluster
After editting configuration is done, you can just hit regular command.

```
$ cd multinode
$ vagrant up --provision aws (--no-parallel)
```

After instances are all up, you can see mesos web UI on: `http://#_public_dns_of_the_master_N_#:5050` if everything went well. 

__CAUTION__: due to [MESOS-672](https://issues.apache.org/jira/browse/MESOS-672), mesos web ui fails redirection to the present leader of mesos master.  So, you need to access each master web ui manually (you can find public dns name for masters on amazon aws console).

#### Destroy a Cluster
this operations terminates all VMs instances forming the cluster.

```
$ cd multinodes
$ ./vagrant destroy
```
