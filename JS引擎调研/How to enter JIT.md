# 如何使JS程序进入即时编译器

> JS程序最初是通过解释器进行解释执行的，当编译器发现某个方法或代码块运行特别频繁的时候，就会认为这是“热点代码”。<br/>为了提升热点代码的执行效率，就会将这些热点代码编译成与本机机器相关的机器码，进行各个层次的优化。完成这个任务的编译器就是即时编译器（JIT）。

## 热点代码

1. 多次被调用的方法
2. 多次被执行的循环体

### 判定方式

1. 基于采样的方式。周期性检测各个线程的栈顶，发现某个方法经常出现在栈顶，就认为是热点方法。
2. 基于计数器的方法。设定计数器，当某个方法超过阈值就认为是热点方法。

## Webkit JIT

### 构成

- LLInt -> baseline JIT -> DFG JIT -> FTL JIT

### JIT 操作

- 无用代码消除（Dead Code Elimination）
- 循环展开（Loop Unrolling）
- 循环表达式外提（Loop Expression Hoisting）
- 消除公共子表达式（Common Subexpression Elimination）
- 常量传播（Constant Propagation）
- 基本块重排序（Basic Block Reordering）
- ...

这些判定可能会存在误判，对于这些会触发JIT 操作的特殊Payload进行Fuzz可能更容易触发漏洞