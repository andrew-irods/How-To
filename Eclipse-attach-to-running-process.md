# How to Attach Eclipse to a Running Process

This document how to attach the Eclipse IDE to a running process. We will first create an eclipse project that targets an existing linux executable, and then create a debug launch configuration which attaches the project to a running executable.


### The Thing with Eclipse ###
 
Eclipse has many features, configuration options, plugins, and supports different platforms and languages. Not all of them work quite as advertised, and it's easy to get lost. It also has many flavors based on the language perspective chosen. The point is (my opinion), load up on patience.  


### Assumptions & Caveats ###

Before tackling this project, it is recommended that you start with the following if you're not familiar with eclipse:
* Download and setup Eclipse. The version used in the making of this document is *Oxygen*, which bundles the C++ perspective, as well as other related plugins with it:

~~~
		Eclipse IDE for C/C++ Developers
		Version: Oxygen.2 Release (4.7.2)
~~~


* Build a small, simple "hello world" application.  Do a google search of "eclipse tutorial", and you'll find dozens of tutorials and examples, including some pretty good video's on youtube.com.


* This how-to document really starts when you have an executable that has been built (it does not have to have been built with Eclipse) in debug mode (-g -O0 compiler flags), and to which you know where the sources are.

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
    while (std::getline((std::cin), line)) {

        std::cout << "Hello world no. " << i << "!!!" << std::endl;
        i++;
    }
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

Don't forget that the delimiter in Makefile is the tab (\t) character, or you will get syntax errors.

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

* From the eclipse File menu, pick the "import..." option  (File --> Import...) 

A new dialog box opens: 

![Import Dialog Box](images/hello-world-1-image1.png "Create an executable project") 




















	 