
## JVM (跨语言的平台)

### 特点 

- 一次编译，到处运行
- 自动内存管理
- 自动垃圾回收功能

### 基本概念

虚拟机的定义有2个，一种是类似Vmware的系统虚拟机，另一种是虚拟机称之为程序虚拟机，诸如JVM，CLR就是最常见到的虚拟机。
CLR：.Net的核心 公共语言运行库 
JVM(Java Virtual Machine)是可运行java代码(不仅仅java,Groovy,Scala,JavaScript等)的假想计算机，包括一套字节码指令集，一组寄存器、一个栈、一个垃圾回收、堆和一个存储方法域 。 JVM运行在操作系统上，与硬件没有直接的交互。
Java HotSpot Virtual Machine 成为java的默认虚拟机
![](/img/java/JVM位置.png)



### java代码执行流程

java源文件 通过编译器 可以生成字节码文件(.class文件) 然后通过不同平台的jvm(java虚拟机)中的解释器，编译成电脑能识别的机器码，从而实现一次编译 多次运行
(任何一个编程语言遵循Java虚拟机规范标准，通过自己的编译器生成的字节码文件遵循Java虚拟机规范标准都可以被java虚拟机解释运行)
也就是:
- java源文件  -> 编译器  ->字节码文件 
- 字节码文件 -> JVM -> 机器码
![](/img/java/classjvm.png)





JVM由三个主要的子系统构成 :
- 类加载机制  Class Loader
- 运行时数据区(内存机构) Runtime Data Area
- 执行引擎 (Execution Engline) 高级语言翻译成机器语言的角色



### 类加载机制

.class文件 研究下机制
加载一个.class文件 需要三个步骤:
- 找到这个类
- 类加载器进行加载




### 双亲委派机制 

向上委托 向加载 
如何打破双亲委派？自己写一个类加载器。例子：tomcat (war包隔离)。热部署。
jdbc的spi


###  JVM运行时数据区分析



### GC日志实战分析