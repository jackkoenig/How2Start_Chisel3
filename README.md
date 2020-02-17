# How2Start_Chisel3
How to start hardware description language of Chisel3 for beginners.

# Purpose of This Document
- Beginners who have not experience about Chisel and Scala programming languages.

# Prerequisites
Prerequisites are;
- Java (for running Scala)
- SBT (Simple Built Tool, to parse Chisel3 grammer on the Scala)

# Installations
- SBT (installation instructions)

https://github.com/freechipsproject/chisel3/wiki/Installation-Preparation

- Chisel3.2

https://github.com/freechipsproject/chisel3/releases/tag/v3.2.0

- Tags for Recent Versions

https://github.com/freechipsproject/chisel3/tags

To install chisel3.2, doing following command in the directory;

```
sbt compile
```

# Project Directory Structure
Required project directory structure is as follows;
```
root--+--build.sbt
      |
      +--project--+--build.properties
      |
      +--src--+--main--+--scala--"YOUR_SOURCE_CODES.scala"
              |
              +--test--+--scala--"YOUR_TEST_CODES.scala"

```
You can download a project template from;

https://github.com/freechipsproject/chisel-template

If you want to use "testers2" (recent scala-based test environment), this is here;

https://github.com/ucb-bar/chisel-testers2

# Set-up to Compile
You must decide following point before starting your project;
1. Project Name (defined in "name"-field in "build.sbt" file),
2. Top module similar to HDL needs (see bellow)
3. "YOUR_TEST_CODES.scala" (see bellow)

# Top module
Top module description bellow shoule be added to your a root-code file.
Replace "ProjectName" with your project's name.
```
object ProjectName extends App {
  chisel3.Driver.execute(args,()=>new ProjectName("_args_"))
}
```
where "_args_" are arguments you defined in your top module (option).
This description is needed to generate Verilog-HDL code.

# YOUR_TEST_CODES.scala
This file is needed to test your code. Set the test file name with ProjectNameMain.scala (replace "ProjectName" with your project's name.)
The file has to have bellow code at least;
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
where "ProjectName" in the code must be replaced with your project's name.

We do **NOT** recommend to modify template file for beginners.


# How to lint
You need to change current directory at root of directory where "build.sbt" file is on. To lint your code, one way is to use test, run following command;
```
sbt 'runMain ProjectNameMain'
```
NOTE: replace "ProjectName" with your project's name.


# How to Test with test-code
Simply run followinf command after making ```YOUR_TEST_CODES.scala```.
```
sbt 'test:runMain ProjectNameMain'
```
NOTE: replace "ProjectName" with your project's name. If you set package name, 
```
sbt 'test:runMain package_name.ProjectNameMain'
```
- You can display hexadecimal number instead of decimal number with following option;
```
--display-base 16
```


# How to generate HDL
To generate Verilog-HDL code, simply run this commands
```
sbt 'runMain ProjectNameMain'
```
NOTE: replace "ProjectName" with your project's name. If you set package name, 
```
sbt 'test:runMain package_name.ProjectNameMain'
```


# Small Tips
- Use of Utilities

If you want to use utility function prepared already such as ```Log2()```, then add following import description in your code;
```
import chisel3.util._
```
Utilities are listed in;

https://www.chisel-lang.org/api/latest/index.html#chisel3.util.package


- Reduction Operation across Vec

For bellow declaration
```
val hoge = Vec(Size, Bool())
```

1. Cast to a UInt and use orR:
```
hoge.asUInt.orR
```

2. Use Scala reduce method (defined on Vec)
```
hoge.reduce(_ || _)
```

- Data Structure with Initialization

You need define class for data set with like this;
```
class datum (DataWidth: Int) extends Bundle {
  val valid = Bool()
  val data  = UInt(DataWidth.W)
}
```
Then you can use the class as follows;
```
val Datum = RegInit(0.U.asTypeOf(new datum(DataWidth)))
```
Both of valid and data "reg"s in RTL is cleared by hardware "reset" signal.
The reset (and also clock) is added automatically to the RTL.

- Multi-Port with Vec()

Port class defines I/O and bundles several I/Os, then we can define Num ports like this;
```
val io = IO(new Bundle {val port = Vec(Num, new Port(Width))})
```

- Multi-Instance with List

When an adder is instance class defining adder logic circuit, then we can define "Num" adders like this;
```
val ADD = List.fill(Num)(Module(new Adder(Width)))
```

# Error Messages
- "OutOfMemory"

JVM needs more heap memory space.
To Specify giving space;
```
sbt -mem 4096
```
This gives 4GiB space.

- "[error] (run-main-0) java.lang.ClassNotFoundException: ProjectName"

Meaning:  There is no ProjectName top module in your source file(s).

Solution: Check top module name.

- "[error] (run-main-0) firrtl.passes.PassExceptions"
```
[info] [time] Done elaborating.
[error] (run-main-0) firrtl.passes.PassExceptions: 
[error] firrtl.passes.CheckInitialization$RefNotInitializedException:  @[YourProject LineNo:ClmNo] : [module TopModuleName]  Reference PortName is not fully initialized.
[error]    : PortName <= VOID
```
If your code does not have unknown state on the port "PortName" (or, wire), the exception might caused by FIRRTL's procedure.
You can temporally fix this issue by assigning "DontCare" which is reserved variable indicating "Do not Care".
You can use it before assigning value to the port, like as follows;
```
PortName := DontCare
```
