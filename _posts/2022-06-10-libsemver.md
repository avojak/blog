---
title: "Libsemver"
description: "A short exercise in library development and unit testing in Vala"
author: avojak
image: https://images.unsplash.com/photo-1556917452-890eed890648
tags:
  - software
  - evergreen
---

As I've been developing apps in Vala and GTK for a few years now, there have been two questions nagging at me the entire time:

1. How libraries are written in Vala?
2. How do you properly unit test code written in Vala?

Frankly the second question is one that I *should* have investigated from the start, but better late than never, no time like the present to learn, and other adages.

To learn by doing, I settled on writing a library to handle Semantic Versions that I aptly named `libsemver`:

{% include github-card.html
  user="avojak"
  repository="libsemver"
%}

## How Are Libraries Written in Vala?

Well the real meat and potatoes is just like writing a library in any other language, but the project setup for a library is incredibly confusing if you're not sure what you're doing. Spoiler: I wasn't sure what I was doing, and frankly I still don't.
The amount of boilerplate code that goes into a library so that it's properly picked up by the system when you try to use it is a bit overwhelming. I can't really write much about this process because like I said, I still don't fully understand
all the ins and outs. I used [Granite](https://github.com/elementary/granite) heavily as a reference since I had used it before, and in the end I was able to successfully pull my library into a test project and use it.

Another great reference that I came across was [Valdo](https://github.com/vala-lang/valdo), which contains a handful of templates for creating various types of projects in Vala.

## How Do I Properly Write Unit Tests in Vala?

GLib Test was the natural starting place, and after poking around on how other people did testing, I landed on the following:

### Directory structure

The root directory for all testing code is `libsemver/tests/`. That directory is included in the `meson.build` file in the root of the project.

In `libsemver/tests/`, we have all of our files containing the test cases. I chose to break it down such that there's a 1:1 relationship between
library sources files and test source files (e.g. `Version.vala` corresponds to `VersionTest.vala`). In my research I found some GNOME projects
that broke out test cases into their own file, but this seemed to have the potential to get out of control very quickly.

In the tests `meson.build`, each test source file is listed and a loop is used to create separate test executables for each file:

```bash
unit_test_files = [
    'Specification',
    'Util',
    'Version'
]

foreach name : unit_test_files
    test(name, executable(name, name + 'Test.vala', libsemver_sources, dependencies: libsemver_deps, install: false))
endforeach
```

Notice that in reality just part of the test source file name is enumerated, and the full name is build later. This lets us have more user-friendly
test case names when we go to run the tests.

### Test Source Files

Lets take a quick look at the `UtilTest.vala` file which contains unit tests for a large-number arithmetic utility class I wrote to safely handle
adding, subtracting, and comparison of numbers larger than what can be held in a `uint64`.

```vala
public class UtilTest : GLib.Object {

    /**
      * Tests large number addition.
      */
    private static void test_addition () {
        assert_true (Util.large_number_addition ("0", "0") == "0");
        assert_true (Util.large_number_addition ("1", "0") == "1");
        assert_true (Util.large_number_addition ("0", "1") == "1");
        assert_true (Util.large_number_addition ("1", "1") == "2");
        assert_true (Util.large_number_addition ("123", "58") == "181");
        assert_true (Util.large_number_addition ("999", "1") == "1000");
        assert_true (Util.large_number_addition ("1", "999") == "1000");
        assert_true (Util.large_number_addition ("99999999999999999999998", "1") == "99999999999999999999999");
        assert_true (Util.large_number_addition ("99999999999999999999999", "1") == "100000000000000000000000");
    }

    /**
      * Tests large number subtraction.
      */
    private static void test_subtraction () {
        assert_true (Util.large_number_subtraction ("0", "0") == "0");
        assert_true (Util.large_number_subtraction ("1", "0") == "1");
        assert_true (Util.large_number_subtraction ("1", "1") == "0");
        assert_true (Util.large_number_subtraction ("2", "1") == "1");
        assert_true (Util.large_number_subtraction ("181", "58") == "123");
        assert_true (Util.large_number_subtraction ("121", "58") == "63");
        assert_true (Util.large_number_subtraction ("58", "121") == "-63");
        assert_true (Util.large_number_subtraction ("0", "1") == "-1");
        assert_true (Util.large_number_subtraction ("99999999999999999999999", "1") == "99999999999999999999998");
    }

    public static void main (string[] args) {
        GLib.Test.init (ref args);
        GLib.Test.add_func ("/util/addition", test_addition);
        GLib.Test.add_func ("/util/subtraction", test_subtraction);
        GLib.Test.run ();
    }

}
```

In the `main` method we initialize GLib Test and then queue up our test functions. It's all pretty straightforward, using assertions to verify
that your code behaves as expected. However when we go to run this test, we get the following output:

```bash
$ ninja -C build test
[0/1] Running all tests.
1/1 Util                                    OK       0.01 s 

Ok:                    1
Expected Fail:         0
Fail:                  0
Unexpected Pass:       0
Skipped:               0
Timeout:               0
```

Well, ok, we can see everything passed, but even though we wrote 18 different assertions across two different test functions, it was all globbed
into a single test entry. Not awful, but not ideal.

Now let's see what happens if we have a failing test:

```bash
$ninja -C build test
[3/4] Running all tests.
1/1 Util                                    FAIL     0.01 s (killed by signal 6 SIGABRT)

Ok:                    0
Expected Fail:         0
Fail:                  1
Unexpected Pass:       0
Skipped:               0
Timeout:               0


The output from the failed tests:

1/1 Util                                    FAIL     0.01 s (killed by signal 6 SIGABRT)

--- command ---
16:07:52 /home/avojak/git/libsemver/build/tests/Util
--- stdout ---
# random seed: R02S9ff2e81c6ab0e5ea9e06492ec046b612
1..2
# Start of util tests
Bail out! ERROR:../tests/UtilTest.vala:14:sem_ver_util_test_test_addition: 'g_strcmp0 (_tmp1_, "1") == 0' should be TRUE
--- stderr ---
**
ERROR:../tests/UtilTest.vala:14:sem_ver_util_test_test_addition: 'g_strcmp0 (_tmp1_, "1") == 0' should be TRUE
-------

Full log written to /home/avojak/git/libsemver/build/meson-logs/testlog.txt
FAILED: meson-test 
/usr/bin/meson test --no-rebuild --print-errorlogs
ninja: build stopped: subcommand failed.
make: *** [Makefile:19: test] Error 1
```

Ok... we can conclude that we failed on the assertion on line 14, but the output is not exactly friendly... Our assertion was:

```vala
assert_true (Util.large_number_addition ("0", "0") == "1"); // Will fail intentionally
```

It's easy to see how in a slightly more complicated test this would not be as simple to figure out. Additionally, if you use any
sort of helper function in your testing to do assertions, there's no trace. That means any tests using that helper function would
show the same line number in case of a failure.

### Conclusion

Writing unit tests with GLib Test feels a bit cludgy.

1. Poor feedback from failed assertions (need better messaging)
2. Poor visibility as to which test case failed (output is only on a class level, so if a class has multiple assertions they all get globbed together)

## What's Next?

### Better Unit Testing

As a next step, I would really like to spend some time digging into a project that I recently discovered called Valadate, which is a testing framework built on top of GLib Test:

{% include github-card.html
  user="chebizarro"
  repository="valadate"
%}

Although it is no longer maintained, I've forked the project and hope that it can still be of use.

I would also like to explore testing of Gtk widgets. 

### Deployment of Documentation

Although the library is currently configured to generate a documentation site using Valadoc, it's not deployed anywhere. I would like to explore some options, including leveraging GitHub Pages for static hosting. One challenge here will be managing multiple versions of the documentation (e.g. `1.0.0` vs. `latest`).