## Overview:

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Terminals / shells / command line](#terminals--shells--command-line)
  - [Terminals](#terminals)
  - [Shells / command line](#shells--command-line)
  - [Shell scripts](#shell-scripts)
  - [Note - End of line encoding:](#note---end-of-line-encoding)
  - [For Windows users: WSL2](#for-windows-users-wsl2)
- [Editors](#editors)
- [Building code](#building-code)
    - [Your turn](#your-turn)
- [Setting up your environment](#setting-up-your-environment)
- [Github codespaces](#github-codespaces)
- [ssh](#ssh)
  - [Linux](#linux)
  - [MacOS X](#macos-x)
  - [Windows](#windows)
  - [Passwords / Keys](#passwords--keys)
  - [Why to use key-based authentication](#why-to-use-key-based-authentication)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
<!-- update -->

## Terminals / shells / command line

### Terminals

These days, probably most, if not all, programs you interact with
feature a GUI (graphical user interface) front end. Back in the day,
though, interacting with programs via text only was state of the art,
and this way of working hasn't died yet (and I don't think they will anytime
soon). In fact, people were quite happy about the convenience that
these interfaces brought about, much nicer then carrying a stack of
[punch cards](http://en.wikipedia.org/wiki/Punched_card) to the
machine room.

Initially, the terminals were actual
[teletypewriters(tty's)](http://en.wikipedia.org/wiki/Teletypewriter).
and they are still called this name today in UNIX/Linux. Things became
even better with the introduction of terminals like the
[VT100](http://en.wikipedia.org/wiki/VT100) that used monitors to
display the text. One could even do
[ASCII art](http://en.wikipedia.org/wiki/ASCII_art). We still have a
VT220 terminal over at the RCC (Resarch Computing Center) in Morse
Hall, and use it occasionally, too. Eventually, people started to
build text-based user interfaces (TUI), but those were quickly
superseded by GUIs as we know them today.

### Shells / command line

A shell is a program that takes care of basic interaction with the
user, it provides the command line interface you have already seen.

The primary function of a shell is to run programs, which as simple as
typing the name of the program, followed by options and arguments, if
needed:

```
[kai@mbpro ~]$ whoami
kai
```

Shells also facilitate basic communication between multiple programs. There's a lot of basic commands which can be called from the shell, a very non-exhaustive list:

```
man who ls cd less cp mv rm echo cat tail bg fg jobs kill top grep wc du df sort
```

And other things:
```
. .. ~ dot-files
```

### Shell scripts

Often, it turns out one wants to do the same thing over and over
again. One thing that's actually quite useful is the "cursor up" key,
which will let you go back in your history of commands, so you don't
have to type a command over again. (There's also "CTRL-R", for
searching your history, which can be quite useful, too.)

But if one has a set of commands one needs to repeat often, shell
scripts are useful. Essentially, those are just a text file with a
bunch of commands, and when you run the script, it behaves just as if
you were typing all those commands.


Some real world examples:

This just saves me some typing when configuring my code. More importantly, I'll inevitably have forgotten how I configured it two days later, so that helps having a record of what I did.

```sh
kai@Kais-MacBook-Pro ~/src/psc/build-mac $ cat cmake.sh
CC=mpicc CXX=mpicxx FC=mpifort \
cmake \
    -DCMAKE_PREFIX_PATH=/Users/kai/src/ADIOS2/build-mac/install \
    -DCPM_gtensor_SOURCE=/Users/kai/src/gtensor \
    -DCMAKE_BUILD_TYPE=Debug \
    -DCMAKE_CXX_FLAGS_RELWITHDEBINFO="-g -O2" \
    -DPSC_USE_PERFETTO=OFF \
    -G Ninja \
    ..
```

A little helper script for comparing some data in HDF5 files.

```sh
#! /bin/bash

if [ $# != 3 ]; then
  echo "Usage: h5diff-fld.sh <FILE1> <FILE2> <FLD>"
  exit 1
fi

FILE1=$1
FILE2=$2
FLD=$3

ID1=`h5ls -r $FILE1 | grep $FLD/p0/3d | cut -d ' ' -f 1`
ID2=`h5ls -r $FILE2 | grep $FLD/p0/3d | cut -d ' ' -f 1`
h5diff -p 1e-6 -v $FILE1 $FILE2 $ID1 $ID2
```

You can use scripts to save you typing the same things all over
again. Here's another example: Making a movie, though using a Makefile
instead might be better:

```
#! /bin/bash

FIRST_FRAME=1
LAST_FRAME=119

python lim1.py
for a in `seq $FIRST_FRAME $LAST_FRAME`; do
    python plot.py data_$a.npy data_$a.png
done

ffmpeg -y -r 8 -i data_%d.png -r 30 -pix_fmt yuv420p lim1.mp4

rm -f *.png *.npy
```

A script is essentially just a text file with the commands you want to execute -- and it should start with the first line shown above, which says which shell to use to run the script. Even without that, you can always run a script by saying, e.g., `bash make_movie.sh`. A useful trick for debugging is to call the script with `bash -x make_movie.sh`, which will show the commands as they are executed.

One should mark scripts as executable, so that they can be called like any other program. You can do it by using the `chmod` command:

```sh
 kai@kais-mbp ~/class/iam851/syllabus/ex_movie main $ ls -l  make_movie.sh
-rw-r--r-- 1 kai 234 Jan 26 10:53 make_movie.sh
 kai@kais-mbp ~/class/iam851/syllabus/ex_movie main $ chmod a+x make_movie.sh
 kai@kais-mbp ~/class/iam851/syllabus/ex_movie main $ ls -l  make_movie.sh
-rwxr-xr-x 1 kai 234 Jan 26 10:53 make_movie.sh
```

I put the movie example code into  the syllabus repository: https://github.com/unh-hpc-2023/syllabus.

A nice, and pretty comprehensive, introduction to shells and the
commandline is at http://linuxcommand.org/. You don't need to know all
of this for this class, but learning the basics will help make your
life easier in the future.

### Note - End of line encoding:
If bash scripts edited using windows apps are not producing correct file names or don't work on Linux or WSL2, problem might be due to a difference in character encoding used in windows editors (End of line encoding - EOL: https://en.wikipedia.org/wiki/Newline). In order to fix this just change the EOL encoding from "Windows (CR LF)" to "Unix (LF)" and save the file. It is usually in the bottom corner of the editor screen.
 <!-- (example screenshot - Notepad++). -->

<!-- _**!!Add Nodepad++ Screenshot here**_ -->

### For Windows users: WSL2

I don't really have much experience with Windows Subsystem for Linux
version 2 (WSL2) myself, but in theory it should work fairly well for
what we need in this class. Here is a tutorial on how to install WSL2
as a
[write-up](https://medium.com/@japheth.yates/the-complete-wsl2-gui-setup-2582828f4577),
and also a [youtube video](https://www.youtube.com/watch?v=X-DHaQLrBi8) demonstrating how to do it. (Google has many
other links, too).

My recommendation here is to install Ubuntu 20.04 or 22.04 as your linux
distribution, though that's also a fairly arbitrary choice. It's
relatively recent, quite stable, pretty popular, and supported for the long
term. The links above provide a way to install the graphical user
interface (GUI) as well, which I think will be helpful.

## Editors

If you want to write a program, or just about anything, you'll need a
text editor. Traditionally, in the UNIX world, there are two
camps: [vi](http://en.wikipedia.org/wiki/Vi)
and [Emacs](http://en.wikipedia.org/wiki/Emacs). These days, there are
probably a hundred text editors out there, and some of them are
actually even user-friendly.

There's always been some competition between using a dedicated editor together with the command line vs an IDE (Integrated Development Environment). It's a choice, but at this point my recommendation would be to try an IDE, but still learn to use the command line, as that is fairly crucial when working on a remote computer.

In recent years, Microsoft Visual Studio Code seems to have become the leader in the field -- it's free and (kinda) open-source. Other options would
be full-blown traditional IDEs (Visual Studio, Xcode, CodeBlocks, Eclipse), though it is important to learn to develop code in a way where you can give it to someone else who does not use the same IDE but will still be able to work with your code.

## Building code

Once you start working on a computational project, you'll need to
frequently compile your code (unless you're using an interpreted
language like python, then you lucked out). It's helpful to compile
code a lot during development, and even more so while trying to find
and fix the bugs...

The most basic way to get that done is to just call the compiler. My
innovative example program is in `hello.cxx`:

```cxx
#include <cstdio>
#include <iostream>

int
main(int argc, char **argv)
{
  printf("Hi there.\n");

  // or if you prefer real C++
  std::cout << "Hi there from C++." << std::endl;

  return 0;
}
```

So you type:
```
[kai@mbpro hello]$ g++ hello.cxx -o hello
```

And it'll compile an executable `hello`, that you can run:
```
[kai@mbpro hello]$ ./hello
Hi there.
Hi there from C++.
```

#### Your turn

* Set up an environment that allows you to write code (editor/IDE), and work on the command line to call a compiler to compile your code.

* Create a source file `hello.cxx` which contains the code above, and
  compile and run it. (Feel free to do so in C or Fortran as well, but for a little while it'll makes sense to stick with C++ even if that's not your language of choice for the later part of the class.)

* Create a script `build.sh` that compiles the code, instead of you calling the compiler by hand


## Setting up your environment

Well, it is hard for me to give you specific instructions because there are so many choices and preferences.

First of all, you do want to be able to work on the command line in a UNIX-like environment. If you're working on Linux or Mac OS, that pretty much already exists, so it's easiest to just use it.

On Windows, while it does have a command line interface, it's sufficiently different that it's not a good choice for what we want to learn in the class, but there are some options.

The first option is to use WSL2, the Windows subsystem for Linux, which essentially runs a Linux machine inside of Windows. When possible, that's quite a good choice -- it can still cause some issues in particular when it comes to running graphical applications under Linux, but it'll do most of what we need out of the box.

An alternative is to do the actual work on a remote Linux machine, and use your local laptop or desktop just to log in and interact with that machine. This is an option for any local operating system (Windows, Linux, Mac). It does need an account on a remote machine -- and if that is your choice, I think we might as well use marvin, the UNH Cray, even though it'll require a bit of work to set up an account.

In terms of accessing a remote machine, the usual way to do it is ssh -- see below. If your choice is to use VS code as your editor / development environment, it has a lot of capability built in that makes life easier when working on a remote machine.

## Github codespaces

A new fancy way of getting a nice development environment is github codespaces. What happens in this case is that the editor (essentially, VS code) runs in your web browser, and the actual development happens on machine in the cloud. This makes things very convenient in that you do not need to install anything on your local machine, a web browser is all that is needed.

One drawback is that it's not (entirely) free. I haven't completely figured out the details, but there is a decent amount of free time one can use per month for regular github accounts: https://docs.github.com/en/billing/managing-billing-for-github-codespaces/about-billing-for-github-codespaces I believe everyone should have a free allowance at the "Github Pro" level.

One of the drawbacks of github codespaces is that it makes you overly reliant on the cloud, you miss out on the experience of solving all the fun problems that happen if one actually tries to do things on more typical local and remote computers. So I think codespaces is a great way to get started, but eventually, everyone should figure out a way of working that works for them.

To give codespaces a try, go to https://github.com/unh-hpc-2023/syllabus, click on the green `Code` button and select the `Codespaces` tab.

## ssh

SSH stands for Secure SHell. It is a program that allows you to
remotely log on to another computer, like fishercat, or the first Exascale supercomputer. It natively
supports command line (shell) access, but can also forward X11, the
Unix/Linux graphical "Windows" system.

ssh is easily supported on Linux and MacOS, while more work is needed
to use it on Windows.

At this time, my plan is for you guys to initially work locally for
the most part, ie., directly on MacOS if you have it, or on Linux /
WSL2 (Windows Subsystem for Linux version 2) if you run Windows.

Since most of you will be working locally, you may not really need SSH
yet. You can try `ssh localhost` instead of trying to connect to
fishercat, but I'm also happy to add an account for you.


### Linux

Open a Terminal, type
```
[kai@linux ~]$ ssh kai@fishercat.sr.unh.edu
Last login: Wed Jan 21 22:29:17 2015 from 10.9.0.1
**
No current scheduled downtime.

Check http://fishercat.sr.unh.edu/trac/fishercat_admin/wiki/DownTime
for announcements

**
[kai@fishercat ~]$
```

You probably want to replace `kai@` with `your_userid@`, or if your userid on the Linux machine is the same as on fishercat, you can skip that altogether.

### MacOS X

Open the Terminal application (which may be hiding in the `Applications/Utilities` folder), type
```
[kai@macbook ~]$ ssh kai@fishercat.sr.unh.edu
Last login: Wed Jan 21 22:29:17 2015 from 10.9.0.1
**
No current scheduled downtime.

Check http://fishercat.sr.unh.edu/trac/fishercat_admin/wiki/DownTime
for announcements

**
[kai@fishercat ~]$
```

You probably want to replace `kai@` with `your_userid@`, or if your userid on the Mac is the same as on fishercat, you can skip that altogether.

Bonus points if you have noticed that the mac instructions are basically copy & pasted
from Linux ;)

### Windows

If you install WSL2, one option is to just do everything from inside
there. Since you're on Linux, you can just follow the Linux
instructions above.

Otherwise, there are a bunch of applications that give you an ssh
client on Windows, ie., gives you Terminal access to a remote machine
via ssh.

If you want to do that, one very commonly used program is called `PuTTY`
(you can find it via Google). But you might as well
give [MobaXterm](https://mobaxterm.mobatek.net/) a shot, since it's a
more complete solution. Yet another alternative is using a VNC client, if you
want to give that a shot, let me know.

### Passwords / Keys

You may notice that in the example above, I did not have to enter my
password, since I set up password-less ssh using public key
authentication.

I find it rather painful to always have to enter your password, so on
your personal laptop I recommend setting things up for password-less
login by setting up keys. For Linux/MacOS that's described below. If
you want to do it for Windows, it's possible to do it with PuTTY,
too - ask me and we can figure it out together (and then maybe update
these instructions).

Setting up keys is also rather useful for git to access remote repositories, in particular the ones on github. The `ssh` way to connect to github requires keys (password doesn't work), though there is the alternative of using `https`.

To set up passwordless ssh, as a first step you need to (once) generate a private/public key pair:

```shell
[kai@mbpro ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/kai/.ssh/id_rsa):
Created directory '/Users/kai/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/kai/.ssh/id_rsa.
Your public key has been saved in /Users/kai/.ssh/id_rsa.pub.
The key fingerprint is:
e4:b1:ed:be:b3:43:d9:11:33:cb:f3:d2:bc:7f:7c:c9 kai@mbpro
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|            +    |
|        o  . =   |
|       o +  =    |
|        S .o *   |
|         .o o +  |
|         ..  ..o.|
|         .o   .E+|
|          +=   .+|
+-----------------+
```

This generates a private key (as you might have guessed, this one you need to keep secret, like a password), and a corresponding public key, which does not include secret information. You can find them in `.ssh/id_rsa` and `.ssh/id_rsa.pub`. Or, if you like to be terse, at the risk of giving away that you're a geek, you just say they're in `.ssh/id_rsa{,.pub}`

You then add the public key into the other computer's `.ssh/authorized_keys` file, for example like so:

```shell
[kai@mbpro ~]$ ssh userid@fishercat.sr.unh.edu "mkdir -p .ssh && chmod 755 .ssh && cat >>.ssh/authorized_keys && chmod 644 .ssh/authorized_keys" < .ssh/id_rsa.pub
```

This should be the last time ever you need to enter your
password. Putting your public key into `.ssh/authorized_keys`
basically means "allow anyone who has the private key corresponding to
this public key to log on without requiring a password".

More information about ssh and keys is available at
https://www.cyberciti.biz/faq/how-to-set-up-ssh-keys-on-linux-unix/, or
all over the internet.

Finally, in the category of tips and tricks: If you're to lazy to type `ssh fishercat.sr.unh.edu` all the time, you can customize your ssh setup by creating a `config` file in your `~/.ssh` directory:

```shell
[kai@mbpro ~]$ cat .ssh/config
Host *
ForwardX11 yes
ForwardX11Trusted yes
Compression yes

Host fc
Hostname fishercat.sr.unh.edu

Host zaphod
Hostname zaphod.sr.unh.edu

Host artemis
Hostname artemis.sr.unh.edu

Host franklin
Hostname franklin.nersc.gov
```

Then just `ssh fishercat` will do.

### Why to use key-based authentication

* It's more convenient (e.g., git usually runs over ssh, too, and it's
a pain having to enter your password all the time).

* It's safer not to have your password leave your own (hopefully
trusted) computer. [Case in point](https://www.zdnet.com/article/this-linux-malware-is-hijacking-supercomputers-across-the-globe/)

## Homework

* set up a working development environment, including the ability to
  * edit code,
  * run commands on a UNIX/Linux command line
  * run a compiler
  * make a simple shell script and run it. (I didn't really get to saying much about shell scripts in class. The notes above have the basics, essentially all you have to do is put your commands into a text file, and then run it, either using `bash build.sh` or making it executable using `chmod`.


* Finish "your turn" from above

* Email me the following:
  * How you set up your environment (what operating system you're using, your editor, etc.)
  * Attach your `hello.cxx`, the `build.sh` script and copy&paste the output from running the program.
