# 计算机系统概述

## 冯*诺伊曼结构计算机

### 存储程序计算机

- 程序由指令构成
- 程序功能通过指令序列描述
- 指令序列在存储器中顺序存放

### 顺序执行指令

- 用指针指示当前被执行的指令
- 从存储器中读出指令执行
- 指针指向下一条指令

## 指令和指令系统

- 计算机系统由硬件和软件两大部分组成。程序由一个序列的计算机指令组成。
- 指令是计算机运行的最小的功能单元，是指挥计算机硬件运行的命令，是由多个二进制位组成的位串，是计算机硬件可以直接识别和执行的信息体。指令中应致命锁完成的操作，并明确操作对象。
- 一台计算机提供的全部指令构成该计算机的指令系统。

## 指令功能分类

- 数据运算指令
  - 算数运算、逻辑运算
- 数据传输指令
  - 寄存器之间、主存/寄存器之间
- 输入/输出指令
  - 与输入/输出端口的数据传输
- 控制指令
  - 转移指令、子程序调用/返回
- 其它指令
  - 停机、开/关终端、空操作、特权、置条件码

## 指令格式

- 指令格式：指令字中操作码和操作数地址的而至今位的分配方案。
- 操作码：指明本条指令的操作功能，每条指令有一个明确的操作码。现在多为定长操作码。
- 操作数地址：说明操作数存放的地址，有时是操作数本身。
- 指令字：完整的一条指令的二进制表示。
- 指令字长：指令字中二进制代码的位数。现在多为定长指令字结构。
- 机器字长：计算机能直接处理的二进制数据的位数。指令字长（字节倍数）多等于机器字长的整数倍。

## 寻址方式

- 寻址方式（又称编址方式）：这是确定本条指令的操作数地址及下一条要执行的指令地址的方法。
- 通常需要在指令中为每一个操作数专设一个地址字段，用来表示数据的来源或去向的地址。在指令中给出的操作数（或指令）的地址被称为形式地址，使用形式地址信息并按照一定规则计算出来或读操作得到的一个数值才是数据（或指令）的实际地址。

## 评价计算机性能的指标

- 吞吐率：单位时间内完成的任务数量
- 响应时间：完成任务的时间
- 衡量性能的指标：MIPS、CPI、CPU time、CPU Clock
- 综合测试程序

## MIPS指令系统

- 无内部互锁流水级的微处理器
- RISC芯片
- 由John L. Hennessy设计

## MIPS指令格式

- 所有的指令都是32位字长。有3种指令格式，及寄存器型、立即数型和转移型。
- 操作数寻址方式有基质加16位位移量的访存寻址、立即数寻址及寄存器寻址3种。
- 这是一个链接 [MIPS指令集](https://www.jianshu.com/p/ac2c9e7b1d8f)。

### 寄存器型（R型）

| 标记 | op | rs | rt | rd | shamt | funct |
| :---- | :---- |:---- | :---- |:---- | :---- |:---- |
| 位数 | 31-26 | 25-21 | 20-16 | 15-11 | 10-6 | 5-0 |
| 功能 | 操作符 | 源操作数寄存器1 | 源操作数寄存器2 | 目的操作数寄存器 | 位移量 | 操作符附加段 |

### 立即数型（I型）

| 标记 | op | rs | rd | im |
| :---- | :---- |:---- | :---- |:---- |
| 位数 | 31-26 | 25-21 | 20-16 | 15-0 |
| 功能 | 操作符 | 源操作数寄存器 | 目的操作数寄存器 | 立即数 |

### 转移型（J型）

| 标记 | op | address |
| :---- | :---- |:---- |
| 位数 | 31-26 | 25-0 |
| 功能 | 操作符 | 地址 |