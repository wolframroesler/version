# Build Number Generation With git And cmake

This example program shows how to achieve the following:

* Manage a project's version number in annotated git tags only (no manual editing in header files when bumping the version number, just create the tag)
* Automatically generate a unique build number with every git commit, which can be used to retrieve the exact source code used in that build
* Compile a "version identifier" that contains the version number, the build number, the build date and time, and platform details (host name, user name, OS version number, etc.)
* Make this version identifier available to the program (e. g. print it when invoked with `--version`)
* Make it possible to extract the version identifier from an executable without having to execute it

## Running The Example Program

```sh
$ mkdir build
$ cd build
$ cmake ..
$ make
```

This creates the `version` program which, when invoked, simply displays its version number and exists. For example:

```sh
$ ./version
Version 1.0-3-g5c4af1f (built 2018-01-08 19:32:17 by wolfram on MacBook-Air with Darwin 16.7.0)
```

Use `strings` and `grep` to extract the version number from the executable, in which it is marked with the magic string `@(#)`. (Or use the ancient `what` program if youre system is old enough to have it.)

```sh
$ strings version | grep '@(#)'
@(#)1.0-3-g5c4af1f (built 2018-01-08 19:32:17 by wolfram on MacBook-Air with Darwin 16.7.0)
```

## How To Make The Build Number

For release versions (that have a git tag with the version number on them), there is no build number beside the version number. For example:

```sh
$ git checkout 1.0
$ make
$ ./version
Version 1.0 (built 2018-01-08 19:50:23 by wolfram on MacBook-Air with Darwin 16.7.0)
```

When new commits are added, the build number indicates the commits since the version tag, plus `-g` followed by the commit ID.

```sh
$ git checkout develop
$ make
$ ./version
Version 1.0-3-g5c4af1f (built 2018-01-08 19:57:34 by wolfram on MacBook-Air with Darwin 16.7.0)
```

When there an uncommitted changes, the build number contains the string `-dirty`. This indicates that it may not be possible to retrieve this version's source code from the repository.

```sh
$ vi main.cpp
$ make
$ ./version
Version 1.0-3-g5c4af1f-dirty (built 2018-01-08 20:03:06 by wolfram on MacBook-Air with Darwin 16.7.0)
```

All this is accomplished with the `git describe` command, which is what we use to construct version number and build number. This means that we don't need to keep the version number anywhere in the source code, it's enough to have it in the git tags.

```sh
$ git describe --dirty
1.0-3-g5c4af1f-dirty

$ ./version
Version 1.0-3-g5c4af1f-dirty (built 2018-01-08 19:32:17 by wolfram on MacBook-Air with Darwin 16.7.0)
```

## How To Add The Build Details

... like time stamp, machine/computer name, etc.

This is accomplished by a shell script, `makeversion.sh`, which uses simple shell tools like `date`, `uname`, etc. (and, of course, `git describe`) to compile the version identifier. The script outputs its result as a C/C++ function that returns the version string.

```sh
$ bash makeversion.sh
char const *Version() {
    return "@(#)1.0-3-g5c4af1f (built 2018-01-08 20:23:33 by wolfram on MacBook-Air with Darwin 16.7.0)" + 4;
}
```

To access the version identifier in the program, simply invoke this function.

```cpp
std::cout << "Version " << Version() << std::endl;
```

The C string that contains the version identifier begins with the "magic string" `@(#)` which we use to retrieve the version number from the executable as described above. The `Version` function, however, skips the magic string and returns only what comes after it.

## How To Put It All Together

This is where the actual magic happens.

When invoked with the `-o` parameter, `makeversion.sh` compiles its C/C++ output into an object file. The cmake file uses `add_custom_command` to invoke `makeversion.sh -o` during each build, and contains the resulting object file in its `target_link_libraries`.

This is done as a "pre link" step, i. e. only when there's actual work to do. Simply typing `make` again (without any source change, `make clean`, etc.) won't trigger a new link and thus won't create a new version number .

## Ressources

* About `git describe`: https://git-scm.com/docs/git-describe
* About cmake's `add_custom_command`: https://cmake.org/cmake/help/v3.0/command/add_custom_command.html
* About `@(#)` and `what`: http://pubs.opengroup.org/onlinepubs/9699919799/utilities/what.html


---
*Wolfram Rösler • wolfram@roesler-ac.de • https://github.com/wolframroesler • https://twitter.com/wolframroesler • https://www.linkedin.com/in/wolframroesler/*
