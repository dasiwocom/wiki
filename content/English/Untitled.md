# 51单片机学习英语单词笔记

---

## 一、硬件与系统

|   |   |   |   |
|---|---|---|---|
|英文|缩写|中文|备注|
|Microcontroller|MCU|微控制器 / 单片机|51 Microcontroller = 51单片机|
|Microprocessor|MP|微处理器|电脑CPU那种，别和MCU搞混|
|Microcontroller Unit|MCU|微控制器单元||
|8051|—|8051内核|原始内核名称|
|STC89C52|—|常见51型号||
|Startup file|—|启动文件|51里是STARTUP.A51|
|Emulator|—|仿真器|硬件调试工具|
|Programmer|—|下载器 / 烧录器|把hex写进芯片|
|RAM|—|随机存取存储器|单片机内部内存|
|ROM|—|只读存储器|存放程序|
|Stack Pointer|SP|堆栈指针|STARTUP初始化|

---

## 二、C语言编程

|   |   |   |   |
|---|---|---|---|
|英文|缩写|中文|备注|
|Modular Programming|—|模块化编程||
|Header File|.h|头文件|只放函数声明|
|Source File|.c|源文件|放函数实现|
|Function Declaration|—|函数声明|只有原型+分号 ;|
|Function Implementation|—|函数实现|带大括号 {}|
|Header Guard|—|头文件保护|#ifndef / #define / #endif|
|Include|—|包含|#include 预编译指令|
|Compile|—|编译|把.c变成机器码|
|Link|—|链接|把多个.obj合成hex|
|Build|—|构建/编译|Keil里点Build按钮|
|Build Target|—|构建目标|"Target 1"|
|Error|—|错误|导致编译失败|
|Warning|—|警告|不阻塞但需处理|
|Hex File|.hex|烧录文件|最终下载到单片机|
|Intrinsic Function|—|内置函数|如 _nop_()|
|Delay|—|延时||
|Nixie Tube|—|数码管||
|Indentation|—|缩进|不影响编译，只管可读性|
|Tab|—|制表符|1字节，宽度可改|
|Space|—|空格|1字节/个|

---

## 三、预编译指令（#开头）

|   |   |   |
|---|---|---|
|英文|中文|说明|
|#include|包含头文件|把另一个文件内容粘贴进来|
|#ifndef|if not defined|如果没有定义|
|#define|定义|定义宏/记号|
|#endif|end if|结束条件编译|
|<xxx.h>|尖括号|去系统库目录找文件|
|"xxx.h"|双引号|优先当前工程目录找文件|

> 口诀：**自带尖括号，自写双引号**

---

## 四、调试与编译输出

|   |   |   |
|---|---|---|
|英文|中文|备注|
|Debug|v. 调试||
|Debugging|n. 调试||
|Debugger|—|调试器（软件）|
|Debug Tool|—|调试工具（统称）|
|Compiling|—|正在编译|
|Linking|—|正在链接|
|Creating hex file|—|正在生成hex文件|
|Target not created|—|目标文件未生成（编译失败）|
|Multiple Public Definitions|—|重复定义（L104错误）|
|Missing function-prototype|—|缺少函数声明|
|Unknown directive|—|未知指令（拼写错误）|
|Misplaced endif|—|endif位置错误|
|Uncalled segment|—|未调用代码段|

---

## 五、其他日常词汇

|   |   |   |   |
|---|---|---|---|
|英文|缩写|中文|备注|
|Wubi|—|五笔输入法|全称 Wubizixing Input Method|
|Input Method|—|输入法||
|Project|—|工程 / 项目|Keil里的Project|
|Module|—|模块||
|Declaration|—|声明||
|Definition|—|定义||
|Variable|—|变量||
|Function|—|函数||
|Loop|—|循环|while / do-while|
|Condition|—|条件判断|if-else|
|Constant|—|常量||
|Register|—|寄存器|51里P0、P1等|
|Port|—|端口|如P0口|
|Pin|—|引脚|芯片上的针脚|
|Schematic|—|原理图|电路图|
|Datasheet|—|数据手册|芯片规格书|

---

## 六、常见缩写速查

|   |   |   |
|---|---|---|
|缩写|全称|中文|
|MCU|Microcontroller Unit|微控制器|
|RAM|Random Access Memory|随机存取存储器|
|ROM|Read-Only Memory|只读存储器|
|SP|Stack Pointer|堆栈指针|
|CPU|Central Processing Unit|中央处理器|
|I/O|Input / Output|输入/输出|
|LED|Light Emitting Diode|发光二极管|
|HEX|Hexadecimal|十六进制|
|.c|C source file|C源文件|
|.h|Header file|头文件|
|.a51|Assembly file|汇编文件|
|.obj|Object file|目标文件|