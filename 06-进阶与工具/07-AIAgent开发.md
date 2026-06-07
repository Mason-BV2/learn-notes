# AI Agent 使用与开发

## 概述

2025-2026 年 AI Agent（智能代理）已深刻改变软件开发方式。在内核开发领域，AI Agent 可以辅助写驱动、分析崩溃、搜索源码、生成补丁。掌握 AI Agent 的使用和开发，已成为高通内核工程师的隐形要求。

> 高通场景：使用 Claude Code/Cursor 辅助内核驱动开发、用 AI 分析 ramdump、自动化补丁 review、构建智能调试助手。

## AI Agent 基础概念

### 什么是 AI Agent

```
AI Agent = 大语言模型（LLM）+ 工具调用（Tools）+ 记忆（Memory）+ 规划（Planning）

与传统 Chat 的区别：
  Chat：你问一句，AI 答一句
  Agent：你给一个目标，AI 自主规划、执行、纠错直到完成
```

### 常用 AI Agent 工具

| 工具 | 类型 | 内核开发用途 |
|------|------|-------------|
| **Claude Code** | CLI Agent | 读/写源码、分析崩溃、重构代码、git 操作 |
| **Cursor** | IDE Agent | 上下文感知编码、代码解释、重构 |
| **GitHub Copilot** | IDE 补全 | 代码自动补全、内联建议 |
| **Aider** | CLI Agent | Git 感知的编码助手 |
| **OpenAI o-series** | 推理模型 | 复杂推理、数学、架构设计 |

## 内核开发中的 AI Agent 实践

### 1. 驱动开发辅助

```bash
# 场景：写一个高通 I2C 驱动

# Claude Code 使用示例：
# prompt: "在 drivers/i2c/busses/ 下创建一个高通 GENI I2C 控制器驱动，
# 支持标准模式和快速模式传输，使用 threaded IRQ。"

# AI Agent 会：
# 1. 读取 drivers/i2c/busses/ 下已有驱动作为参考
# 2. 读取 Documentation/devicetree/bindings/i2c/ 中的 bindings
# 3. 生成驱动框架代码
# 4. 生成对应的 Kconfig 和 Makefile 修改
# 5. 生成 DTS binding 文档

# 好的 AI Agent 提示词（Prompt Engineering）：
"""
请帮我写一个 I2C 控制器驱动，遵循以下要求：
- 平台：Qualcomm SoC，基于 GENI serial engine
- 框架：使用 Linux kernel 6.x I2C subsystem framework
- 特性：支持标准模式(100kHz)和快速模式(400kHz)
- 中断：使用 threaded IRQ
- DMA：支持 DMA 传输（可选）
- 兼容性：需要同时支持 of_match 和 ACPI match
- 代码风格：严格遵循 Linux kernel coding style
"""
```

### 2. 内核代码分析与调试

```bash
# 场景：分析 kernel panic

# 把 panic log 发给 AI Agent：
# "分析这个 kernel panic log，给我 root cause 和修复方案"

# AI Agent 会：
# 1. 解析 PC 地址和调用栈
# 2. 识别异常类型（NULL pointer / Oops / BUG）
# 3. 定位到源码位置
# 4. 分析可能的 root cause
# 5. 给出修复建议和代码示例

# 示例 panic log 分析：
# [22719.265689] Unable to handle kernel NULL pointer dereference at virtual address 0000000000000010
# [22719.265695] Mem abort info:
# [22719.265698]   ESR = 0x96000004
# [22719.265701]   EC = 0x25: DABT (current EL), IL = 32 bits
# [22719.265706]   SET = 0, FnV = 0
# [22719.265709]   EA = 0, S1PTW = 0
# [22719.265712] Data abort info:
# [22719.265715]   ISV = 0, ISS = 0x00000004
# [22719.265718]   CM = 0, WnR = 0
# [22719.265722] user pgtable: 4k pages, 48-bit VAs, pgdp=0000000101fe7000
# [22719.265727] [0000000000000010] pgd=0000000000000000, p4d=0000000000000000
# [22719.265734] Internal: Oops: 96000004 [#1] PREEMPT SMP

# AI 分析结果：
# "这是在访问结构体成员时发生了 NULL pointer dereference，
#  地址 0x10 表示结构体 base + offset 0x10 处为空，
#  建议检查 drivers/i2c/busses/i2c-qcom-geni.c 中 qcom_geni_i2c_probe()
#  中的 dev->base MMIO 映射是否成功。"
```

### 3. 补丁 Review

```bash
# 场景：让 AI Agent review 你的补丁

# "Review this patch for Linux kernel coding style compliance and bugs"

# AI Agent 会检查：
# 1. 代码风格（checkpatch 规则）
# 2. 内存泄漏
# 3. 并发问题（缺少锁）
# 4. 错误处理不完整
# 5. 提交信息格式

# 示例：让 AI 检查补丁
git diff HEAD~1 | claude -p "Review this kernel patch for bugs and style issues"
```

### 4. 设备树生成

```bash
# 场景：为新的硬件生成 DTS 节点

# "基于以下硬件信息，生成 DeviceTree binding 和 DTS 节点：
# - I2C touchscreen, address 0x38
# - 使用 Goodix GT911 控制器
# - 中断 GPIO 42, 上升沿触发
# - 复位 GPIO 43, 低电平有效
# - 电源电压 3.3V"

# AI Agent 生成：
/*
goodix,gt911:
  type: object
  properties:
    compatible:
      const: goodix,gt911
    reg:
      maxItems: 1
    interrupt-parent: true
    interrupts:
      description: GPIO interrupt
    reset-gpios:
      description: Reset GPIO
    vdd-supply: true
*/

&i2c1 {
    touchscreen@38 {
        compatible = "goodix,gt911";
        reg = <0x38>;
        interrupt-parent = <&tlmm>;
        interrupts = <42 IRQ_TYPE_EDGE_RISING>;
        reset-gpios = <&tlmm 43 GPIO_ACTIVE_LOW>;
        vdd-supply = <&pm8998_l17>;
    };
};
```

## AI Agent 开发 — 构建自己的 AI 工具

作为高通内核工程师，你可能会需要构建自定义的 AI 调试助手。

### 1. 使用 Claude API 构建调试助手

```python
#!/usr/bin/env python3
"""
qcom-debug-agent.py — 高通内核调试 AI Agent

功能：分析 kernel panic log、ramdump 摘要、提供修复建议
"""

import os
import sys
import subprocess
from anthropic import Anthropic

client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))

def analyze_panic(log_file):
    """分析 kernel panic log"""
    with open(log_file, 'r') as f:
        log = f.read()
    
    # 提取关键信息
    pc_addr = extract_pc(log)
    stack = extract_stacktrace(log)
    
    prompt = f"""
    你是一个 Linux 内核调试专家，专精于 Qualcomm 平台。
    
    Kernel Panic Log:
    ```
    {log[:8000]}  # 限制长度
    ```
    
    请分析：
    1. Crash 类型和原因
    2. 出错的函数和代码行（参考内核源码）
    3. 可能的 root cause
    4. 修复建议
    5. 是否需要检查 MMIO/寄存器状态
    """
    
    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=2000,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.content[0].text

def extract_pc(log):
    """从 panic log 提取 PC 地址"""
    import re
    match = re.search(r'pc\s*:\s*\[?([0-9a-f]+)\]?', log, re.IGNORECASE)
    return match.group(1) if match else "unknown"

def extract_stacktrace(log):
    """提取调用栈"""
    lines = []
    capture = False
    for line in log.split('\n'):
        if 'Call trace' in line or 'Backtrace' in line:
            capture = True
            continue
        if capture and line.strip() == '':
            break
        if capture:
            lines.append(line.strip())
    return lines

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print("Usage: qcom-debug-agent.py <panic_log>")
        sys.exit(1)
    
    result = analyze_panic(sys.argv[1])
    print(result)
```

### 2. 使用 Agent SDK 构建交互式 Agent

```python
"""
kernel-explorer-agent.py — 交互式内核源码探索 Agent
可以帮你查找内核源码、解释机制、生成测试代码
"""

from claude_agent_sdk import Agent, tool
import subprocess
import os

class KernelExplorer:
    def __init__(self, kernel_path):
        self.kernel_path = kernel_path
    
    @tool
    def search_source(self, pattern: str, path: str = "") -> str:
        """在内核源码中搜索模式"""
        search_path = os.path.join(self.kernel_path, path)
        result = subprocess.run(
            ["grep", "-rn", "--include=*.c", "--include=*.h", 
             pattern, search_path],
            capture_output=True, text=True, timeout=30
        )
        return result.stdout[:3000]  # 限制输出长度
    
    @tool
    def read_source(self, filepath: str) -> str:
        """读取内核源码文件"""
        fullpath = os.path.join(self.kernel_path, filepath)
        with open(fullpath, 'r') as f:
            return f.read()[:5000]
    
    @tool
    def read_doc(self, topic: str) -> str:
        """读取内核文档"""
        doc_path = os.path.join(
            self.kernel_path, 
            "Documentation", 
            topic
        )
        if os.path.isfile(doc_path):
            with open(doc_path, 'r') as f:
                return f.read()[:5000]
        # 尝试搜索文档
        result = subprocess.run(
            ["find", os.path.join(self.kernel_path, "Documentation"),
             "-name", f"*{topic}*", "-type", "f"],
            capture_output=True, text=True, timeout=10
        )
        return result.stdout

# 使用示例
# agent = Agent(
#     model="claude-opus-4",
#     tools=KernelExplorer(kernel_path="/home/user/linux")
# )
# agent.run("解释 Linux 内核中的调度器 CFS 工作原理，并给我看 kernel/sched/fair.c 中的关键函数")
```

### 3. 在 VSCode/Cursor 中配置 AI Agent

```json
// .cursorrules — 为内核开发定制的 AI 规则
{
  "rules": [
    "你是一个 Linux 内核开发专家，专精于 ARM64 + Qualcomm 平台。",
    "代码风格遵循 Linux kernel coding style（缩进8空格 Tab）。",
    "导出的函数和变量需要使用 EXPORT_SYMBOL_GPL。",
    "驱动代码需要包含 Devicetree compatible 匹配表。",
    "函数命名使用 kernel 风格（下划线分隔）。",
    "提交信息格式遵循 kernel 规范（<module>: <description>）。",
    "优先使用内核提供的 API，不要重复造轮子。",
    "错误处理使用 ERR_PTR/PTR_ERR 模式。",
    "使用 devm_* 系列 API 进行资源管理。",
    "不要在内核代码中使用浮点运算。",
    "中断上下文不要使用 mutex，使用 spinlock。",
    "需要包含 MODULE_LICENSE/GPL/DESCRIPTION。"
  ]
}
```

## AI Agent 辅助面试准备

AI Agent 也可以帮你模拟面试：

```python
"""
mock-interview.py — 高通内核面试模拟器
"""

from anthropic import Anthropic

client = Anthropic()

def mock_interview(topic: str):
    """模拟高通面试"""
    prompt = f"""
    你是一个高通面试官，正在面试 Linux 内核工程师岗位。
    
    请就以下主题向我提问 5 个问题：
    {topic}
    
    格式要求：
    - 问题由浅入深
    - 我回答后给我打分（1-10）和反馈
    - 最后给出标准答案
    - 包含实际的高通工作场景
    """
    
    # 交互式模拟
    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1000,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return response.content[0].text

# 使用示例
# mock_interview("Linux 内核中断处理和 threaded IRQ")
```

## AI Agent 使用注意事项

### 内核开发中 AI Agent 的限制

```c
// 1. AI 可能生成不安全的代码
//    × 直接解引用用户态指针（没有 copy_from_user）
//    × 忘记加锁
//    × 在中断上下文调用可能 sleep 的函数
//    
//    解决：始终 review AI 生成的代码，做 code review

// 2. AI 可能引用不存在的 API
//    × 编造不存在的内核函数
//    × 使用已废弃的 API
//    
//    解决：让 AI 提供源码引用，查证后再用

// 3. AI 不理解特定平台的 quirks
//    × 高通特有的寄存器配置
//    × 特定 SoC 版本的 errata
//    
//    解决：结合 Qualcomm 文档和源码 review
```

### 最佳实践

```bash
# 1. AI 辅助 + 人工 review = 最佳效果
# 2. 把 AI Agent 当作"高级 copilot"，不是"替代品"
# 3. 学会写好的 prompt
#    - 具体：说清楚你要什么
#    - 上下文：提供足够的背景信息
#    - 约束：明确编码规范和安全要求
# 4. 让 AI 做它擅长的事
#    ✓ 代码生成（模板代码）
#    ✓ 代码解释
#    ✓ bug 定位
#    ✓ 文档生成
#    ✗ 安全性关键决策
#    ✗ 架构设计（AI 缺乏系统全局观）
```

## 实操练习

### 1. 使用 Claude Code 完成一个内核模块

```bash
# 安装 Claude Code
npm install -g @anthropic-ai/claude-code

# 在内核源码目录中使用
cd linux
claude

# 提示词：
# "Create a simple platform driver in drivers/misc/ that:
# 1. Registers a misc device
# 2. Supports open/read/write/release operations  
# 3. Has devicetree match table
# 4. Follows kernel coding style
# 5. Includes proper Kconfig and Makefile changes"
```

### 2. 使用 AI Agent 分析内核源码

```bash
# 让 AI 解释高通驱动的某个函数
claude -p "Explain the qcom_geni_i2c_xfer function in 
  drivers/i2c/busses/i2c-qcom-geni.c, 
  including the DMA and non-DMA transfer paths"
```

## 真实 GitHub 资源

| 仓库 | 说明 |
|------|------|
| [anthropics/claude-code](https://github.com/anthropics/claude-code) | Claude Code CLI 工具 |
| [anthropics/claude-agent-sdk](https://github.com/anthropics/claude-agent-sdk) | Claude Agent SDK |
| [getcursor/cursor](https://github.com/getcursor/cursor) | Cursor IDE |
| [Aider-AI/aider](https://github.com/Aider-AI/aider) | Aider 编码 Agent |
| [github/copilot](https://github.com/features/copilot) | GitHub Copilot |

## 面试考点

1. **你用过哪些 AI 辅助开发工具？** — 展示你使用 AI Agent 的经验
2. **AI 生成的代码有什么风险？** — 安全性、正确性、许可证合规
3. **如何看待 AI 对内核开发的影响？** — AI 辅助但不会替代内核工程师
4. **如何验证 AI 生成的驱动代码？** — 编译检查、checkpatch、功能测试、安全 review
5. **用过 Claude Code/Cursor 做什么？** — 具体用例（写驱动、分析 panic、生成 dts）
