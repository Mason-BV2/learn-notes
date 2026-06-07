# Trace32 调试

## 概述

Trace32 是 Lauterbach 公司开发的 JTAG/SWD 硬件调试器，是嵌入式开发领域最强大的调试工具之一。在高通平台 bring-up 和深度调试中不可或缺。

> 高通工作场景：当 UART 还没通、系统在启动早期崩溃、需要分析硬件状态时，Trace32 是唯一的选择。JD 要求 "Familiarity with Trace32/Makefile scripting"。

## Trace32 基础

### 连接方式

```
PC ← USB/LAN → Lauterbach Debugger ← JTAG/SWD → Target Board (Qualcomm SoC)
```

### 启动脚本 (.cmm)

```c
; init.cmm — Trace32 初始化脚本
; 分号是注释

; 1. 选择调试接口
SYStem.CPU            ARMv8-A
SYStem.JTAGClock      5MHz        ; 调试时钟
SYStem.Option          POSIX       ; POSIX 线程支持

; 2. 配置调试目标
SYStem.Config.DAP     DAP_AHB_AP   ; 调试访问端口
SYStem.Config.CPUAuto  ON          ; 自动检测 CPU

; 3. 连接目标
SYStem.Up                         ; 连接到目标板

; 4. 初始化调试会话
Data.LOAD.ELF        vmlinux /NoCODE  ; 加载内核符号表
Data.LOAD.ELF        Image            ; 加载内核镜像
```

### 基本调试命令

```c
; 控制 CPU 执行
Go                    ; 运行
Break                 ; 暂停
Step                  ; 单步
StepOver              ; 步过
StepOut               ; 步出

; 设置断点
Break.Set     start_kernel           ; 函数名断点
Break.Set     0x80080000             ; 地址断点
Break.Set     main /Write            ; 数据写断点
Break.Set     my_var /Read           ; 数据读断点
Break.Delete  1                      ; 删除断点

; 查看数据
Data.dump   0x80100000               ; dump 内存（hex）
Data.List   0x80100000               ; 反汇编
Data.Print  jiffies                  ; 查看变量值
Data.Print  &my_struct->field       ; 结构体成员

; 寄存器
Register.view                        ; 查看所有寄存器
Register.Set   x0    0x80000000      ; 设置寄存器值

; 栈回溯
FRAME.view                           ; 显示调用栈
FRAME.returnstack                    ; 显示返回地址链

; 搜索内存
Data.SEARCH 0x80000000 0x90000000 "Linux version"  ; 搜索字符串
```

## 常见调试场景

### 场景1：系统崩溃分析

```c
; crash_analysis.cmm
; 当系统 Hang 或 Panic 后连接 Trace32

; 1. 暂停系统
Break

; 2. 查看当前异常级别
Register.view  CurrentEL

; 3. 查看异常原因
PRINT "ESR_EL1: "
Register.print  ESR_EL1

; 4. 查看 PC 位置
Register.print  PC
PRINT "PC register: "
Register.print  PC
Data.List   %PC     ; PC 附近的代码

; 5. 查看栈回溯
FRAME.view

; 6. 检查关键变量
Data.Print  panic_cpu
Data.Print  panic_smp_self_stop

; 7. 保存日志
PRINT "Crash analysis completed"
PRINT "Saving register dump..."
PRINT %r10  ; 打印 r10~r18
PRINT %r11
PRINT %r12
```

### 场景2：MMU 页表检查

```c
; mmu_check.cmm — 检查页表配置

; 查看 MMU 状态
Register.print  SCTLR_EL1        ; 系统控制寄存器（M位=MMU使能）
Register.print  TTBR0_EL1        ; 用户页表
Register.print  TTBR1_EL1        ; 内核页表
Register.print  TCR_EL1          ; 页表控制

; 查看虚拟地址翻译
MAP.VirtToPhys    0xffffffc010000000  ; 翻译虚拟地址到物理地址
MAP.PhysToVirt    0x80000000          ; 物理地址到虚拟地址

; dump 页表
MAP.PageTable     TTBR1_EL1          ; 查看内核页表
```

### 场景3：中断问题调试

```c
; irq_debug.cmm — 检查中断控制器

; 查看 GIC 状态
Data.dump    0x17a00000          ; GICD 基地址
Data.dump    0x17a60000          ; GICR 基地址

; 检查 GICD_CTLR（GIC 使能状态）
Data.Print   %long(0x17a00000)

; 查看中断使能寄存器
Data.dump    0x17a00100   /Spotlight  ; GICD_ISENABLER
Data.dump    0x17a00180   /Spotlight  ; GICD_ICENABLER

; 检查 CPU 接口
Data.Print   %long(0x17a00000 + 0x6000)  ; ICC_IAR
```

## Trace32 脚本编程

### 脚本基础语法

```c
; 变量定义
VAR &counter  %long  0
VAR &filename %string "dump.bin"

; 循环
WHILE &counter < 100
(
    Data.dump  (0x80000000 + &counter * 0x1000)
    &counter = &counter + 1
)

; 条件判断
IF &counter == 0
(
    PRINT "Counter is zero"
)
ELSE
(
    PRINT "Counter is &counter"
)

; 函数调用
GOSUB *main.read_all_registers
ENDDO

; 函数定义
main.read_all_registers:
    Register.view
    RETURN
```

### 完整调试脚本示例

```c
; qcom_boot_debug.cmm
; 高通平台启动调试脚本

; 配置调试器
SYStem.CPU            ARMv8-A
SYStem.JTAGClock      5MHz  
SYStem.Option          POSIX
SYStem.Config.DAP     DAP_AHB_AP

PRINT "================================================"
PRINT "Qualcomm Boot Debug Script v1.0"
PRINT "================================================"

; 连接到目标
SYStem.Up
IF SYStem.state() != "Up"
(
    PRINT "ERROR: Cannot connect to target!"
    ENDDO
)

; 加载符号
Data.LOAD.ELF  vmlinux /NoCODE
PRINT "Kernel symbols loaded."

; 设置关键断点
Break.Set  start_kernel
Break.Set  setup_arch
Break.Set  mm_init
Break.Set  console_init
Break.Set  kernel_init

PRINT "Breakpoints set."
PRINT "Starting target..."

; 启动运行
Go

; 当命中断点时
WHILE TRUE
(
    WAIT !Break.state()   ; 等待断点命中
    
    PRINT "================================================"
    PRINT "Hit breakpoint at: "
    sYmbol.SOURCE  %PC
    PRINT "================================================"
    
    ; 检查计时器
    Data.Print  jiffies
    
    ; 询问用户是否继续
    PRINT "Press ENTER to continue, or type 'q' to quit"
    ENTER &cmd
    
    IF &cmd == "q"
    (
        PRINT "Debug session ended."
        ENDDO
    )
    
    Go
)
```

## Trace32 快捷键和技巧

```text
F1        帮助文档
F2        打开命令行
F3        运行/继续
F7        单步
F8        步过
F9        运行到光标
Ctrl+F9   设置断点

常用命令缩写：
  G    → Go
  B    → Break
  S    → Step
  D    → Data.dump
  R    → Register.view
  Fr   → Frame.view
  V    → Var.view
```

## 实操练习

### 1. 使用 QEMU + GDB 模拟 Trace32 调试

```bash
# QEMU 的 GDB server 模式可以用来练习
qemu-system-aarch64 \
    -machine virt \
    -smp 1 \
    -m 1G \
    -kernel Image \
    -s -S &   # -s = gdbserver:1234, -S = 暂停启动

# 使用 GDB 连接
aarch64-linux-gnu-gdb vmlinux
(gdb) target remote :1234
(gdb) break start_kernel
(gdb) continue
(gdb) info registers
(gdb) x/10gx 0x80080000
(gdb) backtrace
```

### 2. Trace32 脚本练习

```c
; 练习：编写一个监控变量的脚本
; 监控 jiffies 的变化

VAR &prev %long 0

WHILE TRUE
(
    Data.Print  jiffies
    IF Data.Print(jiffies) == &prev
    (
        PRINT "WARNING: jiffies not incrementing!"
        PRINT "Timer might be stuck!"
    )
    &prev = Data.Print(jiffies)
    WAIT 1.s    ; 等待1秒
)
```

## 真实 GitHub 资源

| 资源 | 说明 |
|------|------|
| [Lauterbach 官方文档](https://www.lauterbach.com/documentation.html) | Trace32 官方文档 |
| [ARM-software/arm-trusted-firmware](https://github.com/ARM-software/arm-trusted-firmware) | ATF 含 Trace32 脚本示例 |
| [QEMU GDB 文档](https://www.qemu.org/docs/master/system/gdb.html) | QEMU GDB 调试 |
| [Trace32 实践指南](https://www.lauterbach.com/training.html) | Lauterbach 培训课程 |

## 面试考点

1. **Trace32 和 GDB 的区别？** Trace32 是硬件调试器（JTAG），可以调试 bootloader、CPU 启动早期；GDB 是软件调试器，需要操作系统已经运行
2. **Watchpoint 和 Breakpoint 的区别？** Watchpoint 监视数据访问（读/写），Breakpoint 监视指令执行
3. **如何使用 Trace32 调试 MMU 相关的问题？** 使用 MAP.VirtToPhys 检查虚拟地址翻译，检查 TTBR/TCR 寄存器
4. **为什么会需要用 Trace32 而不是 UART 日志？** 当 UART 驱动还没初始化时、系统在启动早期就 crash 时、需要看硬件寄存器状态时
5. **如何分析内核 Panic 的调用栈？** 连接 Trace32 → Frame.view 看栈回溯 → PC 和 LR 确定崩溃位置 → ESR 确定异常类型
