# How to Attach Eclipse to a Running Process

This document demonstrates how to attach the Eclipse IDE to a running process. We will first create an eclipse project that targets an existing linux executable, and create a debug launch configuration which attaches the project to a running executable.


### The Thing with Eclipse ###
 
Eclipse has many features, configuration options, plugins, and supports different platforms and languages. Not all of them work quite as advertised, and it's easy to get lost. It also has many flavors based on the language perspective chosen. The point is (in my opinion), load up on patience.  


### Assumptions & Caveats ###

Before tackling this project, it is recommended that you start with the following if you're not familiar with eclipse:
* Download and setup Eclipse. The version used in the making of this document is *Oxygen*, which bundles the C++ perspective, as well as other related plugins with it:

~~~
		Eclipse IDE for C/C++ Developers
		Version: Oxygen.2 Release (4.7.2)
~~~


* Build a small, simple "hello world" application.  Do a google search of "eclipse tutorial", and you'll find dozens of tutorials and examples, including some pretty good video's on youtube.com.


* This how-to document really starts when you have an executable that has been built (it does not have to have been built with Eclipse for this exercise) in debug mode (-g -O0 compiler flags), and to which you know where the sources are.

Lastly, much of what happens next is how I work -- there are many ways to do things, and mine is not necessarily the best for you.  YMMV. 

### Create a simple "hello world" App

For the purpose of this exercise, we're simply going to create a folder with a single Makefile and .cpp file in it - not necessarily in Eclipse:

~~~
	$ mkdir hello-world-1
	$ cd hello-world-1
	$ vi hello-world-1.cpp     # or use your favorite editor.
~~~
Create the following content in hello-world-1.cpp:

~~~
#include <stdio.h>
#include <iostream>
#include <string>

int main(int argc, char **argv)
{
    int i = 1;
    std::string line;

    do {
        std::cout << "Please press ENTER (or ctl^d for EOF): " << std::flush;
        if (!std::getline((std::cin), line)) {
            break;
        }
        std::cout << "Hello world no. " << i << "!!!" << std::endl;
        i++;
    } while (true);
    return 0;
}
~~~

When the program is run, it will echo a line to standard output every time RETURN is hit on the keyboard.  Since it runs until you enter EOF (ctrl^d), it's a good (easy) candidate to attach eclipse to at runtime.  

Now, create the following content in Makefile:

~~~
all: app

app:        hello-world-1.cpp
            g++ --std=c++14 -g -O0 hello-world-1.cpp -o hello-world-1
clean:
            rm -f hello-world-1
~~~

Don't forget that the delimiter in Makefile is the tab (\t) character, or you will get syntax errors. This Makefile will create a single configuration in Debug mode (-g -O0, and the symbols are not stripped from the executable).  When you use Eclipse as a proper IDE, it will have two build modes: Debug and non-Debug (release) modes that you can build and run with.  But that's not important for this exercise.  We just want a debug executable.

Creating the hello-world-1 executable is simply a matter of:

~~~
$ make
g++ --std=c++14 -g -O0 hello-world-1.cpp -o hello-world-1
$ ls -l
total 68
-rwxr-xr-x 1 andrew andrew 60168 Mar  6 13:55 hello-world-1
-rw-r--r-- 1 andrew andrew   266 Mar  6 11:53 hello-world-1.cpp
-rw-r--r-- 1 andrew andrew   153 Mar  6 11:44 Makefile
~~~

So we're now ready to start working with the executable (./hello-world-1):

### Create an Eclipse Executable Project ###

Thus far we have not used eclipse for anything.  We're going to now create an eclipse project which revolves around the executable above (./hello-world-1).   

From the eclipse File menu, pick the "import..." option  (**File --> Import...**). 
A new dialog box opens: 

![Import Dialog Box]
(images/hello-world-1-image1.png 
"Create an executable project") 

Pick the "C/C++ Executable" option, and click "Next". 
A new dialog box opens: 

![Import Executable Path](images/hello-world-1-image2.png "Import C/C++ Executable Files") 

Click on "Select Executable", find the executable we created up above (hello-world-1), and enter its path.  Click "Next".  A new dialog box opens: 

![Choose Project](images/hello-world-1-image3.png "Create an a new executable project") 

Pick a project name, and name the launch configuration (you can leave the names that Eclipse chose for you if they're reasonable). Click the "Create a Launch Configuration" checkbox, and choose  "C/C++ Attach to Application" from the drop-down menu. 

Click Finish. A new dialog box opens: 

![Import Dialog Box](images/hello-world-1-image4.png "Create an executable project") 

A new dialog box opens: 

![Import Dialog Box](images/hello-world-1-image5.png "Create an executable project") 

A new dialog box opens: 

![Import Dialog Box](images/hello-world-1-image6.png "Create an executable project") 

A new dialog box opens: 

![Import Dialog Box](images/hello-world-1-image7.png "Create an executable project") 














