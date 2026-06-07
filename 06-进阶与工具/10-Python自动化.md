# Python 自动化脚本

## 概述

Python 是高通内核开发中的重要辅助工具。JD 明确写了 "Familiarity with automation scripts like Python is a plus"。

> 高通场景：自动化内核编译、批量处理 defconfig、分析日志、操作 Git 仓库、与 Gerrit 交互、解析 trace 数据。

## 内核开发中的 Python 应用

```
内核开发中 Python 的典型用途：

1. 构建自动化
   - 批量编译内核参数组合
   - 自动化 bisect
   - 内核配置管理

2. 日志分析
   - 解析 dmesg
   - 分析 KASAN/panic 日志
   - trace 数据可视化

3. 代码处理
   - 批量修改 DTS 文件
   - 检查代码风格
   - 生成 Makefile/Kconfig

4. 测试自动化
   - 驱动功能测试
   - 压力测试
   - 回归测试

5. Git 操作
   - 批量 cherry-pick
   - 自动化基线升级
   - Gerrit API 交互
```

## 1. 内核构建自动化

```python
#!/usr/bin/env python3
"""
kernel_builder.py — 自动化内核编译脚本
支持多配置、多架构编译
"""

import os
import sys
import subprocess
import argparse
import logging
from pathlib import Path
from concurrent.futures import ThreadPoolExecutor

logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s [%(levelname)s] %(message)s')
log = logging.getLogger(__name__)

class KernelBuilder:
    def __init__(self, kernel_dir: str):
        self.kernel_dir = Path(kernel_dir)
        if not self.kernel_dir.exists():
            raise ValueError(f"Kernel directory not found: {kernel_dir}")
    
    def defconfig(self, arch: str = "arm64",
                   cross_compile: str = "aarch64-linux-gnu-"):
        """运行 defconfig"""
        env = os.environ.copy()
        env["ARCH"] = arch
        env["CROSS_COMPILE"] = cross_compile
        
        log.info(f"Running defconfig for {arch}")
        result = subprocess.run(
            ["make", "defconfig"],
            cwd=self.kernel_dir,
            env=env,
            capture_output=True,
            text=True
        )
        if result.returncode != 0:
            log.error(f"defconfig failed: {result.stderr}")
            return False
        log.info(f"defconfig: {result.stdout.strip()}")
        return True
    
    def build(self, arch: str = "arm64",
               cross_compile: str = "aarch64-linux-gnu-",
               targets: list = None,
               jobs: int = None):
        """编译内核"""
        if targets is None:
            targets = ["Image", "dtbs"]
        
        if jobs is None:
            jobs = os.cpu_count() or 4
        
        env = os.environ.copy()
        env["ARCH"] = arch
        env["CROSS_COMPILE"] = cross_compile
        
        cmd = ["make", f"-j{jobs}"] + targets
        log.info(f"Running: {' '.join(cmd)}")
        
        result = subprocess.run(
            cmd,
            cwd=self.kernel_dir,
            env=env,
            capture_output=True,
            text=True
        )
        
        if result.returncode != 0:
            log.error(f"Build failed:\n{result.stderr[-2000:]}")
            return False
        
        # 提取编译时间
        for line in result.stdout.split('\n'):
            if 'Kernel:' in line or 'DTC:' in line:
                log.info(f"  {line.strip()}")
        
        return True
    
    def build_module(self, module_path: str):
        """编译单个模块"""
        env = os.environ.copy()
        env["ARCH"] = "arm64"
        env["CROSS_COMPILE"] = "aarch64-linux-gnu-"
        
        cmd = ["make", f"M={module_path}", "modules"]
        log.info(f"Building module: {module_path}")
        
        result = subprocess.run(
            cmd, cwd=self.kernel_dir, env=env,
            capture_output=True, text=True
        )
        return result.returncode == 0

def main():
    parser = argparse.ArgumentParser(description="Kernel build automation")
    parser.add_argument("--dir", default=".", help="Kernel source directory")
    parser.add_argument("--arch", default="arm64", help="Target architecture")
    parser.add_argument("--defconfig", action="store_true", help="Run defconfig")
    parser.add_argument("--module", help="Build specific module path")
    parser.add_argument("--jobs", type=int, help="Parallel jobs")
    
    args = parser.parse_args()
    
    builder = KernelBuilder(args.dir)
    
    if args.defconfig:
        builder.defconfig(args.arch)
    
    if args.module:
        builder.build_module(args.module)
    
    # 默认编译内核
    if not args.defconfig and not args.module:
        builder.build(args.arch, jobs=args.jobs)

if __name__ == "__main__":
    main()
```

## 2. 日志分析工具

```python
#!/usr/bin/env python3
"""
dmesg_analyzer.py — 内核日志分析工具
解析 dmesg，提取关键信息
"""

import re
import sys
from collections import defaultdict
from datetime import datetime

class DmesgParser:
    def __init__(self, log_path: str = None):
        self.log = []
        if log_path:
            with open(log_path, 'r') as f:
                self.log = f.readlines()
        else:
            import subprocess
            result = subprocess.run(['dmesg'], capture_output=True, text=True)
            self.log = result.stdout.split('\n')
    
    def extract_panics(self):
        """提取所有 panic/oops"""
        panics = []
        for i, line in enumerate(self.log):
            if re.search(r'(Kernel panic|Oops|BUG:|Unable to handle)',
                         line, re.IGNORECASE):
                # 提取上下文（前后20行）
                start = max(0, i - 5)
                end = min(len(self.log), i + 20)
                panics.append({
                    'line': i + 1,
                    'msg': line.strip(),
                    'context': ''.join(self.log[start:end])
                })
        return panics
    
    def extract_qcom(self):
        """提取高通相关日志"""
        qcom_lines = []
        for line in self.log:
            if re.search(r'(qcom|msm|snapdragon|soc)', line, re.IGNORECASE):
                qcom_lines.append(line.strip())
        return qcom_lines
    
    def extract_errors(self):
        """提取错误级别日志"""
        errors = []
        for line in self.log:
            if re.search(r'(error|failed|timeout|ERR)', line, re.IGNORECASE):
                errors.append(line.strip())
        return errors
    
    def timing_analysis(self):
        """分析启动时间"""
        timestamps = []
        for line in self.log:
            match = re.match(r'\[\s*(\d+\.\d+)\]', line)
            if match:
                time = float(match.group(1))
                # 提取关键阶段
                if 'Booting Linux' in line:
                    timestamps.append(('boot_start', time))
                elif 'Run /init' in line or 'init started' in line.lower():
                    timestamps.append(('init_start', time))
                elif 'Freeing unused kernel memory' in line:
                    timestamps.append(('kernel_done', time))
        
        return timestamps
    
    def summary(self):
        """打印摘要"""
        panics = self.extract_panics()
        errors = self.extract_errors()
        timings = self.timing_analysis()
        
        print("=" * 60)
        print("Kernel Log Analysis Summary")
        print("=" * 60)
        
        if panics:
            print(f"\n❌ Panics/Oops: {len(panics)}")
            for p in panics[:3]:
                print(f"  Line {p['line']}: {p['msg'][:80]}")
        else:
            print("\n✅ No panics found")
        
        print(f"\n⚠️  Errors: {len(errors)}")
        for e in errors[:5]:
            print(f"  {e[:100]}")
        
        if timings:
            print(f"\n⏱️  Boot timing:")
            for name, time in timings:
                print(f"  {name}: {time:.2f}s")

def main():
    if len(sys.argv) > 1:
        parser = DmesgParser(sys.argv[1])
    else:
        parser = DmesgParser()
    
    parser.summary()

if __name__ == "__main__":
    main()
```

## 3. Git 操作自动化

```python
#!/usr/bin/env python3
"""
git_kernel.py — 内核 Git 操作自动化
批量 cherry-pick、基线升级辅助
"""

import subprocess
import os
from pathlib import Path
import logging

logging.basicConfig(level=logging.INFO)
log = logging.getLogger(__name__)

class GitKernel:
    def __init__(self, repo_path: str):
        self.repo = Path(repo_path)
        if not (self.repo / '.git').exists():
            raise ValueError(f"Not a git repo: {repo_path}")
    
    def run_git(self, *args):
        """执行 git 命令"""
        cmd = ['git'] + list(args)
        log.debug(f"Running: {' '.join(cmd)}")
        result = subprocess.run(
            cmd,
            cwd=self.repo,
            capture_output=True,
            text=True
        )
        if result.returncode != 0:
            log.error(f"Git error: {result.stderr}")
        return result
    
    def cherry_pick_series(self, commits: list):
        """批量 cherry-pick"""
        success = []
        failed = []
        
        for commit in commits:
            log.info(f"Cherry-picking: {commit[:12]}")
            result = self.run_git('cherry-pick', commit)
            
            if result.returncode == 0:
                success.append(commit)
            else:
                # 有冲突
                log.warning(f"Conflict with {commit[:12]}")
                self.run_git('cherry-pick', '--abort')
                failed.append(commit)
        
        return success, failed
    
    def bisect_find(self, good: str, bad: str, test_script: str):
        """二分查找引入 bug 的提交"""
        self.run_git('bisect', 'start')
        self.run_git('bisect', 'good', good)
        self.run_git('bisect', 'bad', bad)
        
        while True:
            result = self.run_git('rev-parse', 'HEAD')
            current = result.stdout.strip()
            
            # 运行测试脚本
            test_result = subprocess.run(
                ['bash', test_script],
                cwd=self.repo
            )
            
            if test_result.returncode == 0:
                self.run_git('bisect', 'good')
            else:
                self.run_git('bisect', 'bad')
            
            # 检查 bisect 是否完成
            status = self.run_git('bisect', 'log')
            if 'first bad commit' in status.stdout:
                break
        
        first_bad = self.run_git('bisect', 'bad')
        self.run_git('bisect', 'reset')
        return first_bad.stdout.strip()
    
    def format_patches(self, since: str, output_dir: str):
        """生成补丁"""
        Path(output_dir).mkdir(parents=True, exist_ok=True)
        result = self.run_git(
            'format-patch', '--cover-letter',
            f'--output-directory={output_dir}',
            since
        )
        return result.stdout.strip().split('\n')
    
    def check_patch(self, patch_path: str):
        """运行 checkpatch.pl"""
        script = self.repo / 'scripts' / 'checkpatch.pl'
        if not script.exists():
            log.error("checkpatch.pl not found")
            return None
        
        result = subprocess.run(
            ['perl', str(script), '--strict', patch_path],
            capture_output=True, text=True
        )
        return result.stdout
```

## 4. DTS 处理工具

```python
#!/usr/bin/env python3
"""
dts_tool.py — DeviceTree 文件处理
批量修改、检查、生成 DTS 节点
"""

import re
import os
import sys
from pathlib import Path

class DTSParser:
    """简单的 DTS 解析器（仅供自动化修改，非完整 parser）"""
    
    def __init__(self, dts_path: str):
        with open(dts_path, 'r') as f:
            self.content = f.read()
        self.path = dts_path
    
    def find_node(self, node_name: str) -> list:
        """查找节点及其内容"""
        matches = []
        pattern = rf'(\s*){re.escape(node_name)}\s*{{'
        for match in re.finditer(pattern, self.content):
            start = match.start()
            # 找到匹配的 }
            depth = 0
            pos = match.end() - 1
            while pos < len(self.content):
                if self.content[pos] == '{':
                    depth += 1
                elif self.content[pos] == '}':
                    depth -= 1
                    if depth == 0:
                        end = pos + 1
                        break
                pos += 1
            matches.append(self.content[start:end])
        return matches
    
    def add_property(self, node_path: str, prop_text: str):
        """在指定节点中添加属性"""
        # 找到节点插入点
        old_content = self.content
        # 简单实现：在节点 '{' 后添加
        pattern = rf'({re.escape(node_path)}\s*{{)'
        replacement = rf'\1\n{prop_text}'
        self.content = re.sub(pattern, replacement, self.content)
        return old_content != self.content
    
    def set_status(self, node_path: str, status: str):
        """设置节点 status 属性"""
        pattern = rf'({re.escape(node_path)}\s*{{[\s\S]*?)(status\s*=\s*)"[^"]*"'
        replacement = rf'\1\2"{status}"'
        self.content = re.sub(pattern, replacement, self.content)
        
        # 如果没有 status 属性，添加
        if 'status' not in self.content:
            self.add_property(node_path, f'\tstatus = "{status}";')
    
    def save(self, output_path: str = None):
        """保存修改"""
        path = output_path or self.path
        with open(path, 'w') as f:
            f.write(self.content)
        print(f"Saved to {path}")
    
    def validate(self):
        """基本检查"""
        issues = []
        
        # 检查括号匹配
        braces = 0
        for i, c in enumerate(self.content):
            if c == '{':
                braces += 1
            elif c == '}':
                braces -= 1
            if braces < 0:
                issues.append(f"Line ~{self.content[:i].count(chr(10)) + 1}: Extra '}'")
                break
        
        if braces != 0:
            issues.append(f"Unmatched braces ({braces})")
        
        # 检查常见错误
        if ';' not in self.content:
            issues.append("No semicolons found")
        
        return issues


def batch_set_status(dts_dir: str, node_pattern: str, status: str):
    """批量修改 DTS 文件中节点的 status"""
    dts_dir = Path(dts_dir)
    for dts_file in dts_dir.rglob("*.dts"):
        print(f"Processing: {dts_file}")
        parser = DTSParser(str(dts_file))
        nodes = parser.find_node(node_pattern)
        if nodes:
            parser.set_status(node_pattern, status)
            parser.save()
```

## 5. Gerrit API 交互

```python
#!/usr/bin/env python3
"""
gerrit_tool.py — Gerrit code review 交互工具
"""

import requests
import subprocess
import json
from typing import Optional

class GerritClient:
    def __init__(self, url: str, username: str, api_key: str):
        self.url = url.rstrip('/')
        self.auth = (username, api_key)
    
    def query_changes(self, query: str, limit: int = 10) -> list:
        """查询 Gerrit changes"""
        url = f"{self.url}/changes/"
        params = {
            'q': query,
            'n': limit,
            'o': ['CURRENT_REVISION', 'CURRENT_COMMIT']
        }
        try:
            response = requests.get(
                url,
                params=params,
                auth=self.auth,
                timeout=30
            )
            # Gerrit 返回以 )]}' 开头防 XSSI
            content = response.text
            if content.startswith(")]}'"):
                content = content[4:]
            return json.loads(content)
        except Exception as e:
            print(f"Error: {e}")
            return []
    
    def get_patch(self, change_id: str, revision: str = 'current') -> str:
        """获取 patch 内容"""
        url = f"{self.url}/changes/{change_id}/revisions/{revision}/patch"
        response = requests.get(url, auth=self.auth)
        return response.text
    
    def submit_review(self, change_id: str, revision: str,
                       message: str, labels: dict = None):
        """提交 review"""
        url = f"{self.url}/a/changes/{change_id}/revisions/{revision}/review"
        data = {
            'message': message,
            'labels': labels or {}
        }
        response = requests.post(
            url,
            json=data,
            auth=self.auth,
            headers={'Content-Type': 'application/json'}
        )
        return response.status_code == 200


def gerrit_upload(branch: str, topic: str):
    """使用 git review 上传到 Gerrit"""
    # 确保安装 git-review
    # pip install git-review
    
    cmds = [
        ['git', 'checkout', '-b', topic],
        ['git', 'commit', '--allow-empty', '-m', f'Topic: {topic}'],
        ['git', 'review', branch],
    ]
    
    for cmd in cmds:
        result = subprocess.run(cmd, capture_output=True, text=True)
        print(f"{' '.join(cmd)}: {'OK' if result.returncode == 0 else 'FAIL'}")

if __name__ == '__main__':
    # 使用示例
    client = GerritClient(
        url='https://gerrit.example.com',
        username='your_username',
        api_key='your_api_key'
    )
    
    changes = client.query_changes('project:kernel/msm status:open')
    for change in changes:
        print(f"{change.get('change_id')}: {change.get('subject')}")
```

## 实操练习

### 1. 使用 Python 编译内核并分析日志

```bash
# 编写一个脚本：
# 1. 编译内核
# 2. 启动 QEMU
# 3. 收集 dmesg
# 4. 分析是否有错误

python3 kernel_builder.py --dir ~/linux --defconfig
python3 kernel_builder.py --dir ~/linux
```

### 2. 分析 dmesg

```bash
# 收集 dmesg 并分析
dmesg > dmesg.txt
python3 dmesg_analyzer.py dmesg.txt
```

## 真实 GitHub 资源

| 仓库/资源 | 说明 |
|-----------|------|
| [python/cpython](https://github.com/python/cpython) | Python 官方实现 |
| [requests/requests](https://github.com/psf/requests) | HTTP 库（Gerrit API） |
| [pallets/click](https://github.com/pallets/click) | CLI 工具框架 |
| [Gerrit Code Review](https://www.gerritcodereview.com/) | Gerrit API 文档 |

## 面试考点

1. **Python 在 Kernel 开发中用途？** 构建自动化、日志分析、Git 操作、测试框架、数据处理
2. **subprocess.run 和 os.system 的区别？** subprocess.run 更安全灵活，可以捕获输出、控制输入
3. **如何用 Python 实现自动化 bisect？** 使用 subprocess 调用 git bisect，根据测试结果判断 good/bad
4. **解析 dmesg 时关注哪些信息？** 时间戳、panic 信息、错误调用栈、驱动 probe 结果
