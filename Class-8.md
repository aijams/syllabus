<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Homework etc](#homework-etc)
- [Linear Algebra, testing, debugging, continued](#linear-algebra-testing-debugging-continued)
  - [My work](#my-work)
  - [Dealing with variable vector length](#dealing-with-variable-vector-length)
  - [Structs, argument passing and pointers](#structs-argument-passing-and-pointers)
    - [Your turn](#your-turn)
  - [Introducing `struct vector`.](#introducing-struct-vector)
    - [Your turn](#your-turn-1)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

Today's assignment:

## Homework etc

- You are welcome to work together (in fact, that'd be great) on a shared branch (but if so, please let me know that the pull request is the joined work of students A, B, and C)

- Remember to think about signing up for a [Presentation](Presentations)

## Linear_algebra project in C

Today we'll start working on yet another toy project, but it's getting
a bit more realistic. This work (as it expands over the next couple of
classes) has multiple objectives:

- Become more familiar with cmake and learn a bit more about it (e.g.,
  subdirectories).

- Understand some basic ideas behind object-oriented programming, and
  learn about structuring your code.

- Learn some more about C++ and how it makes things "prettier" than C.

- Understand how multi-dimensional arrays work.

There is rather little in terms of (detailed) instructions today. So
you'll have to use whatever resources are available to you (prior
knowledge, notes from previous clases, your team members, your
friendly instructor, slack, google, whatever...)

### First steps

(Nothing is really new here, but I'll repeat it here as a checklist
which might be useful for future reference until the process becomes
natural.)

- Sign up for the class repository (see link above).

- Clone your newly created github repository onto your work
  machine. You can use git (`git clone <URL-from-github-repo-page>`), VS Code's "Git: Clone / Clone from Github" functionality, or I've in theory added a button on github that says "Open in VS Code". You may still have to reopen the repository in WSL, using the "WSL: Reopen Folder in WSL" command. (All commands can most easily be found, IMHO, by hitting doing Cmd-Shift-P (Mac) or Ctrl-Shift-P (Windows) to open the Command Palette and typing). Or you can use GitHub Desktop or whatever your favorite Git tool is.

  More useful info on VS Code + WSL: <https://code.visualstudio.com/docs/remote/wsl>

- Change into the the directory (`cd class8-<team>`).

- Create and switch to your own branch (`git checkout -b <branchame>`
  or, in modern git, `git switch -c <branchname>`). Or in VS Code, you can find this functionality in the "..." menu in the version control pane, or you can use the Command Palette and "Git: Create Branch".

- Make sure you can build the code: Create a build directory `mkdir build`, change into it `cd build`, configure CMake `cmake ..`, build it `make`, run it. The code is in the `c` subdirectory. Instead of doing it manually, if you have added the CMake Tools extension to VS Code, all you need to do is select a "Kit" (your compiler), and click the "Build" in the status bar on the bottom.

- Make changes and commit as you go. You can create a pull request on
  github after you've made your first commit (and pushed your branch
  to github). This can be useful if you want to write comments as you
  go.

  Generally speaking, if you just want to describe what changes you've
  made to the code, the best place to do so is in the commit
  message. If you just do what I ask for below, it's fine to be brief
  about it ("added vector_add() and a test"). (That test should pass,
  too.)

  If I ask questions ("What happens if you do xyz?") the answers
  are best put into the pull request comments (but I'm not actually
  asking many questions below.)

### Linear Algebra: It starts with a dot product

As opposed to Fortran, neither C nor C++ really have much concept of mathematical objects that are useful for numerical
computations. So the idea here is to see what it takes to add some basic linear algebra capabilities.

In today's repo, I started by writing a (very basic) `vector_dot()` function in `c/vector_dot.c`:

```c
// ----------------------------------------------------------------------
// vector_dot
//
// returns the dot product of the two vectors
// x: first vector
// y: second vector

double vector_dot(double* x, double* y)
{
  double sum = 0.f;
  for (int i = 0; i < N; i++) {
    sum += x[i] * y[i];
  }
  return sum;
}
```

I also add a very basic (and not very automatic test). This is where it all begins...

#### Your turn

- Take the test part of the code out of `vector_dot.c` and make a
  separate `test_vector_dot.c`. Update the `CMakeLists.txt` accordingly.

- Add a new file `vector_add.c`, and write a function `vector_add(x, y, z)` that adds x + y and puts the result into z. Write a test for
 this routine in a separate file.  Update the `CMakeLists.txt`
 accordingly.

- (more challenging) Add a new file `matrix_vector_mul.c`, and write a
  function `matrix_vector_mul(A, x, y)` that calculates y = Ax,
  where A is a N x N matrix. Write a test for it.  Update the
  `Makefile.am` accordingly.

- Add at least one automated test for each of the three functions, which will be
  run by `ctest .`. (You can base these tests on testing code you
  already have.)

- Build a library `linalg` that contains the three linear algebra
  functions above. Change your tests to link with that library.

- Start thinking about how to make this library more generic, in
  particular, how to avoid having it work only with the hardcoded
  vector size N = 3.

## Homework

- As usual, finish the class work above, commit as you go, open a pull request

- Follow the instructions below to get started on structs and pointers. You don't need to hand anything in, but you can add your work to the pull request, and it's probably a good idea to keep a record of what you did and what happened, so I can try to clarify things next class.

### Structs, argument passing and pointers

It seems like a good idea at this point to clarify some C/C++ features that often is
non-intuitive to programmers not experienced in those languages.

#### Your turn (OPTIONAL part of homework)

For the following, write some small pieces of code. You might want to
do it in the `c/` directory, or you can make a whole
new directory for it, but then you'll have to set up cmake for it,
too. Commit the various steps of your test code and copy&paste the
output into your homework pull request, together with some
interpretation of what that output means.

Before you start the work, change back to the main branch (`git
checkout main`) and then create your own branch (`git checkout -b
<name>`).

- Write a function `void foo(double p)`. That function should print `p`,
  then set `p` to 10, then print it again. In the `main` function, make
  a variable `double p = 99.;` and pass it to `foo()`. Print the value
  of `p` in the main program before and after you call `foo(p)`. What happens? Why?

- Rename the variable you're using in `foo()` to `q`. What happens?
  Why?

- Change back to `p`. Instead of having a simple scalar `double p`, change things
  everywhere so that you're using arrays of two doubles, ie., `double
  p[2]`. What happens? Why?

- Finally, let's use a struct -- let's assume p was actually a point,
  with p[0] = x-coordinate and p[1] = y-coordinate. We can make a
  `struct point`:

```c
struct point {
  double x;
  double y;
};
```

To initialize it, you can do

```c
  struct point p;
  p.x = 2.;
  p.y = 3.;
  // or all in one line:
  // struct point p = { .x = 2., .y = 2. };
```

  Pass `p` to your `foo()` function, and print it there, and reassign
  new values. Again, what happens, and why?

- [This is not trivial]. Change your code such that the new values
  assigned in `foo()` actually make it back to the calling `main()`
  function. Do this for both the `double p` and the `struct point p`
  case.

<!--
## Linear Algebra, testing, debugging, continued

If you get the items from last class all done and working, you
can in theory continue in your repository from last time, though you'll also have to add the last bits I've done. So it's probably easiest to start from the new [assignment](https://classroom.github.com/a/fCeXnp0w) which has all the work on `vector_add` and `matrix_vector_mul()` from the last HW. I also did this:

* Start thinking about how to make this library more generic, in
  particular, how to avoid having it work only with the hardcoded
  vector size N = 3.

### My work

You can see my
work [here](https://github.com/unh-hpc-2022/iam851/commits/class8s) --
and it will be in your repo if you sign up for today's assignment.

If you want to go to a specific commit back in history, you can use
`git checkout <hash>`, where `<hash>` is the string of numbers and
letters that identifies a particular commit. You can then look at /
compile / test the code at that point in time.

If you use the `Git Graph` extension in VS code, you can right click on a commit and pick "Checkout".

<!--
### Making a C++ version

The
first
[step]() is
essentially just copying all the files, and renaming `.c` to
`.cxx`. So it's not really using any C++ at all.

The
next
[commit]() uses
googletest rather than our individual `assert`-based tests.

After putting all the tests into a single file, `ctest` only
shows that set of tests as a single test (test #4):

```sh
[kai@macbook build ((9b40a85...))]$ ctest .
Test project /Users/kai/class/iam851/2021/iam851/linear_algebra/build
    Start 1: TestVectorDot
1/4 Test #1: TestVectorDot ....................   Passed    0.01 sec
    Start 2: TestVectorAdd
2/4 Test #2: TestVectorAdd ....................   Passed    0.01 sec
    Start 3: TestMatrixVectorMul
3/4 Test #3: TestMatrixVectorMul ..............   Passed    0.01 sec
    Start 4: TestLinearAlgebra
4/4 Test #4: TestLinearAlgebra ................   Passed    0.01 sec

100% tests passed, 0 tests failed out of 4
```

But ctest can be [made to understand](https://github.com/unh-hpc-2021/iam851/commit/4866918a42a08dac26b79b61419e0e7a1a05b2b2) googletest tests:

```sh
[kai@macbook build (master)]$ ctest .
Test project /Users/kai/class/iam851/2021/iam851/linear_algebra/build
    Start 1: TestVectorDot
1/6 Test #1: TestVectorDot ....................   Passed    0.01 sec
    Start 2: TestVectorAdd
2/6 Test #2: TestVectorAdd ....................   Passed    0.01 sec
    Start 3: TestMatrixVectorMul
3/6 Test #3: TestMatrixVectorMul ..............   Passed    0.01 sec
    Start 4: LinearAlgebra.VectorDot
4/6 Test #4: LinearAlgebra.VectorDot ..........   Passed    0.01 sec
    Start 5: LinearAlgebra.VectorAdd
5/6 Test #5: LinearAlgebra.VectorAdd ..........   Passed    0.01 sec
    Start 6: LinearAlgebra.MatrixVectorMul
6/6 Test #6: LinearAlgebra.MatrixVectorMul ....   Passed    0.01 sec

100% tests passed, 0 tests failed out of 6
```

So other than using googletest, we're not really using any C++ here
yet. So we'll come back to this.

### Dealing with variable vector length

The old-style way of dealing with variable vector length is to pass
the vector length as an additional parameter, as
shown
[here](https://github.com/unh-hpc-2022/iam851/commit/eb15cea3ab77dddbfdd9a216d777c4cc21851837). This
is relatively straightforward, and works fine, as
demonstrated
[next](https://github.com/unh-hpc-2022/iam851/commit/305369c4bbe6a507f6344096f3042cee1b2cdd24).

It's also a bit awkward (but really, there is just no non-awkward way
of handling this in C), as normally a vector is one thing that consists
of a bunch of numbers, but also knows how many numbers are in it. We
can make "objects" that aggregate multiple pieces of information in C
using a `struct`.

-->
