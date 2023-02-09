<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Testing and Debugging](#testing-and-debugging)
  - [Today's assignment](#todays-assignment)
  - [Introduction](#introduction)
  - [The editor / IDE](#the-editor--ide)
  - [The compiler](#the-compiler)
  - [Example of Warnings](#example-of-warnings)
  - [Adding compiler flags with cmake](#adding-compiler-flags-with-cmake)
    - [In-class exercise](#in-class-exercise)
  - [Unit tests](#unit-tests)
    - [In-class exercise](#in-class-exercise-1)
  - [More complex (integration, application-level) testing](#more-complex-integration-application-level-testing)
    - [In-class exercise / Homework](#in-class-exercise--homework)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Testing and Debugging

## Today's assignment

Sign up for the [assignment](https://classroom.github.com/a/T9MVJJK3). We start with the example C++ code in the `hello-again` directory. You can ignore the `bowling` directory for the time being.

## Introduction

Testing and debugging are actually closely related. That is, if you
test your code as you develop it, you may have a lot less debugging to
do later :smirk:.

In general, it is almost never a good idea to write large pieces of
code without testing them as you go.

__"Test early, test often"__ is a popular mantra in the Linux
community, and for good reason.  Another one: __"Premature optimization
is the root of all evil"__.

Testing and debugging can and should happen at many different levels:

* Your editor / IDE
* The compiler (errors and warnings)
* Unit tests
* Complex tests
* Whole application tests
* [more on debugging to follow later...]

## The editor / IDE

Your editor: Your editor, through syntax highlighting, automatic
indentation, etc. can show that something is wrong before you're
even done typing it. Modern IDEs are even better, as they actually
analyze code as you type and alert you if you're calling a function
that doesn't exist, or calling it with the wrong kind of arguments,
etc.

## The compiler

If you mess something up badly enough, your compiler will throw an error
(or dozens of them), trying to tell you what the problem is. In general,
you want to start at the top of the list trying to understand what's
wrong, since subsequent errors can be misleading as they are caused by
the compiler being confused from that first mistake.

In addition to real errors, the compiler can also warn you that while some construct is legal, you probably meant to write something else. These days, compilers tend to give useful warnings by default already, but it may be worthwhile to add more useful warnings, enabled by `-Wall`. As usual, the man page for the compiler (`man gcc`) will tell you a lot more.

## Example of Warnings

On my system (using clang), I get a bunch of warnings from today's sample code:

```sh
kai@kais-mbp  ~/class/iam851/classes/class-6/hello-again/build   main  make
[ 20%] Building CXX object CMakeFiles/stuff.dir/greeting.cxx.o
[ 40%] Building CXX object CMakeFiles/stuff.dir/factorial.cxx.o
/Users/kai/class/iam851/classes/class-6/hello-again/factorial.cxx:6:9: warning: using the result of an assignment as a condition without
      parentheses [-Wparentheses]
  if (n = 0) {
      ~~^~~
/Users/kai/class/iam851/classes/class-6/hello-again/factorial.cxx:6:9: note: place parentheses around the assignment to silence this warning
  if (n = 0) {
        ^
      (    )
/Users/kai/class/iam851/classes/class-6/hello-again/factorial.cxx:6:9: note: use '==' to turn this assignment into an equality comparison
  if (n = 0) {
        ^
        ==
/Users/kai/class/iam851/classes/class-6/hello-again/factorial.cxx:11:1: warning: non-void function does not return a value [-Wreturn-type]
}
^
2 warnings generated.
[ 60%] Linking CXX static library libstuff.a
[ 60%] Built target stuff
[ 80%] Building CXX object CMakeFiles/hello.dir/hello.cxx.o
[100%] Linking CXX executable hello
[100%] Built target hello
```

The code also doesn't exactly work all that great...

## Adding compiler flags with cmake

The following is a quick and dirty way to manipulate compiler flags
with cmake. It requires a specific user intervention, rather than
happening automatically which is not all that great, but it'll do for
now, and it'll show you how to manually experiment with flags. Instead
of simply calling `cmake ..`, you can set all kinds of options when
calling cmake, in particular

```sh
kai@lolcat:~/src/iam851/class6/build$ cmake -DCMAKE_CXX_FLAGS="-Wall" -S ..
-- Configuring done
-- Generating done
-- Build files have been written to: /home/kai/src/iam851/class6/build
```

Depending on your compiler, you might already get all the relevant warnings without adding `-Wall`. But it doesn't hurt to learn
how to set custom flags.

### In-class exercise

* What are the warning messages?
* What do they mean?
* How do you fix this code?

Fix the code and commit the changes.

## Unit tests

Unit tests are tests designed to test one particular piece of your
code, a "unit". This may well be one function, or maybe one class.
It's often important to cover posssible corner cases as well.

A unit test for the factorial function might be as simple as

```cxx
  printf("4! = %d\n", factorial(4));
  printf("1! = %d\n", factorial(1));
  printf("0! = %d\n", factorial(0));
```

[or you might C++'s `std::cout`] This isn't really all that great, though, since it not only requires you to run the test by hand, but you also need to look at the output and check that it's correct (and remember for the factorial function is defined for a zero argument...)

So a relatively simple way to improve on this is to do this:

```cxx
  assert(factorial(4) == 24);
  assert(factorial(1) == 1);
  assert(factorial(0) == 1);
  // we don't have a good way to handle negative arguments, so we don't.
  // Hence it doesn't make sense to test this case, either. FIXME
```

Fortran doesn't have anything quite like this built in (and really, I'm abusing assertions above, anyway). But you could do something like

```fortran
  if (factorial(4) .ne. 24) stop 'error: factorial(4) does not return 24'
```

### In-class exercise

* Create a `test_factorial` executable that tests correctness of the factorial function as shown above. (Hint: You may encounter trouble using the `assert()` functionality in C -- `man assert` might be helpful.)

What would be even better is for those tests to run automatically -
clearly, it's not a big deal if you just have a single test, but once
you have a bunch, it'd be nice to have a single command run them all. cmake has its own testing framework, called ctest. This is how it's done:

This is how to do automated tests in cmake:

```diff
diff --git a/class6/CMakeLists.txt b/class6/CMakeLists.txt
index 06bf783..7b2febf 100644
--- a/class6/CMakeLists.txt
+++ b/class6/CMakeLists.txt
@@ -3,8 +3,13 @@ cmake_minimum_required (VERSION 3.17)

 project(Hello)

+enable_testing()
+
 add_library(stuff greeting.cxx factorial.cxx)

 add_executable(hello hello.cxx)
 target_link_libraries(hello stuff)

+add_executable(test_factorial test_factorial.cxx)
+target_link_libraries(test_factorial stuff)
+add_test(FactorialTest test_factorial)
```

To run the test(s), use "ctest .".

```sh
[kai@macbook build (master *)]$ ctest .
Test project /Users/kai/class/iam851/iam851/class6/build
    Start 1: FactorialTest
1/1 Test #1: FactorialTest ....................***Exception: SegFault  0.05 sec

0% tests passed, 1 tests failed out of 1

Total Test time (real) =   0.05 sec

The following tests FAILED:
	  1 - FactorialTest (SEGFAULT)
Errors while running CTest
```

### In-class exercise

Try out `ctest .`. If you've fixed the `factorial()` bugs previously, it should succeed, otherwise you'll at least get confirmation that it's still broken.

## More complex (integration, application-level) testing

Sometimes, you want to not just test units, but make sure those units work
together properly, and of course you can create tests for that, too. This is
highly problem-specific, and maybe overkill for most cases in
scientific computing -- Test your application as it gets completed
instead. Keeping track of test cases you can run with your application
as you go is very useful, even if it involves a manual process.

Testing unfortunately does not avoid debugging completely, but it may
help narrowing down where things go wrong. We'll talk about debugging
more next time.

### In-class exercise / Homework

* Here's a catch on (ab)using `assert` for testing. Build your factorial unit test with `cmake -DCMAKE_BUILD_TYPE=Debug` and verify that the testing works as intended (ie., tests that are supposed to pass, pass; tests that are supposed to fail, fail). Now do so again with `-DCMAKE_BUILD_TYPE=Release`. What do you observe? Challenge: Can you figure out why?

* Read the [Googletest primer](https://github.com/google/googletest/blob/master/docs/primer.md) through "Simple Tests".

* Read [Barely Sufficient Software Engineering](BarelySufficientSoftwareEngineering.pdf) Take note if you see something in there that relates to your prior experiences or that you find particularly interesting.

* In your assignment repo, complete the in-class exercises given above, and write a brief report. Work in your own branch and commit as you go, and push your branch to github and make a pull request to submit.

  *Writing a report / notes*: As I mentioned in class, there are multiple ways of handling this, with neither being clearly superior. If you're describing an actual piece of coding you've done, the most natural place to do so is the commit message. Unfortunately, it's also a place that doesn't allow for markdown or other formatting, and it's not that straightforward for me to comment on it. Nevertheless, I think it's the right way to go for basic descriptions of changes that you've made.

  If there is something that's worth a lengthier note, copy&pasting of output or even a picture, you could do things in a separate (e.g., Word) document, or in a `README.md` file in the repo. However, those again aren't straightforward for me to comment on. So a better option is to write such comments into the PR (pull request), which fundamentally is a natural place to have a conversation about a given PR, and it even allows inserting picture easily by drag&drop, etc.

  There's a complication, though: You can't make a PR before you've actually made changes to your branch. However, I think this is a feasible way of going about it: After you've made your first commit to your branch, you can push that branch back to github, and open a PR, and then you can add a comment in the PR (if that's warranted for the current step of your assignment.) As you make more changes and have more things to write, you can add them as PR comments as you go, and push the code changes (commits) to github, either as you go, or when you're done. Those additional commits to the same branch will automatically become part of the PR. If you want to be fancy about it, you can mark your PR as a "draft" while you're working on it, and marking it ready when you're done.

* If you don't already know them, figure out the scoring rules for the game of ten-pin bowling (spares, strikes, etc).
