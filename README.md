# How2Start_Chisel3
How to start hardware description language for biginners.

# Purpose of This Document
- Biginners who have not experience about chisel and scala.

# Prerequisists
Prerequisists are
- Java (for running Scala)
- SBT (Simple Built Tool to parse Chisel3 grammer on the Scala)

See for sbt installation;
'Installation Preparation'

https://github.com/freechipsproject/chisel3/wiki/Installation-Preparation

# Project Directory Structure
Required project directory structure is as follows;
```
root--+--build.sbt
      |
      +--project--+--build.properties
      |           |
      |           +--target
      |
      +--src--+--main--+--scala--"YOUR_SOURCE_CODES.scala"
      |       |
      |       +--test--+--scala--"YOUR_TEST_CODES.scala"
      |
      +--target
```
You can download a project template from;

https://github.com/freechipsproject/chisel-template

# Set-up to Compile
You must decide following point at first;
1. Project Name (defined in "name"-field in "build.sbt" file),
2. Top module similar to HDL needs (see bellow)
3. "YOUR_TEST_CODES.scala"

# Top module
Replace "ProjectName" with your project's name.
```
object ProjectName extends App {
  chisel3.Driver.execute(args,()=>new ProjectName(args))
}
```
where "args" are arguments you defined in your top module (option).

# YOUR_TEST_CODES.scala
Set the test file name with ProjectNameMain.scala (replace "ProjectName" with your project's name.)
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

# How to compile
At root of directory where "build.sbt" file is on, run following command;
```
sbt 'test:runMain ProjectNameMain'
```
NOTE: replace "ProjectName" with your project's name.

# Small Tips
If you want to use utility function prepared already such as ```Log2()```, then add following import description in your code;
```
import chisel3.util._
```
Utilities are listed in;

https://www.chisel-lang.org/api/latest/index.html#chisel3.util.package
