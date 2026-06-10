# 第1周学习笔记 — Linux 开发环境 + /proc 文件系统

## 学习日期
2026-06-10

## 本机环境
- 内核版本：Linux 6.8.0-90-generic，Ubuntu 24.04
- CPU：AMD EPYC 9655P，96核/192线程，4.5 GHz
- 内存：1.5 TB 物理内存

---

## 核心概念：/proc 是什么

`/proc` 不是真实的磁盘文件系统，是内核在内存里动态生成的**虚拟文件系统**。
内核通过它把自己的运行状态暴露给用户态程序。

**规律**：想知道内核/系统状态，先去 `/proc` 找。

---

## 今天学会的命令

| 命令 | 作用 |
|------|------|
| `pwd` | 显示当前所在目录 |
| `ls` | 列出目录内容 |
| `ls -l` | 详细列表（权限/大小/时间） |
| `cd 目录` | 进入目录 |
| `cd ..` | 返回上一级 |
| `cd ~` | 回到家目录 |
| `cat 文件` | 显示文件内容 |
| `cat 文件 \| head -N` | 只看前N行 |

---

## /proc 下的关键文件

| 文件 | 内容 | 常用场景 |
|------|------|----------|
| `/proc/version` | 内核版本、编译器版本 | 确认内核版本 |
| `/proc/cpuinfo` | 每个CPU核的详细信息 | 查CPU型号/特性/频率 |
| `/proc/meminfo` | 内存使用详情 | 查内存/Slab/虚拟内存 |
| `/proc/<pid>/status` | 某个进程的详细状态 | 查进程内存/调度/权限 |

---

## /proc/meminfo 关键字段

| 字段 | 含义 |
|------|------|
| `MemTotal` | 物理内存总量 |
| `MemFree` | 完全空闲（通常很小，正常） |
| `MemAvailable` | 实际可用内存（这个才是真实可用量） |
| `Slab` | 内核自己用的内存（驱动数据结构从这里分配） |
| `VmallocTotal` | 内核虚拟地址空间总量（不是物理内存） |

---

## /proc/1/status 关键字段

| 字段 | 含义 |
|------|------|
| `Name` | 进程名 |
| `Pid` | 进程ID |
| `PPid` | 父进程ID（PID 1 的 PPid=0，它是祖先） |
| `State` | 进程状态（S=sleeping，R=running，D=不可中断睡眠） |
| `VmRSS` | 实际占用物理内存 |
| `VmSize` | 虚拟地址空间大小 |
| `Cpus_allowed_list` | 允许运行的CPU核范围 |
| `voluntary_ctxt_switches` | 主动让出CPU次数（多=老实进程） |

---

## 重要直觉

1. **`MemFree` 小不代表内存不够** — Linux 会把空闲内存用作缓存，看 `MemAvailable`
2. **`/proc` 下的数字目录 = 活着的进程** — `ls /proc | grep -E "^[0-9]+"` 能看到所有进程
3. **后面写驱动也可以在 `/proc` 下创建文件** — 让用户态读取驱动内部状态（第6周）

---

## 待学内容（下次继续）
- [ ] 文件权限（`chmod`/`chown`）
- [ ] 进程管理命令（`ps`/`top`/`kill`）
- [ ] 工具链安装（`gcc`/`make`/`gdb`）
- [ ] 第1周验收：在 QEMU 启动 ARM64 内核


---

## Day 2 — 文件权限 + 进程管理

### 文件权限

权限格式：`[类型][所有者][同组][其他人]`

| 文件类型 | 符号 |
|----------|------|
| 普通文件 | `-` |
| 目录 | `d` |
| 字符设备 | `c` |
| 块设备 | `b` |

权限计算：r=4，w=2，x=1，相加得数字
- `rw-r--r--` = 644
- `rwxr-xr-x` = 755
- `rw-------` = 600
- `rw-rw-rw-` = 666

常用命令：
```bash
chmod 644 文件    # 修改权限
ls -l 文件        # 查看权限
```

**驱动开发关键点**：
- `/dev/` 下的设备文件是 `c`（字符设备）或 `b`（块设备）开头
- 设备文件有主设备号和次设备号（如 `1, 3`），内核用它找对应驱动
- `/proc/` 下文件大小永远是 0，内容是内核动态生成的

---

### 进程管理命令

| 命令 | 作用 |
|------|------|
| `ps aux` | 列出所有进程 |
| `top` | 实时查看进程状态 |
| `cat /proc/$$/status` | 查看当前 shell 的进程信息 |
| `$$` | 特殊变量，代表当前 shell 的 PID |

**ps aux 列含义**：

| 列 | 含义 |
|----|------|
| `PID` | 进程ID |
| `%CPU` | CPU占用率 |
| `%MEM` | 物理内存占用率 |
| `VSZ` | 虚拟内存大小（大不代表真占那么多） |
| `RSS` | 实际占用物理内存 |
| `STAT` | 进程状态 |

**STAT 状态含义**：

| 状态 | 含义 |
|------|------|
| `S` | sleeping，等待事件 |
| `R` | running，正在运行 |
| `I` | idle，内核线程空闲 |
| `Z` | zombie，僵尸进程（父进程未回收） |

**重要直觉**：
- 方括号进程（如 `[kthreadd]`）= 内核线程，VSZ/RSS 为 0，跑在内核空间
- `VIRT` 大不代表真的占那么多，看 `RES` 才是实际物理内存
- 僵尸进程数量应接近 0，多了说明程序退出处理有问题


---

## Day 3 — 工具链 + 第一个 C 程序

### 环境确认
- gcc 13.3.0 ✅
- make ✅
- gdb ✅
- objdump ✅

### gcc 编译四步流程

```
源代码(.c)
   ↓ 预处理（展开 #include / #define）
   ↓ 编译（C代码 → 汇编）
   ↓ 汇编（汇编 → 机器码 .o）
   ↓ 链接（.o + 库 → 可执行文件）
可执行文件
```

常用命令：
```bash
gcc source.c -o output      # 一步完成编译+链接
gcc -c source.c -o output.o # 只编译不链接，生成 .o
ldd output                  # 查看可执行文件依赖的动态库
```

### 用户态 vs 内核态的关键区别

| | 用户态程序 | 内核模块 |
|--|-----------|---------|
| 标准库 | 可以用 libc（printf/fopen） | 不能用，要用内核函数（printk/filp_open） |
| 崩溃后果 | 进程挂掉 | 整机 kernel panic |
| 内存泄漏 | 进程退出后OS回收 | 一直泄漏直到重启 |

### 两类最常见 bug（内核里更致命）

**1. 空指针解引用**
```c
// 错误写法
fp = fopen(...);
fgets(line, sizeof(line), fp);  // fopen失败返回NULL，这里直接崩溃

// 正确写法
fp = fopen(...);
if (fp == NULL)
    return -1;
fgets(line, sizeof(line), fp);
```

**2. 资源泄漏**
```c
// 错误写法：打开了不关
fp = fopen(...);
// ... 用完没有 fclose

// 正确写法：用完必须释放
fp = fopen(...);
// ... 使用
fclose(fp);  // 必须关闭
```

**内核开发铁律**：每个函数调用后必须检查返回值，每个资源申请后必须有对应释放。


---

## Day 4 — gcc 编译 + Makefile

### gcc 常用参数

| 命令 | 作用 |
|------|------|
| `gcc source.c -o output` | 编译+链接，生成可执行文件 |
| `gcc -c source.c -o output.o` | 只编译不链接，生成目标文件 |
| `gcc -Wall source.c -o output` | 开启所有警告 |

**`-c` 很重要**：没有 `-c` 生成的是可执行文件，有 `-c` 生成的才是 `.o` 目标文件。

### Makefile 基本结构

```makefile
目标: 依赖
	命令（必须用Tab缩进，不能用空格）
```

### Makefile 工作原理

- make 比较目标和依赖的**修改时间**
- 依赖比目标新 → 重新执行命令
- 依赖没变 → 跳过，输出 `up to date`
- 只重新编译有变化的文件，50个文件改1个只编译1个

### 最小 Makefile 模板

```makefile
program: program.o
	gcc program.o -o program

program.o: program.c
	gcc -c program.c -o program.o

clean:
	rm -f program program.o
```

### 常见错误

| 错误 | 原因 |
|------|------|
| `cannot use executable file as input to a link` | 编译 .o 时忘了加 `-c` |
| `missing separator` | Makefile 命令行用了空格而不是 Tab |

