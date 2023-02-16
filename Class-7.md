<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Testing and debugging continued](#testing-and-debugging-continued)
  - [Test-driven development](#test-driven-development)
  - [Bowling game](#bowling-game)
  - [Set up the project](#set-up-the-project)
  - [Write the first test](#write-the-first-test)
  - [Make sure test fails](#make-sure-test-fails)
  - [Write the code / fix the test](#write-the-code--fix-the-test)
  - [Refactor / clean up](#refactor--clean-up)
  - [Write another test (that fails)](#write-another-test-that-fails)
  - [Fix the second test](#fix-the-second-test)
  - [Refactor / clean up](#refactor--clean-up-1)
  - [Repeat](#repeat)
    - [Your turn / Homework](#your-turn--homework)
  - [More complex (integration, application-level) testing](#more-complex-integration-application-level-testing)
  - [VS code notes](#vs-code-notes)
  - [Your turn / Homework](#your-turn--homework-1)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

Today's [assignment](https://classroom.github.com/a/Ym-dA7bi)

## Testing and debugging continued

### Test-driven development

People's approach to testing varies wildly. It is not uncommon to not
test a resarch code in detail at all, other than running entire
simulations and checking for results to look reasonable. This
obviously save lots of time writing tests, finding bugs, and fixing
them. On the other hand, lots of bugs still happen, and finding them
when running the whole application is often very hard and
time-intensive, and some bugs may never be found a all.

The other extreme is to test everything, ie., achieve "100% coverage"
of your test suite, which means that there is a test for every code
path. While commercial software aims to go this way, this is often not
feasible for science/research project, and probably not worth the
effort. Nevertheless, I highly recommend doing lot of testing as you
write code, and if those tests can be formalized into a test suite,
that helps future development / maintenance a lot. A side effect is
that in order to be able to test small units at a time, one is
essentially forced to design smaller, independent structures /
components / modules, which greatly helps keeping a code manageable.

Traditionally, one writes code first, and then tests to make sure that
it works afterwards. An interesting alternative approach is
test-driven development (TDD), where one writes the tests first, and
then the code to satisfy the tests next.

In a nutshell, test-driven developments follows these steps:

* Add a test
* Run all tests and make sure the new test fails
* Write the code
* Run all tests and make sure they now all pass
* Refactor / clean up
* Repeat until your project is done


### Bowling game

I will demonstrate this process (or at least the beginning of it) on a
classical example, ie., let's implement a function that scores a game
of bowling.

Here are the scoring rules for 10-pin bowling (straight from
Wikipedia):

> In traditional scoring, one point is scored for each pin that is
knocked over, and when less than all ten pins are knocked down in two
rolls in a frame (an open frame), the frame is scored with the total
number of pins knocked down. However, when all ten pins are knocked
down with either the first or second rolls of a frame (a mark), bonus
pins are awarded as follows.

> * Strike: When all ten pins are knocked down on the first roll (marked
  "X" on the scoresheet), the frame receives ten pins plus a bonus of
  pinfall on the next two rolls (not necessarily the next two
  frames). A strike in the tenth (final) frame receives two extra
  rolls for bonus pins.

> * Spare: When a second roll of a frame is needed to knock down all ten
  pins (marked "/" on the scoresheet), the frame receives ten pins
  plus a bonus of pinfall in the next roll (not necessarily the next
  frame). A spare in the first two rolls in the tenth (final) frame
  receives a third roll for bonus pins.

### Set up the project

I already set things up for you in today's assignment. In particular, I added a `CMakeLists.txt` that builds the
`bowling` executable, but `bowling.cxx` which is still basically empty. It will have the `bowlingScore()` function, as well as contain the
tests for this function. (Generally, one would probably want to have
the actual function in a separate file from the tests, but starting
out with everything together makes things a bit simpler.)

I'm using googletest as a testing framework, and the cmake stuff I put
in there should make sure that it automatically gets downloaded if
it's not already found on your system.

### Write the first test

One of the simplest cases is doing all gutter rolls, in which case the
final score should be 0.

```cxx
TEST(Bowling, AllZeros)
{
  std::vector<int> rolls = { 0,0, 0,0, 0,0, 0,0, 0,0,
 			                 0,0, 0,0, 0,0, 0,0, 0,0, };
  EXPECT_EQ(bowlingScore(rolls), 0);
}

```

I roll 20 zeros in a row, and I expect to get a zero. In order to
store my 20 zeros, I'm using `std::vector<int>`, which is essentially
the basic C++ data type for an array of ints.

### Make sure test fails

Well, when run this test, one does not get very far, since it already
fails at the compilation stage. It fails with an unpleasant error, but
the reason is simple: We're calling the `bowlingScore()` function,
which we haven't written yet at all.

So let's add a simple, and intentionally wrong, first attempt at
providing the function:

```cxx
int bowlingScore(const std::vector<int>& rolls)
{
  return -1;
}
```

Now we can compile the code, and when running it, the test fails, as
it is supposed to:

```sh
[kai@macbook build (master|REBASE-i 4/8)]$ ./bowling
Running main() from gtest_main.cc
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from Bowling
[ RUN      ] Bowling.AllZeros
/Users/kai/class/iam851/iam851/class8/bowling.cxx:25: Failure
      Expected: bowlingScore(rolls)
      Which is: -1
To be equal to: 0
[  FAILED  ] Bowling.AllZeros (0 ms)
[----------] 1 test from Bowling (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test case ran. (0 ms total)
[  PASSED  ] 0 tests.
[  FAILED  ] 1 test, listed below:
[  FAILED  ] Bowling.AllZeros

 1 FAILED TEST
```

This is where googletest helps presenting some good information on
what's going on. It's running our single test, and it fails. And it
also shows the problem: `bowlingScore(rolls)` returns -1, but we
expect the final score to be 0.

### Write the code / fix the test

There isn't a whole lot needed to fix the test -- just returning 0
will do it:

```diff
diff --git a/class8/bowling.cxx b/class8/bowling.cxx
index bbb8cbb..6b0e7e3 100644
--- a/class8/bowling.cxx
+++ b/class8/bowling.cxx
@@ -12,7 +12,7 @@

 int bowlingScore(const std::vector<int>& rolls)
 {
-  return -1;
+  return 0;
 }

 // ======================================================================
```

And in fact the test now passes:

```sh
[kai@macbook build (master|REBASE-i 5/8)]$ ./bowling
Running main() from gtest_main.cc
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from Bowling
[ RUN      ] Bowling.AllZeros
[       OK ] Bowling.AllZeros (0 ms)
[----------] 1 test from Bowling (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test case ran. (0 ms total)
[  PASSED  ] 1 test.
```

### Refactor / clean up

There isn't really anything to clean up yet, so we're going to repeat
the process.

### Write another test (that fails)

So let's assume I improved a little bit and occasionally manage to get
the ball to knock over some pins:

```cxx
TEST(Bowling, RegularGame)
{
  std::vector<int> rolls = { 0,0, 2,3, 0,0, 0,0, 0,0,
                             0,0, 0,0, 0,0, 0,0, 3,6, };
  EXPECT_EQ(bowlingScore(rolls), 14);
}
```

Note that instead of just picking, say rolling 20 times "3", I picked
something with a bit more variation, in order to make the test a bit more
meaningful.

Once I add this test, it fails, as it should:

```sh
[kai@macbook build (master|REBASE-i 6/8)]$ ./bowling
Running main() from gtest_main.cc
[==========] Running 2 tests from 1 test case.
[----------] Global test environment set-up.
[----------] 2 tests from Bowling
[ RUN      ] Bowling.AllZeros
[       OK ] Bowling.AllZeros (0 ms)
[ RUN      ] Bowling.RegularGame
/Users/kai/class/iam851/2020/iam851/class8/bowling.cxx:32: Failure
      Expected: bowlingScore(rolls)
      Which is: 0
To be equal to: 14
[  FAILED  ] Bowling.RegularGame (0 ms)
[----------] 2 tests from Bowling (0 ms total)

[----------] Global test environment tear-down
[==========] 2 tests from 1 test case ran. (1 ms total)
[  PASSED  ] 1 test.
[  FAILED  ] 1 test, listed below:
[  FAILED  ] Bowling.RegularGame

 1 FAILED TEST
```

### Fix the second test

The test expects that score should be 14 (which is correct), but our
`bowlingScore()` function apparently returns 0, and that's not too
surprising, considering it doesn't look at the array of rolls that it
gets passed at all.

A good start to actually scoring a bowling game would be to sum up all
the pins that were knocked over in each roll:

```diff
diff --git a/class8/bowling.cxx b/class8/bowling.cxx
index c5cbdbb..bf0f841 100644
--- a/class8/bowling.cxx
+++ b/class8/bowling.cxx
@@ -12,7 +12,11 @@

 int bowlingScore(const std::vector<int>& rolls)
 {
-  return 0;
+  int score = 0;
+  for (int roll : rolls) {
+    score += roll;
+  }
+  return score;
 }

 // ======================================================================

```

This does in fact work:
```sh
[kai@macbook build (master|REBASE-i 7/8)]$ ./bowling
Running main() from gtest_main.cc
[==========] Running 2 tests from 1 test case.
[----------] Global test environment set-up.
[----------] 2 tests from Bowling
[ RUN      ] Bowling.AllZeros
[       OK ] Bowling.AllZeros (0 ms)
[ RUN      ] Bowling.RegularGame
[       OK ] Bowling.RegularGame (0 ms)
[----------] 2 tests from Bowling (0 ms total)

[----------] Global test environment tear-down
[==========] 2 tests from 1 test case ran. (0 ms total)
[  PASSED  ] 2 tests.
```

### Refactor / clean up

There still isn't really any need to refactor at this point, it's about as concise as it can be. So I'm doing this instead:

The above function uses a somewhat fancy "range-based for loop", which
is a newer C++ feature. It's nice and compact, and I would think
pretty intuitive to understand, but it'll be hard using that approach
to look at results frame-by-frame, as we will probably have to do once
it comes to handling spares and strikes correctly. So let's refactor
the code into a more traditional for loop for now:

```diff
diff --git a/class8/bowling.cxx b/class8/bowling.cxx
index bf0f841..0916195 100644
--- a/class8/bowling.cxx
+++ b/class8/bowling.cxx
@@ -13,8 +13,9 @@
 int bowlingScore(const std::vector<int>& rolls)
 {
   int score = 0;
-  for (int roll : rolls) {
-    score += roll;
+  int n_rolls = rolls.size();
+  for (int i = 0; i < n_rolls; i++) {
+    score += rolls[i];
   }
   return score;
 }
```

### Repeat

And the story continues, but now it's your turn:

#### Your turn / Homework

Get started on doing your own test-driven development. Starting from the current state of your assignment repo, perform the steps above to get started, and then keep following the
process of test-driven development to finish the `bowlingScore`
function to handle all the possible scenarios that can happen in a
real game.

In order for me to follow your steps, please commit at every step, including commit where you add the test and make sure it fails, then commit how you fix it, then commit again after refactoring (if there is anything to refactor).

**Note**: In a real bowling game, not every frame does have two rolls and there may be extra rolls at the end, too. Rolls that don't happen **are not** part of the input. E.g., the first two frames of a game may look like `{ 10, 3, 4 }`, ie., a strike, and then 3 and 4 pins falling in the next frame.

### More complex (integration, application-level) testing

Sometimes, you want to not just test units, but make sure those units
work together properly, and of course you can create tests for that,
too. This is highly problem-specific, and maybe overkill for most
cases in scientific computing -- Test your application as it gets
completed instead. Keeping track of test cases you can run with your
application as you go is very useful, even if it involves a manual
process.

Testing unfortunately does not avoid debugging completely, but it often
helps narrowing down where things go wrong.


### VS code notes

If you're using VS code, here are some notes that may be of interest:

* If you use Windows/WSL, it integrates very nicely in VS code, to the point where one
barely notices that the work involves different operating systems. To get started, use the `Remote-WSL` extension.
I can help with issues further down the road.

* I showed how to build code directly inside of VS code, which usually works pretty nicely since it supports CMake as a build system. You'll have to install the `CMake Tools` extension -- and the `CMake` extension probably doesn't hurt, either.



### Your turn / Homework

* Finish the bowling project.

* It may not hurt to throw some arbitrary tests from the internet on it. E.g., here's one: https://www.myactivesg.com/sports/bowling/how-to-play/bowling-rules/how-are-points-determined-in-bowling. Of course, you'll need to translate it into input for your code, and remember that only actually rolls should be in the input. So the score card from this link should start like this: `{ 10, 9,1, 5,5, 7,2, 10, 10, ...}`.

* Sign up for a [Presentation](Presentations)
