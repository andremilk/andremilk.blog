---
published: true
layout: post
date: {}
---



The following posts will be an adaptation of a tutorial I've created for a intro lecture I gave about Docker and it's uses to help Computer Science students in their tasks. 

## Intro to Docker

It was created heavily based on the official Docker documentation (that you should definitely read) but in portuguese and with some minor aditions of some explaning with my words. The original document can be found [here](https://docs.google.com/document/d/1R2_vhAWnj8AqAtSOZX4WBiBPgxlnA4p0QW0Y0WvQN50/edit?usp=sharing).

## Introduction (brief explanation)
Containers are not some sort of [new technology and neither a linux invention](https://docs.google.com/document/d/1R2_vhAWnj8AqAtSOZX4WBiBPgxlnA4p0QW0Y0WvQN50/edit?usp=sharing). 

However Docker has made it fairly simple to create and use linux containers and so in the last couple of years there's been a huge hype around it. 
I've discovered Docker by accident and thought that perhaps it would be useful for me to solve a dependency problem that a project I was working with had. It was not on my top list of solutions for that problem at the time, but I was interested. 
After playing around with it a little I was told to look for other linux container technologies and have a [look at them...](http://slides.com/andreleite/teste-de-apresentacao/#/) I was in awe of how easy using docker was compared to the alternatives I've tried ([lxc](https://linuxcontainers.org/lxc/introduction/), [linux-vserver](http://linux-vserver.org/Welcome_to_Linux-VServer.org) and [lmctfy](https://github.com/google/lmctfy)).

The thing about (docker) containers is that they aim to create an isolation between processes, namespaces and so on. It does it without the need to add another operating system on top of it and not to mention any hardware at all.
***
## Dockerhub
[Dockerhub](https://hub.docker.com) is a repository of docker images packing a bunch of features and integrations with services like [github](https://github.com/) or [bitbucket](https://bitbucket.org/) to use [automated builds](https://docs.docker.com/docker-hub/builds/) and so on. They offer a free plan that, at the time I'm writing this, enables you to have one private repository. 
You may deploy your own Docker image [registry](https://docs.docker.com/registry/) if you desire to do so.
***
## Installation
The docker documentation about different [installation procedures](https://docs.docker.com/installation/) for different linux distributions or even Windows or OSX.

I am using Arch Linux and installed docker through pacman:

	$ sudo pacman -S docker
If you like you could install the AUR package:

	$ yaourt -S docker-git

After installing docker you may choose to add your user to the docker group in order to use it without sudo:

	$ sudo usermod -aG docker $(whoami)
    $ newgrp docker

After that you should be able to run docker corretly.. If by some reason you can't you may try to restart the docker service:

	$ sudo systemctl restart docker

Let's see: 

	[dekozo@dekarch ~]$ docker ps
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
	[dekozo@dekarch ~]$ 


Yep, it works!
