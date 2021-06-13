# GraalVM getting started

Here you will find information about tools needed to build `native-image` tool which can compile Java programs into native executables. Also you will learn how to install and use those tools, how to build executables for Java programs. Important thing is to learn how to debug some specific piece of code, which will also be presented in this document. There are two types of code in GraalVM project, hosted and non-hosted. You will learn the difference here.

**GraalVM** is a virtual machine which can run different languages. First idea was to make Java faster, but soon it became much more. One runtime for all languages. You can read more about GraalVM [here](https://www.graalvm.org/docs/introduction/).

## Prerequisites

It is recommended to use a **Linux-based** operating system. This guide assumes you are running Ubuntu, however the Ubuntu-specific commands such as package installations can be translated to an arbitrary distribution.

In order to successfully build and run Graal projects, one needs to have the following dependencies satisfied:
- `git` 
- `build-essential`
- Python 2.7, required in order to run our build tool
- [`zlib`](https://www.zlib.net/), for Ubuntu the package is named `zlib1g` 
- an IDE for Java development

### Setting up IntelliJ IDEA

Download and install the latest [IntelliJ IDEA Community Edition]( https://www.jetbrains.com/idea/download/).
Change the IntelliJ maximum memory to 2 GB or more. As per the [instructions](https://www.jetbrains.com/idea/help/increasing-memory-heap.html#d1366197e127), from the main menu choose **Help | Edit Custom VM Options** and modify the **-Xmx** and **-Xms** options.

Enable parallel builds in **Preferences > Build, Execution, Deployment > Compiler > Compile independent modules in parallel**.

Open IntelliJ and go to **Preferences > Plugins > Browse Repositories**. Install the following plugins:

* [Eclipse Code Formatter](https://plugins.jetbrains.com/plugin/6546): formats code according to Eclipse
* [Checkstyle-IDEA](https://plugins.jetbrains.com/plugin/1065): runs style checks as you develop
* [Save Actions](https://plugins.jetbrains.com/plugin/7642): allows code reformatting on save similar to Eclipse
* [FindBugs-IDEA](https://plugins.jetbrains.com/plugin/3847): looks for suspicious code
* [Python Plugin](https://plugins.jetbrains.com/idea/plugin/631-python): python plugin
* [Markdown Navigator](https://plugins.jetbrains.com/plugin/7896-markdown-navigator): markdown plugin

Check that the bundled Ant plugin is enabled in **Preferences > Plugins > Installed** (you may get `Unknown artifact properties: ant-postprocessing.` errors in your project artifacts otherwise).

You can read more setting up IntelliJ IDEA [here](https://github.com/graalvm/mx/blob/master/docs/IDE.md).

## Installation

### Clone graal repository
First, clone the entire [Graal](https://github.com/oracle/graal/) repository:
```sh
$ git clone https://github.com/oracle/graal
```

This repository consists of several subdirectories:
* [GraalVM SDK](https://github.com/oracle/graal/blob/master/sdk/README.md) contains long term supported APIs of GraalVM.
* [GraalVM compiler](https://github.com/oracle/graal/blob/master/tree/master/compiler/README.md) written in Java that supports both dynamic and static compilation and can integrate with
the Java HotSpot VM or run standalone.
* [Truffle](https://github.com/oracle/graal/blob/master/truffle/README.md) language implementation framework for creating languages and instrumentations for GraalVM.
* [Tools](https://github.com/oracle/graal/blob/master/tools/README.md) contains a set of tools for GraalVM languages
implemented with the instrumentation framework.
* [Substrate VM](https://github.com/oracle/graal/blob/master/substratevm/README.md) framework that allows ahead-of-time (AOT)
compilation of Java applications under closed-world assumption into executable
images or shared objects.
* [Sulong](https://github.com/oracle/graal/blob/master/sulong/README.md) is an engine for running LLVM bitcode on GraalVM.
* [GraalWasm](https://github.com/oracle/graal/blob/master/wasm/README.md) is an engine for running WebAssembly programs on GraalVM.
* [TRegex](https://github.com/oracle/graal/blob/master/regex/README.md) is an implementation of regular expressions which leverages GraalVM for efficient compilation of automata.
* [VM](https://github.com/oracle/graal/blob/master/vm/README.md) includes the components to build a modular GraalVM image.
* [VS Code](https://github.com/oracle/graal/blob/master/vscode/README.md) provides extensions to Visual Studio Code that support development of polyglot applications using GraalVM.

In order to successfully build Graal, you will need a command-line tool called `mx`.


### Clone mx repository

`mx` is a command line based tool for managing the development of (primarily) Java code. It includes a mechanism for specifying the dependencies as well as making it simple to build, test, run, update, etc the code and built artifacts. `mx` contains support for developing code spread across multiple source repositories. `mx` is written in Python and is easily extendable.

First, clone the `mx` repository:
```sh
$ git clone https://github.com/graalvm/mx/
```

`mx` can be run directly (i.e., `python mx/mx.py ...`), but is more commonly invoked via the `mx/mx` bash script. Adding the `mx/` directory to your `PATH` simplifies executing `mx`.

```sh
$ export PATH=/path/to/mx/directory:$PATH
```

You can also add this line to your shell configuration file. For `bash`, you can add it to your `.bashrc` file. Don't forget to reload your shell configuration after adding those changes.

Alternatively, you can also add an alias to your shell configuration file so you can use `mx` from anywhere:
```sh
$ alias mx=/path/to/mx/executable 
``` 

### Installing JDK with JVMCI

In order to build Graal components, you will need a JDK with [JVMCI](https://openjdk.java.net/jeps/243) enabled. These specific JDK versions can be either downloaded from [GraalVM organization](https://github.com/graalvm), or using `mx`. `JAVA_HOME` should point to a JDK with JVMCI enabled. It is recommended to use `labs-openjdk-11`.

#### Downloading labs-openjdk-11 from GitHub repository

Pick a release from [labs-openjdk-11 repository](https://github.com/graalvm/labs-openjdk-11/releases) with regards to your OS. You will need to extract the downloaded archive and set the `JAVA_HOME` to the extracted directory, for example (if you extracted `labs-openjdk-11` to `/usr/lib/java/labs-openjdk-11`):
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

The organizing principle of `mx` is a _suite_. A suite is both a directory and the container for the components of the suite. A suite may import and depend on other suites.

The definition of a suite and its components is located in a file named `suite.py` in the `mx` metadata directory of the primary suite. This is the directory named `mx.<suite name>` in the suite's top level directory. For example, for the compiler suite, it is `mx.compiler`.

Relevant commands for build process:
- `mx build` command builds one suite (if you want to build a specific Graal subproject like Truffle, Sulong, compiler etc, you should go to the specific directory and then invoke `mx build` from there)
- `mx graalvm-home` shows the path to the latest build output directory, which is located in `sdk/` directory - you should have 2 symbolic links pointing to `latest_graalvm` and `latest_graalvm_home`. `latest_graalvm_home` points to the latest build output.

The `/vm` suite allows you to build custom GraalVM distributions by specifying which components you wish to include. This can be done via command-line arguments, environment variables or files.

### Specifying components to build via command-line arguments

You can specify a list of components to include via `--dynamicimports` (`--dy` for short) option when invoking `mx build`, for example:
```sh
$ mx --dynamicimports /substratevm,/sulong,/truffle build
```
Note that we are referencing the components relatively to the root of the Graal repository. 

### Specifying components to build via environment variables

You can also specify which components you wish to include via `DYNAMIC_IMPORTS` environment variable, for example:
```sh
$ export DYNAMIC_IMPORTS=/substratevm,/sulong,/truffle
$ mx build
```

Or, even simpler:
```sh
$ DYNAMIC_IMPORTS=/substratevm,/sulong,/truffle mx build
```

### Specifying components to build via environment files

`mx` metadata directory contains environment files where you can specify you own environment variable set. For example, the `mx.vm` metadata directory inside the `/vm` component contains `native-ce` environment file with the following contents:
```sh
DYNAMIC_IMPORTS=/substratevm
DISABLE_INSTALLABLES=true
EXCLUDE_COMPONENTS=pbm
NATIVE_IMAGES=native-image,lib:native-image-agent
```

You can specify which environment file `mx` should use by passing it as an argument to `--env` option, for example:
```sh
$ mx --env ni-ce build
```

### Building substratevm

Invoking `mx build` from `/substratevm` directory builds `substratevm` project. After the build is completed, you can find `native-image` executable inside `latest_graalvm_home/bin` directory. You can also add this directory to your `PATH` in order to invoke `native-image` from anywhere on the system. Invoking `java` or `javac` will then invoke GraalVM `latest_graalvm_home/bin/java` or `latest_graalvm_home/bin/javac`. You can also use `mx graalvm-home` to find the path to the `latest_graalvm_home` directory, which you can then use in your shell configuration files.

Now you can test the `native-image` tool that we just built.


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
$ java HelloWorld.java
```

You can verify that the compilation was successful by running the compiled class:
```sh
$ time java HelloWorld
Hello World!
real	0m0.229
user	0m0.094s
sys	0m0.039s
```

Now you can make a native image (we will explain these phases in the following sections):
```sh
$ native-image HelloWorld
// TODO add output
```

This creates an executable named `helloworld` which does not require a JVM to be run, which significantly reduces execution time, which you can simply verify:
```sh
$ time ./helloworld
Hello World!
real	0m0.009s
user	0m0.001s
sys	0m0.008s
``` 

## Native image build phases (short):

// TODO 

- **Analysis**: going through all Java functions nedeed in program. You cannot take whole Java standard library and put it into executable, so this phase finds out all important and used functions and takes them, compiles them and make binaries from them which are part of created executable.

You can distinct **hosted** and **non-hosted** code. Hosted code is Java code that executes during building of native image of the program. On the other hand non-hosted code is part of code that runs during program execution. There is some code that executes only during build, and some code is used only in execution. To distinct those codes, annotations are used. Some classes are partially hosted, partially non-hosted, so annotations help to distinct those types of codes.


## Debugging
Debugging hosted and non-hosted code is different. To debug hosted code you can use IDE debugger and if you want to debug non-hosted code, you have to use debugger that can debug executables (gdb for example).

### Debugging non-hosted code
If you want to debug non-hosted code, you should use gdb, or any other debugger thath can debug binary code (executables). To build  image with debug symbols add flag `-g` to native-image compilation process. 
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

Command `next` gdb looks one line as an instruction and executes it as one instruction. Similar as `step over` command in IntelliJ.

Command `step` executes line but if the instruction represents call of a function, `step` will go into function body and will execute function instructions one by one. Similiar to `step into` command in IntelliJ. 

Command `bt` gives us backtrace of current stack.

### Debugging hosted code

If you want to debug hosted code, you can use standard debugger from IDE. During build of image using `native-image` tool, you have to specify that we want debugging of hosted code using `--debug-attach`.
```sh
$ native-image --debug-attach HelloWorld
```
*Note: You have to set breakpoint inside IDE, use `native-image` command with mentioned option, and then run debug mode inside IDE to debug hosted code.*

To see how to get native code from Java code you can see internal Graal representation using [IdealGraphVisualizer](https://www.oracle.com/downloads/graalvm-downloads.html). Just unzip archive, position to directory where you unzipped it, go to `bin` directory and then run `idealgraphvisualizer`:
```sh
$ ./idealgraphvisualizer
```
To build an executable and see it's internal representation type the following:
```sh
$ native-image HelloWorld -H:Dump=:3 -H:MethodFilter=HelloWorld.main -H:Optimize=0 -H:PrintGraph=Network
```

## More
You can read more about GraalVM community contributors [here](https://www.graalvm.org/community/contributors/).
You can read more about GitHub[here](https://docs.github.com/en/github/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests).
