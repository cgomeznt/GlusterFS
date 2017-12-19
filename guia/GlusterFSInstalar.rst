Como instalar GlusterFS con una replica de volumen en dos nodos con Debian Wheezy
=================================================================================

In this tutorial I will explain GlusterFS configuration in Ubuntu 14.04. GlusterFS is an open source distributed file system which provides easy replication over multiple storage nodes. Gluster File System is a distributed filesystem allowing you to create a single volume of storage which spans multiple disks, multiple machines and even multiple data centres.


IP 192.168.0.100, hostname server1.example.com

IP 192.168.0.101, hostname server2.example.com


GlusterFS server installation
+++++++++++++++++++++++++++++

Gluster File System is a distributed files system allowing you to create a single volume of storage which spans multiple disks, multiple machines and even multiple data centres.

Install the required packages using apt-get on both Ubuntu machines. If you have more than two servers, perform this command on all of the servers required for the volume.::

	# apt-get install glusterfs-server

After successful installation, open the terminal and type following command and check if the installation was successful::

	root@server1:~#  glusterfs --version
	glusterfs 3.4.2 built on Jan 14 2014 18:05:35
	Repository revision: git://git.gluster.com/glusterfs.git
	Copyright (c) 2006-2013 Red Hat, Inc. <http://www.redhat.com/>
	GlusterFS comes with ABSOLUTELY NO WARRANTY.
	It is licensed to you under your choice of the GNU Lesser
	General Public License, version 3 or any later version (LGPLv3
	or later), or the GNU General Public License, version 2 (GPLv2),
	in all cases as published by the Free Software Foundation.
	root@server1:~#

	root@server2:~#  glusterfs --version
	glusterfs 3.4.2 built on Jan 14 2014 18:05:35
	Repository revision: git://git.gluster.com/glusterfs.git
	Copyright (c) 2006-2013 Red Hat, Inc. <http://www.redhat.com/>
	GlusterFS comes with ABSOLUTELY NO WARRANTY.
	It is licensed to you under your choice of the GNU Lesser
	General Public License, version 3 or any later version (LGPLv3
	or later), or the GNU General Public License, version 2 (GPLv2),
	in all cases as published by the Free Software Foundation.
	root@server2:~#

Now it is mandatory that both machines must listen to each other with their hostname, so I will update both Ubuntu 14.04 machines with the entries in /etc/hosts::

	# vi /etc/hosts

	192.168.0.100	server1.example.com	gluster1
	192.168.0.101	server2.example.com	gluster2

Now in both machines run the command  gluster peer probe I will run this on first Ubuntu 14.04 (server1.example.com) machine as follows::

	root@server1:~# gluster peer probe gluster1
	peer probe: success: on localhost not needed

further::

	root@server1:~# gluster peer probe gluster2
	peer probe: success

Now I will check the status for peer as::

	root@server1:~# gluster peer status
	Number of Peers: 1
	Hostname: gluster2
	Port: 24007
	Uuid: 5c5a045c-34b9-44ac-b5c0-8acb461d8523
	State: Peer in Cluster (Connected)
	root@server1:~#

Again same thing I will repeat on second Ubuntu 14.04 (server2.example.com) machine::

	root@server2 ~ # gluster peer probe gluster1
	peer probe: success

Next::

	root@vboxtest ~ # gluster peer probe gluster2
	peer probe: success: on localhost not needed

& check the peer status as follows::

	root@server2 ~ # gluster peer status
	Number of Peers: 1

	Hostname: gluster1
	Port: 24007
	Uuid: 8d865314-af12-4950-a784-6a5308ec501b
	State: Peer in Cluster (Connected)
	root@vboxtest ~ #

 

This is a good idea::

	service glusterd stop
	echo "UUID=$(uuidgen)" > /etc/glusterd/glusterd.info
	service glusterd start

Now I will create a common folder on both Ubuntu 14.04 machines viz /mnt/gluster::

	mkdir -p /mnt/gluster

If you wish you can use any other mount points on both machines.

Now we need to create the volume where the data will reside. The volume will be called datapoint. Now run on any machine:

gluster  volume create datapoint replica 2 transport tcp  gluster1:/mnt/gluster  gluster2:/mnt/gluster force::

	root@server1:~# gluster  volume create datapoint replica 2 transport tcp  gluster1:/mnt/gluster  gluster2:/mnt/gluster
	volume create: datapoint: success: please start the volume to access data
	root@server1:~#

Now we need to start the volume::

	root@server1:~# gluster volume start datapoint
	volume start: datapoint: success
	root@server1:~#

Running either of the below commands should indicate that GlusterFS is up and running. The ps command should show the command running with both servers in the argument. netstat should show a connection between both nodes.::
 
	root@server1:~# ps aux | grep gluster
	root      2041  0.0  0.8 391892 16252 ?        Ssl  11:49   0:00 /usr/sbin/glusterd -p /var/run/glusterd.pid
	root      2865  0.0  1.0 451692 19464 ?        Ssl  14:06   0:00 /usr/sbin/glusterfsd -s gluster1 --volfile-id datapoint.gluster1.mnt-gluster -p /var/lib/glusterd/vols/datapoint/run/gluster1-mnt-gluster.pid -S /var/run/d317967a0e3119238993e1580556da73.socket --brick-name /mnt/gluster -l /var/log/glusterfs/bricks/mnt-gluster.log --xlator-option *-posix.glusterd-uuid=8d865314-af12-4950-a784-6a5308ec501b --brick-port 49152 --xlator-option datapoint-server.listen-port=49152
	root      2875  0.0  2.8 277732 53404 ?        Ssl  14:06   0:00 /usr/sbin/glusterfs -s localhost --volfile-id gluster/nfs -p /var/lib/glusterd/nfs/run/nfs.pid -l /var/log/glusterfs/nfs.log -S /var/run/d3557e241e521ea123bcdfb9ed54e30f.socket
	root      2882  0.0  1.2 295436 23492 ?        Ssl  14:06   0:00 /usr/sbin/glusterfs -s localhost --volfile-id gluster/glustershd -p /var/lib/glusterd/glustershd/run/glustershd.pid -l /var/log/glusterfs/glustershd.log -S /var/run/f06a6deb150e1c5c0e607ec357f085f4.socket --xlator-option *replicate*.node-uuid=8d865314-af12-4950-a784-6a5308ec501b
	root      2900  0.0  0.0  11744   924 pts/0    S+   14:09   0:00 grep --color=auto gluster
	root@server1:~#

::

	root@server1:~# netstat -tap | grep glusterfsd
	tcp        0      0 *:49152                 *:*                     LISTEN      2865/glusterfsd 
	tcp        0      0 server1.example.c:49152 server2.example:1020 ESTABLISHED 2865/glusterfsd 
	tcp        0      0 server1.example.co:1019 server1.example.c:24007 ESTABLISHED 2865/glusterfsd 
	tcp        0      0 server1.example.c:49152 server1.example.co:1023 ESTABLISHED 2865/glusterfsd 
	tcp        0      0 server1.example.c:49152 server2.example:1014 ESTABLISHED 2865/glusterfsd 
	tcp        0      0 server1.example.c:49152 server1.example.co:1022 ESTABLISHED 2865/glusterfsd 
	root@server1:~#

 

As a final test, to make sure the volume is available, run gluster volume info. As shown below::

	root@server1:~# gluster volume info
	 
	Volume Name: datapoint
	Type: Replicate
	Volume ID: 3fd7bcea-3ee5-41b4-9336-880a5c1527b7
	Status: Started
	Number of Bricks: 1 x 2 = 2
	Transport-type: tcp
	Bricks:
	Brick1: gluster1:/mnt/gluster
	Brick2: gluster2:/mnt/gluster
	root@server1:~#

 

By default, all clients can connect to the volume. If you want to grant access to client1.example.com (= 192.168.0.102) only, run::

	# gluster volume set testvol auth.allow 192.168.0.102

Please note that it is possible to use wildcards for the IP addresses (like 192.168.*) and that you can specify multiple IP addresses separated by comma (e.g. 192.168.0.102,192.168.0.103).

The volume info should now show the updated status::

	root@server1:~# gluster volume info

	Volume Name: testvol
	Type: Replicate
	Status: Started
	Number of Bricks: 2
	Transport-type: tcp
	Bricks:
	Brick1: server1.example.com:/data
	Brick2: server2.example.com:/data
	Options Reconfigured:
	auth.allow: 192.168.0.102
	root@server1:~#

 

 
Setting Up The GlusterFS Client

client1.example.com:

On the client, we can install the GlusterFS client as follows:

apt-get install glusterfs-client

Then we create the following directory:

mkdir /mnt/glusterfs

That's it! Now we can mount the GlusterFS filesystem to /mnt/glusterfs with the following command:

mount.glusterfs server1.example.com:/testvol /mnt/glusterfs

(Instead of server1.example.com you can as well use server2.example.com in the above command!)

You should now see the new share in the outputs of...

mount

root@client1:~# mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,relatime,size=10240k,nr_inodes=126813,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,noexec,relatime,size=102704k,mode=755)
/dev/mapper/server1-root on / type ext4 (rw,relatime,errors=remount-ro,user_xattr,barrier=1,data=ordered)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
tmpfs on /run/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=205400k)
/dev/sda1 on /boot type ext2 (rw,relatime,errors=continue)
rpc_pipefs on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw,relatime)
server1.example.com:/testvol on /mnt/glusterfs type fuse.glusterfs (rw,relatime,user_id=0,group_id=0,default_permissions,allow_other,max_read=131072)
fusectl on /sys/fs/fuse/connections type fusectl (rw,relatime)
root@client1:~#

... and...

df -h

root@client1:~# df -h
Filesystem                    Size  Used Avail Use% Mounted on
rootfs                         29G  1.2G   26G   5% /
udev                           10M     0   10M   0% /dev
tmpfs                         101M  240K  101M   1% /run
/dev/mapper/server1-root       29G  1.2G   26G   5% /
tmpfs                         5.0M     0  5.0M   0% /run/lock
tmpfs                         201M     0  201M   0% /run/shm
/dev/sda1                     228M   18M  199M   9% /boot
server1.example.com:/testvol   29G  1.2G   26G   5% /mnt/glusterfs
root@client1:~#

 

Instead of mounting the GlusterFS share manually on the client, you could modify /etc/fstab so that the share gets mounted automatically when the client boots.

Open /etc/fstab and append the following line:

vi /etc/fstab

[...]
server1.example.com:/testvol /mnt/glusterfs glusterfs defaults,_netdev 0 0

(Again, instead of server1.example.com you can as well use server2.example.com!)

To test if your modified /etc/fstab is working, reboot the client:

reboot

After the reboot, you should find the share in the outputs of...

df -h

... and...

mount

 
4 Testing

Now let's create some test files on the GlusterFS share:

client1.example.com:

touch /mnt/glusterfs/test1
touch /mnt/glusterfs/test2

Now let's check the /data directory on server1.example.com and server2.example.com. The test1 and test2 files should be present on each node:

server1.example.com/server2.example.com:

ls -l /data

root@server1:~# ls -l /data
total 0
-rw-r--r-- 1 root root 0 Sep 30 17:53 test1
-rw-r--r-- 1 root root 0 Sep 30 17:53 test2
root@server1:~#

Now we shut down server1.example.com and add/delete some files on the GlusterFS share on client1.example.com.

server1.example.com:

shutdown -h now

client1.example.com:

touch /mnt/glusterfs/test3
touch /mnt/glusterfs/test4
rm -f /mnt/glusterfs/test2

The changes should be visible in the /data directory on server2.example.com:

server2.example.com:

ls -l /data

root@server2:~# ls -l /data
total 8
-rw-r--r-- 1 root root 0 Sep 30 17:53 test1
-rw-r--r-- 1 root root 0 Sep 30 17:54 test3
-rw-r--r-- 1 root root 0 Sep 30 17:54 test4
root@server2:~#

Let's boot server1.example.com again and take a look at the /data directory:

server1.example.com:

ls -l /data

root@server1:~# ls -l /data
total 0
-rw-r--r-- 1 root root 0 Sep 30 17:53 test1
-rw-r--r-- 1 root root 0 Sep 30 17:53 test2
root@server1:~#

As you see, server1.example.com hasn't noticed the changes that happened while it was down. This is easy to fix, all we need to do is invoke a read command on the GlusterFS share on client1.example.com, e.g.:

client1.example.com:

ls -l /mnt/glusterfs/

root@client1:~# ls -l /mnt/glusterfs/
total 8
-rw-r--r-- 1 root root 0 Sep 30 17:53 test1
-rw-r--r-- 1 root root 0 Sep 30 17:54 test3
-rw-r--r-- 1 root root 0 Sep 30 17:54 test4
root@client1:~#

Now take a look at the /data directory on server1.example.com again, and you should see that the changes have been replicated to that node:

server1.example.com:

ls -l /data

root@server1:~# ls -l /data
total 0
-rw-r--r-- 1 root root 0 Sep 30 17:53 test1
-rw-r--r-- 1 root root 0 Sep 30 17:54 test3
-rw-r--r-- 1 root root 0 Sep 30 17:54 test4
root@server1:~#
