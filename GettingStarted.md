# Understanding GraalVM and native-image - Overview, Building & Contributing

**GraalVM** is a virtual machine which can run different languages. First idea was to make Java faster, but soon it
became much more: one runtime for many languages. You can read more about
GraalVM [here](https://www.graalvm.org/docs/introduction/).

In this document, you will find information about:

- Tools needed to build `native-image`, a tool which can compile Java programs into native executables
- How to install and use these tools
- How to build executables for Java programs
- How to debug some specific piece of code
- The difference between hosted and non-hosted types of code.

## Prerequisites

It is recommended to use a **Linux-based** operating system. This guide assumes you are running Ubuntu, however the
Ubuntu-specific commands such as package installations can be translated to an arbitrary distribution.

In order to successfully build and run Graal projects, one needs to have the following dependencies satisfied:

- `git`
- `build-essential`
- Python 2.7, required in order to run our build tool
- [`zlib`](https://www.zlib.net/), for Ubuntu the package is named `zlib1g`

## Installation

### Clone the graal repository

First, clone the entire [Graal](https://github.com/oracle/graal/) repository:

```sh
$ git clone https://github.com/oracle/graal
```

You can learn more about its subdirectories [here](https://github.com/oracle/graal#repository-structure).

In order to successfully build Graal, you need a command-line tool called `mx`.

### Clone the mx repository

[`mx`](https://github.com/graalvm/mx) is a command line based tool for managing the development of (primarily) Java
code. It includes a mechanism for specifying the dependencies as well as making it simple to build, test, run, update,
etc. the code and built artifacts. `mx` contains support for developing code spread across multiple source
repositories. `mx` is written in Python and is easily extendable.

First, clone the `mx` repository:

```sh
$ git clone https://github.com/graalvm/mx/
```

`mx` can be run directly (i.e., `python mx/mx.py ...`), but is more commonly invoked via the `mx/mx` bash script. Adding
the `mx/` directory to your `PATH` simplifies executing `mx`.

```sh
$ export PATH=/path/to/mx/directory:$PATH
```

You can also add this line to your shell configuration file.

### Installing the JDK with JVMCI

In order to build Graal components, you will need a JDK with [JVMCI](https://openjdk.java.net/jeps/243) enabled. These
specific JDK versions can be either downloaded from [GraalVM organization](https://github.com/graalvm), or using `mx`
. `JAVA_HOME` should point to a JDK with JVMCI enabled. It is recommended to use `labs-openjdk-11`.

#### Downloading labs-openjdk-11 from the GitHub repository

Pick a release from [labs-openjdk-11 repository](https://github.com/graalvm/labs-openjdk-11/releases) with regards to
your OS. You will need to extract the downloaded archive and set the `JAVA_HOME` to the extracted directory, for
example (if you extracted `labs-openjdk-11` to `/usr/lib/java/labs-openjdk-11`):

```sh
$ export JAVA_HOME=/usr/lib/java/labs-openjdk-11
```

You can also add this line to your shell configuration file.

#### Downloading labs-openjdk-11 using mx

You can also use `mx fetch-jdk` to download a JVMCI enabled JDK. For example:

```sh
$ mx fetch-jdk --to /where/you/wish/to/install
```

You will be prompted to select a JDK version.

*Note: Invoke `mx fetch-jdk` outside of the Graal repository.*

## Building Graal projects

The organizing principle of `mx` is a _suite_. More about suites you can
learn [here](https://github.com/graalvm/mx#suites).

Relevant commands for build process:

- `mx build` command builds one suite (if you want to build a specific Graal subproject like Truffle, Sulong, compiler
  etc, you should go to the specific directory and then invoke `mx build` from there)
- `mx graalvm-home` shows the path to the latest build output directory, which is located in `sdk/` directory - you
  should have two symbolic links pointing to `latest_graalvm` and `latest_graalvm_home`. `latest_graalvm_home` points to
  the latest build output.

The `/vm` suite allows you to build custom GraalVM distributions by specifying which components you wish to include.
This can be done via command-line arguments, environment variables or files. More about that you can
read [here](https://github.com/oracle/graal/blob/master/vm/README.md#vm-suite).

### Building substratevm

Invoking `mx build` from `/substratevm` directory builds `substratevm` project. After the build is completed, you can
find `native-image` executable inside `latest_graalvm_home/bin` directory. You can also add this directory to
your `PATH` in order to invoke `native-image` from anywhere on the system. Invoking `java` or `javac` will then invoke
GraalVM `latest_graalvm_home/bin/java` or `latest_graalvm_home/bin/javac`. You can also use `mx graalvm-home` to find
the path to the `latest_graalvm_home` directory, which you can then use in your shell configuration files.

Now you can test the `native-image` tool that you have just built.

## Creating a native image

Suppose you have a file `HelloWorld.java` with the following contents:

```java
import java.lang.*;

public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

You can start by compiling the source:

```sh
$ javac HelloWorld.java
```

You can verify that the compilation was successful by running the compiled class:

```sh
$ time java HelloWorld
Hello World!
real	0m0.229s
user	0m0.094s
sys	0m0.039s
```

Now you can make a native image:

```sh
$ native-image HelloWorld
[helloworld:6530]    classlist:  10,554.19 ms,  1.18 GB
[helloworld:6530]        (cap):     483.33 ms,  1.18 GB
[helloworld:6530]        setup:   3,861.20 ms,  1.18 GB
[helloworld:6530]     (clinit):     210.05 ms,  2.30 GB
[helloworld:6530]   (typeflow):   7,437.99 ms,  2.30 GB
[helloworld:6530]    (objects):   6,442.54 ms,  2.30 GB
[helloworld:6530]   (features):   5,034.89 ms,  2.30 GB
[helloworld:6530]     analysis:  19,454.27 ms,  2.30 GB
[helloworld:6530]     universe:     854.96 ms,  2.30 GB
[helloworld:6530]      (parse):   1,441.33 ms,  2.30 GB
[helloworld:6530]     (inline):   2,931.44 ms,  2.33 GB
[helloworld:6530]    (compile):  15,920.09 ms,  2.87 GB
[helloworld:6530]      compile:  21,238.28 ms,  2.87 GB
[helloworld:6530]        image:   3,215.30 ms,  2.87 GB
[helloworld:6530]        write:   2,310.63 ms,  2.87 GB
[helloworld:6530]      [total]:  61,977.20 ms,  2.87 GB
```

This creates an executable named `helloworld` which does not require a JVM to be run, which significantly reduces
execution time, which you can simply verify:

```sh
$ time ./helloworld
Hello World!
real	0m0.009s
user	0m0.001s
sys	0m0.008s
``` 

## Hosted and non-hosted code

You can distinct **hosted** and **non-hosted** code. Hosted code is Java code that executes during building of native
image of the program. On the other hand non-hosted code is part of code that runs during program execution. There is
some code that executes only during build, and some code is used only in execution. To distinct those codes, annotations
are used. Some classes are partially hosted, partially non-hosted, so annotations help to distinct those types of codes.

## Debugging

Debugging hosted and non-hosted code is different. To debug hosted code you can use IntelliJ IDEA debugger for Java
code. If you want to debug non-hosted code, you have to use debugger that can debug executables (gdb for example).

### Debugging non-hosted code

If you want to debug non-hosted code, you should use gdb, or any other debugger that can debug binary code (executables)
. To build image with debug symbols add flag `-g` to native-image compilation process.

```sh
$ native-image HelloWorld -g
```

If you want to turn off optimizations you can add flag `-H:Optimize=0` during compilation.

```sh
$ native-image HelloWorld -g -H:Optimize=0
```

#### Adding breakpoint using gdb

To invoke gdb:

```sh
$ gdb ./helloworld
```

To run program in debug mode using gdb type run

```sh
(gdb) run
```

You can add a breakpoint in `gdb` using `b` option:

```
b ClassName::functionName
```

For example:

```
(gdb) b HelloWorld::main
```

##### gdb layouts

Too see source file and line where execution stopped type `layout src`:

```sh
(gdb) layout src
```

To see assembly code type `layout asm`:

```sh
(gdb) layout asm
```

To see memory registers type `layout reg`:

```sh
(gdb) layout reg
```

Command `next` gdb looks one line as an instruction and executes it as one instruction. Similar as `step over` command
in IntelliJ.

Command `step` executes line but if the instruction represents call of a function, `step` will go into function body and
will execute function instructions one by one. Similar to `step into` command in IntelliJ.

Command `bt` gives us backtrace of current stack.

### Debugging hosted code

If you want to debug hosted code, you can use IntelliJ IDEA debugger for Java code. During build of image
using `native-image` tool, you have to specify that you want debugging of hosted code using `--debug-attach`.

```sh
$ native-image --debug-attach HelloWorld
```

By default native-image tool is listening for transport on port 8000 on localhost. If you want to change the port, you
can do that by specifying the port in the following way (for example, port 8080)

```sh
$ native-image --debug-attach:8080 HelloWorld
```

You will get the message

```sh
Listening for transport dt_socket at address: 8080
```

To debug hosted code using IntelliJ, go to `Run` and choose `Attach to Process`. You will be offered to choose the
process and a corresponding host port.

To be able to debug hosted code using IntelliJ, you have to enable remote debugging inside IDE. This can be done
manually or by using command `mx intellijinit`. If you want to do this manually, you can find more information at the
following [link](https://www.jetbrains.com/help/idea/tutorial-remote-debug.html#99f9e7ee).

## Visualization

To understand better the process of getting native code from Java code, you can visualize internal Graal representation
using [IdealGraphVisualizer](https://www.oracle.com/downloads/graalvm-downloads.html). First unzip the archive, then
position to the directory where you unzipped it, go to `bin` directory and then run `idealgraphvisualizer`:

```sh
$ ./idealgraphvisualizer
```

To build an executable and see it's internal representation type the following:

```sh
$ native-image HelloWorld -H:Dump=:3 -H:MethodFilter=HelloWorld.main -H:Optimize=0 -H:PrintGraph=Network
```

You can find more about
IdealGraphVisualizer [here](https://docs.oracle.com/en/graalvm/enterprise/19/guide/tools/ideal-graph-visualizer.html).

## How to contribute to GraalVM?

When you solve some issue or make a contribution that you want to share, you should find the appropriate way to
contribute to GraalVM. Read about common ways to collaborate on
GraalVM [here](https://www.graalvm.org/community/contributors/). If you want to propose changes with pull requests,
learn more about the GitHub pull
requests [here](https://docs.github.com/en/github/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests)
.

### Creating and maintaining pull request (PR) conventions

The name of the new branch should be **<github-username>/<what-branch-do>**. Along with PR, the contributor should
provide short description of what code is trying to solve. </br>
If PR is about a bug, then it should contain:

- what was the problem (stacktrace?) and how it's solved,
- command(s) to reproduce the issue,
- if it's possible, add a simplified version of the problem as a native image test.

If PR is a proposal, then it should contain:

- goal,
- context,
- impact on image size and execution and
- summary

When we successfully create PR, we should maintain them:

- The PR should be rebased (https://git-scm.com/book/en/v2/Git-Branching-Rebasing) on master daily. Stale code can cause
  merging conflict and weight reviewing.
- Number of commits in one PR should be kept low. Multiple commits can make it hard to do a rebase for you and the
  reviewer as well.
- Last but not least, commit messages should be descriptive (remember, maybe one day someone will need to search through
  commit history). See how to write good commit messages:
  (https://chris.beams.io/posts/git-commit/)


