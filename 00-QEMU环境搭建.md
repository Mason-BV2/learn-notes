# ⚡ QEMU ARM64 实验环境搭建

> 第1周必须完成的**第一个项目**。
> 这是整个 28 周课程的实验基地，所有驱动都在 QEMU 上调试。

---

## 一、安装 QEMU

### Ubuntu/Debian
```bash
sudo apt update
sudo apt install qemu-system-arm qemu-efi-aarch64 \
                 qemu-system-gui \
                 aarch64-linux-gnu-gcc \
                 aarch64-linux-gnu-binutils \
                 gcc-arm-linux-gnueabihf \
                 libncurses-dev flex bison \
                 openssl libssl-dev dkms \
                 libelf-dev libudev-dev libpci-dev \
                 libiberty-dev autoconf \
                 fakeroot bc \
                 cpio \
                 build-essential
```

### Windows (WSL2)
```powershell
# PowerShell 管理员模式
wsl --install -d Ubuntu-24.04
# 重启后进入 WSL2，然后执行上面的 Ubuntu 命令
```

## 二、下载内核源码

```bash
# 完整内核（推荐，约 2GB）
git clone https://github.com/torvalds/linux.git
cd linux

# 或者浅克隆（只下载最新版本，约 500MB）
git clone --depth=1 https://github.com/torvalds/linux.git
cd linux
```

## 三、编译 ARM64 内核

```bash
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-

# 使用默认配置
make defconfig

# 启用内核模块和调试选项（驱动开发必需）
make menuconfig
# 确保打开以下选项：
#   CONFIG_MODULES=y          — 模块支持
#   CONFIG_DEVTMPFS=y         — /dev 自动管理
#   CONFIG_DEVTMPFS_MOUNT=y   — 自动挂载 devtmpfs
#   CONFIG_IKCONFIG=y         — 内核配置
#   CONFIG_IKCONFIG_PROC=y
#   CONFIG_DEBUG_FS=y         — debugfs
#   CONFIG_DYNAMIC_DEBUG=y    — 动态调试

# 快速保存配置
make savedefconfig
cp defconfig arch/arm64/configs/qemu_defconfig

# 编译内核
make -j$(nproc) Image.gz dtbs

# 编译内核模块
make modules
```

## 四、制作根文件系统

```bash
# 使用 BusyBox 制作最小根文件系统

# 1. 下载 busybox
wget https://busybox.net/downloads/busybox-1.36.1.tar.bz2
tar xf busybox-1.36.1.tar.bz2
cd busybox-1.36.1

# 2. 配置静态编译
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
sed -i 's/# CONFIG_STATIC is not set/CONFIG_STATIC=y/' .config
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- install

# 3. 制作 rootfs
mkdir -p _install/{dev,etc,proc,sys,lib,lib64,mnt,root,tmp,var}
# 创建设备节点
sudo mknod _install/dev/console c 5 1
sudo mknod _install/dev/null c 1 3
sudo mknod _install/dev/ttyAMA0 c 204 64

# 4. init 脚本
cat > _install/init << 'INIT'
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev
echo "=== QEMU ARM64 Ready ==="
echo "Welcome to Qualcomm Linux Kernel Learning!"
echo ""
exec /bin/sh
INIT
chmod +x _install/init

# 5. 打包 rootfs
find _install | cpio -H newc -o | gzip > ../rootfs.cpio.gz
cd ..
```

或者使用现成的 Buildroot（更简单）：

```bash
# 使用 Buildroot 生成完整的根文件系统
git clone https://github.com/buildroot/buildroot
cd buildroot
make qemu_aarch64_virt_defconfig
make -j$(nproc)
# 输出: output/images/rootfs.cpio.gz
```

## 五、启动验证

```bash
# 从内核源码目录执行
qemu-system-aarch64 \
  -machine virt \
  -cpu cortex-a72 \
  -smp 2 \
  -m 1G \
  -kernel arch/arm64/boot/Image \
  -initrd /path/to/rootfs.cpio.gz \
  -append "console=ttyAMA0 root=/dev/ram rdinit=/init" \
  -nographic

# 看到以下输出说明成功：
# [    0.000000] Booting Linux on physical CPU 0x0000000000 [0x410fd083]
# ...
# [    1.234567] Freeing unused kernel memory: 2048K
# [    1.234567] Run /init as init process
# === QEMU ARM64 Ready ===
# Welcome to Qualcomm Linux Kernel Learning!
# / #
```

## 六、加载内核模块测试

```bash
# 在 QEMU 中测试内核模块加载
# 编译一个 hello.ko
cat > hello.c << 'EOF'
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>

static int __init hello_init(void)
{
    pr_info("Hello from QEMU ARM64!\n");
    return 0;
}

static void __exit hello_exit(void)
{
    pr_info("Goodbye from QEMU ARM64!\n");
}

module_init(hello_init);
module_exit(hello_exit);
MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Hello from QEMU");
EOF

cat > Makefile << 'MAKEFILE'
obj-m += hello.o

KDIR ?= /path/to/linux

all:
	$(MAKE) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -C $(KDIR) M=$(PWD) modules

clean:
	$(MAKE) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -C $(KDIR) M=$(PWD) clean
MAKEFILE

make

# 将 hello.ko 放入 rootfs 或使用 QEMU 的 9p 共享
# 在 QEMU 启动参数添加：
# -fsdev local,security_model=passthrough,id=fsdev0,path=/path/to/modules \
# -device virtio-9p-device,fsdev=fsdev0,mount_tag=modshare

# 在 QEMU 中挂载并测试：
mount -t 9p modshare /mnt
insmod /mnt/hello.ko
dmesg | tail
rmmod hello
dmesg | tail
```

## 七、开发板投资建议

当 QEMU 实验做够了，是时候上真硬件：

| 板子 | 价格 | 适合阶段 |
|------|------|----------|
| Raspberry Pi 5 | ¥400-500 | 第5周后：练手模块和字符驱动 |
| DragonBoard 845c | ¥1000-1500 | 第17周后：高通平台实战 |
| Qualcomm RB5 | ¥3000+ | 进阶：完整高通平台开发 |

## 八、常见问题

**Q: QEMU 启动卡住了？**
```bash
# 去掉 -nographic，改用 -serial stdio
qemu-system-aarch64 ... -serial stdio
# 或者检查 -append 参数是否正确
```

**Q: insmod 报错 "Invalid module format"**
```bash
# 模块和内核对不上版本
# 确保用和 QEMU 内核相同的源码编译模块
# 检查 modinfo hello.ko 的 vermagic 是否匹配
```

**Q: 如何快速调试内核崩溃？**
```bash
# QEMU 启动参数添加：
# -append "... panic=1"  # 崩溃后等待 debug
# -s -S                  # 启动后暂停，等待 GDB 连接

# 在另一个终端：
aarch64-linux-gnu-gdb vmlinux
(gdb) target remote :1234
(gdb) continue
```

## ✅ QEMU 环境搭建检查清单

```
□ qemu-system-aarch64 安装成功
□ ARM64 交叉编译工具链安装成功
□ 内核编译成功（make Image）
□ 根文件系统制作成功
□ QEMU 启动到 shell 提示符
□ 能加载和卸载 .ko 模块
□ 9p 文件共享正常工作
```

**这是第1周周末验收标准。没跑通之前，不要进入第2周。**
