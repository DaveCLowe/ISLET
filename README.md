Isolated, Scalable, & Lightweight Environment for Training
=========

Make training a smooth process...#NoMoreVMs <br>

A container system for teaching Linux based software with minimal participation and <br>
configuration effort. The participation barrier is set very low, students only need an SSH client.

![ISLET Screenshot](http://jonschipp.com/islet/islet.png)

#### Uses

* Event training
* Staff training
* Capture the flag competitions
* Trying out tools in a containerized environment
* Development environments

## Demo
You can quickly try out ISLET on some of my dev systems. Password is demo
```shell
ssh demo@islet1.jonschipp.com
ssh demo@islet2.jonschipp.com
```

## Design

####Simplified Diagram
![ISLET Diagram](http://jonschipp.com/islet/islet_diagram.jpg)

####Detailed Flowchart
![ISLET Flowchart](http://jonschipp.com/islet/islet_flowchart.png)

## Installation

Installation of ISLET is very simple and it can be done in two ways:

On the host operation system
```shell
make install
```
Or as a Docker container which requires little to no modification to the host
```shell
make install-contained
```

![ISLET Make Screenshot](http://jonschipp.com/islet/islet_make.png)

Target:         |    Description:
----------------|----------------
install         | Install ISLET: install-files + configuration
install-contained | Install ISLET as a container, no modification to host system
update		| Downloads and install new code (custom changes to default files will be overwritten)
uninstall       | Uninstall ISLET (Recommended to backup your stuff first)
mrproper 	| Removes files that did not come with the source
install-docker  | Installs latest Docker from Docker repo (Debian/Ubuntu only)
docker-config   | Reconfigures Docker storage backend to limit container and image sizes
user-config     | Configures a user account called demo w/ password dem
security-config | Configures sshd and pam_limits with islet relevant security in mind
iptables-config | Installs iptables ruleset

GNU make accepts arguments if you want a customized installation (*not supported*):
```shell
make install INSTALL_DIR=/usr/local/islet USER=training
make user-config INSTALL_DIR=/usr/local/islet USER=training PASS=training
make security-config INSTALL_DIR=/usr/local/islet USER=training
make uninstall INSTALL_DIR=/usr/local/islet USER=training
```

Variable:       |    Description:
----------------|----------------
CONFIG_DIR      | ISLET config files directory (def: /etc/islet)
INSTALL_DIR     | ISLET installation directory (def: /opt/islet)
CRON		| Directory to place islet crontab file (def: /etc/cron.d)
USER		| User account created with user-config target (def: demo)
PASS		| User account password created with user-config target (def: demo)
SIZE		| Maximum container and image size with configure-docker target (def: 2G)
IPTABLES	| Iptables ruleset (def: /etc/network/if-pre-up.d/iptables-rules)
NAGIOS      | Location of nagios plugins (def: /usr/local/nagios/libexec)
PORT        | The SSH port on the host when installing ISLET as a container (def: 2222)
PACKAGE     | Type of package to build for `make package` (def: deb)

## Updating

Updating an existing ISLET installation is very simple:

For an existing host installation (`make install`):
```shell
make update
```
For an existing container installation (`make install-contained`):
```shell
docker pull jonschipp/islet
```
### Dependencies

* Linux, Bash, Cron, OpenSSH, Make, SQLite, and Docker

The configure script will check for dependencies
```shell
./configure
```

![ISLET Configure Screenshot](http://jonschipp.com/islet/islet_configure.png)

Typically all you need is make, sqlite, and docker (for Debian/Ubuntu):
```shell
apt-get install make sqlite
make install-docker
```

The included installation scripts are designed to work with Debian/Ubuntu systems.

**Note:** Installing ISLET as container (`make install-contained`) only requires Docker

#### Ubuntu

The following make targets will install docker and configure the system with security in mind for the Docker process.
It is designed to be a quick way to get a working system with a good configuration.

Install ISLET on the host:
```shell
make install-docker	# Installs latest Docker
make configure-docker   # Limits image and container sizes by rebuilding storage backend (Skip if using Docker 1.4+)
make security-config    # Configure islet relevant security with sshd and pam_limits
```

Install ISLET as a container on the host:
```shell
make install-docker	# Installs latest Docker
make install-contained	# Installs ISLET as a container
```

#### Manual

For manual installation and configuration of dependencies to your liking i.e. not using the system make targets.

* Install Docker:
```shell
apt-get install docker
yum install docker
```

* Configure user account for training (this is given to students to login):
```shell
useradd --create-home --shell /opt/islet/bin/islet_shell training
echo "training:training" | chpasswd
groupadd docker
groupadd islet
gpasswd -a training docker
gpasswd -a training islet
```

See the SECURITY file more information on manually securing the system.

### Post-Install First Steps

Post-installation first steps

1. Set STORAGE_BACKEND in /etc/islet/islet.conf to match your Docker storage driver
```
docker info | grep Storage
```
2. Change the password for the islet user (default: demo)
```
passwd demo
```
3. Create a Docker image for your training environment (see Adding Training Environments)
```
cat <<EOF > Dockerfile
# Build image for C programming
FROM      ubuntu
MAINTAINER Jon Schipp <jonschipp@gmail.com>

RUN adduser --disabled-password --gecos "" demo
RUN apt-get update -qq
RUN apt-get install -y build-essential
RUN apt-get install -y git vim emacs nano tcpdump gawk rsyslog
RUN apt-get install -y --no-install-recommends man-db
EOF

docker build -t gcc-training - < Dockerfile
```
4. Create an ISLET configuration file for the Docker image (see Adding Training Environments)
```
make template > /etc/islet/gcc.conf
vim /etc/islet/islet/gcc.conf
# Set IMAGE variable to name of docker image (e.g. gcc-training)
# Set VIRTUSER variable to name of user in docker image that the student will become (e.g. demo)
```

# Adding Training Environments

See Docker's [image documentation](http://docs.docker.com/userguide/dockerimages)

 1. Build or pull in a new Docker image

 2. Create an ISLET config file for that image. You can use `make template` for an example.

 3. Place it in /etc/islet with a .conf extension.

 It should now be available from the selection menu upon login.

![ISLET Configs Screenshot](http://jonschipp.com/islet/islet_configs.png)

# Administration

* Global configuration file: */etc/islet/islet.conf*
* Per-image configuration file: */etc/islet/$IMAGE.conf*

Per-image configs overwrite the variables specified in the global config file.
For each Docker image you want available for use by ISLET, create an image file with a .conf extension and place it in the /etc/islet/ directory.
These images will be selectable from the ISLET menu after authentication via SSH.

Common Tasks:

* Add another system account for ISLET (used to remotely access e.g. ssh)

```
useradd --create-home --shell /opt/islet/bin/islet_shell training
echo "training:training" | chpasswd
gpasswd -a training docker
gpasswd -a training islet
```

* Change the password of a container user (Not a system account).

```
    $ PASS=$(echo "newpassword" | sha1sum | sed 's/ .*//)
	$ sqlite3 /var/tmp/islet.db "UPDATE accounts SET password='$PASS' WHERE user='jon';"
	$ sqlite3 /var/tmp/islet.db "SELECT password FROM accounts WHERE user='jon';"
	aaaaaaa2a4817e5c9a56db45d41ed876e823fcf|1413533585

```

* Configure container and user lifetime (e.g. conference duration)

  1. Specify the number of days for user account and container lifetime in:

```
        $ grep ^DAYS /etc/islet/islet.conf
        DAYS=3 # Length of the event in days
```

  Removal scripts are cron jobs that are scheduled in /etc/cron.d/islet

* Allocate more or less resources for containers, and control other container settings.
  These changes will take effect for each newly created container.
  - System and use case dependent

```
    $ grep -A 5 "Container config /etc/islet/brolive.conf
	# Container Configuration
	VIRTUSER="demo"                                         # Account used when container is entered (Must exist in container!)
	CPU="1"                                                 # Number of CPU's allocated to each container
	RAM="256m"                                              # Amount of memory allocated to each container
	HOSTNAME="bro"	                                      	# Set hostname in container. PS1 will end up as $VIRTUSER@$HOSTNAME:~$ in shell
	NETWORK="none"                                          # Disable networking by default: none; Enable networking: bridge
	DNS="127.0.0.1"                                         # Use loopback when networking is disabled to prevent error messages from resolver
	MOUNT="-v /exercises:/exercises:ro"			# Mount point(s), sep. by -v: /src:/dst:attributes, ro = readonly (avoid rw)
	OPTIONS="--cap-add=NET_RAW --cap-add=NET_ADMIN"		# Apply any other options you want passed to Docker run here
	MOTD="Training materials are in /exercises"             # Message of the day is displayed before container launch and reattachment
```

* Adding, removing, or modifying exercises

  1. Make changes in /exercises on the host's filesystem

  *  Changes are immediately available for new and existing containers


# Case Study

The precursor to ISLET was used to aid the instructers in teaching the Bro platform at BroCon14.

Workflow:
* Install ISLET and dependencies
* Build Docker image containing Bro (docker pull broplatform/brolive)
* Write a ISLET config file for the Bro image
* Set a banner in the ISLET config file for light branding (logo)
* Hand out the demo account credentials to your students so they can SSH in
* Instruct them on the software

Here's a brief demonstration:

```
        $ ssh demo@live.bro.org

        Welcome to Bro Live!
        ====================

            -----------
          /             \
         |  (   (0)   )  |
         |            // |
          \     <====// /
            -----------

        A place to try out Bro.

        Are you a new or existing user? [new/existing]: new

        A temporary account will be created so that you can resume your session. Account is valid for the length of the event.

        Choose a username [a-zA-Z0-9]: jon
        Your username is jon
        Choose a password:
        Verify your password:
        Your account will expire on Fri 29 Aug 2014 07:40:11 PM UTC

        Enjoy yourself!
        Training materials are located in /exercises.
        e.g. $ bro -r /exercises/beginner/http.pcap

        demo@bro:~$ pwd
        /home/demo
        demo@bro:~$ which bro
        /usr/local/bro/bin/bro
```

More info:
[Mailing List] (https://groups.google.com/d/forum/islet)
