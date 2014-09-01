```
      .-'\
   .-'  `/\                     BOOTCHAIN
.-'      `/\
\         `/\                   - Running multiple bootstraps -
 \         `/\                  
  \    _-   `/\       _.--.     * Unify programs & scripts of different 
   \    _-   `/`-..--\     )      languages and frameworks
    \    _-   `,','  /    ,')   * Use BASH scripts as bootstraps
     `-_   -   ` -- ~   ,','    * Make a BASH script into a bootstrap by
      `-              ,','        simply adding one line of code
       \,--.    ____==-~        * Intuitive since it uses your project's
        \   \_-~\                 directory hierarchy
         `_-~_.-'
          \-~
```


#Bootchain?!
Bootchain is essentially a simple BASH script which runs other scripts
recursively. Visually that would look like a chain of scripts.

Why not call it Scriptchain then? Well, you could. However Bootchain was
made with the initial goal to unify the test files in a big project.
Imagine having a bunch of test files, each in a different language or
using a totally different framework. You either need a thorough
documetnation on how tests are run or you need some sort of tool that
keeps meta-data about your tests. We chose the second choice, where the
meta-data are just simple BASH-scripts which know how to run each test or
group of tests.

An other way to think of these BASH-scripts is as bootstraps. Bootstraps
are generally scripts or programs that run other scripts or programs.
You can think of them as wrappers. Since these wrappers are just BASH-scripts
in our case, we can do whatever we want with them. You can setup an
environment where you want your ultimate program/script to run for example
or you could just try and collect the return value of each program/script
in a single file.



#The benefits
**Intuitive**
You use your already (hopefully logical) hierarchy of the folders in
your project. Each bootstrap script you place in a folder is going to be
used by all files underneath it (recursively).

**Simplicity**
The scripts you make, are simple BASH scripts with the only addition of
a simple bootstrap_wait command. This command just tells the script to
wait for the next script to finish before continuing. Simple, right?

**Always a return value**
You always get a return value from the last bootstrap. The last bootstrap
is essentially the one that runs your program, script or whatever. You
obviously want to get the return value of that. Luckily you get that
in each individual bootstrap when the latter one has finished. You just
need to `exit` as you would in any BASH script or use Bootchain's
function `set_return_value` to set a value explicitly.


**System close**
Since bootstraps are shell scripts, we have a very big range of what
we can achieve with a bootstrap.



#Usage
1. Copy bootchain in a folder.
2. Create at least one bootstrap between that folder and the target
   file (the one you would like to run with bootchain)
3. Make sure that the last bootstrap uses `set_return_value` or `exit`
   if you are going to use the return value of your target.
4. Run the target file/files with bootchain:

    ./bootchain <target1> <target2> ..



##Deeper
You run BOOTCHAIN with:

    ./bootchain <target1> <target2> ..

where <target1>, <target2> etc. are paths to the scripts or programs
you are trying to run. Use absolute paths or relative to the bootchain.

You should have at least one bootstrap file between the target and the
bootchain. The reason is that bootchain doesn't know how to run
the target file. It needs a bootstrap to tell it how.

If you want to run a python script for example you would probably
want a bootstrap that at some point has:

    python $TARGET
    echo ?$

The second line just echoes back the return value of the script run. This
is the default way of returning values to bootchain.



##A bit deeper

At least one bootstrap file is needed somewhere in the path of the file
you are trying to run with bootchain.

However you can have multiple bootstrap files in between to create a
chain of bootstraps being run.

Let's have a look at the /tests folder of an actual project:


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

You can see two bootstrap files. The first will run for all tests (since
it's at root compared with bootchain). The second will run for everything
under the 'gui' folder.

The bootstrap files could look very different depending on what you want
them to do. For example in this case maybe we want to create a Docker
container for all our tests so then the first bootstrap would create a
container and then destroy it:




All this is pure BASH. If it's a bit intimidating, don't worry. You
just have to google around for your case scenario.



BOOTCHAIN will run any bootsrap file it finds on its way (as long as it
has a bootstrap_wait line) and run it in a recursive fashion.


        
#Examples
There are examples that you can use to see how things work or use
them as templates in your projects. 

project1 - running a few (really) simple tests in python

		   Run with: ./bootchain examples/project1/tests/gui/logged_in/*.py

project2 - running tests in a docker container. you need to have Pithikos's
           docker-enter installed for this to work.
           
           Run with: 

project3 - very deeply nested bootstraps
           
           Run with: ./bootchain examples/project3/a/b/c/d/e/test.py



#Running bootchain
./bootchain a/b/c/      --> Runs all bootstraps between folder a and c
./bootchain a/b/c/*     --> Runs all files in folder c (even the
                            bootstrap itself if there is one there)
./bootchain a/b/c/*.py  --> Runs all python files in folder c



#API
The only mandatory thing is that you use bootstrap_wait in the
intermediate bootstraps so that a chain of bootstraps is made where
each bootstrap waits for the next one to finish.

The last bootstrap will automatically omit bootstrap_wait if it is
present.


    # SETUP STUFF
	
    bootstrap_wait
    
    # CLEANUP STUFF



##Functions

bootstrap_wait     Pauses the script until the next bootstrap has finished.
                   Omitting this, makes bootchain assume that this is
                   as the last script and thus will not run any other
                   ones.
                   
                   This can be handy if you're debugging as you just
                   need to remove this line and the bootstrap will be
                   omitted.
                   
set_return_value   This can be used in the last bootstrap to set a
				   specific value to RETURN_VALUE. It's essentially
				   an alternative of using BASH's `exit`.



##Variables

The below variables are visible from all bootstraps.

TARGET             The name of the file that was given to bootchain.

                   For example if you run ./bootchain a/b/c/script then
                   the TARGET would be 'script'
                   
TARGET_PATH        The path to the target file relative to bootchain.
                   
                   For example if you run ./bootchain a/b/c/script then
                   the TARGET_PATH would be 'a/b/c/script'

RETURN_VALUE       This is empty until you set a value to it in the last
                   bootstrap. This comes in handy if you want to save
                   the return value of the script/program that the
                   last bootstrap runs.
                   
				   After that you can use the RETURN_VALUE in every
				   bootstrap (just after bootstrap_wait).
				   
				   Notice that if you don't have bootstrap_wait in a
				   between-bootstrap then the value can't be read from
				   the parent bootstraps of that. So if you want to pass
				   down the return value from the last, to the first
				   bootstrap, make sure you use bootstrap_wait in every
				   single bootstrap involved.


* Notice that all variables passed to a bootstrap are in capital to make
  it more aparent that they are part of bootchain.



Scope
------------------------------------------------------------------------
Each bootstrap runs as a separate process. Therefore if you want to achieve
communication between bootstraps, you'll need to use some type of
inter-process communication like signals, FIFOs, files, sockets, etc.

If you are going to use signals, keep in mind that SIGCONT
is already being used by BOOTCHAIN for synchronisation between proc-
esses.



How it works
========================================================================

If you are calling `./bootchain`

The flow of how things are being called is:

```
 BOOTCHAIN            bootstrap                  bootstrap
                      script1                     script2
 ___________           __________________         __________________
|           |         |                  |       |                  |
|           |---------->    do stuff    ---------->   do stuff      |
|           |         |  bootstrap_wait  |       |                  |
|           |<----------    do stuff    <----------    exit 0       |
|___________|         |__________________|       |__________________|
```` 

In the last bootstrap we omit bootstrap_wait as it's the last bootstrap.
If this is the last bootstrap but you have placed a bootstrap_wait,
the wait will silently be omitted.

bootstrap_wait separates what it has to be done once the bootstrap has
loaded from the things that have to be done when just before it exists (
in most cases cleanup).



Multiple targets
========================================================================
Target is the path you use with bootchain

Essentially you run BOOTCHAIN with a list of paths. The path should
be to the program or script you are trying to run.

BOOTCHAIN will automatically look for bootstrap scripts between itself
and the 



Common uses
========================================================================
This a list of practical uses to give you an idea of when BOOTCHAIN
can be used. Most uses involve running tests since that was the initial
purpose of BOOTCHAIN.

* You want to run tests based on different languages & frameworks from a
  centralized script.

* If you want to setup an environment before running something. For
  example if you want to run many tests in a virtual machine or container.



How it works
========================================================================

Bootchain is simply a script that looks for other scripts named "bootstrap".
The bootstrap files can have whatever extension but their barename should
be bootstrap and nothing more. (This makes it possible for these scripts
to be written in various languages while making it visible what language
they are written in.)

Each bootstrap script is run one-by-one and in the end the testfile is run.
Bootstrap scripts can be written in whatever scripting language you want.

The idea is that each bootstrap script closest to the testfile will have
set the environment correctly for the testfile to run.

The is no constraint as to how many bootscripts you can have. However it's
best to keep it as small as possible for organisational reasons.

The purpose of this design is to make it possible to have a centrelized place
to run all your tests without needing to know how each individual test
should be run. That's up to each bootstrap script to solve for you.



Return value
========================================================================
Normally the last bootstrap's return value will be returned to all
other bootstraps. For this reason if you want the return value to corre-
spond to the value of the program or script you ran, you should use BASH's
`exit` or the API's set_return_value.

These functions will work only on the last bootstrap. Putting them in any
intermediate bootstraps will have no effect on the RETURN_VALUE. This is
desired as in that way, you can have `exit` in your cleanup without
altering the return value of the main program/script you just ran.

If you want to return more information then you could use *Unix standard
ways like files, named pipes, sockets etc.



FAQ & Design notes
========================================================================

##Do bootstraps run recursively?

Bootstraps do behave as if they are running recursively. However
in practice Bootchain acts as the parent process that synchronises
everything. This implementation makes the code more robust.

Keep in mind that in this case no data is passed from one bootstrap to
the next.


##Why execute bootstraps linearly and not recursively?

The initial idea was to execute the bootstrap scripts recursively. However
that becomes very complex if many files are run. Therefore a linear
way was chosen. Furthermore it's easier to have a communication link
between parent and child.
Even if the implementation is linear, the behaviour is recursive.
bootstrap_wait does wait for the next bootstrap to finish. The only
difference with real recursion is that there is no implicit inheritance
between the bootstraps (which is favorable to not mess things up).

A plus with linearly, is that we don't have to wait for the children,
to do their cleanup before getting the return value. We can get the
return value directly from the last child and thus have that value before
the cleanup of each child.


##Why ./ (background process) and no . (source)?

Source can't be used because each bootstrap has to be running (even if
they are suspended) at the same time. By default `source` can't do that.

Ofcourse you could run source with an ampersand (&) to run the whole
thing in a separate process. But then you get the same behaviour as if
you ran the whole thing as a background process from the beginning.


##Parent/children synchronisation

The parent and children communicate in two different way. For synchroni-
sation purposes they use the signal SIGCONT. This is used by any of the
two when they want to tell the other partner to continue, if they are
"suspended".

Since signals can't propagate values, a second method is nessasary.
To return values from a child to a previous child, named pipes are used.
This is only in case the last child exits with a number code. (In BASH
this is by using 'exit 0' for example.)

Signals can be concidered redudndant since we could have used named pipes
for the synchronisation as well. However, having both, makes the program
to work in the same way even if we don't return a return command  at the
last bootstrap.

##Signals vs Named pipes

Named pipes can't be used by bootstrap_wait because then we would need
a unique pipe for each individual child which becomes cumbersome.

For this reason signals have to combine with pipes so that there is only
one process at a time waiting at a pipe end.

In practice, signals can be used for the synchronisation until we reach
the latest bootstrap. After that we need to combine signals with pipe.
Namely we signal the previous child that we have created a pipe that it
can read.
