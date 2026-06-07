# Upstream 提交

## 概述

将代码提交到 Linux 内核主线（upstream）是内核开发者的重要技能。高通工程师经常需要将驱动代码 upstream，以减轻维护负担并符合 GKI 要求。

> 高通工作场景：JD 要求 "commit upstream"。高通许多驱动都 upstream 了（如 qcom_geni_serial, sdhci-msm），维护 upstream 代码是高通软件工程师的日常工作之一。

## 补丁格式

### 提交信息规范

```
<module>: <简短描述（不超过 70 字符）>

<详细描述，解释为什么这个补丁是必要的>
<描述修改的内容和原因>
<每行不要超过 72 字符>

Signed-off-by: Your Name <your.email@example.com>
```

```text
// 好的提交信息示例：
//
// i2c: qcom-geni: add support for I2C Master Hub serial engine
//
// I2C Master Hub is a new serial engine IP which is derived from
// existing GENI IP. It supports DMA mode transfer only and has a
// different interrupt path compared to the legacy GENI.
//
// Add support for this new variant by extending the existing
// qcom-geni I2C driver with new compatible and operations.
//
// Signed-off-by: John Doe <john@qualcomm.com>
```

### 补丁生成

```bash
# 生成针对主线的补丁
git format-patch --subject-prefix="PATCH" master
# 输出: 0001-i2c-qcom-geni-add-support-for-I2C-Master-Hub.patch

# 如果是一个系列（patch series）
git format-patch --cover-letter -v2 -n master
# 输出：
#   v2-0000-cover-letter.patch
#   v2-0001-i2c-qcom-geni-add-support-for-I2C-Master-Hub.patch
#   v2-0002-...

# 补丁 Header 示例
# From: John Doe <john@qualcomm.com>
# Date: Mon, 15 Jan 2024 14:30:00 +0800
# Subject: [PATCH v2 1/2] i2c: qcom-geni: add support for ...
```

## 补丁提交流程

```
1. 代码开发
   └── 在 linux-next 分支上开发（不要基于久的分支）
   
2. 自检
   └── make C=1 (sparse 静态检查)
   └── make W=1 (额外警告)
   └── scripts/checkpatch.pl (补丁格式检查)
   └── 编译通过
   └── 功能测试通过

3. 选择目标内核版本
   └── 修复 → merge window 期间通过 fixes 标签
   └── 新功能 → merge window 开放时提交
   └── 驱动新硬件 → 在 -next 树中先合入

4. 发送补丁
   └── 通过邮件发送到对应的 mailing list
   └── 抄送维护者 (MAINTAINERS 文件)

5. Review 周期
   └── 维护者/Reviewer 给出反馈
   └── 修改并发送 v2, v3...
   └── Review 可能数周到数月

6. 合入
   └── 维护者 ack 后合入 subsystem tree
   └── 进入 linux-next 测试
   └── 最终被 Linus 合入主线
```

## 相关标签

```text
// 必备标签
Signed-off-by:           // 开发者证书（DCO），表示你有权提交

// 审核标签
Reviewed-by:             // 代码审查通过
Acked-by:                // 模块维护者同意
Tested-by:               // 测试过
Reported-by:             // 问题报告者（修复 bug 时）

// 修正标签
Fixes: abcd1234 ("module: commit message")  // 指出修复的提交
Cc: stable@vger.kernel.org                   // 需要 backport 到 stable 内核

// 版本标签
v1, v2, v3...            // 补丁版本号
RFC                      // Request For Comments（征求反馈，非正式提交）
```

## checkpatch.pl 常用检查

```bash
# 运行 checkpatch
./scripts/checkpatch.pl --strict 0001-my-patch.patch

# 常见错误：
# ERROR: code indent should use tabs where possible
#    → 内核代码用 Tab 缩进（8字符宽）
#
# WARNING: line over 80 characters
#    → 尽量保持在 80 字符以内（偶尔可以超过）
#
# WARNING: Missing Signed-off-by
#    → 每个补丁必须包含 Signed-off-by
#
# CHECK: braces {} should be used
#    → 风格一致性
```

## Maintainer 选择

```bash
# 查看 MAINTAINERS 文件确定维护者
./scripts/get_maintainer.pl drivers/i2c/busses/i2c-qcom-geni.c
# 输出类似：
# Bjorn Andersson <andersson@kernel.org> (supporter:ARM/QUALCOMM...)
# Andy Gross <agross@kernel.org> (supporter:ARM/QUALCOMM...)
# linux-arm-msm@vger.kernel.org (open list:ARM/QUALCOMM...)
# linux-kernel@vger.kernel.org (open list)
```

## Upstream 常见拦截点

```
1. 缺少 Signed-off-by → 补丁被拒绝
2. 格式不符合规范 → checkpatch 错误
3. 没有 CC 给正确的人 → 被忽略
4. 基于错误的分支 → cherry-pick 冲突
5. 提交消息不清晰 → reviewer 不理解
6. 代码风格不行 → 重新格式化
7. 功能不完整 → 需要补充
8. 与其他子系统冲突 → 需要协调
```

## 实操练习

### 1. 给 Linux 主线提交一个测试补丁

```bash
# 先探索：找到可以改进的地方
# 1. 在 Documentation/ 中找到拼写错误
# 2. 或者修复一个 Kconfig 中的描述错误
# 3. 或者清理编译警告

# 示例：修复文档中的拼写
git clone https://github.com/torvalds/linux
cd linux
# 修改文档
# 生成补丁
git format-patch HEAD~1
# 检查格式
./scripts/checkpatch.pl 0001-*.patch
# 发送
git send-email 0001-*.patch
```

### 2. 使用 b4 工具管理补丁系列

```bash
# b4 是一个方便的补丁管理工具
# https://github.com/mricon/b4

# 下载某个补丁系列
b4 am 20240115.12345@qualcomm.com

# 发送补丁
b4 send --to linux-arm-msm@vger.kernel.org
```

## 真实 GitHub 资源

| 仓库/资源 | 说明 |
|-----------|------|
| [torvalds/linux/Documentation/process/](https://github.com/torvalds/linux/tree/master/Documentation/process) | 内核开发流程 |
| [torvalds/linux/Documentation/process/submitting-patches.rst](https://github.com/torvalds/linux/blob/master/Documentation/process/submitting-patches.rst) | 补丁提交指南（必读） |
| [torvalds/linux/scripts/checkpatch.pl](https://github.com/torvalds/linux/blob/master/scripts/checkpatch.pl) | 补丁检查脚本 |
| [mricon/b4](https://github.com/mricon/b4) | 补丁管理工具 |
| [git/git](https://github.com/git/git) | Git 源码（了解 git send-email） |

## 面试考点

1. **Signed-off-by 的法律意义？** 开发者原创声明（DCO），表示有权提交此代码
2. **Fixes 标签的作用？** 帮助 stable 内核团队知道哪些补丁需要 backport
3. **为什么补丁要 CC 给维护者？** 维护者负责该子系统的代码审查和合并
4. **checkpatch.pl 最常见的警告？** 行太长（80列）、Tab 缩进、缺少 Signed-off-by
5. **linux-next 的作用？** 各子系统的补丁集成测试树，发现集成问题
