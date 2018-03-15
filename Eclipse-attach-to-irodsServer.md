# How to Attach Eclipse to a Running irodsServer Process

Assuming a system with an installed iRODS build (and runtime system), this document demonstrates how to create an Eclipse debug session attached to the running /usr/bin/irodsServer process.

This operation will not build the irodsServer app at all, simply debug it. 

The eclipse debugger will be run as the user **root**.  As such it will have access to all source files maintained by the developer, and most importantly, be able to attach to the running irodsServer process. 

**Important Note**:  

sudo echo 0 > /proc/sys/kernel/yama/ptrace_scope



https://docs.irods.org



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

### Start the debugger

Right click on the **ireg-[x86_64le]** line in the project explorer, and choose "Debug As   ->", and then pick "Local C/C++ Application".  What opens up is this dialog, asking whether you want to switch from the C/C++ perspective to the Debug perspective.  Choose Yes (you might want to first click on the "Remember my decision" checkbox to avoid getting this dialog box again:

![Switch Perspectives](images/debug-icmds-image11.png "Switch Perspectives: click Yes") 

What opens up is the Debug Perspective, with the program running, stopped at a breakpoint just inside the main() function:

![Choose Project](images/debug-icmds-image12.png "Create an a new executable project") 

So **ireg** the program is running, under our control.  We are stopped right at the beginning of the program, and shouldn't really continue much further, since the command line parameters entered into Eclipse, were quite imaginary. 

The actual debugging of the program is beyond the scope of this document. 
