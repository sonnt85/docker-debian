## Debian Wheezy/Jessie version with some extra commands
 [![Docker Hub; nimmis/debian](https://img.shields.io/badge/dockerhub-nimmis%2Fdebian-green.svg)](https://registry.hub.docker.com/u/nimmis/debian)

This is a docker images with different versions of Debian with a working init process and syslog

### Why use this image

The unix process ID 1 is the process to receive the SIGTERM signal when you execute a 

	docker stop <container ID>

if the container has the command `CMD ["bash"]` then bash process will get the SIGTERM signal and terminate.
All other processes running on the system will just stop without the possibility to shutdown correclty

### my_init init script

In this container i have a scipt that handles the init process an uses the [supervisor system](http://supervisord.org/index.html) to start
the daemons to run and also catch signals (SIGTERM) to shutdown all processes started by supervisord. This is a modified version of
an init script made by Phusion. I've modified it to use supervisor in stead of runit. There are also two directories to run scripts
before any daemon is started.

#### Run script once /etc/my_runonce

All executable in this directory is run at start, after completion the script is removed from the directory

#### Run script every start /etc/my_runalways

All executable in this directory is run at every start of the container, ie, at `docker run` and `docker start`

#### Permanent output to docker log when starting container

Each time the container is started the content of the file /tmp/startup.log is displayed so if your startup scripts generate 
vital information to be shown please add that information to that file. This information can be retrieved anytime by
executing `docker logs <container id>`

### cron daemon

In many cases there are som need of things happening att given intervalls, default no cron processs is started
in the standard Debain image. In this image cron is running together with logrotate to stop the logdfiles to be
to big on log running containers.

### syslog-ng

No all services works without a syslog daemon, if you don't have one running those messages is lost in space,
all messages sent via the syslog daemon is saved in /var/log/syslog

### Docker fixes 

Also there are fixed (besideds the init process) assosiated with running Debian inside a docker container.

### New commands autostarted by supervisord

To add other processes to run automaticly, add a file ending with .conf  in /etc/supervisor/conf.d/ 
with a layout like this (/etc/supervisor/conf.d/myprogram.conf) 

	[program:myprogram]
	command=/usr/bin/myprogram

`myprogram` is the name of this process when working with supervisctl.

Output logs std and error is found in /var/log/supervisor/ and the files begins with the <defined name><-stdout|-stderr>superervisor*.log

For more settings please consult the [manual FOR supervisor](http://supervisord.org/configuration.html#program-x-section-settings)

#### starting commands from /etc/init.d/ or commands that detach with my_service

The supervisor process assumes that a command that ends has stopped so if the command detach it will try to restart it. To work around this
problem I have written an extra command to be used for these commands. First you have to make a normal start/stop command and place it in
the /etc/init.d that starts the program with

	/etc/init.d/command start or
	service command start

and stops with

        /etc/init.d/command stop or
        service command stop

Configure the configure-file (/etc/supervisor/conf.d/myprogram.conf)

	[program:myprogram]
	command=/my_service myprogram

There is an optional parameter, to run a script after a service has start, e.g to run the script /usr/local/bin/postproc.sh av myprogram is started

        [program:myprogram]
        command=/my_service myprogram /usr/local/bin/postproc.sh

### Output information to docker logs

The console output is owned by the my_init process so any output from commands woun't show in the docker log. To send a text from any command, either
at startup och during run, append the output to the file /var/log/startup.log, e.g sending specific text to log

	echo "Application is finished" >> /var/log/startup.log

or output from script

	/usr/local/bin/myscript >> /var/log/startlog.log


	> docker run -d --name debian nimmis/debian
	> docker logs debian
	*** open logfile
	*** Run files in /etc/my_runonce/
	*** Run files in /etc/my_runalways/
	*** Running /etc/rc.local...
	*** Booting supervisor daemon...
	*** Supervisor started as PID 9
	2015-08-04 11:34:06,763 CRIT Set uid to user 0
	*** Started processes via Supervisor......
	crond                            RUNNING    pid 13, uptime 0:00:04
	syslog-ng                        RUNNING    pid 12, uptime 0:00:04

	> docker exec debian sh -c 'echo "Testmessage to log" >> /var/log/startup.log'
	> docker logs debian
        *** open logfile
        *** Run files in /etc/my_runonce/
        *** Run files in /etc/my_runalways/
        *** Running /etc/rc.local...
        *** Booting supervisor daemon...
        *** Supervisor started as PID 9
        2015-08-04 11:34:06,763 CRIT Set uid to user 0
        *** Started processes via Supervisor......
        crond                            RUNNING    pid 13, uptime 0:00:04
        syslog-ng                        RUNNING    pid 12, uptime 0:00:04

	*** Log: Testmessage to log
        >
### Added som normaly used commands

There are a number of commands that most uses and adds to their build, in this build I've added som commonly used packages

Extra included packages are

- wget
- curl
- zip/unzip
- git
- vim
- nano
- psmisc
- less
- patch
- software-properties-common

### Installation

This continer should normaly run as a daemon i.e with the `-d` flag attached

	docker run -d nimmis/debian

but if you want to check if all services has been started correctly you can start with the following command

	docker run -ti nimmis/debian

the output, if working correctly should be

	docker run -ti nimmis/debian
	*** open logfile
	*** Run files in /etc/my_runonce/
	*** Run files in /etc/my_runalways/
	*** Running /etc/rc.local...
	*** Booting supervisor daemon...
	*** Supervisor started as PID 11
	2015-01-02 10:45:43,750 CRIT Set uid to user 0
	*** Started processes via Supervisor......
	crond                            RUNNING    pid 15, uptime 0:00:02
	syslog-ng                        RUNNING    pid 14, uptime 0:00:02

pressing a CTRL-C in that window  or running `docker stop <container ID>` will generate the following output

	*** Shutting down supervisor daemon (PID 11)...
	*** Killing all processes...

you can the restart that container with 

	docker start <container ID>

Accessing the container with a bash shell can be done with

	docker exec -ti <container ID> /bin/bash

### TAGs

This image only contains the 2 latest versions of Debian Wheezy and Jessie, the versions are
nimmis/debian:<tag> where tag is

- latest -  this gives the latest debian version (Jessie)
- jessie -  this gives the Debian Jessie version
- wheezy -  this gives the Debain Wheezy version


