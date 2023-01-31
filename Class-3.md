
# Overview

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Where things are going this week (and next)](#where-things-are-going-this-week-and-next)
- [Writing code](#writing-code)
- [Building code](#building-code)
  - [Using a script to build the code](#using-a-script-to-build-the-code)
- [A real app](#a-real-app)
- [Compilers](#compilers)
- [Using a script to build the code](#using-a-script-to-build-the-code-1)
- [Header files](#header-files)
  - [The `-Wall` compiler flag](#the--wall-compiler-flag)
  - [Fixing the warning](#fixing-the-warning)
  - [Not quite perfect yet](#not-quite-perfect-yet)
    - [Your turn](#your-turn)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Where things are going this week (and next)

The next couple of classes, we'll deal with organizing and building
code -- first manually, and then using tools like `make` and
`cmake`. As we go, we'll get started on version management, too.

## Writing code

As I touched upon last time, a lot of time is spent in an editor where
one can do the actual code writing. That editor may be stand-alone, or
it may be part of a bigger Integrated Development Environment
(IDE). That choice is yours. In either case, you'll likely want to use
use an editor with a graphical user interface.

Usually, it's easiest and preferable to run your editor on the machine
you're working on, ie., directly in Windows if you're on Windows and
on the Mac if you're using a Mac, and same for Linux, too. There is
the alternative of running the editor on the system you're using for
development (e.g., WSL2) and just have the display happen on your host
machine, though.

If the editor is not running on the same system where you compile and
run the code, there is a potential issue of keeping the code you're
working on in sync. Traditionally, one might edit the code on Windows,
then copy it to Linux, compile it there, find an error, edit on
Windows again, copy again, etc., which is painful. Most fancier
editors actually allow to work on remote files in various ways, which
is an option. But for the common case in this class, ie., WSL2 I
suggest the following:

On your host (Windows) system, you have the `C:` drive (and maybe
`D:`). The filesystem that Linux sees is a different one, but those
windows drives get "mounted" into Linux under `/mnt/c/` and
`/mnt/d/`. So if you use your editor the edit
`C:\Users\abc\iam851\hello.cxx`, you'll find that very same file on
Linux in `/mnt/c/Users/abc/iam851/hello.cxx`. Note how conveniently
(not!) one is using backslashes and the other forward slashes.

Since those windows paths are a lot of typing in Linux, one can use a
"symlink" (symbolic link) to make things a bit easier:

```sh
$ ln -s /mnt/c/Users/abc/iam851 .
$ cd iam851 # much more convenient to type than "cd /mnt/c/Users/abc/iam851"
```


## Building code

Once you start working on a computational project, you'll need to
frequently compile your code (unless you're using an interpreted
language like Python, then you lucked out). During development, and
even more so while trying to find and fix the bugs...

The most basic way to get that done is to just call the compiler by hand, as
you've seen in the last class. Slightly more sophisticated is to have
a script that takes care of calling the compiler.

### Using a script to build the code

As you may have done already at the end of the previous class, I
created a little `build.sh` script -- for two reasons: First, because
that way I could add it to the code repository, so that I have a
record of how I compiled the code. But also because the command to
compile the code is going to become more complex, so it's not fun to
keep typing it on the command line -- and even worse, I may have
forgotten how to do it by next week. Having it in a script means that
you not only can look at it and remind yourself, you can also just run
the script and don't care how it's done.

My original `build.sh` file looked like this:
```sh
#! /bin/bash

g++ hello.cxx -o hello
```

Again, there's a number of ways to run the script. You can just say

```sh
[kai@mbpro hello4]$ bash build.sh
```

which tells the shell `bash` to
run the script `build.sh`. An advantage of doing it this way is that
you can add the option `-x`: `bash -x build.sh` to see what's happening.

In general, it's a good habit to mark scripts as executable,
like compiled programs are. Then they can be run just like any other program:
```
[kai@mbpro hello4]$ chmod a+x build.sh # mark as executable
[kai@mbpro hello4]$ ./build.sh # from now on, we can just run it like this
```

## A real app

Alright, maybe the simple hello world program is not actually a
realistic example of a complex real application. So let's do this in
`class3/hello.c`: (Since this is a new "application", and I don't want
to get confused with my existing one, I'm doing this in an new
subdirectory `class3/`.)

```c
#include <stdio.h>

void
greeting(void)
{
  printf("Hi there.\n");
}

int
factorial(int n)
{
  int i;
  int retval = 1;

  for (i = 1; i <= n; i++) {
    retval *= i;
  }

  return retval;
}

int
main(int argc, char **argv)
{
  greeting();

  printf("10 factorial is %d\n", factorial(10));
}
```

Um, still not a real application? Well, it'll have to do for now. This isn't
about coding, it's about organizing / building code, so I
intentionally keep the code itself simple. I also switched back to C
for now, for a particular reason. We can break this program up, and
demonstrate what happens when building a more complicated project.

So let's just use our imagination, let's say this file has became way
too large and we want to split it up. I'll pull out the `greeting()`
function, you'll get to do `factorial()`. The goal is to put the
`greeting()` function into a new file `greeting.c`:

```c
#include <stdio.h>

void
greeting(void)
{
  printf("Hi there.\n");
}
```

The `main` function remains in `hello.c`. This isn't quite perfect
yet. But first, let's see how to actually build this program.

## Compilers

It's worth mentioning that there are many C/C++ compilers out there,
and I've just used `gcc`/`g++`. These are the GNU C/C++ compilers, they
are the most well known open source compilers. In recent years, the
clang/llvm compiler infrastructure has also become popular -- you can
use them (if they are installed) by calling `clang`/`clang++`.

There also many other compilers available, e.g., from Intel, Cray,
PGI, etc. They do basically the same thing, but there are often
noticable differences in how to call them, what kinds of feature they
support, how well they optimize, etc.

The Fortran landscape is somewhat similar -- there is `gfortran` as part of the GNU compiler suite. There is `flang` as part of clang/llvm. I don't personally have experience with it, but from what I've heard I'm not sure it's ready for primetime. There are a lot of vendor-supported proprietary compilers, like from Intel, Cray, PGI / Nvidia, which tend to be pretty good, but they are not necessarily free and don't work everywhere, so I'd recommend focusing on gfortran.

## Using a script to build the code

To compile the project, I'll now have to go through some more effort,
or at least I should. So let's work on a `build.sh` again.

My original `build.sh` file looked like this:
```sh
#! /bin/bash

gcc hello.c -o hello
```

Again, there's a number of ways to run the script. You can just say

```sh
[kai@mbpro hello4]$ bash build.sh
```

which tells the shell `bash` to
run the script `build.sh`. An advantage of doing it this way is that
you can add the option `-x`: `bash -x build.sh` to see what's happening.

In general, it's a good habit to mark scripts as executable,
like compiled programs are. Then they can be run just like any other program:
```
[kai@mbpro hello4]$ chmod a+x build.sh # mark as executable
[kai@mbpro hello4]$ ./build.sh # from now on, we can just run it like this
```

Anyway, now that my source code is spread amongst two source files, I
need to tell the compiler. The easiest way is to say

```sh
#! /bin/bash

gcc hello.c greeting.c -o hello
```

Actually, that's just shorthand for telling the `gcc` to compile
`hello.c`, then compile `greeting.c`, then link the resulting object
files together into an executable called `hello`.

So what really happens is the following, and I prefer to be explicit
about it, in particular because it'll help understand where we're
going with using Makefiles.

```sh
#! /bin/bash

gcc -c hello.c
gcc -c greeting.c
gcc hello.o greeting.o -o hello
```

## Header files

Maybe you already got a warning like this from the compiler:

```c
[kai@macbook hello (class-3)]$ ./build.sh
hello.c:20:3: warning: implicit declaration of function 'greeting' is invalid in C99
      [-Wimplicit-function-declaration]
  greeting();
  ^
1 warning generated.
```

Warnings are not errors, that is, the compiler will keep on going, but
what the compiler really means is "while this is technically
acceptable, I think you may have made a mistake here". And most of the
time, the compiler is right, and you should fix things.

### The `-Wall` compiler flag

I'll actually change one more thing, I'll add `-Wall` as a compiler
flag. (Actually, I already did in the class repository). This tells the
compiler to turn on (almost) all warnings, and these warnings are
useful in finding problems in your code. If you hadn't already gotten
the warning above anyway, you should definitely see it now.

```
#! /bin/bash

gcc -Wall -c hello.c
gcc -Wall -c greeting.c
gcc hello.o greeting.o -o hello
```

### Fixing the warning

The issue is that we're now calling the `greeting()` function from
`hello.c`, but the compiler doesn't know anything about it, because
it's now defined in `greeting.c`. It's legal (and useful) to define
functions in separate source files, but one should at least tell the
compiler that the function exists before calling it. Technically, one
should at least "declare" the function before uing it.

We'll do that in what's called a "header file". Here's `hello.h`:

```c

#ifndef HELLO_H
#define HELLO_H

void greeting(void);

#endif

```

The greeting function is now declared in `hello.h`, which doesn't
quite help yet, because we need to know that declaration when
compiling `hello.c`. But now that we have it in the header, we can
include it there:

```diff
diff --git a/hello/hello.c b/hello/hello.c
index 4272def..ddf0bd8 100644
--- a/hello/hello.c
+++ b/hello/hello.c
@@ -1,4 +1,6 @@

+#include "hello.h"
+
 #include <stdio.h>

 int
```

### Not quite perfect yet

There are no more warnings, but there's still an issue that should be
avoided:

Say I find my code being just a bit too boring and simple, so I want
to personalize my greeting:

```diff
diff --git a/hello/greeting.c b/hello/greeting.c
index 2adcc82..b62a9f1 100644
--- a/hello/greeting.c
+++ b/hello/greeting.c
@@ -2,7 +2,7 @@
 #include <stdio.h>

 void
-greeting(void)
+greeting(const char *name)
 {
-  printf("Hi there.\n");
+  printf("Hi there, %s!\n", name);
 }

```

What happens when we compile and run the code now? Why?

#### Your turn

[This will be your homework. Make sure you keep notes, and cut&paste what you did, and what happens, in particular errors or surprising results. You may not always be able to answer the "why" for sure, but you can at least speculate.]

* Follow the steps above, ie., start with a single `hello.c`, make a build script, and then split off `greeting.c`, etc.

* Make the change to the greeting function above. What happens? Why?
  What can you do about it, and how could you avoid this happening in
  the first place?

* Separate out the `factorial()` function into its own file
  `factorial.c`, and adapt `build.sh` accordingly. Make sure
  everything still compiles and runs correctly.

* Finally, redo all of this using C++, starting from scratch in a new subdirectory.
  In particular, start over with
  `hello.cxx`, and generally, make your source files have the `.cxx`
  extension. (Some people like to use `.hxx` for C++ header files,
  though most, like me, still use `.h`. That's up to you.)

  For some additional fun, you can change the output to using `<iostream>` along the way.

  More importantly, though, what changes when you follow the same process in C++? Any idea why?
