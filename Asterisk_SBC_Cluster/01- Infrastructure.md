

# choosing a Session Border Controller (SBC)

```text
	Compatibility	: compatible with Asterisk and supports the necessary protocols and standards.
	Features		: security, protocol translation, session management, call routing, and media handling
	Scalability		: handle increased traffic loads.
	Reliability		: high availability and failover
	Cost			: both initial costs and ongoing costs
```




	

# VOIP traffic load balancer(PBX load balancing):
```text
The SBC acts as a gateway, directing incoming calls to one of the available PBXs based on various factors, such as the availability of PBX resources, call routing policies, or the load on each PBX.

```

​	

# Commecial SBC:
```text
	AudioCodes: 			leading provider
	Sangoma: 				well-known provider
	Digium: 				creator of Asterisk
	Ribbon Communications:  compatible with Asterisk
```

​	

# open source SBC:

		

```text
Kamailio: 	high-performance, rich set of features
OpenSBC: 
			(Based on the OpenSIPS) easy to use and can be integrated with a wide range of VoIP platforms.
			firewall and intrusion detection, support for various signaling protocols.
			support for various signaling protocols, and call routing and management capabilities. 
			
FreeSWITCH: multi-protocol, has a large and active community of developers,
OpenSIPS: 	highly customizable
LiberSBC:  	designed for use with Asterisk, pre-configured templates, ease of use and flexibility.
```

# NAT for VOIP traffic: 

* An SBC can perform both inbound and outbound NAT	

	* nathelper: modifies the SIP headers to include the public IP address of the Kamailio server and updates the Contact, Via, and Record-Route headers as necessary. 
 * ​	The media stream is handled by a separate module, such as rtpproxy, which forwards the media packets between the endpoints.
   ​	

	Kamailio: nathelper and rtpproxy
	OpenSBC:  nathelper and rtpproxy
	FreeSWITCH: 
		mod_sofia module support NAT traversal and RTP proxying
		mod_nat, mod_extaddr, and mod_rtp_multicast.
	OpenSIPS: 

# NAT traversal technologies:


	STUN (Simple Traversal of UDP through NATs) is a protocol that helps to determine the public IP address and type of NAT used by a device. STUN allows devices behind a NAT to discover their public IP address and port mapping.
		
	TURN (Traversal Using Relays around NAT) is a protocol that provides a way to relay network traffic between devices that are behind NATs. TURN is used when STUN fails, and it allows devices to communicate with each other even if they are behind different NATs.
	
	ICE (Interactive Connectivity Establishment) is a framework that uses STUN and TURN to establish a direct communication path between two devices, even if they are behind NATs. ICE is commonly used in VoIP and WebRTC applications.
	
	pjSIP is an open-source multimedia communication library written in C language that implements the SIP (Session Initiation Protocol) and various other protocols related to real-time multimedia communication. It is a library and not an SBC itself, but it can be used to build SBC applications.
	pjSIP supports NAT traversal through STUN and TURN protocols, and it can also be integrated with ICE (Interactive Connectivity Establishment) for more robust NAT traversal. Additionally, pjSIP can be used in combination with other technologies, such as RTPproxy, for more advanced media handling and media processing.


​	
-----------------------------------------------------------------

# SOP

![Scenario](/home/hojat/Downloads/Timor.jpg)



- To create a high available and failover VoIP network with two Asterisk PBX systems and two Kamailio SBCs, you can follow the below steps:

##  Configure both Asterisk PBX systems as highly available cluster and act as one PBX:

- Install and configure Asterisk PBX on both servers.
- Configure the SIP extensions, trunks, and dial plans on both PBX systems.
- Ensure that both PBX systems are reachable from each other and from the outside network.

1. Synchronize the configurations: Both PBX systems should have the same configuration files and settings. You can use tools like rsync to keep the configuration files in sync.
2. Configure Network: Both PBX systems should be connected to the same network and have access to the same VoIP devices and services.
3. Configure DNS: Set up a DNS server to provide the IP address of the active PBX.
4. Configure Heartbeat: Use Heartbeat to monitor the state of the PBX systems and switch between active and standby systems in case of a failure.
5. Configure failover: Configure failover of SIP sessions to the standby PBX in case the active PBX fails.
6. Test the setup: Test the failover and failback by simulating a failure on the active PBX.

##  basic configuration guide to set up a high-availability cluster with two Asterisk PBXs on Ubuntu 22.04 LTS:

1. Install the necessary packages:

```bash
sudo apt-get update
sudo apt-get install asterisk asterisk-dahdi asterisk-core-sounds-en-gsm asterisk-core-sounds-en-wav asterisk-moh-opsound-wav asterisk-dev asterisk-doc
```

1. Create a configuration file for both servers. Here, we’ll call it `ha.conf`. Put the following contents in the file:

```
makefileCopy code[general]
Failover=Primary
Heartbeat=10.0.0.1
[Primary]
Address=10.0.0.2
Port=5038
[Secondary]
Address=10.0.0.3
Port=5038
```

1. Copy the file to both servers:

```ruby
scp ha.conf username@10.0.0.2:/etc/asterisk/
scp ha.conf username@10.0.0.3:/etc/asterisk/
```

1. On both servers, edit the `/etc/asterisk/manager.conf` file to allow remote connections:

```bash
[general]
enabled = yes
port = 5038
bindaddr = 0.0.0.0

[ha-peer]
secret = password
read = system,call,log,verbose,command,agent,user,config
write = system,call,log,verbose,command,agent,user,config
```

1. On the primary server, set the manager.conf setting to be writable:

```bash
chmod 660 /etc/asterisk/manager.conf
chown asterisk:asterisk /etc/asterisk/manager.conf
```

1. On both servers, edit the `/etc/asterisk/asterisk.conf` file:

```
makefileCopy code[options]
manageruser=ha-peer
managerpassword=password
```

1. Restart both servers:

```
Copy code
sudo service asterisk restart
```





## Configure both Kamailio SBCs as highly available cluster and act as one SBC:

- Install and configure Kamailio SBC on both servers.
- Configure Kamailio to handle incoming and outgoing SIP traffic and perform necessary functions, such as NAT traversal, call routing, and security.
- Ensure that both Kamailio SBCs are reachable from each other and from the outside network.

1. Install Kamailio on both servers and make sure they are running the same version.
2. Configure the network settings of each server so that they can communicate with each other. This includes configuring the same IP addresses and subnet masks, and setting up a shared storage mechanism such as NFS or iSCSI.
3. Configure both servers as nodes in a Kamailio cluster. This includes setting up the Kamailio clustering protocol, configuring the cluster's name, and specifying which node is the active node and which node is the standby node.
4. Configure the Kamailio SIP service on each node. This includes setting up the SIP domain, setting up SIP user accounts, and specifying which IP address and port the SIP service should listen on.
5. Configure load balancing and failover for the SIP service. This involves setting up rules for distributing SIP traffic between the two nodes, and specifying how the active node should take over if the standby node fails.
6. Test the configuration by sending SIP traffic to the cluster and verifying that it is being handled by the correct node.

* Base Configuration

- The exact code for configuring Kamailio SBCs in a highly available cluster depends on the specific setup and requirements of the network. However, some general steps to achieve this are:

1. Install Kamailio on both Ubuntu 22.04 LTS servers:

```sql
sudo apt update
sudo apt install kamailio
```





## basic process to install and configure FreeNAS as an NFS server on an Ubuntu 22.04 system:

1. Install FreeNAS on a server: FreeNAS can be installed as a virtual machine or on dedicated hardware.
2. Create a FreeNAS dataset: In the FreeNAS web interface, navigate to the "Storage" section, then click on "Datasets." Create a new dataset for NFS sharing.
3. Enable NFS sharing: In the FreeNAS web interface, navigate to the "Services" section, then click on "NFS." Enable NFS sharing, then add the dataset created in step 2 as a shared directory.
4. Configure NFS on the Ubuntu 22.04 system: On the Ubuntu 22.04 system, install the NFS client packages by running the following command:

```bash
sudo apt-get install nfs-common
```

1. Mount the NFS share: On the Ubuntu 22.04 system, create a mount point for the NFS share, then mount the NFS share using the following command:

```bash
sudo mkdir /mnt/freenas
sudo mount <FreeNAS IP address>:<FreeNAS NFS share path> /mnt/freenas
```

1. Verify the NFS mount: On the Ubuntu 22.04 system, use the following command to verify that the NFS share is mounted correctly:

```bash

df -h
```



3. configuration code to set up a shared database for Kamailio nodes to store configuration and real-time information on the PostgreSQL server:

```perl
# Database setup in PostgreSQL
CREATE DATABASE kamailio;
CREATE USER kamailio WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE kamailio TO kamailio;

# Kamailio configuration
modparam("db_postgres", "db_url", "postgres://kamailio:password@localhost/kamailio")

# Load the required database modules
loadmodule "db_postgres.so"

# Use the appropriate database table names in Kamailio configuration
route{
  ...
  # Kamailio database operations
  ...
}
```



## Configure Kamailio on both nodes to use the freeNAS shared storage

The configuration of Kamailio to use the FreeNAS shared storage involves the following steps:

1. Install NFS client packages on both Kamailio nodes:

```bash
sudo apt-get update
sudo apt-get install nfs-common
```

1. Create a mount point for the NFS share on both Kamailio nodes:

```bash
sudo mkdir /mnt/nfs
```

1. Mount the NFS share on both Kamailio nodes:

```bash
sudo mount <FreeNAS IP address>:/<NFS share name> /mnt/nfs
```

1. Modify the `/etc/fstab` file on both Kamailio nodes to mount the NFS share at boot time:

```bash
<FreeNAS IP address>:/<NFS share name> /mnt/nfs nfs defaults 0 0
```

1. Configure Kamailio to use the NFS share as the database storage location. This can be done by modifying the database section in the Kamailio configuration file, typically located at `/etc/kamailio/kamailio.cfg`. The following example shows how to configure Kamailio to use a PostgreSQL database stored on the NFS share:

```bash
modparam("db_postgres", "db_url", "postgresql://kamailio:kamailiopass@<FreeNAS IP address>/kamailio")
```

1. Verify that both Kamailio nodes can access the NFS share and the database by starting Kamailio and checking the logs for any errors.



##  Set up load balancing and failover:

- Use a load balancing solution, such as HAProxy or LVS, to distribute incoming SIP traffic to both Kamailio SBCs.
- Set up a keepalived or similar solution to monitor the health of both Kamailio SBCs and switch over to the backup SBC in case of failure.
- Ensure that the load balancing solution and the failover mechanism are highly available and redundant.

1. Install HAproxy:

```bash
sudo apt-get update
sudo apt-get install haproxy
```

1. Configure HAproxy:

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

1. Add the following configuration to the file:

```bash
global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode tcp
    option tcplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000

listen sip
    bind :5060
    mode tcp
    balance roundrobin
    server kamailio1 IP_ADDRESS_OF_KAMAILIO1:5060 check
    server kamailio2 IP_ADDRESS_OF_KAMAILIO2:5060 check
```

Replace IP_ADDRESS_OF_KAMAILIO1 and IP_ADDRESS_OF_KAMAILIO2 with the IP addresses of your Kamailio nodes.

1. Start HAproxy:

```bash
sudo service haproxy start
```

1. Verify that HAproxy is running:

```bash
sudo service haproxy status
```

Now, incoming SIP traffic will be distributed to the two Kamailio nodes using HAproxy



## Configure the Kamailio nodes to communicate with each other for real-time information sharing and failover



1. Install Kamailio on both nodes and make sure they are operational.
2. In the Kamailio configuration file of each node, specify the remote node as a neighbor:

```bash
# Node 1
neighbor = 192.168.1.100:5060

# Node 2
neighbor = 192.168.1.101:5060
```

1. In the Kamailio configuration file of each node, specify a shared database for storing real-time information:

```perl
# Node 1
modparam("tm", "db_url", "mysql://kamailio:kamailio@127.0.0.1/kamailio")
modparam("tm", "failover_db_url", "mysql://kamailio:kamailio@192.168.1.101/kamailio")

# Node 2
modparam("tm", "db_url", "mysql://kamailio:kamailio@127.0.0.1/kamailio")
modparam("tm", "failover_db_url", "mysql://kamailio:kamailio@192.168.1.100/kamailio")
```

1. In the Kamailio configuration file of each node, load the failover module:

```bash
# Node 1
loadmodule "failover.so"

# Node 2
loadmodule "failover.so"
```

1. Restart both Kamailio nodes.

Now, the two Kamailio nodes are configured to communicate with each other for real-time information sharing and failover. In case of a failure in one node, the other node will take over.





## Connect both PBX systems to both SBCs:

### Configure both Asterisk PBX systems to use both Kamailio SBCs as their SIP proxies.



1. On each Asterisk PBX system, open the sip.conf file and create a new peer for each Kamailio SBC. Here's an example:

```bash
[kamailio1]
type=peer
host=kamailio1_ip_address
port=5060
qualify=yes

[kamailio2]
type=peer
host=kamailio2_ip_address
port=5060
qualify=yes
```

1. In the sip.conf file, create a new entry for each peer and set the `outboundproxy` option to the IP address of each Kamailio SBC. Here's an example:

```bash
[general]
context=internal
outboundproxy=kamailio1_ip_address

[Peers]
peer=kamailio1
peer=kamailio2
```

1. Restart the Asterisk service on both PBX systems.
2. Verify the configuration by making a test call and checking the SIP messages in the Asterisk logs.



## Ensure that both PBX systems can register with both SBCs and can make and receive calls.



1. Define the Kamailio SBC IP addresses in the Asterisk PBX configuration files, for example, sip.conf.
2. Configure the Kamailio SBCs to accept incoming connections from the Asterisk PBX systems. This can be done by defining the PBX IP addresses in the Kamailio SBC's configuration files, for example, kamailio.cfg.
3. Define the routing rules in the Kamailio SBCs to route incoming SIP traffic to the correct PBX system. This can be done using the "route" or "rewrite" directives in the Kamailio SBC's configuration files.
4. Test the connection between the PBX systems and the SBCs to ensure that SIP traffic is being properly routed.

