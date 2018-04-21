### THIS PAGE IS UNDER CONSTRUCTION -- PLEASE IGNORE FOR NOW ###

# How to Attach Eclipse to a Remote Running iRODS System #

This document demonstrates how to attach the Eclipse IDE to an iRODS system (icommand client and server) on a remote system. 

The local system (**akelly1** in this document) is the development system, and includes a full set of iRODS sources (from github) that are built as a full package build, to provide us with debug objects to install and run on the target remote system (**akellylt1** in this document).  Since both **akelly1** and **akellylt1** are Ubuntu 16.04 systems, the packages we focus on will be .deb files.  An equivalent process for Redhat derived systems (like CentOS, RHEL, etc) would involve .rpm packages.

The connection to the remote system will be based on **ssh**.  It is recommended that ssh keys be installed to avoid being challenged for a password on every connection.  YMMV.

Following the process in this document, we will create an eclipse project that targets an existing icommand linux executable (**iput**), and create a remote debug launch configuration which can launch and debug the utility on a remote system.  We will then create a separate project for the **irodsServer** process as an executable, and then create a remote debug configuration that will allow us to attach eclipse to the remote running **irodsServer** which interacts with our **iput** utility.  None of these Eclipse configurations allows building the actual sources - that's not the intent.  They're used for the purpose of allowing the debugger access to the sources as programs are being debugged.


### The Thing with Eclipse ###
 
Eclipse has many features, configuration options, plugins, and supports different platforms and languages. Not all of them work quite as advertised, and it's easy to get lost. It also has many flavors based on the language perspective chosen. The point is (in my opinion), load up on patience.  


### Assumptions, Caveats and References ###

Before tackling this project, it is recommended that you start with the following knowledge base:

[Installing and running iRODS](https://docs.irods.org/master/getting_started/installation/ "https://docs.irods.org/master/getting_started/installation/")

[Building from source: A guide for developers who want to further work on iRODS](https://github.com/d-w-moore/irods-dev-orientation/blob/master/README.md "https://github.com/d-w-moore/irods-dev-orientation/blob/master/README.md")

**Eclipse How-To's.**  

In fact, we rely on the setup for the projects described below in order to start this project:

[Eclipse Odds and Ends](https://github.com/andrew-irods/How-To/blob/master/Eclipse-notes.md "https://github.com/andrew-irods/How-To/blob/master/Eclipse-notes.md")

[How to Create an Eclipse project for /usr/bin/ireg](https://github.com/andrew-irods/How-To/blob/master/Eclipse-attach-to-icommand-executable.md)

[How to Attach Eclipse to a Running irodsServer Process](https://github.com/andrew-irods/How-To/blob/master/Eclipse-attach-to-irodsServer.md)

**Also, these won't hurt:**

[How to Attach Eclipse to a Running Process](https://github.com/andrew-irods/How-To/blob/master/Eclipse-attach-to-running-process.md "https://github.com/andrew-irods/How-To/blob/master/Eclipse-attach-to-running-process.md")

[Youtube video:  GDB Remote Debugging using LUbuntu Linux and Eclipse CDT Neon 3](https://www.youtube.com/watch?v=KbN1UyybsV4  "https://www.youtube.com/watch?v=KbN1UyybsV4")


### Versions ###

The eclipse version used here is:

~~~
Eclipse IDE for C/C++ Developers
Version: Oxygen.3 Release (4.7.3RC3)
~~~

Running with Oracle Java 9:

~~~
$ java --version
java 9.0.4
Java(TM) SE Runtime Environment (build 9.0.4+11)
Java HotSpot(TM) 64-Bit Server VM (build 9.0.4+11, mixed mode)
~~~

This document assumes sufficient familiarity with Eclipse so that long winded descriptions with screen-shots showing where menus are and how to invoke them are not included.  To find a specific option or menu, examine the referenced How-To's listed above (they are long winded with screen-shots).


### Build and copy all iRODS .deb files to remote system ###

Once we have built the irods server and icommands, we should have two directories, one for each, as well as the corresponding build directories with the results of each build. For the purpose of this document, we'll use the environment variable **$work** as the base directory for all source and build directories (on **akelly1** this is "~/github"):

(These could be any directory path names):

**$work/irods** - the directory where the irods server sources were installed (from [iRODS on Github](https://github.com/irods/irods "https://github.com/irods/irods")).

**$work/bld_irods** - the directory where the artifacts from the build of the irods server reside.

**$work/irods\_client\_icommands** - the directory where the irods icommand sources were installed (from [iRODS Client iCommands on Github](https://github.com/irods/irods_client_icommands "https://github.com/irods/irods_client_icommands")). 

**$work/bld_icmds** - the directory where the artifacts from the build of the irods icommands reside. 

Also, there is an expectation here that the remote system (**akellylt1** in this case) has been installed with the standard iRODS packages in a regular configuration.  This includes an iRODS administration user called **irods** which exists on that system with the home directory of **/var/lib/irods** (typically with an unknown password); binaries installed under **/usr/bin**, **/usr/sbin**, **/var/lib/irods/**, etc. 

In preparation of the coming debug session, the target system **akellylt1** should be accessed, a password for the **irods** user added, and ssh keys installed under **~/.ssh/**.

**Note:** There are other ways to do this (for example, logging in as another user who has iRODS admin authority, but for this project we're simply following this process.

For example, access the remote irods system (below, 172.20.0.104 is the ip address for akellylt1, where andrew is a regular linux user with "**sudo**" privileges):

~~~
$ ssh andrew@172.20.0.104
# uname -n                         # This is our remote system
akellylt1
$ sudo su
<password>
# passwd irods
<negotiate the new irods password>
# su - irods
$ pwd
/var/lib/irods
$ ./irodsctl status
irodsServer :
  Process 1431
  Process 1432
irodsReServer :
  Process 1434
$ 
~~~

As you can see above, you are logged in as the user **irods**, with the current working directory set at **/var/lib/irods**, and you can proceed with entering ssh keys on both the target system, as well as on your own development system if you wish.

This is a good point in the process to adjust the Yama linux security module ptrace scope variable (ref. [https://www.kernel.org/doc/Documentation/security/Yama.txt](https://www.kernel.org/doc/Documentation/security/Yama.txt)).  While you are still superuser on the target system, do this:

~~~
$ uname -n                          $ This is our remote system
akellylt1
$ whoami
$ irods
$ exit                              $ Go back to being superuser
#
# sysctl kernel.yama.ptrace_scope
kernel.yama.ptrace_scope = 1
#
~~~

If the value displayed is "0", that is the current value of the ptrace_scope variable in the kernel, and you don't have to do anything else.  If it is any other value, do this:

~~~
# sysctl kernel.yama.ptrace_scope=0
kernel.yama.ptrace_scope = 0
#
# exit
$ whoami
andrew
$
~~~

This will allow the debugging session (local or remote) to attach gdb and eclipse to the running server process that's detached.  You might have to do this again after the next time the remote system is rebooted. 

Log out from the remote iRODS system (**akellylt1**) and back to your development system at the directory where iRODS was built from source. Of course, you can keep the remote ssh session open, and simply use another window for the local development system -- this might be a good idea, since we still have to install the .deb packages that will be transferred over from the development system:

On the local development system (**akelly1**):

~~~
$ uname -n            # This is our development system
akelly1
$ cd ~/github         # Enter your own base path for the source/build directory
$ ls -l
drwxrwxr-x  5 akelly akelly 4096 Apr 17 10:29 bld_icmds
drwxrwxr-x  8 akelly akelly 4096 Apr 17 10:29 bld_irods
drwxrwxr-x 15 akelly akelly 4096 Apr 14 06:25 irods
drwxrwxr-x 10 akelly akelly 4096 Apr  8 10:39 irods_client_icommands
. . . 
  
~~~
Of course there would be other files and/or directories there in most cases.

Finally, copy and transfer the .deb files from under the two bld_ directories, to the remote iRODS system:

~~~
$ uname -n            # This is our development system
akelly1
$ cd ~/github
$ ls -l bld_icmds/*.deb bld_irods/*.deb
-rw-rw-r-- 1 akelly akelly  3152538 Apr 17 10:29 bld_icmds/irods-icommands_4.2.3~xenial_amd64.deb
-rw-rw-r-- 1 akelly akelly  2657322 Apr 17 10:29 bld_irods/irods-database-plugin-mysql_4.2.3~xenial_amd64.deb
-rw-rw-r-- 1 akelly akelly  2656750 Apr 17 10:29 bld_irods/irods-database-plugin-oracle_4.2.3~xenial_amd64.deb
-rw-rw-r-- 1 akelly akelly  2685302 Apr 17 10:29 bld_irods/irods-database-plugin-postgres_4.2.3~xenial_amd64.deb
-rw-rw-r-- 1 akelly akelly 10481568 Apr 17 10:29 bld_irods/irods-dev_4.2.3~xenial_amd64.deb
-rw-rw-r-- 1 akelly akelly 48034298 Apr 17 10:29 bld_irods/irods-runtime_4.2.3~xenial_amd64.deb
-rw-rw-r-- 1 akelly akelly 19888640 Apr 17 10:29 bld_irods/irods-server_4.2.3~xenial_amd64.deb
$ mkdir packages
$ cp bld_icmds/*.deb bld_irods/*postgres*.deb bld_irods/irods-{dev,runtime,server}*.deb packages
$  
~~~

Notice that we did not copy the packages destined for oracle or mysql based systems -- we are using postgres on the remote system **akellylt1**. Now, lets copy the files to the remote system:

~~~
$ ls -l packages
-rw-rw-r-- 1 akelly akelly  2685302 Apr 19 19:25 irods-database-plugin-postgres_4.2.3~xenial_amd64.deb
-rw-rw-r-- 1 akelly akelly 10481568 Apr 19 19:25 irods-dev_4.2.3~xenial_amd64.deb
-rw-rw-r-- 1 akelly akelly  3152538 Apr 19 19:25 irods-icommands_4.2.3~xenial_amd64.deb
-rw-rw-r-- 1 akelly akelly 48034298 Apr 19 19:25 irods-runtime_4.2.3~xenial_amd64.deb
-rw-rw-r-- 1 akelly akelly 19888640 Apr 19 19:25 irods-server_4.2.3~xenial_amd64.deb
$
$ scp -r packages irods@172.20.0.104:/tmp           # Enter the correct name/ip-address for your system 
irods-icommands_4.2.3~xenial_amd64.deb                                    100% 3079KB   3.0MB/s   00:00    
irods-database-plugin-postgres_4.2.3~xenial_amd64.deb                     100% 2622KB   2.6MB/s   00:00    
irods-dev_4.2.3~xenial_amd64.deb                                          100%   10MB  10.0MB/s   00:00    
irods-server_4.2.3~xenial_amd64.deb                                       100%   19MB  19.0MB/s   00:01    
irods-runtime_4.2.3~xenial_amd64.deb                                      100%   46MB  45.8MB/s   00:00    
$
~~~

You will be challenged for a password if you did not install ssh keys on the remote system.  This operation will place a new directory **/tmp/packages** on the remote system with the .deb files created from the builds of the irods server and icommands.


### Install the iRODS .deb files on the remote system ###

Back on the remote system, become superuser.  You cannot do this as the user **irods** with the **sudo** command, because as installed from packages, this user does not have **sudoer** priviliges.  

In my case, I log in as **andrew** (who does have **sudoer** priviliges) on the remote system, and run the following commands as superuser:

~~~
$ uname -n                     # This is our remote system
akellylt1
$ whoami
andrew
$
$ # First, stop the irods servers:
$
$ sudo su - irods -c "./irodsctl stop"
Stopping iRODS server...
Success
$
$ # Now, install the packages transfered to /tmp/packages, all in one command line:
$
$ sudo dpkg -i /tmp/packages/*.deb
(Reading database ... 267288 files and directories currently installed.)
Preparing to unpack .../irods-database-plugin-postgres_4.2.3~xenial_amd64.deb ...
Unpacking irods-database-plugin-postgres (4.2.3) over (4.2.3) ...
Preparing to unpack .../irods-dev_4.2.3~xenial_amd64.deb ...
Unpacking irods-dev (4.2.3) over (4.2.3) ...
Preparing to unpack .../irods-icommands_4.2.3~xenial_amd64.deb ...
Unpacking irods-icommands (4.2.3) over (4.2.3) ...
Preparing to unpack .../irods-runtime_4.2.3~xenial_amd64.deb ...
Unpacking irods-runtime (4.2.3) over (4.2.3) ...
Preparing to unpack .../irods-server_4.2.3~xenial_amd64.deb ...
No iRODS servers running.
Upgrading Existing iRODS Installation
Unpacking irods-server (4.2.3) over (4.2.3) ...
Setting up irods-dev (4.2.3) ...
Setting up irods-runtime (4.2.3) ...
Setting up irods-icommands (4.2.3) ...
Setting up irods-server (4.2.3) ...
Setting up irods-database-plugin-postgres (4.2.3) ...
Processing triggers for man-db (2.7.5-1) ...
Processing triggers for systemd (229-4ubuntu21.1) ...
Processing triggers for ureadahead (0.100.0-19) ...
ureadahead will be reprofiled on next reboot
Processing triggers for libc-bin (2.23-0ubuntu10) ...
$
$ # Restart the irods servers:
$
$ sudo su - irods -c "./irodsctl start"
Updating /var/lib/irods/VERSION.json...
Validating [/var/lib/irods/.irods/irods_environment.json]... Success
Validating [/var/lib/irods/VERSION.json]... Success
Validating [/etc/irods/server_config.json]... Success
Validating [/etc/irods/host_access_control_config.json]... Success
Validating [/etc/irods/hosts_config.json]... Success
Ensuring catalog schema is up-to-date...
Catalog schema is up-to-date.
Starting iRODS server...
Success
$
~~~

It is important that the "dpkg -i " command executed above be done all in one command line (this allows dpkg to resolve dependencies appropriately).  

Still on the remote system, prepare a small file that can be used as a parameter to the **iput** command in the upcoming eclipse session on the development system.

~~~
$ uname -n                     # This is our remote system
akellylt1
$ whoami
andrew
$ sudo su - irods
[sudo] password for andrew: 
<enter password>
$ whoami
$ irods
$ date > /tmp/testfile1
$ cat /tmp/testfile1
Fri Apr 20 03:07:56 EDT 2018
$ iput /tmp/testfile1
$ ils -l 
/tempZone/home/rods:
  rods              0 demoResc           29 2018-04-20.03:10 & testfile1
$
$ # and remove it from irods...
$
$ irm testfile1
$ ils -l
/tempZone/home/rods:
$
~~~

This small segment above assures us that the irods installation worked fine, and also this is the very same command we will execute remotely from eclipse on the development system.  If it worked here, it should also work there, all things being equal.


### Create an Eclipse Executable Project on the Development System ###

Everything is now ready on the remote system for our eclipse session. Start eclipse:

~~~
$ uname -n
akelly1
$ /opt/eclipse/eclipse       # Enter your own path to the eclipse executable
. . .
~~~

We're going to create an eclipse C++ executable project for the **iput** utility.  The focus of this project will be the iput executable created locally during the build of the **irods_client_icommand** sources.

We're going to assume a blank eclipse screen - with no previous projects present.

Click:

~~~
File --> Import... --> C/C++ --> C/C++ Executable 
(click Next)

Select Executable:  Enter the full path for $work/bld_icmds/iput
(click Next)

New project name: IPUT_remote_debug
Create a Launch Configuration:  C/C++ Remote Application
Name: IPUT_remote_debug
(click Finish)
~~~

The dialog box for "Create, Manage, and run configurations" opens, and in the left pane, IPUT_remote_debug should be highlighted.

In the right pane, in the "Arguments" tab, enter: **/tmp/testfile1**

(We created the **/tmp/tesfile1** file in a previous step, on **akellylt1** while logged in as **irods**)

Still on the right pane, in the **"Source"** tab:

~~~ 
Click Add... --> File System Directory --> and enter the full pathname for $work/irods_client_icommands/src

and again: 

Click Add... --> File System Directory --> and enter the full pathname for $work/irods
~~~

Click "Apply" on the "Create, Manage, and run configurations" dialog box, and then click "Close".

Then, in the left (Project Explorer" pane of eclipse: 

~~~
Right-click on the project name "IPUT_remote_debug", then choose:

"Configure >"  -->  "Configure and Detect Nested Projects..."
~~~

In the "Import Projects from File System or Archive" dialog box that opens. enter:

Import source:  **Enter the full pathname for $work/irods**

Make sure that the "Search for nested projects" checkbox is checked, as well as the "Detect and configure project natures" checkbox.

Click **"Finish".**





























