```
title: 初识JDK、JRE、JVM
tags: JDK,JRE,JVM
grammar_zjwJava: true
```

- JVM、JRE、JDK概括
  - JDK：Java 虚拟机（Java Virtual Machine），它认识xxx.class类型的文件，并且能够将 class 文件中的字节码指令进行识别并调用操作系统向上的 API 完成动作，是java可以跨平台的核心。
  - JRE：Java 运行时环境（Java Runtime Environment），它包含JVM标准实现及Java核心类库。
  - JDK：Java 开发工具包，它是整个java开发的核心，它包含了JAVA的运行环境（JVM+Java系统类库）和JAVA工具。
- JDK与JRE区别
  - JRE是java运行的环境，而JDK是java的开发工具包，它包含所有JRE拥有的，而且可以进行创建新程序。如果只关心在计算机上运行java程序，那么只安装JRE就够了，但是如果有编程的需要，那么就需要安装JDK，但是在有一些特殊情况下，只运行java程序可能会也要JDK支持，如果文件都是编译好了的，那么可以不用安装JDK。
- Java 为什么能跨平台，实现一次编写，多处运行？
  - Java跨平台的核心是JVM，不同的操作系统向上的API是不同的，如果我们想要调用不同操作系统的同一个功能，那么可能需要针对不同操作系统写不同的代码来实现，JVM能识别字节码文件，然后解释到操作系统的API调用，不同的操作系统有不同的JVM，java代码的字节码都是一样的，通过不同操作系统的JVM解析后，向上调用API的动作是一样的，所以相同的java代码在不同平台上可以完成相同的动作。