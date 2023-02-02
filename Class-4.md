
## Overview
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Getting code from GitHub](#getting-code-from-github)
  - [Doing an assignment on github classroom](#doing-an-assignment-on-github-classroom)
- [Ways to interact with git repositories](#ways-to-interact-with-git-repositories)
  - [Your IDE (e.g., VS code)](#your-ide-eg-vs-code)
  - [GitHub Desktop](#github-desktop)
  - [git on the command line](#git-on-the-command-line)
    - [Your turn](#your-turn)
- [make and Makefiles](#make-and-makefiles)
    - [Your turn](#your-turn-1)
  - [Variables](#variables)
  - [Special variables](#special-variables)
  - [Wildcard rules](#wildcard-rules)
    - [Your turn](#your-turn-2)
  - [Header files](#header-files)
  - [Header file dependencies](#header-file-dependencies)
- [More on make, and some of its limitations](#more-on-make-and-some-of-its-limitations)
- [Homework](#homework)
  - [Sample report bit](#sample-report-bit)
- [<!--](#--)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

* Office hours: Mon 2-3pm

## Getting code from GitHub

In order to do the in-class exercises (and homeworks in the future),
you often need to start from an existing piece of code. So this is a
good time to learn about using GitHub a bit more. As usual, there are
many ways to get the job done -- the traditional way is to use git on
the command line, but there are also a bunch of GUI applications that
make life easier. I'll introduce *GitHub Desktop*, which is available
for Mac and Windows. The command line tool is available for all
platforms.

### Doing an assignment on github classroom

Use this link to sign up for the "Class-4" assignment:
https://classroom.github.com/a/mNWe30n1

Please work in teams of
2-3 students, so one member of your group should make a new team, and the
others should then join that team. What you will end up with is
a link to the github repository that has a piece of starter code, and you'll submit your work into
that repository, too.

## Ways to interact with git repositories

If you're quite new to git and github. You may find this tutorial useful: https://classroom.github.com/a/Cz8pr0in

### Your IDE (e.g., VS Code)

If you're using a modern (or even not so modern) IDE/editor, it probably has support for git already built in, but it may require some setup.

VS Code comes with all the basics already built in. There are also various extensions for doing fancier git things (e.g., Git Lens), and also an extension to interact with github directly through VS Code, rather than through your web browser.

In VS Code, if you open a new Window (from the File menu), there should be an option to clone a git repository, though you can also always directly use the "Git: Clone" Command (use ctrl-shift-p, or command-shift-p on Mac to get the command palette). [More details](https://learn.microsoft.com/en-us/azure/developer/javascript/how-to/with-visual-studio-code/clone-github-repository?tabs=create-repo-command-palette%2Cinitialize-repo-activity-bar%2Ccreate-branch-command-palette%2Ccommit-changes-command-palette%2Cpush-command-palette)

### GitHub Desktop

GitHub Desktop is available [here](https://desktop.github.com/). It's
specifically designed to support the GitHub work flow, so it certainly
doesn't support all that git can do, but it makes life easier for
standard stuff.

To get a repository from GitHub onto your local computer, you want to
"clone a repository", and it should offer the repository you
created in github classroom. Select the directory on your local
computer where it should go (by default, it'll be put into
`~/Documents/GitHub` on Mac). After it's done syncing, you'll have the code to work
with in whichever directory you selected, and you can use GitHub
Desktop to look through the history. (You can also do that
on
[github.com](https://github.com). Go to the repository of interest and then click on
`<n> commits`, and it'll show you a list of commits.)

Since we are using private repositories for assignments etc, one needs
to authenticate with GitHub to be able to access these. With GitHub
Desktop, this is taken care of by authenticating with github.com.

### git on the command line

Go to your classroom repository on github. Click the `Clone or
Download` button, then copy the URL. You have the choice of using
`https` or `ssh`. Personally, I use `ssh` -- after a one-time key
setup, you'll never need to enter your account / password again -- in
fact, on github this method of authenticating has now been disabled
(it's less safe). So you'll need to set up key-based
authentication. There is a pretty extensive
[documentation](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh)
on that provided by GitHub, but if you get stuck, I'll be glad to help.

On the command line, go to the directory where you want your code to
(it'll go into a subdirectory). I tend to put stuff into `~/src`:

```sh
[kai@macbook ~]$ mkdir src # if the directory doesn't exist yet
[kai@macbook ~]$ cd src

### It's in general "git clone <URL>", where you paste the URL you copied above.
[kai@macbook src]$ git clone git@github.com:unh-hpc-2023/class-4.git #
use the URL from you assignment
Cloning into 'class'...
remote: Counting objects: 60, done.
remote: Compressing objects: 100% (37/37), done.
remote: Total 60 (delta 19), reused 56 (delta 17), pack-reused 0
Unpacking objects: 100% (60/60), done.

### Today's sample code is now in the "class-4" subdirectory
### It's really the code from last class, so as such, nothing new here...
[kai@macbook src]$ cd class-4
[kai@macbook class-4 (master)]$ ./build.sh
[kai@macbook class-4 (master)]$ ./hello
Hi there!
10 factorial is 3628800
```

#### Your turn

Get the code from github, either using the command line or Github
Desktop, or some other git interface of your choice, if you happen to
be familiar with one.

## make and Makefiles

The build script works perfectly fine, but for large projects it's not a good
solution, since even if we just make one little change in one file,
the whole project will be recompiled, which is slow. Instead we'll
be using the `make` command, which is the traditional way of building
code, and still the most commonly used build tool, though it may be
hiding under additional layers.

You generally call `make <target>`, and the make command will then do
whatever is needed to create the `<target>` -- if it can figure out
what to do. Normally, one writes a file called `Makefile` in the
current directory, which contains rules that tells make what command
to run to do the job of creating a target. For example:

```
hello: hello.o factorial.o greeting.o
	gcc hello.o factorial.o greeting.o -o hello
```

The first thing, listed before the colon, is the name of the
target. After the colon follows a list of prerequisites, also called
dependencies -- these need to exist and be up-to-date before the
target can be built. Finally, in the next line is the command that
will actually turn the prerequisites and create the target. Note that
there needs to be a TAB in front of the command, spaces won't work.

So, generally:

```
TARGET: PREREQ1 PREREQ2 ...
	COMMAND
```

At this point, the Makefile knows how to create "hello" from the
object (.o) files, but we haven't told it how to make the object
files, ie., by asking the compiler to compile the source files:

```
hello: hello.o factorial.o greeting.o
	gcc hello.o factorial.o greeting.o -o hello

hello.o: hello.c
	gcc -c hello.c

greeting.o: greeting.c
	gcc -c greeting.c
[...]
```

#### Your turn

Write a `Makefile` like the above (it needs to be completed!), and run
```
[kai@mpro] $ make
```

Run make repeatedly and pay attention to what commands it does / does
not rerun. Edit a file (you can just make a trivial change, like
adding a space or a comment), and run make again. Observe which
commands get executed.

Commit this step to your git repository. Make sure that you have made
a new branch to work on first, using `git checkout -b <new-branch>`, or equivalently  `git switch -c <new-branch>`.

### Variables

The main ingredient in Makefiles are rules like the above -- but we
can also use variables. For example, we can assign `gcc` to the
variable `CC`, and then use that instead, so that when you want to
change to a different compiler later, e.g., `clang`, you only have to
change `gcc` -> `clang` in one place. `CC` is the standad name for the C
compiler, and we'll also use `CFLAGS` for additional flags to the
C compiler, like `-Wall`, which turns on lots of often useful
warnings. (For C++, the corresponding variables are `CXX` and `CXXFLAGS`.)

```
# for C
CC = gcc
CFLAGS = -Wall
# for C++
CXX = g++
CXXFLAGS = -Wall

hello: hello.o factorial.o greeting.o
	$(CC) hello.o factorial.o greeting.o -o hello

hello.o: hello.c
	$(CC) $(CFLAGS) -c hello.c

[...]
```

Variables are assigned using, e.g., `CC = gcc`, that means the
variable `CC` now has the value `gcc`. When using a variable, you have
to put `$(...)` around it, e.g., `$(CC)` will be replaced by `gcc` if that's
the value we set previously.

### Special variables

There are a number of special variables that can be used in Makefile rules:

   * `$@` target
   * `$<` (first) prerequisite
   * `$^` all prerequisites

We can use these so we don't have to put redundant information into
the command part of a rule:

```
CC = gcc
CFLAGS = -Wall

hello: hello.o factorial.o greeting.o
	$(CC) $^ -o $@

hello.o: hello.c
	$(CC) $(CFLAGS) -c $<

[...]
```

### Wildcard rules

Now we notice that the last three rules are pretty much the same:
`something.o` depends on `something.c`, and the command is also the
same. Make allows wildcard rules using `%` that we can use to write a
general compile .c -> .o rule:

```
CC = gcc
CFLAGS = -Wall

hello: hello.o factorial.o greeting.o
	$(CC) $^ -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $<
```

As it turns out, make actually already knows a lot of standard rules,
For example, it also knows how to link multiple object files into an
executable -- but we still need to list the prerequisites so that make
knows what object files to link together. So we can skip that rule. It
also knows how to compile `.c` files into `.o` files using the C compiler,
or how to compile `.cpp` files into `.o` files using the C++ compiler.
rather short Makefile. Unfortunately, our C++ files have the .cxx
extension which make does not know about by default, so we have to add
that rule when switching to C++ (or we could use the `.cpp` extension).

```
CC = gcc
CFLAGS = -Wall

hello: hello.o factorial.o greeting.o

```

#### Your turn

Try out the simplified version of the Makefile above.

### Header files

As we've seen, C++ uses header files, just like C. Fortran77 on the
other hand doesn't have a concept of header files, and it doesn't need
/ want functions (or subroutines) to be declared before using
them. However, that means it's easily possible to call a function with
the wrong kind or the wrong number of arguments, and the compiler is
fine with that. Unfortunately, if you run the code, it's highly
unlikely that it works right, and fairly likely that it'll just crash
without giving much of a clue as to what's wrong.

Fortran90 (and C++20) supports "modules", which basically create something like a
header file automatically as they get compiled. This helps in ensuring
that one uses functions correctly. It does cause complications when
using make/Makefiles, though, and there are better tools out there
(like cmake, as we'll see.)

### Header file dependencies

Back to the `Makefile`: Things are pretty good already, but there's
one problem: make doesn't know anything about header files, unless we
tell it about the dependencies explicitly. Next time we'll see how
some tools can help out with that, but for now we're stuck listing
dependencies on header files explicitly, which is not so much fun...

```
CC = gcc
CLAGS = -Wall

hello: hello.o factorial.o greeting.o

hello.o: hello.h
greeting.o: hello.h
factorial.o: hello.h

```


## More on make, and some of its limitations

`make` and `Makefile`s are a standard tool that has been around for a
long time and can be found on pretty much any operating system. As
often the case for things that have been around for a while, there are
different versions of the make command around, and while the basic
feature set is always the same and compatible between versions,
specific versions of make, like, e.g. the GNU make, support additional
features that might be useful, but if using those, one should be aware
that a non-GNU make might choke on such a Makefile.

As usual for these kind of basic tools, there is a wealth of
information available on the internet that goes into a lot more depth,
for example the GNU
make [manual](http://www.gnu.org/software/make/manual/).

There are a number of limitations to the make utility, in particular
its lack of handling of dependencies on header files, and the fact
that it cannot automatically adapt to the specifics of a given
machine. For example, I hardcoded my compiler to be `gcc`,
so that same Makefile wouldn't work on another system where the
compiler might be called `pgcc` or `clang`.

## Homework

Check back here.

### Sample report bit

I took the `greeting()` function and moved it into `greeting.c`. When
I tried to compile the program, this happened:

```
$ gcc hello.c -o hello
hello.c: In function 'main':
hello.c:20:3: warning: implicit declaration of function 'greeting' [-Wimplicit-function-declaration]
   20 |   greeting();
      |   ^~~~~~~~
Undefined symbols for architecture x86_64:
  "_greeting", referenced from:
      _main in cckcj8CI.o
```

The compiler warns that it doesn't know anything about the
`greeting()` function I was calling, but it accepted it. However, when
trying to link the final executable program, it failed since the
`greeting()` function wasn't defined anywhere, so it couldn't be
called. (It was now defined in `greeting.c`, but I didn't tell the
compiler to compile that file. I fixed that in the next step)

[...]

<!--
---

*  You can, but don't have to, change the output to using
  `<iostream>`, which is the traditional C++ way of doing output, and
  it's maybe a bit nicer than `printf()` in C, but still kinda
  cumbersome.

* In the end, after making sure that you're done a series of commits
  showing your work, push the result back to github, which is where I
  will take a look at it.

The usual way of doing this is

```sh
$ git push
```

This will push your current branch back to where it came from, that
is, the github repository that has your group's assignment.

But, as usual, things get a little bit more complicated. The above is
just short-hand for

```sh
$ git push origin main
```

which more explicitly says to push to `origin`, which is where the
repository came from in the first place (ie., your assignment repo on
github), and specifies that the branch to push is `main`, which is
what the default branch is usually named (well, or `master`, which is
the older default).

Now if you're working in a team, and everyone is making their own
changes, the first person will have local `main` branch with their
changes, and they'll be successful in pushing it back. However, the
next person will also have a local `main` branch, but the changes in
their will be different. If both people pushed to the same remote
`main` branch, one would overwrite what the other did, and git doesn't
want that to happen, so the 2nd person's push will be rejected.

We'll deal with this more later, but for now a way to work around this
is do something like

```sh
# make a new branch called `kai`
$ git switch -c kai
# push that to branch to github, and remember that choice
$ git push -u origin kai
```

-->
