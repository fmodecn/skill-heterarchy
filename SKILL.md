---
name: skill-heterarchy
description: "内异层认知协同 — 在单一 Hermes Agent 内部，分化多心智并行驱动 Claude Code，主控不阻塞、子单元互校验、阻塞自愈。实操技能。"
version: 0.0.10
author: Yuyang001 (FmodeAgent)
license: MIT
tags: [heterarchy, dispatch, claude-code, hermes-tools, cognitive-collaboration, parallel]
---

# skill-heterarchy — 对内分思实操技能

> Delegate 是对外派活，Heterarchy 是对内分思。
> 本技能用 Hermes 底层工具在单一 Agent 内部分化多心智并行思考。
> 加载方式：`skill_view(name='skill-heterarchy')`

---

## 零、何时触发本技能

当你（Hermes）收到**一个复杂任务，明显可以拆成多个独立维度并行处理**时：

| 触发词 | 示例 |
|--------|------|
| "多个主题需要完成" | 课程的四个主题、PPT多章节 |
| "同时做这几件事" | 调研+写代码+出图 |
| "这几个部分需要..." | 多模块开发、多文案撰写 |
| "尽量并行" | 用户明确要求加速 |

**判断原则：** 如果任务可以自然拆成 2-5 个独立认知维度，且每个维度需要整体思考而不是简单脚本执行 → **启用 Heterarchy**。

**何时不用：**
- 单一步骤命令 → 直接 `terminal()` 或 `execute_code()`
- 纯对外输出（发消息、写简单文件）→ 直接本技能完成
- 需要跨 Profile 协作 → 用 `kanban_create` 委派，不是 Heterarchy

---

## 一、任务拆解（认知分化 — Hermes 自己做）

这是唯一不能交给 CC 的步骤。Hermes 必须亲自：
1. 理解用户真实需求
2. 识别可并行的独立维度（每个维度 = 一个独立认知子单元）
3. 判断子单元之间是否有依赖关系
4. 确定项目工作目录（每个 CC 进程必须在正确的 git 仓库根目录启动）

**实操模板（Hermes 的思考过程）：**

```
收到："四个主题的 HTML PPT 需要图文并茂增强"

拆解：
├→ 主题一：数字分身（独立 → 无前置依赖）
├→ 主题二：Harness（独立 → 无前置依赖）
├→ 主题三：Loop（独立 → 无前置依赖）
├→ 主题四：RSI（独立 → 无前置依赖）
└→ 依赖：无（四主题可完全并行）

项目根目录：/opt/data/git-repos/skill-present
CC 必须在项目根目录启动，否则读不到 CLAUDE.md 的项目规则。
```

---

## 二、任务书编写（心智分化）

每个认知子单元需要一份独立的任务书，包含：

```
1. 背景上下文（用户原始需求）
2. 本次要做什么（明确范围，不要多做）
3. 要保留什么（已有的内容别动）
4. inline 素材（路径、关键描述）
5. 纪律约束：
   - 框架不可改
   - 自绘 SVG 保留
   - 案例 CDN 图优先用已有
   - skill-image 只用于核心架构图/场景图
6. 验收标准（git commit、Content-Type 验证、URL 200）
7. 禁止事项（不要重写、不要改标题、不要改结构）
```

任务书写入临时文件 `/tmp/cc-xxx-task.md`，用 `cat` 管道喂入 CC，不用 `$(cat)` 避免路径问题：

**正确姿势：**
```bash
cat /tmp/cc-xxx-task.md | /opt/data/npm-global/bin/claude -p --dangerously-skip-permissions > /tmp/cc-xxx.log 2>&1 &
```

**错误姿势（踩过坑）：**
```bash
# 不要用 $(cat file) —— 文件路径在后台进程可能不同
# 不要省略绝对路径 —— exit 127 找不到 claude
```

---

## 三、Hermes 协同工具调用表（核心实操）

### 3.1 启动心智 — terminal + background + notify

```python
# Hermes 代码中：
terminal(
    command=f"""
    export PATH="/opt/data/npm-global/bin:$PATH"
    cd {project_root}
    cat {task_file_path} | /opt/data/npm-global/bin/claude \\
      -p --dangerously-skip-permissions \\
      > {log_path} 2>&1
    """,
    background=True,
    notify=True,
    timeout=1800,
    workdir=project_root
)
# → 返回 session_id 用于后续监控
```

### 3.2 阻塞监控 — process_manage + sleep

```python
# 启动后 60s 首次健康检查
terminal("sleep 60 && pgrep -f 'claude -p' | wc -l", timeout=90)

# 或主动轮询
tool_call(calls=[{"name": "process_manage", "arguments": {"action": "list"}}])
# → 检查 uptime_seconds 是否在增长
# → 检查 output_preview 是否为空
```

### 3.3 心智收敛 — git log + curl Content-Type

```python
# 检查是否 commit
terminal("cd {repo} && git log --oneline -3", timeout=10)

# 验证图片 Content-Type（重要！只查 HTTP 200 会被 404 首页骗）
terminal(f"""
for url in $(grep -roh 'https://fmode.cn[^"'"'"' )]*\\.png' {repo}/*.html 2>/dev/null | sort -u); do
  CT=$(curl -s -m 10 -o /dev/null -w "%{{content_type}}" "$url")
  [[ "$CT" == image/* ]] || echo "❌ 图裂: $CT $url"
done""", timeout=30)
```

### 3.4 横向协商（可选）— 前一子单元的产物成为后一子单元的输入

```python
# 如果主题一的 commit 包含架构图，主题二需要引用
terminal("cd {repo} && git diff HEAD~1 --name-only", timeout=10)
# 读文件内容作为 inline 素材塞进主题二的任务书
read_file(path="{repo}/path/to/architecture.md")
```

### 3.5 模型配置检查

```python
# 检查 settings.json 确保模型名正确
# 用户钦点：Hermes=deepseek/deepseek-v4.1-flash, CC=deepseek/deepseek-v4.1-flash[1m]
terminal("cat {project_root}/.claude/settings.json", timeout=5)
# 如发现模型名不对，patch 修正
```

---

## 四、派发策略决策树

```
收到复杂任务
│
├→ 可以拆成 2-5 个独立维度？
│   ├→ 否 → 单一 CC 进程处理 或 本 Agent 直接完成
│   └→ 是 → 继续
│
├→ 子单元之间有依赖关系？
│   ├→ 是 → 拓扑排序：先做无依赖的，串行分批
│   │   ├→ 所有独立 → 全并行（最多 5 个同时）
│   │   └→ 部分依赖 → 分批并行：依赖关系跨批次传递
│   └→ 否 → 全并行启动
│
├→ 每个子单元的工作量？
│   ├→ 简单（< 50 行代码/单页）→ 不启 CC，本 Agent 直接写
│   ├→ 中等（需要完整思考/调研/创作）→ 独立 CC 进程
│   └→ 复杂（需要跨文件多模块/项目级）→ 优先用 kanban_create 委派给 fullstack profile
│
├→ 项目目录正确吗？
│   ├→ 每个 CC 必须在项目根目录启动
│   └→ 错目录 → CC 读不到 CLAUDE.md → 规则丢失 → 出次品
│
└→ 启动后：
    ├→ 60s 后检查存活
    ├→ 存活 → 等待 notify → 收敛验收
    └→ 全死 → 执行恢复流程
```

---

## 五、恢复流程（阻塞自愈 — 实操版）

一旦 `process_manage(action="list")` 显示所有进程已退出，按以下顺序处理：

### 5.1 查日志尾巴

```python
terminal(f"tail -30 {log_path}", timeout=10)
```

### 5.2 根据错误分类处理

| 日志症状 | 诊断 | 处置 |
|----------|------|------|
| `command not found: claude` 或 exit 127 | claude 不在 PATH | 用 `/opt/data/npm-global/bin/claude` 绝对路径 |
| `503 No available channel` 或 exit 1 | 模型渠道不可用 | 检查 `~/.claude/settings.json` 模型名（少 `deepseek/` 前缀会 503） |
| `401` 或 `Not logged in` | Token 失效 | 检查 ANTHROPIC_AUTH_TOKEN=$FMODE_API_KEY |
| 日志为空 + exit 0 | 假成功 | 从 `$(cat)` 改 `cat pipe`方式重派 |
| 日志有内容但 exit 1 | 任务执行中出错 | 读日志末尾判断：模型退火 / 文件读写错 / 权限问题 |
| 通知没来、长时间无响应 | 进程阻塞 | `pgrep -f claude` 如果还在跑则等待；否则杀进程重派 |

### 5.3 重派策略

```python
# 1. 如果是"续做"（之前的部分产物可用）
#    在新的任务书开头写明：
#    "已完成：xxx（state） 继续做：yyy（未完成部分）"

# 2. 如果是"从零重做"（完全失败）
#    缩小范围：大任务拆小
#    禁用 CC 的某些能力：机器产出内容需要再确认

# 3. 连败 3 次同域 → 停止 CC 重试
#    本 Agent 直接处理 或 拆成更细的原子指令
```

---

## 六、完整实操流程（以 V6 四主题图文增强为例）

```
Step 1: 认知拆解
Hermes 消化用户需求："四个主题需要图文并茂增强，框架不动，加图"
→ 四主题可完全并行，项目根目录=skill-present

Step 2: 心智分化  
Hermes 为每个主题编写独立任务书（/tmp/cc-v6-0x-resume.md）
每份包含：当前 commit hash + 结构框架 + 纪律 + 验收标准

Step 3: 并行启动
terminal(background=True, notify=True, workdir=skill-present,
  command=f"cat /tmp/cc-v6-01-resume.md | /opt/data/npm-global/bin/claude -p --dangerously-skip-permissions > /tmp/cc-v6-01-r2.log 2>&1")
terminal(background=True, notify=True, workdir=skill-present,
  command=f"cat /tmp/cc-v6-02-resume.md | ...")  # 主题二
terminal(background=True, notify=True, workdir=skill-present,
  command=f"cat /tmp/cc-v6-03-resume.md | ...")  # 主题三
terminal(background=True, notify=True, workdir=skill-present,
  command=f"cat /tmp/cc-v6-04-resume.md | ...")  # 主题四

Step 4: 阻塞自愈（首次 60s 后）
process_manage(action="list") → 检查所有子单元存活

Step 5: 心智收敛（逐个子单元 notify）
每个 notify 到达：
  └→ git log --oneline 检查 commit →
  └→ curl Content-Type 验证图片 →
  └→ 记录产出摘要

Step 6: 统一交付
push 所有 commit → OBS sync → CDN 刷新 → URL 验证 → 发用户
```

---

## 七、常见陷阱（踩坑记录）

### 7.1 任务书喂入方式
- ❌ `claude -p "$(cat task.md)"` → 路径丢失，后台进程找不到文件
- ✅ `cat task.md | claude -p` → 管道直送，无文件路径依赖

### 7.2 项目根目录
- ❌ 在父目录启动 CC → CC 读不到 `.claude/rules/`
- ✅ 必须在 git 项目根目录启动（读得到 CLAUDE.md + rules + settings.json）

### 7.3 图片验证只看 200
- ❌ `curl -o /dev/null -w "%{http_code}"` → 200 可能是 404 首页
- ✅ `curl -o /dev/null -w "%{content_type}"` → 必须是 `image/*`

### 7.4 版本号
- ❌ 跳 1.0.0、5.0.0 大版本
- ✅ 0.0.1 → 0.0.12 → 0.0.99 → 0.1.1 小步迭代

### 7.5 配置 files 字段
- ❌ 不设 `files: ["lib/", "bin/", ...]` → npm publish 把无关文件全打包
- ✅ 精确声明要发布的文件

---

## 八、技能与本 Agent 内置工具的关系

本技能**不替代** Hermes 内置工具，而是**编排**它们：

```
Hermes 收到任务
│
├→ skill-heterarchy（本技能）→ 决策是否启用异层认知
│   └→ 认知分化 → 写任务书
│
├→ terminal（Hermes 工具）→ 后台启动 CC 进程
├→ process_manage → 监控存活
├→ read_file / patch / git → 收敛验证
├→ web_search → 前置调研
└→ delegate_task → 轻量子任务（与 Heterarchy 可组合使用）
```

**何时用 delegate_task 而不是 Heterarchy？**
- 子任务是"帮我查点东西"（轻量检索）→ `delegate_task`
- 子任务是"完整思考 + 产出可交付物" → Heterarchy CC

**何时用 kanban_create 而不是 Heterarchy？**
- 任务需要跨 Profile 的独立 MCP 环境 → `kanban_create`
- 任务在本 Agent 的能力范围内，只是需要并行 → Heterarchy