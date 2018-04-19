### THIS PAGE IS UNDER CONSTRUCTION -- PLEASE IGNORE FOR NOW ###

# How to Attach Eclipse to a Remote Running iRODS System

This document demonstrates how to attach the Eclipse IDE to a running icommand and server on a remote system. We will first create an eclipse project that targets an existing linux executable, and create a debug launch configuration which attaches the project to a running executable.


### The Thing with Eclipse ###
 
Eclipse has many features, configuration options, plugins, and supports different platforms and languages. Not all of them work quite as advertised, and it's easy to get lost. It also has many flavors based on the language perspective chosen. The point is (in my opinion), load up on patience.  


### Assumptions & Caveats ###

Before tackling this project, it is recommended that you start with the following How-To's.  In fact, we rely on the setup for the projects described below in order to start this project:

[Eclipse Odds and Ends](https://github.com/andrew-irods/How-To/blob/master/Eclipse-notes.md "https://github.com/andrew-irods/How-To/blob/master/Eclipse-notes.md")

[How to Attach Eclipse to a Running Process](https://github.com/andrew-irods/How-To/blob/master/Eclipse-attach-to-running-process.md "https://github.com/andrew-irods/How-To/blob/master/Eclipse-attach-to-running-process.md")

[How to Create an Eclipse project for /usr/bin/ireg](https://github.com/andrew-irods/How-To/blob/master/Eclipse-attach-to-icommand-executable.md)

[How to Attach Eclipse to a Running irodsServer Process](https://github.com/andrew-irods/How-To/blob/master/Eclipse-attach-to-irodsServer.md)

The eclipse version used here is:

~~~
		Eclipse IDE for C/C++ Developers
		Version: Oxygen.2 Release (4.7.2)
~~~


So we're now ready to start working with the executable (./hello-world-1):

### Create an Eclipse Executable Project ###

Thus far we have not used eclipse for anything.  We're going to now create an eclipse project which revolves around the executable above (./hello-world-1).   

From the eclipse File menu, pick the "import..." option  (**File --> Import...**). 
A new dialog box opens: 

![Import Executable Path](images/hello-world-1-image1.png "Choose the C/C++ Executable Files target") 

Pick the "C/C++ Executable" option, and click "Next". 
A new dialog box opens: 

![Import Executable Path](images/hello-world-1-image2.png "Import C/C++ Executable Files") 

Click on "Select Executable", find the executable we created up above (hello-world-1), and enter its path.  Click "Next".  A new dialog box opens: 

![Choose Project](images/hello-world-1-image3.png "Create an a new executable project") 

Pick a project name, and name the launch configuration (you can leave the names that Eclipse chose for you if they're reasonable). Click the "Create a Launch Configuration" checkbox, and choose  "C/C++ Attach to Application" from the drop-down menu. 

Click Finish. A new dialog box opens: 

![Debug Configurations](images/hello-world-1-image4.png "Name the debug configuration") 

Nothing has to change in the Debugger, Source, or Common tabs.  We will add source files to the project in the next step from the project explorer. 

Click "Close". This should take you back to the Project Explorer window: 

![Project Explorer](images/hello-world-1-image5.png "Project Explorer") 

Click on the small arrow next to "**hello-world-1 - [x86_64le]**" to show the list of sources found for this executables in the previous step.  Notice that the **hello-world-1.cpp** source file is present in the list of files.  You can double-click and edit it, but this project is not geared for building the executable - just running it.

At this point, we can add more source files to the project by telling eclipse to look in additional source folders if needed.  Right-click on the project name in the expolorer pane, and click on the **"Debug As" -->  "Debug Configurations..."** submenus that pop up.
 
A new dialog box opens (the same dialog box we saw above, but with the ability to make changes): 

![Debug Configurations Again](images/hello-world-1-image6.png "Modify this configuration if needed") 

Notice that you can pick additional debug configurations at this point. We'll stick to the **"C/C++ Attach to Application"** configuration for this exercise: Highlight the name of the configuration as shown above, and click the "Source" tab in the right pane.

If you don't see all your sources in the explorer pane on the left (and we do - we just have our hello-world-1.cpp file), you can add source directories for eclipse to look for additional sources. 

Click the **"Add..."** button, and a new dialog box opens: 

![Add Sources](images/hello-world-1-image7.png "Don't click OK since we're going to cancel out of this.") 

Pick the "File System Directory" in the "Add Source" dialog, and then enter the full path to the additional folder you want scanned for sources.  Note the "Search subfolders" checkbox at the bottom of the dialog box. 

**For this exercise, we do not need additional source directories scanned by Eclipse, so click Cancel, and return to the main Explorer window.**


### Run the hello-world-1 program ###

Open a "terminal" (aka a shell window).  This can be done inside of eclipse, or outside of it using any terminal or ssh window.  We'll use eclipse for this.  

![Set up runtime environment](images/hello-world-1-image8.png "Start a shell in the bottom pane") 

The bottom horizontal pane in the eclipse workspace should have a "Terminal" tab.  If it doesn't, enter "terminal" in the "Quick Access" field on the top right of the workspace.

A list of suggestions pops out. Choose "Commands | Open Local Terminal" to make this tab visible and start a shell going.  Notice that a shell prompt appears within this pane, though not necessarily in the correct directory.  Use "cd" to get to where the executable is, just like you would in any terminal window. 

Start the hello-world-1 program:

![Start the program](images/hello-world-1-image9.png "Start the program in the Terminal pane") 

Focus your cursor within the pane, and click ENTER a couple of times.  You'll notice that the program is now running, and waiting for your next ENTER click.

Time to attach the eclipse debugger to the running program. 

### Attach Eclipse Debugger to the hello-world-1 Program ###
 
At this point, your terminal pane should show the hello-world-1 program waiting for you to click the ENTER key.  Don't do that. We're now going to attach to the running program. 

Right click on the project name in the Project Explorer pane on the left.  From the drop-down menu that shows up, pick "Debug As", and then "Local C/C++ Application" from the second drop-down menu that shows up. A new dialog box then appears:

![Start the debugger](images/hello-world-1-image10.png "Start the debugger") 

The dialog box contains all the processes currently running, in alphabetized order.  Use the down-arrow to navigate to the "hello-world-1" process (pid = 22631, running single threaded on core# 2 at this time).  Highlight the process (by navigating to it), and click OK.  

The debugger is now started, attached to the running hello-world-1 process, and eclipse switches us from the C/C++ perspective to the Debug perspective:

![Debug perspective](images/hello-world-1-image11.png "Program is paused waiting for std::getline() to return") 

Notice that we have switched perspectives - everything looks different here. At the top right hand corner of the workspace, there is a highlighted button (debug perspective) which we are currently in.  If we want to do something in the sources while the debugger is running, we can, by switching back to the C/C++ perspective, by clicking on the button to the left of the debug perspective (hover over the button to see what each button is).  We could then go back to the debugger by clicking the debug perspective button. 

As soon as the debugger starts, it pauses all the the running threads in the process (with a breakpoint) - in our case there is a single thread which is paused.  It displays the stack for each thread in the top-left pane, various information (see tabs) in the top-right pane, and the source line equivalent to the paused thread at the top of the stack (which we don't have sources to, as you can see). 

In order to step through our code, and/or set breakpoints, etc - we need to switch to the stack level we want, and that we have sources for.  In this case, it's the bottom of the stack, at hello-world-1.cpp line 12.  So highlight that:

![Go the the bottom of the stack](images/hello-world-1-image12.png "Examine our sources at the breakpoint") 

There's a lot going on here, now that we are looking at a stack level that we have sources for.  If we continued to step through the code, you could see the program counter progressing through the code, variable values changing, and observe what's happening to the stack.

However, we are at an end here - we've attached eclipse to a running sources, and for the rest of what's possible here - it's way outside the scope of this document.  Back to the google machine (search for how to debug a program using eclipse).




