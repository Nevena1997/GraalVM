# GraalVM
## Getting Started

Here you will find information about tools needed to build native-image tool which can compiler Java programs into native executables. Also you will learn how to install and use those tools, how to build executables for Java programs. Important thing is to learn how to debug some specific piece of code, which will also be presented in this document. There are two types of code in GraalVM project, hosted and non-hosted. You will learn the difference here.

**GraalVM** is virtual machine which can run different languages. First idea was to make Java faster, but soon it became much more. One runtime for all languages. More about GraalVM you can find at the following link: https://www.graalvm.org/docs/introduction/.

## Installation

### Clone graal project
First clone graal project from https://github.com/oracle/graal.
This is directory which contains all files and directories nedeed for GraalVM.

To build graal and native-image tool (which is used to build executable files written in Java and other languages) you will need ```mx``` tool.

### mx

You need to clone mx tool from https://github.com/graalvm/mx. 
When you clone project, inside mx folder you will find  executable file named mx. That is pre-built mx tool which can be used for building graal project, native images, and far more.

You can add alias to your .bashrc file so you can use command ```mx``` from anywhere. 
Path to .bashrc file is ~/.bashrc or /home/user/.bashrc. You open that file and in the bottom you add alias like this: 
```
alias mx=PATH_TO_MX_EXECUTABLE 
```
After that inside terminal type ```source ~/.bashrc```. 
Now you can use command ```mx <command>``` from any location.

### Building native images:
Tools and repos nedeed:

https://github.com/oracle/graal - Directory which contains all files and directories nedeed for GraalVM.
https://github.com/graalvm/mx - mx build tool (not only a build tool, but general purposes tool).

- ```mx build``` command builds one graal release (if you want to build one specific graal subproject like truffle, sulong, compiler etc, you should go to the specific directory and then run command ```mx build``` from that directory)
- to find release you built just type ```mx graalvm-home```
- to find latest built release go to graal/sdk directory, you should have 2 symbolic link pointing to ```latest_graalvm``` and ```latest_graalvm_home```. latest_graalvm_home directory is the directory where the latest build is (no matter from where in project command mx build was executed).

>**IMPORTANT:** 
In terminal in which you want to build graal, JAVA_HOME should point to downloaded labs JDK. 
In terminal in which you want to run native-image command (to build native images/executables) JAVA_HOME can point to latest_graalvm_home, but it is not necessary, but you have to add latest_graalvm_home/bin to PATH. It is good to do that in ~/.bashrc so it is always set.

Suites are subprojects that can be built using ```mx``` build command. All directories that includes directory named mx.NAME_OF_THE_CURRENT_DIRECTORY are suites. For example vm is suite because inside vm directory we have mx.vm directory. Inside those directories, mx finds some meta data that mx builder uses to build project.

Some subprojects have dependencies on other subprojects (for example, if we run ```mx build``` from substratevm directory, it will include all other subprojects on which this subproject is dependent on). Subproject vm has no dependencies on other subprojects, so if we run ```mx build``` from this directory we won't get much. If we want to build some other subprojects too, we can do that in two ways:

1. Type ```mx --dynamicimports /PATH_TO_OTHER_SUBPROJECT build```. Note that / here represents root of the graal project. 
```
mx --dynamicimports /substratevm build 
```

If you need more than one suite you should separate their names by comma
``` 
mx --dynamicimports /substratevm,/sulong,/truffle build
```
You can also use dy instead of dynamic-imports. 

```
mx --dy /substratevm,/sulong,/truffle build
```

2. Make enviorement. There are some predefined enviorements in ```mx.vm``` directory. Content of native image community edition enviorement is:
```
DYNAMIC_IMPORTS=/substratevm
DISABLE_INSTALLABLES=true
EXCLUDE_COMPONENTS=pbm
NATIVE_IMAGES=native-image,lib:native-image-agent
```
If you want to use enviorement to build graal, just type ```mx --env NAME_OF_THE_ENVIOREMENT build```.
```
mx --env ni-ce build
```

### JDK
JDK with JVMCI - so-called labs JDK - https://github.com/graalvm/labs-openjdk-11/releases. JAVA_HOME should be set to this JDK. 
You can do this in two ways:

1. From a website https://github.com/graalvm/labs-openjdk-11/releases, pick a release for your Operating system and distribution  (Linux for example). Then unzip archive. If you are using **Linux which is highly recommended** command is ```tar -xf name_of_the_archive.tar.gz```. After that, type ```export JAVA_HOME=ABSOLUTE_PATH_TO_UNZIPPED_DIRECTORY```.

2. If you have ```mx``` installed and configured, just type ```mx fetch-jdk --to ABSOLUTE_PATH_TO_DIRECTORY```. You should get console menu where you choose Java 11. Java 8 is also supported, but you need Java 11 most of the time. This command will find, download and extract needed archive. After that just type ```export JAVA_HOME=ABSOLUTE_PATH_TO_UNZIPPED_DIRECTORY```.


After building GraalVM release, go to bin directory inside latest_graalvm_home. There you can see different executables including javac and java. Those are executables that can be used instead of default javac and java executables. Too see which executable is set by  default type which java command in terminal.

```which java```
Output: ```/usr/bin/java```

If you want to make your graalvm java executable deafult, you should add latest_graalvm_home/bin to PATH and set JAVA_HOME to  latest_graalvm_home.

```export PATH=PATH_TO_LATEST_GRAALVM_HOME_DIRECTORY/bin```

Now, if you type "which java" you will get the latest built graalvm release. Additionally, if you make any changes and then rebuild graalvm, it will still point to the latest version because symbolic link we set JAVA_HOME to always points to the latest built release. 

After we build graal and native image tool (for building native images/executables) we need one Java class to test everything.

-------------------------------------------------------------------------------------------------------------------------------------------------------
### Example: HelloWorld.java


```Java
import java.lang.*;

public class HelloWorld
{
    public static void main(String args[])
    {
        System.out.println("Hello World!");
    }
}
```



Command ```javac HelloWorld.java ``` will make ```.class``` file.
Then we can run it using command ```java HelloWorld```.

If you want to make executable java file with native-image to build it, use command ```javac HelloWorld.java``` and then ```native-image HelloWorld```

After this command we have an executable named ```helloworld```.

If we measure execution time for those commands we get the following:

```time java HelloWorld```
Output:
```
Hello World!
```
real	0m0.229
user	0m0.094s
sys	0m0.039s

```time ./helloworld```
Output:
```
Hello World!
```
real	0m0.009s
user	0m0.001s
sys	0m0.008s

The difference is obvious because with native images we are running executables, similar to C executables which is much faster.

### Building phases (short):

- **Analysis**: going through all Java functions nedeed in program. You cannot take whole Java standard library and put it into executable, so this phase finds out all important and used functions and takes them, compiles them and make binaries from them which are part of 
created executable.

We can distinct **hosted** and **non-hosted** code. Hosted code is Java code that executes during building of native image of the program. On the other hand non-hosted code is part of code that runs during program execution. There is some code that executes only during build, and some code is used only in execution. To distinct those codes, annotations are used. Some classes are partially hosted, partially non-hosted, so annotations help to distinct those types of codes.

## Debugging
Debugging hosted and non-hosted code is different. To debug hosted code you can use IDE debugger and if you want to debug non-hosted code, you have to use debugger that can debug executables (gdb for example).

#### Debugging non-hosted code
If you want to debug non-hosted code, you should use gdb, or any other debugger thath can debug binary code (executables). To build  image with debug symbols add flag ```-g``` to native-image compilation process. 
```
native-image HelloWorld -g
```
If you want to turn off optimizations you can add flag ```-H:Optimize=0``` during compilation.
```
native-image HelloWorld -g -H:Optimize=0
```

**Adding breakpoint using gdb:**

b ClassName::functionName
Ex: ```HelloWorld::main```

Running code using gdb:
Ex: ```gdb ./helloworld```

To run program in debug mode using gdb type run
Ex: ```(gdb) run```

**GDB Layouts:**

Too see src file and line where execution stopped type layout src:
Ex: ```(gdb) layout src```

To see assembly code type layout src:
Ex: ```(gdb) layout asm```

To see memory registers type layout reg:
Ex: ```(gdb) layout reg```

Command ```next``` gdb looks one line as an instruction and executes it as one instruction. Similar as ```step over``` command in IntelliJ.

Command ```step``` executes line but if the instruction represents call of a function, ```step``` will go into function body and will execute function instructions one by one. Similiar to ```step into``` command in IntelliJ. 

Command ```bt``` gives us backtrace of current stack.

#### Debugging hosted code

If you want to debug hosted code, you can use standard debugger from IDE. During build of image using native-image command, we have to specify that we want debugging of hosted code using ```--debug-attach```.
```
native-image --debug-attach HelloWorld
```
> **IMPORTANT:**
You have to set breakpoint inside IDE, use native-image command with mentioned option, and then run debug mode inside IDE to debug hosted code.

To see how to get native code from Java code you can see internal Graal representation using IdealGraphVisualizer. This software can be downloaded from https://www.oracle.com/downloads/graalvm-downloads.html. Just unzip archive, position to directory where you unzipped it, go to bin directory and then run idealgraphvisualizer executable.
```
./idealgraphvisualizer
```
To build executable and see it's internal representation type the following:
```
native-image HelloWorld -H:Dump=:3 -H:MethodFilter=HelloWorld.main -H:Optimize=0 -H:PrintGraph=Network
```
More about IdealGraphVisualizer you can find at https://docs.oracle.com/en/graalvm/enterprise/19/guide/tools/ideal-graph-visualizer.html.