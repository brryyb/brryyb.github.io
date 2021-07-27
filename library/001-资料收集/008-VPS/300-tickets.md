### installation###
We do offer free Master and Slave installation as per the Install it For me service request.

We will be here for 24x7.

We offer free Master installation and 2 slave installation and after that you have to do it as per the documentation.

https://support.solus.io/hc/en-us/articles/360015047832-How-to-deploy-SolusVM-Master-node-

https://support.solus.io/hc/en-us/articles/360015107872-How-to-install-SolusVM-on-a-slave-node-

We have an article for this which you could try from your end.
https://documentation.solusvm.com/display/DOCS/WHMCS

I have installed the slave with KVM virtualization and configured the KVM bridge by following the articles.

https://support.solus.io/hc/en-us/articles/360015107872-How-to-install-SolusVM-on-a-slave-node-

https://support.solus.io/hc/en-us/articles/360020999631-How-to-configure-bridge-interface-for-KVM-slave-node-

You are able to add the slave node by following the article with ID key & ID password.


ID Key .......... : feMqMA884iwQ5yvXhviXO3tC4wOv6jkbhQu88ZUOP8DGWTW9P0
ID Password ..... : ZYk6VXHDZlcdVmjvgH9HoVNVV3l6Tj6iwen4WNUdw6QSluQwB8


https://support.solus.io/hc/en-us/articles/360025346392-How-to-add-slave-SolusVM-node-to-a-Master-SolusVM-node-

You are able to reset the admin user login bu following the article.

https://support.solus.io/hc/en-us/articles/360021875832-How-to-generate-a-new-admin-password-for-SolusVM-

Please contact us when the servers are ready.

### remove an unavailable node###

to remove an unavailable node from SolusVM - execute the command below on Master node:

php /usr/local/solusvm/scripts/deletenode.php --level=force --comm=delete --id=NODEID

NODEID replace with actual id - can be seen in SolusVM > Nodes page

It will also remove relevant VPS from SolusVM interface - they still will be running on slave node itself.

To manually remove KVM VPS from slave node - execute on slave node:

virsh undefine kvmXXX
rm -rf /home/kvm/kvmXXX
lvremove /dev/your_vg/kvmXXX

change kvmXXX to the actual ID - for example, kvm101 - can be found in the output of the command

virsh list --all

your_vg can be found in the output of command "vgs".

### VPSes not starting after creation###
As we discussed in the chat you had an issue with VPSes not starting after creation:

Error: could not find capabilities for domaintype=kvm

To fix this you contacted your DC to enable VT support.

Please let me know if you have any additional questions.

### connect to new KVM VPS###
it is not possible to connect to new KVM VPS. VNC console shows that VPS not booted.
such behavior was caused by incorrect template configuration and missing template file on the slave node:

    There is no a template file on the slave server
    On the template settings page architecture set as i386, but the template file is for x86_64

once template settings were updated and media synced, VPS created successfully
each TDN template has first-boot.sh script that performs several important tasks in order to prepare VPS. It is launched 1 time on the first boot of the VPS. One of the operations the script executes is installing system updates, for CentOS-like system it runs "yum -y update"
As per your request, I removed installing of system updates for the CentOS 7 template on the slave server:
/home/solusvm/kvm/template/linux-centos-7-x86_64-minimal-latest.gz

    Created a temporary directory:
    mkdir /mnt_point
    mounted the template file to this directory to gain access to the content inside of the template
    guestmount -a /home/solusvm/kvm/template/linux-centos-7-x86_64-minimal-latest.gz -i --rw /mnt_point/
    found script
    /mnt_point/usr/lib/virt-sysprep/scripts/0001-swapoff--dev-vda2-mkswap--dev-vda2-swapon--dev-vda2-resize2f
    it has several actions that not recommended to remove, like resizing partition. So it is not recommended to remove them.
    Opened this file with text editor and removed "yum -y update" part. Saved the changes.
    And unmounted template file to complete all actions:
    fuser -u /mnt_point/

    You have created a KVM VPS on your new node with specific IP range - network did not work in this VPS.

However, the same IP range with the same gateway worked on another node.

During the chat I have tested routing on both servers - added the same IP address to both nodes and checked its availability:

on the node where IP range is working:

[root@CTG583 ~]# ifconfig br0:0 103.146.90.61 netmask 255.255.255.248 up
[root@CTG583 ~]#

then pinging it from another server

ping 103.146.90.61

PING 103.146.90.61 (103.146.90.61) 56(84) bytes of data.<br />
64 bytes from 103.146.90.61: icmp_seq=1 ttl=49 time=214 ms<br />
64 bytes from 103.146.90.61: icmp_seq=2 ttl=49 time=214 ms<br />
64 bytes from 103.146.90.61: icmp_seq=3 ttl=49 time=214 ms<br />
^C<br />
--- 103.146.90.61 ping statistics ---<br />
3 packets transmitted, 3 received, 0% packet loss, time 2467ms<br />
rtt min/avg/max/mdev = 214.216/214.249/214.303/0.535 ms<br />


Now when I add the same IP to your second node:

[root@GDI2076 ~]# ifconfig br0:0 103.146.90.61 netmask 255.255.255.248 up
[root@GDI2076 ~]#

ip is not pingable - instead, it shows error:

ping 103.146.90.61

PING 103.146.90.61 (103.146.90.61) 56(84) bytes of data.<br />
From 23.225.35.153 icmp_seq=1 Time to live exceeded<br />
From 23.225.35.153 icmp_seq=2 Time to live exceeded<br />
From 23.225.35.153 icmp_seq=3 Time to live exceeded<br />


which means that there is no proper route to this server

I suggest contacting the provider of your IP range to make sure that it is properly routed to your server 23.224.147.106
You can show them the above information to prove that there is something wrong with routing.

we have installed SolusVM KVM slave node and configured bridge.

Then you have attached slave node to Master node (https://docs.solusvm.com/display/BET/Connecting+SolusVM+KVM+Slave+to+SolusVM+Master+node) (updated LV group in SolusVM > Nodes > Edit Node next to the added node), then added templates to master node and synced them with slave node:
https://docs.solusvm.com/display/BET/About+TDN+Templates
https://docs.solusvm.com/display/BET/Uploading+the+templates+to+the+Master+Node

After that you have created a test VPS, however, it did not have a network. It was due to incorrect Gateway set in SolusVM > IP Blocks > Edit Block next to the required IP Block - it was IP address of VPS, however should have been Gateway of the node itself, as the IP Block was routed to your server.

After fixing that and reconfiguring VPS through SolusVM > Virtual Servers > VPS > Re-Configure - VPS became available.

You are able to remove the slave node by following the article. While removing the slave node from the master, the VPSs on the node also will be removed from the master.

https://support.solus.io/hc/en-us/articles/360018312972-How-to-forcefully-delete-a-slave-node-from-Master-SolusVM-

As the master and slave node are running with different SolusVM versions, you may face a password reset issue. You are able to update the SolusVM version by running the below commands on the master server.

upcp
/usr/local/solusvm/tmp/update/update

You have reported the network issue on the new VPSs with the range 103.146.90.xx. I could see that this is a different range of IPs on the slave node. I have bound the IP 103.146.90.60 & 103.146.90.63 on the slave node as a secondary IP temporarily but its not working from the slave node so that it should be a routing issue and suggest you to contact your DC to route this range of IPs to your slave node 23.224.147.106 then have a try further.
### not enough free memory###
VPS provisioning from WHMCS failed with "not enough free memory" error. It was caused by default zero values for Max Disk and Max Memory fields for the node settings in SolusVM.
You have updated them as below:
Max Memory: there are 31G of total amount of RAM on the node.
It is recommended to leave about 2G of RAM for system services and other memory you can use for VPSes
29696 value has been set as Max Memory
Max Disk: Volume Group size is 855040 MB, so you can use this value as Max Disk

After that you successfully provisioned VPS from WHMCS.
Let me know should you have additional questions.
### df -h###
SolusVM displays the df -h output of the Node in Dashboard page and the Virtual Disk usage (i.e) Volume Group usage shows in the Manage Node page.

Your total server disk usage = Dashboard Disk usage report + Manage Node's Virtual Disk Usage Report

### bridge configuration.###
I have just followed the article to assist you with the bridge configuration.

https://docs.solusvm.com/display/BET/Bridge+configuration+for+KVM+Slave

You can add the slave node to the master server by following the article.

https://support.solus.io/hc/en-us/articles/360025346392-How-to-add-slave-SolusVM-node-to-a-Master-SolusVM-node-

You are able to add the IP block and templates by following the articles respectively.

https://support.solus.io/hc/en-us/articles/360025561712-How-to-create-an-IP-block-in-SolusVM-
https://support.solus.io/hc/en-us/articles/360021214032-How-to-add-a-KVM-template-on-SolusVM-

I have upgraded the e2fsprogs on the host node by following the article to make the CentOS 7 VM work on the CentOS 6 host node.

https://support.solus.io/hc/en-us/articles/360024478692-Virt-resize-fails-on-the-CentOS-6-node-dev-sda1-has-unsupported-feature-s-64bit-e2fsck-Get-a-newer-version-of-e2fsck-

There is no possibility to use our Ubuntu & Debian templates from TDN on CentOS 6 nodes since guestmount version doesn't support ext4 on it. You can create a VM using ISO with an ext3 filesystem then convert it to a template by following the document else you can go with CentOS 7 for the host node to avoid like this issue.

https://docs.solusvm.com/display/BET/Custom+templates
