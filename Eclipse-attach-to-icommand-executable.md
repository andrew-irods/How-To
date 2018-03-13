# How to Create an Eclipse project for /usr/bin/ireg

This document demonstrates how to create an eclipse project revolving around the **ireg** app executable, which, with the available sources for the project, will allow the developer to start the app, step through code, and examine variable and object values, in short - debug the process.

This operation will not build the ireg app at all, simply debug it.  This can be applied to any of the icommands in /usr/bin.

The eclipse debugger will be run as the user **irods**.


### The Thing with Eclipse ###
 
In this case, the thing with Eclipse is the same thing as with any debugger for this project -- it should be run as the user **irods**, since that sets up the environment needed for the app to run.

As such, the debugger running as **irods**, will not have ready-made access to the source files which are persumably under the control of git, somewhere on the system, created by another user (you, the developer, not **irods**).

Therefore, unless you change permissions on the files, the user **irods** will not be able to access them.  But **don't change the permissions**, since then git will see every file with modified access as being "changed" and in need of staging, committing, pushing, etc.  So don't do that. 

The solution chosen here is simple.  The icommand sources are simply copied to a different location on disk, but not the **irods** home directory, since that directory can be overwritten during an installation.  Somewhere else, say, /tmp/irods (more on that below).  The sum total of space currently taken up by the *irods_client_icommands/* repository is about 2.5Mb. So this is not a costly exercise in terms of time or space.


### Assumptions & Caveats ###

Before tackling this project, it is recommended that you start with the following if you're not familiar with eclipse:
* Download and setup Eclipse. The version used in the making of this document is *Oxygen*, which bundles the C++ perspective, as well as other related plugins with it:

~~~
		Eclipse IDE for C/C++ Developers
		Version: Oxygen.3 Release (4.7.3RC3)
~~~

* Build a small, simple "hello world" application.  Do a google search of "eclipse tutorial", and you'll find dozens of tutorials and examples, including some pretty good video's on youtube.com.


* Put together your **irods** development environment. See https://github.com/d-w-moore/irods-dev-orientation as a starting point. 


* Go over this HOWTO: [How to Attach Eclipse to a Running Process](https://github.com/andrew-irods/How-To/blob/master/Eclipse-attach-to-running-process.md). 

Lastly, much of what happens next is how I work -- there are many ways to do things, and mine is not necessarily the best for you.  YMMV. 

### Preparations 

Lets assume your developer user name is "akelly", and that your git repository sources are under the path "/home/akelly/src/renci/".  

You first have to create the runtime environment for eclipse under the user identity (authority) of "irods".  

This means that the user "irods" exists and has a sane user environment (i.e. all the prerequisite packages to run iRODS on your system have been installed, and that the "irodsServer" process is running.  See the links in the previous section).

We're going to use a somewhat temporary folder "/home/irods-persistent" to house both the eclipse workspace, as well as the copy of the sources we're going to make, for the "irods" user:

~~~
$ sudo su
<password>
# cd ~akelly/src/renci
# ls -l
total 36
drwxrwxr-x  9 akelly akelly 4096 Mar 11 13:21 ./
drwxrwxr-x 15 akelly akelly 4096 Mar 10 16:34 ../
drwxrwxr-x  8 akelly akelly 4096 Mar 12 15:23 bld_irods/
drwxrwxr-x  5 akelly akelly 4096 Mar 12 15:27 bld_irods_client_icommands/
drwxrwxr-x  4 akelly akelly 4096 Mar 10 22:01 How-To/
drwxr-xr-x  4 akelly akelly 4096 Mar 10 16:34 How-To.bak/
drwxrwxr-x 15 akelly akelly 4096 Mar 10 21:55 irods/
drwxrwxr-x 10 akelly akelly 4096 Mar 10 22:06 irods_client_icommands/
drwxrwxr-x  5 akelly akelly 4096 Mar 11 13:21 irods_training/
~~~

The sources we want (for /usr/bin/ireg) are in the "irods\_client\_icommands/" folder.  Create the target folder if it doesn't exist already, and copy the sources there.

~~~
# mkdir -p /home/irods-persistent
## --->  this is optional: ## rm -rf /home/irods-persistent/irods_client_icommands  
# cp -r irods_client_icommands/ /home/irods-persistent 
# chown -R irods:irods /home/irods-persistent
~~~

Now, become the user "irods", find the "ireg" executable, and start eclipse (from the command line). Still as superuser:

~~~
# su - irods
irods@akellydt1:~$ pwd
/var/lib/irods
irods@akellydt1:~$ which ireg
/usr/bin/ireg
irods@akellydt1:~$ /opt/eclipse/eclipse   # Your installation folder might be different.

~~~

At this point, eclipse comes up -- the terminal you invoke it from is waiting for it to exit, and you will see many messages from eclipse.  Those may safely be ignored.

The first thing that happens, is a dialog box that eclipse shows:

![Workspace](images/debug-icmds-image1.png "Choose a workspace for the irods user") 

It suggests to use "/var/lib/irods/eclipse-workspace" as the folder it should use for the workspace.  Override this by updating the path to "/home/irods-persistent/eclipse-workspace", and click "Launch".

The next screen you see is the opening window.  I tend to dismiss that window forever, and go to the actual workbench:

![Splash](images/debug-icmds-image1-1.png "Move on to the worksbench") 

Now that the workbench is open, import the executable.  Click "File" --> "Import...":

![Import](images/debug-icmds-image1-5.png "Import the executable /usr/bin/ireg") 

Click on the arrow next to C/C++:
 
![Import Executable](images/debug-icmds-image2.png "Choose C/C++ Executable") 

Choose "C/C++ Executable".  Click "Next". 

![Choose Project](images/debug-icmds-image3.png "Type in /usr/bin/ireg") 

Type in "/usr/bin/ireg", and click "Next".

![Choose Project](images/debug-icmds-image4.png "Enter project name and launch configuration") 

In the dialog above, you can use the names eclipse sets up, or enter your own, then click "Finish".

The next dialog pops up. This is where you can enter information about the runtime environment for the debugged executable, as well as where the sources are:

First we'll set up the runtime command line parameters.  Click on the "Arguments" tab:

![Enter runtime program arguments](images/debug-icmds-image6.png "Enter runtime program arguments") 

The information shown is imaginary of course.  Before we run the debugger for real, we will need to refer to an existing iRODS resource, an existing user directory or file, and a real target. Enter the information, and then click on the "Source" tab:

![Add source folder(s)](images/debug-icmds-image7.png "Enter sources' paths") 

Click and highlight Debug\_ireg.  Click "Add...", pick "File System Directory" from the next dialog box, and finally type in or browse to the source directory for irods_client_icommands.

Click "OK". After the two dialog boxes disappear, click on "Apply" and "Close" on the remaining dialog box.  (Do not click "Debug").

![Project Explorer file listing](images/debug-icmds-image8.png "Explorer File Listing") 

Click on the small arrow next to **ireg-[x86_64le]** in the Project Eplorer pane on the left.  This will expand and show the list of source files gathered in the previous step. Opening any one of these files (double-click, for example), will display the source file in the main center-right pane.



![Choose Project](images/debug-icmds-image11.png "Create an a new executable project") 

Temporary text while under construction


![Choose Project](images/debug-icmds-image12.png "Create an a new executable project") 

Temporary text while under construction


However, we are at an end here - we've attached eclipse to a running sources, and for the rest of what's possible here - it's way outside the scope of this document.  Back to the google machine (search for how to debug a program using eclipse).




