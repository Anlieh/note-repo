本章从分析CPU的功能和内部结构入手，详细套路机器完成一条指令的全过程，为了提高数据的处理能力、开发系统的并行性所采取的流水技术，还概括了中断技术在提高整机系统效能方面的作用。

![](https://ypic.oss-cn-hangzhou.aliyuncs.com/202211041743922.png)

# 8.1 CPU的结构
## 1.CPU的功能
指令控制、操作控制、时间控制、数据加工、处理中断

1.控制器的功能
- 取指令
- 分析指令
- 执行指令
- 控制程序输入及结果的输出
- 总线管理
- 处理异常情况和特殊请求
- 总线管理
- 处理异常情况和特殊请求

2.运算器的功能
实现算术运算和逻辑运算

## 2.CPU结构框图 
1. CPU与系统总线
![](https://ypic.oss-cn-hangzhou.aliyuncs.com/202211041525616.png)

2. CPU的内部结构
![CPU的内部结构](https://ypic.oss-cn-hangzhou.aliyuncs.com/202211041523855.png)

## 3.CPU中的寄存器
1. 用户可见寄存器
| 类型         | 存放数据类型                         | 用途                                 |
| ------------ | ------------------------------------ | ------------------------------------ |
| 通用寄存器   | 存放操作数                           | 可作某种寻址方式所需的专用寄存器     |
| 数据寄存器   | 存放操作数（满足各种数据类型）       | 两个寄存器拼接存放双倍字长数据       |
| 地址寄存器   | 存放地址（其位数应满足最大地址范围） | 用于特殊的寻址方式，如段基址、栈指针 |
| 条件码寄存器 | 存放条件码                           | 可作程序分支的依据，如正、零、溢出等 |

2. 控制和状态寄存器
| 控制寄存器 | 作用                                                                   |
| ---- | ---------------------------------------------------------------------- |
| PC   | 程序计数器，存放现行指令的地址                                         |
| MAR  | 存储器地址寄存器，用于存放将被访问的存储单元的地址                     |
| MDR  | 存储器数据寄存器，用于存放欲存入存储器的数据或最近从存储器中读出的数据 |
| IR   | 指令寄存器，存放当前欲执行的指令                                                                       |
其中，MAR、MDR、IR用户不可见，PC用户可见
| 状态寄存器 | 作用       |
| ---------- | ---------- |
| 状态寄存器 | 存放条件码 |
| PSW寄存器  | 存放程序状态字           |

3. 控制单元CU和中断系统
CU 产生全部指令的微操作命令序列，俩种方式：
- 组合逻辑设计，硬连线逻辑
- 微程序设计，存储逻辑

中断系统主要用于处理计算机的各种中断

# 8.2 指令周期
![](https://ypic.oss-cn-hangzhou.aliyuncs.com/202211041838071.png)

## 1.指令周期的基本概念
- 指令周期的基本概念：从CPU中取出并执行一条指令所需的全部时间
- 指令周期常用机器周期（即CPU周期）表示
- 一个时钟周期包含若干时钟周期（CPU操作的基本单位）


**2.不同指令的指令周期**
每个指令周期内机器周期数可以不等，每个机器周期内的节拍数也可以不等
完成一条指令：取值周期（取值、分析），执行周期（执行）

![](https://ypic.oss-cn-hangzhou.aliyuncs.com/202211041752862.png)

CPU的工作周期：
- 取值周期：取指令
- 间址周期：取有效地址
- 执行周期：去操作数（当指令为访存指令时）
- 中断周期：保存程序断点

![](https://ypic.oss-cn-hangzhou.aliyuncs.com/202211041757025.png)

## 2.指令周期的数据流
假设CPU中有MAR、MDR、PC和IR

### 1.取值周期的数据流
1. 当前指令地址送至存储器地址寄存器，记做：（PC)→MAR
2. CU发出控制信号，经控制总线传到主存，这里是读信号，记做：1→R
3. 将MAR所指主存中的内容经数据总线送入MDR，记做：M(MAR）→MDR
4. 将MDR中的内容(此时是指令)送入IR，记做：（MDR)→IR
5. CU发出控制信号，形成下一条指令地址，记做：（PC)+1→PC

![](https://ypic.oss-cn-hangzhou.aliyuncs.com/202211041823035.png)

### 2.间址周期的数据流
1. 将指令的地九码送入MAR，记做：Ad(IR）→ MAR或Ad(MDR)→ MAR
2. CU发出控制信号，启动主存做读操作，记做：1→R
3. 将MAR所指主存中的内容经数据总线送入MDR，记做：M(MAR）→MDR
(不一定)4. 将有效地圳送至指令的地址码字段，记做:MDR→Ad(IR)

![](https://ypic.oss-cn-hangzhou.aliyuncs.com/202211041822170.png)

### 3.执行周期的数据流
执行周期的任务是根据IR中的指令字的操作码和操作数通过ALU操作产生执行结果。
不同指令的执行周期操作不同，因此没有统一的数据流向。

### 4.中断周期的数据流
**中断**：暂停当前任务去完成其他任务，为了能够恢复当前任务，需要保存断点。
一般使用堆栈来保存断点，这里用SP表示栈顶地址，假设SP指向栈顶元素，进栈操作是先修改指针，后存入数据

1. CU控制将SP减1，修改后的地址送入MAR，记做：（SP)-1 → SP，（SP)→ MAR
     本质上是将断点存入某个存储单元，设其地址为a，故可记做：a→MAR
2. CU发出控制信号，启动主存做写操作，记做：1→ W
3. 将断点(PC内容)送入MDR，记做：（PC)→ MDR
4. CU控制将中断服务程序的入口地址（由向量地址形成部件产生）送入PC，记做：向量地址→PC

![](https://ypic.oss-cn-hangzhou.aliyuncs.com/202211041831333.png)

## 3.指令执行方案
一个指令周期通常包括几个时间段（执行步骤），每个步骤完成指令的一部分功能，几个依次执行的步骤完成这条指令的全部功能。

![](https://ypic.oss-cn-hangzhou.aliyuncs.com/202211041837358.png)

**方案1 单指令周期**
对所有指令都选用相同的执行时间来完成。
指令之间串行执行：指令周期取决于执行时间最长的指令的执行时间。
对本来可以在更短时间内完成的指令，要使用这个较长的周期来完成，会降低整个系统的运行速度。

**方案2 多指令周期**
对不同类型的指令选用不同的执行步骤来完成，指令之间串行执行
可选用不同个数的时钟周期来完成不同指令的执行过程

**方案3 流水线方案**
在每一个时钟周期启动一条指令，尽量让多条指令同时运行，但各自处在不同的执行步骤中。
指令之间并行执行

# 8.3 指令流水



# 8.4 中断系统
