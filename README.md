# Introduction to Chisel for Biginners
How to start hardware description language of Chisel3 for beginners.

## Purpose of This Document
Beginners who have not experience about Chisel and Scala programming languages.

## What is Chisel?
Chisel is to generate HDL code from high-level language, which is made with wrapping Scala language. Current version is Chisel3. **Class** is converted to HDL.

## Prerequisites

Prerequisites are;
- Java Runtime Environment (for running Scala)
- SBT (Simple Built Tool, to compile chisel code)

## Installations

There are two user types. One is **normal** user who want to compile their chisel code, so they want to use chisel as a common tool, they should use **released** chisel. Other one is **deveropper** user who needs advanced tool which is not yet released (ex. **rc** version), and or want to costomize the tools.

### Common Installations for Users

- Java Runtime Environment (JRE)
  
  JRE is probably installed in your computer. You can check it through terminal;
```
  java -version
```

- SBT (installation instructions)

  https://github.com/freechipsproject/chisel3/wiki/Installation-Preparation


### Installation for Normal Users

  By using template of **build.sbt**, SBT invokes necessary **released** tool softwares defined in the build.sbt. Template is always upto-date, so normal users need not to manual installation works.

  https://github.com/freechipsproject/chisel-template

By doing **copy and paste** the template's build.sbt into your project directory, you can do your compilation!


### Installation for Developpers

To install, doing following command in the directory;
```
  sbt compile
```

- Stable Version of Recent Chisel
  
  Stable Recent Version: 3.4.0
  https://github.com/freechipsproject/chisel3/releases/tag/v3.4.0

- Tags for Recent Versions

  You can see recent versio at here;
  https://github.com/freechipsproject/chisel3/tags


- iotesters

  iotesters is old version of verification tool which run on Scala and can generate VCD file.

  https://github.com/freechipsproject/chisel-testers/tree/master/src/main/scala/chisel3/iotesters

- tester2 (chisel-testers)

  tester2 is second generation of verification tool, currently this is on a main stream.

  https://github.com/freechipsproject/chisel-testers


## Project Directory Structure
Preferred project directory structure is as follows;
```
root--+--build.sbt
      |
      +--src--+--main--+--scala--"YOUR_SOURCE_CODES.scala"
              |
              +--test--+--scala--"YOUR_TEST_CODES.scala"

```
This is **strict** *constraint*, all chisel codes belong to this directory structure.

## Compilation

At the compilation, SBT checks directory structure, and so, you do not need to specify which file(s) should be compiled. SBT traces a  *class-dependency* from top-class as a root.

### Before Your Compilation

You must decide following point before starting your compilation;
1. Project Name (OPTION)
  This name can be defined build.sbt, "name"-field. You can see the name at the compilation, this helps that you work on what project.
2. Top *class* similar to HDL needs (see bellow)
  At the compilation, you need to set **which class** is a *top* module of HDL. So, you can choose preferred class as top module of HDL by specifying the name.
3. "YOUR_TEST_CODES.scala" (see bellow)
  This is needed for using **iotesters**.

### Top Class
Top class description bellow should be added to your a *top-class* file, or a file you want to test/generate HDL. Replace "ProjectName" with your project's name.
```
object ProjectNameMain extends App {
  chisel3.Driver.execute(args,()=>new ProjectName("_args_"))
}
```
where ```_args_``` are arguments you defined in your class as a succeeded parameter(s) (option). You can do naming this object's name ```ProjectNameMain```, freely. This description is needed to generate HDL.

### How to Compile Your Code
Compiling on a terminal is simply as follows;
```
sbt 'runMain ProjectNameMain'
```
**NOTE**: replace "ProjectName" with your project's name, and if you have set package name ```package_name```;
```
sbt 'runMain package_name.ProjectNameMain'
```

The ```runMain``` invokes ```ProjectNameMain``` object and chisel works for this object.

### How to Test Your Code

Test on a terminal is simply as follows;
```
sbt 'test:runMain ProjectNameMain'
```


### YOUR_TEST_CODES.scala for iotesters
This file is needed to test your code on **iotesters** (**not tester2**). Set the test file name with ProjectNameMain.scala (replace "ProjectName" with your project's name). The file has to have bellow code at least;
```
import chisel3._

object ProjectNameMain extends App {
  iotesters.Driver.execute(args, () => new ProjectName) {
    c => new ProjectNameUnitTester(c)
  }
}

object ProjectNameRepl extends App {
  iotesters.Driver.executeFirrtlRepl(args, () => new ProjectName)
}
```
where "ProjectName" in the code must be replaced with your project's name. We do **not** recommend to modify this template file for beginners.


## Small Tips

- **Use of Utilities**

  If you want to use utility function prepared already such as ```Log2()```, then add following import description in your code;
```
  import chisel3.util._
```
  Utilities are listed in;

  https://www.chisel-lang.org/api/latest/index.html#chisel3.util.package

- **Displaying Hexadecimal Number on iotersters**
  
  You can display hexadecimal number instead of decimal number with following option (**only** for iotesters);
```
  --display-base 16
```

- **Reduction Operation across Vec**

  Let us see bellow declaration
```
  val hoge = Vec(Size, Bool())
```
And we want to do OR-reduction. There are two ways;
1. Cast to a UInt and use orR;
```
  hoge.asUInt.orR
```
orR is OR-reduction.

2. Use Scala's reduction method;
```
  hoge.reduce(_ || _)
```

- **Initialization of Bundled Primitive**

  Let us see following code;
```
  class datum (DataWidth: Int) extends Bundle {
    val valid = Bool()
    val data  = UInt(DataWidth.W)
}
```

Class ```datum``` has to primitives; ```valid``` (bool type) and ```data``` (unsigned DataWidth-bit integer). Then you can use the class as a register with Initialization as follows;
```
  val Datum = RegInit(0.U.asTypeOf(new datum(DataWidth)))
```
  Both of valid and data "reg with init var"s in RTL is *zero`` cleared (unsigned zero) by hardware "reset" signal. You can specify your own value.The reset (and also clock) is added automatically to the HDL.

- **Multi-Primitive**
  
  You might want to have multiple instances. ```Vec`` method help us to coding without redundant efforts.
  For example, port class which defines I/O and bundles several I/Os, then we can define Num ports like this;
```
  val io = IO(new Bundle {val port = Vec(Num, new Port(Width))})
```

  Then we can specify identical object such by ```io.port(3).xxx``` for fourth object. **NOTE** that ```3``` is **not** Chisel's data-type, it is Scala's data-type. So, you can combine this description with Scala's coding style (because of Chisel uses Scala language), such as for-loop.

- **Multi-Instance with List**

  When an adder ```Adder()``` is an instance class defining adder logic circuit, then we can define "Num" adders like this;
```
  val ADD = List.fill(Num)(Module(new Adder(Width)))
```
  Then we can also specify identical object such as ```ADD(3)```.


## Error Messages
- "[error] *OutOfMemory*"

  **Meaning**: JVM needs more heap memory space.

  **Solution**: To Specify giving space, add this option at compilation;
```
  -mem 4096
```
  This gives 4GiB space in terms of MiB.

- "[error] *java.lang.ClassNotFoundException: ProjectName*"

  **Meaning**: There is no ProjectName top module in your source file(s).

  **Solution**: Check top module name.

- [exception] "*CheckInitialization$RefNotInitializedException*"

  **Meaning**: If your code does have unknown state on the port "PortName" (or, wire) in switch statement, the exception might caused by FIRRTL's procedure. So, register does not make this error.

  **Solution**: You can temporally fix this issue by assigning "*DontCare*" which is reserved variable indicating "**Do not Care**". You can use it before assigning value to the port, like as follows;
```
  Port/Wire := DontCare
```

  **NOTE**: the ```DontCare``` works every where. If you use this assignment for other purpose then you will meet failed HDL generation.
