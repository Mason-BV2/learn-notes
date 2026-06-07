# Linux 内核模块编程

## 概述

内核模块（Loadable Kernel Module, LKM）是 Linux 内核动态扩展的机制。模块可以在不重新编译内核、不重启系统的情况下加载和卸载。这是内核驱动开发的基础。

> 高通工作场景：你需要为 Qualcomm 平台编写内核模块来支持新的硬件外设，或者调试/验证内核功能特性。

## 核心概念

### 1. 模块结构

每一个内核模块都有两个入口点：

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

// 模块加载时调用
static int __init my_module_init(void)
{
    printk(KERN_INFO "Hello, Qualcomm!\n");
    return 0; // 0 表示成功，负数表示失败
}

// 模块卸载时调用
static void __exit my_module_exit(void)
{
    printk(KERN_INFO "Goodbye, Qualcomm!\n");
}

module_init(my_module_init);
module_exit(my_module_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("A simple Qualcomm-style kernel module");
```

### 2. 编译模块 — Makefile

```makefile
obj-m += hello.o

KDIR ?= /lib/modules/$(shell uname -r)/build

all:
	$(MAKE) -C $(KDIR) M=$(PWD) modules

clean:
	$(MAKE) -C $(KDIR) M=$(PWD) clean
```

编译命令：
```bash
make
sudo insmod hello.ko    # 加载模块
sudo rmmod hello        # 卸载模块
dmesg | tail            # 查看 printk 输出
```

### 3. 模块参数

向模块传递参数，类似内核 cmdline：

```c
static int debug_level = 0;
static char *device_name = "default";

module_param(debug_level, int, 0644);
module_param(device_name, charp, 0644);
MODULE_PARM_DESC(debug_level, "Debug level (0-3)");
MODULE_PARM_DESC(device_name, "Device name string");

// 加载时：insmod mymod.ko debug_level=2 device_name="qcom_dev"
```

### 4. printk 日志级别

```c
printk(KERN_EMERG   "0 - 系统不可用\n");
printk(KERN_ALERT   "1 - 需要立即处理\n");
printk(KERN_CRIT    "2 - 严重状态\n");
printk(KERN_ERR     "3 - 错误\n");
printk(KERN_WARNING "4 - 警告\n");
printk(KERN_NOTICE  "5 - 正常但重要\n");
printk(KERN_INFO    "6 - 信息\n");
printk(KERN_DEBUG   "7 - 调试\n");
```

高通调试建议：使用 `pr_info()`、`pr_err()`、`pr_debug()` 等封装宏代替原始 printk。

### 5. 模块导出符号

```c
// 在模块中导出函数，供其他模块使用
int qcom_smmu_map_region(unsigned long phys, size_t size)
{
    // ...
    return 0;
}
EXPORT_SYMBOL_GPL(qcom_smmu_map_region);  // 其他模块可以用这个函数
```

## 实操练习

### 练习1：Hello World 模块

```c
// hello.c — 带模块参数和 /proc 接口
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/proc_fs.h>
#include <linux/uaccess.h>

#define PROC_NAME "qcom_hello"

static struct proc_dir_entry *proc_entry;
static char msg[256] = "Hello from Qualcomm!\n";

static ssize_t proc_read(struct file *file, char __user *buf,
                          size_t len, loff_t *off)
{
    return simple_read_from_buffer(buf, len, off, msg, strlen(msg));
}

static ssize_t proc_write(struct file *file, const char __user *buf,
                           size_t len, loff_t *off)
{
    if (len >= sizeof(msg))
        len = sizeof(msg) - 1;
    if (copy_from_user(msg, buf, len))
        return -EFAULT;
    msg[len] = '\0';
    return len;
}

static struct proc_ops proc_fops = {
    .proc_read = proc_read,
    .proc_write = proc_write,
};

static int __init hello_init(void)
{
    proc_entry = proc_create(PROC_NAME, 0666, NULL, &proc_fops);
    if (!proc_entry)
        return -ENOMEM;
    pr_info("qcom_hello: module loaded\n");
    return 0;
}

static void __exit hello_exit(void)
{
    remove_proc_entry(PROC_NAME, NULL);
    pr_info("qcom_hello: module unloaded\n");
}

module_init(hello_init);
module_exit(hello_exit);
MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Qualcomm Hello World with proc interface");
```

### 练习2：使用 QEMU 测试模块（不依赖真实硬件）

```bash
# 使用 cirosantilli 的 LKMC 项目
git clone https://github.com/cirosantilli/linux-kernel-module-cheat
cd linux-kernel-module-cheat

# 构建并运行 QEMU
./build -a arm64
./run -a arm64

# 在 QEMU 中测试模块
insmod /mnt/9p/your_module.ko
rmmod your_module
dmesg
```

## 真实 GitHub 资源

| 仓库 | 用途 | 推荐理由 |
|------|------|----------|
| [sysprog21/lkmpg](https://github.com/sysprog21/lkmpg) | 权威模块编程指南 | 适配 6.x 内核，在线可读 |
| [cirosantilli/linux-kernel-module-cheat](https://github.com/cirosantilli/linux-kernel-module-cheat) | QEMU 实验环境 | 最完整的模块测试环境，含 100+ 示例 |
| [martinezjavier/ldd3-examples](https://github.com/martinezjavier/ldd3-examples) | LDD3 示例代码 | 经典教材配套，适合入门 |
| [satoru-takeuchi/linux-kernel-mod-sample](https://github.com/satoru-takeuchi/linux-kernel-mod-sample) | 现代内核模块示例 | 简洁清晰，适配新内核版本 |
| [torvalds/linux/samples/](https://github.com/torvalds/linux/tree/master/samples) | 内核官方示例 | 最权威的参考，`samples/kobject/` 等 |

## 面试考点

1. **module_init 的执行顺序** — 依赖优先级、initcall 级别
2. **__init 和 __exit 宏的作用** — 释放初始化内存
3. **EXPORT_SYMBOL_GPL 和 EXPORT_SYMBOL 的区别** — GPL 兼容性要求
4. **内核模块与用户态程序的区别** — 没有标准库、不能使用浮点运算、直接访问硬件
5. **为什么模块不能使用 libc** — 内核是独立的环境，有自己的一套 API

## 高通面试可能问到

- 如何调试模块加载失败？（查看 dmesg、检查符号依赖、确认内核版本匹配）
- ```
  Unknown symbol xxx (err -2)
  ```
  的含义是什么？（模块依赖的符号未导出或未加载）
- 如何确认模块是否与当前内核版本兼容？（`modinfo` 检查 vermagic）
