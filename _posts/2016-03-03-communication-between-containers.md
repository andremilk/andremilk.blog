---
published: false
layout: post
---

My container is lonely... it would like to talk to other containers. How could I arrange that?

## Communication between containers and more

It's possible to have applications running inside different containers talk to each other. This could be achieved through different ways and depending on what you want to accomplish one way might be better than the others.

***
### Port forwarding

You can have access to a service running inside a container making use of port forwarding

	[dekozo@dekarch ~]$ docker run -d -p 80:5000 training/webapp python app.py
	bcc27f2b253e9533f2d934cc92d5e4478a79c0e753603374a3c73c457d2ac01c
	[dekozo@dekarch ~]$ docker ps
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
	bcc27f2b253e        training/webapp     "python app.py"     8 seconds ago       Up 7 seconds        0.0.0.0:80->5000/tcp   cocky_torvalds
	[dekozo@dekarch ~]$ docker port cocky_torvalds
	5000/tcp -> 0.0.0.0:80
	[dekozo@dekarch ~]$ curl localhost:80
	Hello world![dekozo@dekarch ~]$
    
What happened was that docker ran a container in the background using the **training/webapp** image, with the command `python app.py` and establishing that whatever gets to the port 80 on your **host** is forwarded inside the container to port 5000.
The command **docker port** shows us what ports are exposed on a given container.

***

But what if I want to have two containers talking to each other? How can i do that?
You could either use links or create a network for your containers.

### [Links](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/#communication-across-links)

Links are setup when creating containers and they work through modifications on **/etc/hosts** along with environment variables. 

The downside of links is that if you have a containerA linked to containerB, containerA can be accessed by containerB but not the other way around. Let's see.

	[dekozo@dekarch ~]$ docker run -dt --name containerA ubuntu /bin/bash
	f3f191e842ead41ebd496fd96478215c93a34c9730478b61298876bb70962524
	[dekozo@dekarch ~]$ docker run -dt --name containerB --link containerA:containerA ubuntu /bin/bash
	6ea6cd1d99acbc15a78341bdade325792afcc3bbf680600832b4737010547b0a
	[dekozo@dekarch ~]$ docker exec containerB /bin/ping containerA
	PING containerA (172.17.0.3) 56(84) bytes of data.
	64 bytes from containerA (172.17.0.3): icmp_seq=1 ttl=64 time=0.072 ms
	64 bytes from containerA (172.17.0.3): icmp_seq=2 ttl=64 time=0.054 ms
	64 bytes from containerA (172.17.0.3): icmp_seq=3 ttl=64 time=0.043 ms
	^C[dekozo@dekarch ~]$
    
and 

	[dekozo@dekarch ~]$ docker exec containerB cat /etc/hosts
	127.0.0.1    localhost
	::1    localhost ip6-localhost ip6-loopback
	fe00::0    ip6-localnet
	ff00::0    ip6-mcastprefix
	ff02::1    ip6-allnodes
	ff02::2    ip6-allrouters
	172.17.0.3    containerA f3f191e842ea
	172.17.0.4    6ea6cd1d99ac
	[dekozo@dekarch ~]$
    
but...

	[dekozo@dekarch ~]$ docker exec containerA /bin/ping containerB
	ping: unknown host containerB
	[dekozo@dekarch ~]
    
***
### Networks

Docker has three default networks when you've just installed it. 
	
    [dekozo@dekarch ~]$ docker network ls
	NETWORK ID          NAME                DRIVER
	db5a0a41aa68        bridge              bridge              
	4d81ccf2a281        none                null                
	29222e749f5f        host                host                
	[dekozo@dekarch ~]$
    
- bridge is the default network for the docker0 interface
- none is a network without any interface
- host is the network which the host system is


If you add a container to the **host** network, you shall have something like this:

	[dekozo@dekarch ~]$ docker run -it --name test-host-network --net=host ubuntu /bin/bash
	root@dekarch:/# cat /etc/hosts
	#
	# /etc/hosts: static lookup table for host names
	#

	#<ip-address>    <hostname.domain.org>    <hostname>
	127.0.0.1    localhost.localdomain    localhost
	::1        localhost.localdomain    localhost

	# End of file
	root@dekarch:/# ifconfig
	docker0   Link encap:Ethernet  HWaddr 02:42:af:89:63:63  
	          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
    	      inet6 addr: fe80::42:afff:fe89:6363/64 Scope:Link
	          UP BROADCAST MULTICAST  MTU:1500  Metric:1
	          RX packets:69 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:39 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:0
	          RX bytes:4542 (4.5 KB)  TX bytes:2817 (2.8 KB)

	lo        Link encap:Local Loopback  
	          inet addr:127.0.0.1  Mask:255.0.0.0
	          inet6 addr: ::1/128 Scope:Host
	          UP LOOPBACK RUNNING  MTU:65536  Metric:1
	          RX packets:473 errors:0 dropped:0 overruns:0 frame:0
    	      TX packets:473 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:0
	          RX bytes:36321 (36.3 KB)  TX bytes:36321 (36.3 KB)
	
	wlp3s0    Link encap:Ethernet  HWaddr 5c:e0:c5:c7:c4:1e  
	          inet addr:192.168.25.227  Bcast:192.168.25.255  Mask:255.255.255.0
          	inet6 addr: fd09:b8c1:65cf:1234:5ee0:c5ff:fec7:c41e/64 Scope:Global
          	inet6 addr: 2804:7f7:d083:c413:5ee0:c5ff:fec7:c41e/64 Scope:Global
          	inet6 addr: fe80::5ee0:c5ff:fec7:c41e/64 Scope:Link
          	UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          	RX packets:280257 errors:0 dropped:0 overruns:0 frame:0
          	TX packets:124965 errors:0 dropped:0 overruns:0 carrier:0
          	collisions:0 txqueuelen:1000
          	RX bytes:349922940 (349.9 MB)  TX bytes:17093372 (17.0 MB)

	root@dekarch:/#


which is exactly the same as my host configurations.
You can inspect networks on docker using `docker network inspect name-of-network`.

#### User defined networks
Another way is to have your containers in the same user defined network. The simplest network kind you can make is a **bridge** type. All of your containers should be on the same docker host and they may communicate between them just by referencing their names. For instance

	[dekozo@dekarch ~]$ docker run --net=bridge-test -itd --name container1
	
	[dekozo@dekarch ~]$ docker run --net=bridge-test -itd --name container1 ubuntu
	51e3a5ff9481dd2657566f13472be63fe64094fb1003cf2bccf78f35857ea256
	[dekozo@dekarch ~]$ docker run --net=bridge-test -itd --name container2 ubuntu
	0d317a819fae98780503206f44fb0c2dfde9de0ba7de537e53eaec8276e1622c

	[dekozo@dekarch ~]$ docker exec -it container1 ping -c 3 container2
	PING container2 (172.18.0.3) 56(84) bytes of data.
	64 bytes from container2.bridge-test (172.18.0.3): icmp_seq=1 ttl=64 time=0.038 ms
	64 bytes from container2.bridge-test (172.18.0.3): icmp_seq=2 ttl=64 time=0.064 ms
	64 bytes from container2.bridge-test (172.18.0.3): icmp_seq=3 ttl=64 time=0.092 ms
	
	--- container2 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 1998ms
	rtt min/avg/max/mdev = 0.038/0.064/0.092/0.023 ms
	[dekozo@dekarch ~]$ docker exec -it container2 ping -c 3 container1
	PING container1 (172.18.0.2) 56(84) bytes of data.
		64 bytes from container1.bridge-test (172.18.0.2): icmp_seq=1 ttl=64 time=0.043 ms
	64 bytes from container1.bridge-test (172.18.0.2): icmp_seq=2 ttl=64 time=0.047 ms
	64 bytes from container1.bridge-test (172.18.0.2): icmp_seq=3 ttl=64 time=0.040 ms
	
	--- container1 ping statistics ---
	3 packets transmitted, 3 received, 0% packet loss, time 1998ms
	rtt min/avg/max/mdev = 0.040/0.043/0.047/0.006 ms
	[dekozo@dekarch ~]$


Another type of network **(that I did not include on the lecture I gave to simplify things)** is called an **overlay network**. I intend to make some experiments with this type of network later on, but that's for another time.

