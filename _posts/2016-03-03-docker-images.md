---
published: true
layout: post
date: 2016-03-03T12:00:00.000Z
---







What exactly are Docker images? Where and how do they live? How do they reproduce? 

## Docker images

[Docker images](https://docs.docker.com/engine/userguide/containers/dockerimages/) can be seen as __"recipes"__ of how to create a container.

Whenever we enter a command like:

	[dekozo@dekarch ~]$ docker run ubuntu:14.04 /bin/echo "Hello World"
    
An image __(in this case, ubuntu with the tag 14.04)__ will be used to create a new container and run something inside of it. **Tags** can be understood as nicknames for certain a __"version"__ of our image. 

If you create an image without specifying a tag, **latest** will be the default tag. You should always tag your images, so that way you can easily use the right __"version"__ of your image. 

We can create images in docker in two ways:

- Using [**docker commit**](https://docs.docker.com/engine/reference/commandline/commit/) on a container
- [Dockerfiles](https://docs.docker.com/engine/reference/builder/)

The Dockerfile alternative is considered a better alternative for it is easier to audit what goes on and modify anything on the building process of an image.
Dockerfiles are simple text files with instructions of how to build an image. 
We will get to that later...

How can I see what images are available to me locally?

	[dekozo@dekarch ~]$ docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
	gliderlabs/alpine   latest              0fbc31a65b4b        10 days ago         4.79 MB
	ubuntu              latest              3876b81b5a81        6 weeks ago         187.9 MB
	[dekozo@dekarch ~]$

What if I want to find other images? Well, I would search for it on the [Dockerhub](http://hub.docker.com/):
![Screenshot2016-45-0309:45:47.png]({{site.baseurl}}/_posts/Screenshot2016-45-0309:45:47.png)

But you could search for them using the docker client:
	
	[dekozo@dekarch ~]$ docker search redis
	NAME                      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
	redis                     Redis is an open source key-value store th...   1729      [OK]       
	torusware/speedus-redis   Always updated official Redis docker image...   27                   [OK]
	sameersbn/redis                                                           25                   [OK]
	bitnami/redis             Bitnami Redis Docker Image                      13                   [OK]
	cleardevice/redis         Redis Dockerfile for trusted automated Doc...   10                   [OK]

And to download the image for later use:

	[dekozo@dekarch ~]$ docker pull redis
	Using default tag: latest
	latest: Pulling from library/redis
	7268d8f794c4: Pull complete 
	a3ed95caeb02: Pull complete 
	9759e73902d9: Pull complete 
	b7d5780e9418: Pull complete 
	ddc594ddd54a: Pull complete 
	3b65d25d9dbb: Pull complete 
	0a23eeaae213: Pull complete 
	6457592763e5: Pull complete 
	8f359895dbf8: Pull complete 
	Digest: sha256:11b4db3e6e6215399c62b2430e77ae1d910669e7be16d4e8bc77783c400be47a
	Status: Downloaded newer image for redis:latest
	[dekozo@dekarch ~]$
    
[Docker images work with layers](https://docs.docker.com/engine/understanding-docker/#how-does-a-docker-image-work). If you have an image with different tags and want to download the next one, docker will not have to download the whole image again, it's optimized to keep images small in size.

It's possible to create an image from a container with: 

	$ docker commit id image_name:tag_name
    
For instance:

	[dekozo@dekarch ~]$ docker ps -a
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
	859d005f0f17        ubuntu              "/bin/echo 'Hello Wor"   4 seconds ago       Exited (0) 3 seconds ago                       gigantic_murdock
	[dekozo@dekarch ~]$ 	

Here we have an exited container and if we would like to commit it and make a new image:
    
	[dekozo@dekarch ~]$ docker commit gigantic_murdock my-image:my-tag
	sha256:f31a3e61b4c34bf117dcb76506317a44b2373ac242cf740017b9823aa72f1fe5
	[dekozo@dekarch ~]$ docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
	my-image            my-tag              f31a3e61b4c3        4 seconds ago       187.9 MB
	redis               latest              9afdbcd3766e        7 days ago          151.3 MB
	gliderlabs/alpine   latest              0fbc31a65b4b        10 days ago         4.79 MB
	ubuntu              latest              3876b81b5a81        6 weeks ago         187.9 MB
	[dekozo@dekarch ~]$ 
    
***

### Dockerfiles

The best way to share and create images is using Dockerfiles. 

[This Dockerfile](https://hub.docker.com/r/ingensi/play-framework/~/dockerfile/) enables us to build an image to run Java [Playframework!](https://www.playframework.com/) apps inside a container without having to worry about setting up the whole environment.

Let's examine it:

	FROM ingensi/oracle-jdk 
	MAINTAINER Ingensi labs <contact@ingensi.com> 
	RUN yum update -y && yum install -y unzip 
	RUN curl -O http://downloads.typesafe.com/typesafe-activator/1.3.6/typesafe-activator-1.3.6.zip 
	RUN unzip typesafe-activator-1.3.6.zip -d / && rm typesafe-activator-1.3.6.zip && chmod a+x /activator-1.3.6/activator 
	ENV PATH $PATH:/activator-1.3.6 
	EXPOSE 9000 8888 
	RUN mkdir /app 
	WORKDIR /app 
	CMD ["activator", "run"]
    
The **FROM** instruction will tell docker which __base image__ will be used. In this case it will be the image **oracle-jdk** from the user **ingensi** on dockerhub.

**MAINTAINER** the way to tell who created the dockerfile. It's common practice to use your name and your email.

**RUN** will execute given commands. On the example above there are four RUN instructions. You can use the && operators to have one command being executed after the other (if the previous command performed correctly) or you can have multiple commands in the form:

	RUN command1 \
        command2 \
        .        \
        .        \
        commandN
You can have the **RUN** instruction in two forms too:

	RUN command
    
or 

	RUN ["command", "parameter1", "parameter2", ... "parameterN"]
    
The main difference is that **the first alternative** will run your command through __/bin/bash__ just like `/bin/bash -c "command"` and **the second alternative** will run your command like `/bin/command paramater1 parameter2 ... parameterN`.

**ENV** sets environment variables for the context of building the image. It has two forms:

	ENV VARIABLE VALUE
    
and

	ENV VARIABLE=VALUE
    
Here the differences are that **the first alternative** can only set a variable per line, in contrast **the second alternative** can set multiple variables per line. You can also use variables inside your Dockerfile and use a [few bash modifiers](https://docs.docker.com/engine/reference/builder/#environment-replacement).

**EXPOSE** will tell which ports are to be exposed at runtime. **Attention:** this instruction will not expose the ports but merely tell docker what ports are to be exposed if you use **docker run** with __-p__ or __-P__ parameters (**more on that later**).

**WORKDIR** is an instruction to set the current working directory for the context of building an image. It can work in two different ways:
- Having it set an absolut path
- Having it set a relative path

For instance:

	WORKDIR /app
	/app   WORKDIR abc
	└── abc   WORKDIR def
	    └── def (We end up here, at /app/abc/def)
        
**CMD** will tell docker what command should be executed when you create a container using this image without passing any command. For instance:

	[dekozo@dekarch ~]$ docker run -d redis
	4214e9b9e9c78d5c7de86e3047d9373637c925d5832db404ff3282fd26bebb7c
	[dekozo@dekarch ~]$ docker ps
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
	4214e9b9e9c7        redis               "/entrypoint.sh redis"   4 seconds ago       Up 2 seconds        6379/tcp            sick_bell
	[dekozo@dekarch ~]$ 
    
We can see that no command was given when creating and running this container (that uses redis image... the __-d__ flag tells docker to run it dettached, in the background). The command set on the redis Dockerfile might be something like

	CMD ["/entrypoint.sh", "redis"]
    
If we wish to execute another command we could simply do:

	[dekozo@dekarch ~]$ docker run redis /bin/echo "This is a test"
	This is a test
	[dekozo@dekarch ~]$ 
    
There is also an instruction named **ENTRYPOINT** which gives a little bit more work to change during runtime (but it is possible). You could understand **CMD** as the default parameters to **ENTRYPOINT**

If we had in our Dockerfile the following:

	ENTRYPOINT ["/bin/echo"]
    CMD ["this is the default value"]
    
We could use the same image to echo anything we choose at runtime just by doing:

	docker run our-image new string
    
It would evaluate to `/bin/echo new string`.

There's a good explanation [here](https://www.ctl.io/developers/blog/post/dockerfile-entrypoint-vs-cmd/).

There are a bunch of Dockerfile examples available online. You can(should) also see the [Dockerfile best practices](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/) created by Docker.
   
Ok, but how do I use a Dockerfile to actually build an image?

	docker build -t user/image-name:tag path/to/Dockerfile
    
You can tag the same built image with different tags at the same time. This is useful if you want to have a **latest** tag and another specific tag:

	docker build -t user/image-name -t user/image-name:my-tag path/to/Dockerfile

When we execute this command all the steps listed on the Dockerfile will be taken and your image will be created. If we wish to make our image available to others we can push it to dockerhub. But first we have to login:

	[dekozo@dekarch ~]$ docker login
	Username: andreguimaraesleite
	Password:
	Email: andre.guimaraes.leite@gmail.com
	WARNING: login credentials saved in /home/dekozo/.docker/config.json
	Login Succeeded
	[dekozo@dekarch ~]$
    
and after that we can push our image:

	docker push image-name:tag
    
If you wish to **remove an image locally** you can do this just like:

	[dekozo@dekarch ~]$ docker rmi -f redis
	Untagged: redis:latest
	Deleted: sha256:9afdbcd3766e4564e5a420d6890ab09e3d1ee1ed8bfb04a58af7b4a62dee9402
	[dekozo@dekarch ~]$
    
the __-f__ option will force the image to be removed. If there is any container created (exited or running) using that image docker will report it and won't delete the image.
