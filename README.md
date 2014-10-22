```
      .-'\
   .-'  `/\                     BOOTCHAIN
.-'      `/\
\         `/\                   - Running multiple bootstraps -
 \         `/\                  
  \    _-   `/\       _.--.     * Unify programs & scripts of different 
   \    _-   `/`-..--\     )      languages and frameworks
    \    _-   `,','  /    ,')   * Simply BASH - No need to learn a new framework
     `-_   -   ` -- ~   ,','    * Can run existing scripts without a
      `-              ,','        single change
       \,--.    ____==-~        * Intuitive behaviour
        \   \_-~\
         `_-~_.-'
          \-~
          
          (for thorough documentation check in doc folder)
```


#Bootchain?!

Bootchain is essentially a simple BASH script which runs other scripts
recursively. Visually that would look like a chain of scripts.

Why not call it Scriptchain then? Well, you could. However Bootchain was
made with the initial goal to unify the test files in a big project.
Imagine having a bunch of test files, each in a different language or
using a totally different framework. You either need a thorough
documentation on how to run each individual test or some sort of tool that
keeps meta-data about your test files. Bootchain takes the second approach.
However instead of using some text files defining things, it uses
simple BASH-scripts. In that way you don't need to learn anything new.
You just make a BASH script and make sure that it knows how to run your
test file/files.

An other way to think of these BASH-scripts is as bootstraps. Bootstraps
are generally scripts or programs that run other (bigger) scripts or programs.
Since these wrappers are just BASH-scripts in our case, we have quite some
freedom and power as to what we can do with them. You can setup an
environment where you want your ultimate program/script to run for example
or you could just try and collect the return value of each program/script
in a single file.



#Benefits of using Bootchain

**Intuitive**
Your project is already stored in folders and subfolders (hopefully in a nice
organized manner). Bootchain takes advantage of that since it uses the
already hierarchy tree of your project. So you just have to add a
bootstrap in the folder that makes sense to you and IF you want.
The bootstrap has then an effect on everything underneath it
(recursively).

**Simplicity**
The scripts you make, are simple BASH scripts with the only addition of
a simple bootstrap_wait command (and even that is optional). This
command just tells the script to wait for the next script to finish
before continuing. Simple, right? Ofcourse you are given some more
variables and functions you can use but EVERYTHING is optional!

**Idiot-safe**
The user has little to take into concideration when making a bootstrap.
As an example when the last child has finished, the return value will
be propagated automatically from child to child as long as the user
doesn't add an exit command explicitly.

**Permissive**
Bootstrap can be anything from a bootstrap in BASH to a normal program.
The only advantage of using a BASH as bootstrap is that you then use
the API and thus achieve full recursion (in practice return values and
cleanups).

**Recursion when it makes sense**
You can achieve full recursion behaviour by using the API. However
sometimes some scripts don't need the recursion hillibilly while others do.
So how do you combine them? Well since the implementation of Bootchain
is linear, there is no problem. For example you can completely omit the
bootstrap_wait function in one BASH script without screwing up the flow
of the rest of bootstrap scripts.

**System close**
Since bootstraps are shell scripts, we have a very big range of what
we can achieve with a bootstrap. You can make system calls, run programs
and what not.



#Usage
1. Copy bootchain in a folder.
2. Create at least one bootstrap between that folder and the target
   file (the one you would like to run with bootchain)
3. Make sure the bootstrap is executable (chmod +x).
3. Use the API of Bootchain if you like
4. Run the target file/files with bootchain:

    ./bootchain <target1> <target2> ..

where <target1>, <target2> etc. are paths to the scripts or programs
you are trying to run. Use absolute paths or relative to the bootchain.



#An example

At least one bootstrap file is needed somewhere in the path of the file
you are trying to run with bootchain. That's because Bootchain itself
doesn't know how to run your <target> file. It relies on bootstraps for
that.

However you can have multiple bootstrap files in between to create a
chain of bootstraps being run.

Let's have a look at the /tests folder of an actual project where we
use Bootchain:


```
/tests
  | 
  |-- bootstrap                                   <-- First bootstrap
  |
  |-- bootchain                                   <-- The bootchain script
  |
  '-- gui
  |    |
  |    |-- bootstrap                              <-- Second bootstrap
  |    |
  |    |-- forum
  |    |     |
  |    |     |-- profile.py                       <-- Test file
  |    |     '-- threads.py                       <-- Test file
  |    |
  :    :
  .    .
```

We could run both test files (profile.py and threads.py) with

```
./bootchain gui/forum/*.py
```

That will run the first bootstrap and then the second one.

If these bootstraps don't make use of any of the API, they will run like
normal BASH scripts one after the other. Keep in mind that the first
bootscrap will then finish before the second one being launched.

In most cases you would want the last bootstrap to run the actual target.
In our case the second bootstrap should have at some point

    python $TARGET
    exit ?$

$TARGET is the relative path from the bootstrap to the target file we
want to run.


Now, if we use bootstrap_wait in the first bootstrap, then the flow
of execution will look like below.


```
./bootchain            bootstrap 1                bootstrap 2
 ___________           __________________         __________________
|           |         |                  |       |                  |
|           |---------->    do stuff    ---------->   do stuff      |
|           |         |  bootstrap_wait  |       |                  |
|           |<----------    do stuff    <----------   do stuff      |
|___________|         |__________________|       |__________________|
```` 

Without bootstrap_wait however we just treat the bootstrap as a normal
BASH script:


```
./bootchain            bootstrap 1                bootstrap 2
 ___________           __________________         __________________
|           |         |                  |       |                  |
|           |---------->    do stuff    ---------->   do stuff      |
|           |         |                  |       |                  |
|           |<--.     |     do stuff     |    .-----  do stuff      |
|___________|    .    |__________________|   .   |__________________|
                  '-------------------------'
```` 



        
#Examples
There are examples that you can use to see how things work or use
them as templates for your projects. 


```
project1 - Simple demonstration

           Run with: ./bootchain examples/project1/tests/gui/logged_in/*.py

project2 - Running tests in a docker container. you need to have Pithikos's
           docker-enter installed for this to work.
           
           Run with: <not fully working yet>

project3 - Deeply nested bootstraps
           
           Run with: ./bootchain examples/project3/a/b/c/d/e/*.py
           
project4 - Changing the RETURN_VALUE from an intermediate bootstrap

           Run with: ./bootchain examples/project4/a/b/c/d/e/*.py
```



#API
All functions and variables are optional.


##Functions

````
bootstrap_wait          Pauses the script until the next bootstrap has
                        finished. Omitting this, will execute the whole
                        script without waiting for the next bootstrap.
                        
                        If you use this function don't include any of the
                        below in your bootstrap code:
                              * trap on EXIT pseudosignal
                              * use SIGCONT signal
                        
export_var NAME VALUE   Similar to BASH' export. This function will set
                        a variable to be inherited to ALL sub-bootstraps.
````


##Variables

The below variables are visible from all bootstraps.

````
TARGET             Full path to the target file.

ROOT               Full path to bootchain directory.

RETURN_VALUE       Every bootstrap after the bootstrap_wait function
                   gets this variable. To change this variable you simply
                   have to use BASH' exit command. It's common to exit
                   at the last bootstrap with the exit code of the program
                   or script ran. In that way, RETURN_VALUE will hold
                   the exit code of the target we just ran.
                   
                   RETURN_VALUE remains unchanged until a bootstrap uses
				   BASH' exit again.
````
