# AI 辅助内核开发 — 2026 年全景

> 归档时间：2026-06-11
> 用途：了解当前 AI 在系统软件/内核开发领域的实际应用，消除"被替代"焦虑，指导学习方向

---

## 一、通用场景：大家都在用 AI 干什么

| 场景 | 普及度 | 说明 |
|------|--------|------|
| CRUD / 前端 / API 开发 | ⚡ 主力 | 自然语言直接生成，大部分中小需求不用手写 |
| 写单元测试 | ⚡ 主力 | 人写代码，AI 补测试，覆盖率大幅提升 |
| 代码审查 | 🔥 快速增长 | 内核社区也在用（见下文） |
| 重构 / 迁移 | 🔥 常见 | Python 2→3、Java 8→21、框架升级等 |
| 文档 / 注释 / Commit 信息 | ✅ 标配 | 没人手写 commit message 了 |
| 架构设计 / 系统设计 | 🟡 辅助 | 给初稿和选项，但决策仍要人来定 |
| 调试复杂问题 | 🟡 辅助 | 缩小范围，根因分析仍需经验 |
| 系统软件 / C 编译器 | 🟢 前沿 | 16 个 AI Agent 协作写出能编译 Linux 内核的 C 编译器 |

---

## 二、内核开发工程师实际工作中的 AI 应用

### 2.1 代码审查 — 内核社区已在用

2026 年最重要的趋势：**Linux 内核社区主动将 AI 嵌入 patch review 流程**。

**Chris Mason**（Meta，Btrfs 创建者）
- 创建了 AI code review prompt 仓库
- 将 review 拆为子任务：chunk review → lore thread 检查 → syzkaller 修复验证 → 最终报告
- 降低 token 消耗，覆盖更多 bug 类型

**David Airlie**（Red Hat，DRM 显示驱动联合维护者）
- 在 DRM 显示驱动子系统跑 Claude Opus 4.6 patch 审查实验
- 审核结果发布到专门邮件列表

**Roman Gushchin**（Google）— Sashiko 工具
- Rust 编写的 AI code review 工具，扫描内核邮件列表
- 测试结果：**1000 个近期上游 bug 中，找到 53%——全部被人类 reviewer 漏掉**
- 使用 Gemini 3.1 Pro，同时支持 Claude

**Konstantin Ryabitsev**（Linux 基金会）— b4 review TUI
- b4 工具新增 TUI 界面，集成 Claude Code 做 AI code review
- 接近 pre-alpha 发布

### 2.2 GPU/加速器内核开发 — AI 在写底层算子

**LinkedIn（Liger Kernel）**
- 3 个生产级 Agent 工作流
- 将 PyTorch op / 自然语言自动转为优化后的 Triton kernel
- 成果：**1.9–3.2× 加速**，内部模型 **encoder 10× 加速**，**64.7% GPU 工时节约**

**AWS（Neuron Agentic Development）**
- 5 个 Agent 技能覆盖：写→调试→profile→分析 Trainium/Inferentia kernel
- 开发者无需深入芯片知识即可从 PyTorch 生成硬件优化 kernel

**Google（MaxKernel）**
- Agentic 框架将 JAX / CUDA / Triton 转为 TPU Pallas kernel
- 成果：**比人工 kernel 降低 8.7% 延迟，提升 9% 吞吐**（v5p TPU）

### 2.3 跨平台内核移植（KernelGen）

FlagOS 社区的 KernelGen：
- **世界首个多芯片 Triton kernel 自动生成工具**
- 输入自然语言 / 数学公式 / 现有实现 → 输出验证 + 性能测试的 kernel
- 支持华为昇腾、寒武纪、沐曦等国产芯片
- 目标：打破 CUDA 厂商锁定

### 2.4 学术前沿：Agent 在新 ISA 上生成 kernel

KernelCraft 基准测试：
- 评估 LLM Agent 在**全新 ISA 架构**上生成底层 kernel 的能力
- 结果：顶级 Agent 能在几步内产出功能正确的 kernel
- 对硬件验证和早期原型开发有重要意义

### 2.5 日常辅助场景

内核工程师高频使用的 AI 场景：

```
📝 patch commit message 润色（内核有固定提交风格）
🐛 解释 Oops 栈回溯（贴 dmesg → AI 辅助分析）
🔍 找代码位置（"interrupt handler 注册流程在哪"）
📖 翻译/解释内核文档（大量英文技术细节）
📋 生成 DeviceTree 节点（减少重复手写）
🔄 代码风格检查（checkpatch 自动修正）
```

---

## 三、AI 替代了什么？替代不了什么？

### 已经被"吃掉"的工作（体力活 → AI）

| 工作 | 替代程度 |
|------|----------|
| patch 格式检查、commit message 润色 | ✅ 可完全交给 AI |
| driver skeleton 初稿（你改对就行） | ✅ AI 生成 80% 模板 |
| 代码审查第一遍扫描（找明显 bug） | ✅ 已在生产中 |
| 内核文档检索和初步解读 | ✅ 比 man grep 快 |
| DeviceTree 节点生成 | ✅ 减少重复手写 |

### 仍需要人的领域（护城河）

| 工作 | 为什么 AI 不行 |
|------|---------------|
| Oops/traceback 深层根因分析 | corner case 数据稀缺 |
| 新硬件 bring-up 调试 | AI 不会摸示波器、读逻辑分析仪 |
| 架构决策（锁机制 / 数据结构选择） | 需要理解业务场景和系统约束 |
| 性能调优中的直觉判断 | 依赖对硬件的深刻理解 |
| 社区协作和评审 | 需要人的判断力和政治智慧 |

---

## 四、标志性事件：16 个 Agent 写出能编译 Linux 的 C 编译器

Anthropic 实验（2026-02）：

| 指标 | 数据 |
|------|------|
| Agent 数 | 16 个 Claude Opus 4.6 实例 |
| 代码语言 | Rust |
| 代码量 | **10 万行** |
| GCC torture test 通过率 | **99%** |
| 支持架构 | x86、ARM、RISC-V |
| 能编译的项目 | Linux 6.9、Doom、PostgreSQL、Redis、FFmpeg、SQLite、QEMU |
| 总花费 | ~$20,000（14 万 RMB） |
| 总耗时 | 2 周，近 2000 个 session |

**协作方式（无人类管理）：**
- 没有"老板 Agent"，16 个实例各自独立运行
- 共享 Git 仓库，自己找任务、创建 lock 文件、写代码、跑测试、推代码
- 遇到 merge 冲突？自己解决
- **没有一次站会，没有一个需求文档**

**局限：**
- 生成代码效率较低（不如 GCC -O0 的效果）
- 16 位 x86 实模式代码搞不定
- 仍依赖 GNU 工具链

> *"I did not expect this to be anywhere near possible so early in 2026."*
> — Nicholas Carlini，Anthropic 研究员

---

## 五、对内核开发工程师的启示

### 5.1 内核开发是护城河最深的领域之一

AI 替代的难易程度与以下因素负相关：
1. **数据稀缺度** — 内核调试的高质量数据互联网上很少
2. **环境复杂度** — 需要真实硬件、外设、示波器
3. **后果严重性** — 内核崩了就崩了，不是"重新生成一次"能解决的

### 5.2 正确的竞争策略

> **内核工程师里最懂 AI 的，AI 用户里最懂内核的。**

这意味着：
- 用 AI 加速学习中（你现在正在做的事 ✅）
- 用 AI 生成驱动模板/设备树/测试用例（减少体力活，聚焦核心问题）
- 用 AI 做代码审查辅助（跟内核社区最新实践一致）
- 把自己的经验写成可复用的 Skill（如 Matt Pocock 的 `write-a-skill`）

### 5.3 未来面试会问的

2026 年的内核岗位面试，不会问"你用过 AI 吗"，而是：
- "你如何在驱动开发中利用 AI？"
- "你怎么用 AI 辅助内核调试？"
- "你用 AI 写过 DeviceTree 节点吗？"

---

## 六、参考链接

- [Phoronix: AI Code Review Prompts 倡议进入 Linux 内核](https://www.phoronix.com/news/AI-Code-Review-Prompts-Linux)
- [The Register: Sashiko — AI 代码审查工具，发现 53% 被漏掉的 bug](https://www.theregister.com/software/2026/03/20/linux-kernel-engineer-introduces-sashiko-code-review-system/)
- [Phoronix: Linux DRM 子系统实验性 AI 代码审查](https://www.phoronix.com/news/Linux-DRM-AI-Code-Review)
- [腾讯云: 16 个 AI Agent 协作从零写出能编译 Linux 内核的 C 编译器](https://cloud.tencent.cn/developer/article/2629270)
- [AWS: Neuron Agentic Development — AI Agent 加速 Trainium 内核开发](https://aws.amazon.com/cn/blogs/machine-learning/stop-hand-tuning-kernels-how-neuron-agentic-development-accelerates-aws-trainium-tunings/)
- [arXiv: KernelCraft — 面向新兴硬件的 Agentic 内核生成基准](https://export.arxiv.org/abs/2603.08721)
- [LinkedIn: Liger Kernel — Agentic 工作流加速 GPU 内核工程](https://www.zenml.io/llmops-database/ai-agents-accelerating-gpu-kernel-engineering-for-llm-infrastructure)
- [Google: MaxKernel — 自动化 Pallas Kernel 生成与优化](https://discuss.google.dev/t/maxkernel-automating-pallas-kernel-generation-and-optimization-via-agentic-systems/366686)
- [mattpocock/skills — 124K Stars 的 Agent Skill 工程化仓库](https://github.com/mattpocock/skills)
