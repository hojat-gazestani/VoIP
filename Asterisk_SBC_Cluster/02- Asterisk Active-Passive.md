In an Active-Passive configuration, there is one primary server that is responsible for processing requests, and a secondary server that is passive and only becomes active if the primary server fails. Here is an example configuration for a basic Active-Passive setup for Asterisk High Availability:

1. First, install and configure Asterisk on both servers as needed.

2. On the primary server, set up a floating IP address (VIP) that will be used by clients to connect to the cluster. This IP address will be moved to the secondary server if a failover occurs. You can do this with the following command:

   ```bash
   ip addr add <VIP>/<netmask> dev <interface>
   
   ## OR 
   
   vim /etc/netplan/00-installer-config.yaml 
   # This is the network config written by 'subiquity'
   network:
     version: 2
     ethernets:
       ens160:
        dhcp4: no
        addresses: [172.25.23.71/24]
        gateway4: 172.25.23.254
        nameservers: 
          addresses: [172.25.23.254, 8.8.8.8]
   #  version: 2
   
   ```

   Replace `<VIP>` with the IP address you want to use for the virtual IP, `<netmask>` with the appropriate netmask for your network, and `<interface>` with the network interface you want to use.

3. Install and configure corosync and pacemaker on both servers. These are the tools that will manage the failover process. You can do this with the following commands:

   ```bash
   sudo apt-get install corosync pacemaker
   sudo systemctl enable corosync.service
   sudo systemctl start corosync.service
   sudo systemctl enable pacemaker.service
   sudo systemctl start pacemaker.service
   ```

4. On the primary server, configure the ha.cf file to specify the cluster name, the IP address of the secondary server, and the failover method. Here is an example configuration:

   ```bash
   debugfile /var/log/ha-debug
   logfile /var/log/ha-log
   logfacility local0
   keepalive 1
   deadtime 10
   initdead 120
   bcast eth0
   udpport 694
   auto_failback on
   node <primary_server> 
   node <secondary_server> secondary
   ```

   Replace `<primary_server>` and `<secondary_server>` with the hostnames or IP addresses of your servers.

5. On the secondary server, configure the ha.cf file as follows:

   ```bash
   debugfile /var/log/ha-debug
   logfile /var/log/ha-log
   logfacility local0
   keepalive 1
   deadtime 10
   initdead 120
   bcast eth0
   udpport 694
   auto_failback on
   node <secondary_server>
   node <primary_server> primary
   ```

   Replace `<primary_server>` and `<secondary_server>` with the hostnames or IP addresses of your servers.

6. Create a resource configuration file for Asterisk, which specifies the commands to start, stop, and monitor Asterisk on both servers. Here is an example:

   ```bash
   primitive p_asterisk ocf:heartbeat:asterisk \
     op start timeout=120 \
     op stop timeout=120 \
     op monitor interval=30 timeout=30 depth=0 \
     meta is-managed=true
   ```

7. On the primary server, create a resource group that includes the Asterisk resource and the VIP resource. Here is an example:

   ```bash
   group g_asterisk p_asterisk vip <VIP>
   ```

   Replace `<VIP>` with the virtual IP address you created earlier.

8. On the secondary server, create the same resource group, but with the VIP resource set to "secondary". Here is an example:

   ```bash
   group g_asterisk p_asterisk vip <VIP> secondary
   ```

9. Start the resource group on the primary server with the following command:

   ```bash
   crm resource start g_asterisk
   ```

10. Test the failover by stopping Asterisk on