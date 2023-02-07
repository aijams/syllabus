
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Overview: advanced build systems](#overview-advanced-build-systems)
- [cmake](#cmake)
  - [Building an executable](#building-an-executable)
    - [Your turn](#your-turn)
    - [Adding `CMakeLists.txt`](#adding-cmakeliststxt)
    - [Your turn](#your-turn-1)
  - [Building a library](#building-a-library)
    - [Your turn](#your-turn-2)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

Today's assignment: https://classroom.github.com/a/sotaz0kO

Remember to switch to your own branch before making any commits (`git checkout -b <branch-name>` or the "Create branch" option in VS Code).

## Overview: advanced build systems

The `make` tool that you've got introduced to is most definitely a legacy tool. It mostly focuses on one particular thing, that is, given proper dependency information, it will rebuild all the needed parts, to update the target(s) requested.

It handles some more basics of adapting/customizing the build process to fit a particular environment, but it's quite lacking as a complete build system.

Historically, in the open source software world, the [autotools](https://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html) were developed as a layer on top of `make` to fix many of its shortcomings. If you install software from source you may well come across it (on the user side) -- here's an [example](https://github.com/unh-hpc-2023/syllabus/blob/main/hello-0.02.tar.gz).

```sh
[kai@macbook class.wiki (master)]$ tar zxvf hello-0.02.tar.gz
[kai@macbook class.wiki (master)]$ cd hello-0.02
[kai@macbook hello-0.02 (master)]$ ./configure
[...]
[kai@macbook hello-0.02 (master)]$ make
/Applications/Xcode.app/Contents/Developer/usr/bin/make  all-am
  CC       hello.o
  CC       greeting.o
  CC       factorial.o
  CCLD     hello
```

On the other hand, modern IDEs like Microsoft Visual Studio or Apple's Xcode typically provide their own ways of building projects. These are much more advanced, but often proprietary and not portable.

## cmake

[cmake](http://cmake.org) is an alternative to the autotools, which has been gaining in popularity. Its goals are quite similar to the autotools, but the underlying technology is quite different.

The main goal is to enable the developer to provide code that will work on different platforms without having to customize the build and related tasks for each system. The autotools are built on top of traditional Makefiles, while cmake is more general and has a number of choices for its backend -- one can just use regular Makefiles again, but one can also use IDEs like Microsoft Visual Studio or Apple's XCode.

The autotools are built using existing languages (shell, m4, perl in the background), while cmake has its own DSL (domain specific language) to describe what to build and how.

### Building an executable

We will now build a basic project (the `hello` project yet again) using cmake. You can look at the [cmake tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html) for going into a bit more depth.

First of all, before we start using a new way of building code, let's clean house.

#### Your turn

We're still working on the same code example (last time -- I promise ;) ), that is, in the `class3/` directory in your assignment repo, where the C sources are (Alternatively, you can work in the `cxx/` subdirectory, if you prefer C++. If you prefer Fortran, this may be a good opportunity to start your own little Fortran project -- cmake will handle it, with some minor adaptations). The `class3/` dir still has `build.sh` and `Makefile` from the previous classes. They're checked in, so they'll be there in the history forever, if one ever wanted to revisit them. But to move on, let's remove them, and also remove any object files, executables, etc., you may still have lying around.



#### Adding `CMakeLists.txt`

Instead of a `Makefile`, cmake uses another file that provides info on what to build, it's called `CMakeLists.txt`.
```
cmake_minimum_required (VERSION 3.16)

project(hello)

add_executable(hello hello.c greeting.c factorial.c)
```

This is mostly boilerplate code. You tell cmake that it should be at least version 3.16, so that if someone uses, e.g., an ancient cmake 2.8, they'll get a useful error rather than all kinds of things going wrong.

The project is then given a name (which for now isn't really being used). Finally, the last line does the real work. It tells cmake that you want an executable to the things (more properly named "targets" that you want to build). That executable is named `hello`, and it's to be built from the three `.c` sources stated.

cmake supports "out of tree" builds. They're optional, but you should definitely get in the habit of using them. That is, instead of putting all kinds of object files, executables, etc in the same directory as your sources, you can have all that stuff happen in a separate directory.

#### Your turn

* Create the `CMakeLists.txt` file with the contents shown above, and use it to build the code. It should go as shown below. Note that I'm first creating a separate build directory, and then call cmake and do the build from there, which is much preferred over building the code directly in the same directory as the sources. Once you got it working, commit your work. You do not want to commit the actual build files in the `build/` directory, though, just the `CMakeLists.txt` (and other changes you made to the code, if any.)

```sh
 kai@Kais-MacBook-Pro  ~/class/iam851/iam851/class3 $ mkdir build
 kai@Kais-MacBook-Pro  ~/class/iam851/iam851/class3 $ cd build
 kai@Kais-MacBook-Pro  ~/class/iam851/iam851/class3/build $ cmake -S ..
-- The C compiler identification is AppleClang 12.0.0.12000032
-- The CXX compiler identification is AppleClang 12.0.0.12000032
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /Users/kai/class/iam851/iam851/class3/build
 kai@Kais-MacBook-Pro  ~/class/iam851/iam851/class3/build $ make
[ 25%] Building C object CMakeFiles/hello.dir/hello.c.o
[ 50%] Building C object CMakeFiles/hello.dir/greeting.c.o
[ 75%] Building C object CMakeFiles/hello.dir/factorial.c.o
[100%] Linking C executable hello
[100%] Built target hello

```

[As may have happened before, as we're using more tools, I'm kinda just hoping that they are present on your system. But actually, these days, even for Linux, most people using Linux aren't programmers, so often those development tools aren't installed by default. On Ubuntu you may see the helpful message `To install cmake, use the command "sudo apt install cmake"`. On Mac OS, one can certainly find all these packages in many variants on the web, but I recommend using either https://macports.com or https://brew.sh as package managers -- once those are set up, installing a package is typically as simple as `sudo port install cmake`.]

So what happens is that you create and change into a separate directory called `build/` here. You then run `cmake -S ..`, which invokes `cmake` and tells it to find the sources into the parent directory `..`. What cmake does is, essentially, to create a custom Makefile, so you don't have to write it yourself. Then you just call `make` and that'll build your code. From this point on, as you develop / debug your code, you only need to call `make` again. As you can see, it figured out what compiler to use, and it also handles dependencies on header files automatically.

### Building a library

While not exactly called for here in this simple example code, I'd like to introduce the concept of (software) libraries. In a nutshell, a library is a collection of various routines that are packaged together and where one can easily pull out those that one wants to use (and one doesn't have to do it manually -- the linker will do it). A very commonly used example is the C standard library, which contains functions like `printf`, `fopen`, and hundreds others. Whenever you use `printf` in you program, the compiler/linker will just pull out the actual `printf` code from the library and link it in.

There are numerous third-party libraries, and since it's generally a good idea to avoid reinventing the wheel, we'll use some as we go. Sometimes building libraries is also a good way of organiziing your own code. Let's pretend the `factorial()` and `greeting()` functions are actually useful and you want to package in a library, called `libstuff`. This is how to change your `CMakeLists.txt`:

```
cmake_minimum_required (VERSION 3.16)

project(Hello)

# It's not a real meaningful library, so I'm not giving it a real
# meaningful name ("stuff") either...
add_library(stuff greeting.c factorial.c)

add_executable(hello hello.c)
target_link_libraries(hello stuff)
```

`add_library` is very similar to `add_executable`, it's just that it creates library rather than an executable from the provided sources. In order to use a library (`hello` needs it), one uses `target_link_libraries`.

#### Your turn

Make the changes above to build a library containing the `greeting()` and `factorial()` functions. Make sure everything still works.

## Homework

* I told you that cmake will automatically take care of header file dependencies, even if the header files (like `hello.h`) are never even mentioned in `CMakeLists.txt`. Come up with a plan to verify that this is in fact the case, and try it.

* (A bit of a challenge) Figure out how to build the C++ test program as well. There are multiple ways of doing so -- the nicest way involves cmake's `add_subdirectory()` functionality.

* To submit the homework, create a pull request on github. This is how to do it:
  * First of all, make sure that everyone in the group is working on their own individual branch, not everyone on the same `main` branch.
  * If you have forgotten to make your own branch and already committed onto the `main` branch, this will fix things:

  ```sh
  $ git branch <branch-name>
  $ git reset --hard origin/main
  $ git checkout <branch-name>
  ```

  * Now push your branch back to github into your group's repository:

  ```sh
  $ git push -u origin <branch-name>
  ```

  Or use the "Push" menu item in VS Code.

  * Finally, go to github.com, into your group's repository, and choose the "Pull Requests" tab and create a new pull request.

  For more info on pull requests, see e.g. this write-up: https://medium.com/@urna.hybesis/pull-request-workflow-with-git-6-steps-guide-3858e30b5fa4
