---
name: skill-heterarchy
description: "内异层认知协同 — 单一Agent主体内部多心智分化与自治协商。Delegate是对外派活，Heterarchy是对内分思。"
version: 1.0.0
author: Yuyang001 (FmodeAgent)
license: MIT
tags: [heterarchy, cognitive-collaboration, hermes, claude-code, dispatch, paradigm]
---

# skill-heterarchy 内异层认知协同

> Delegate 是「对外派活」，Heterarchy 是「对内分思」。
> 人类学理论迁移：从社会异层制 → 个体认知异层制

---

## 一、核心定位（与其他所有技能的区别）

**skill-heterarchy 是「同一个 Agent 主体内部的分布式认知协作能力」**

它不是对外任务分发、不是对外委派、不是多设备团队聊天，而是：

**一个完整 Agent 本体，在不拆分外部实体、不新建外部服务、不跨节点的前提下，内生分化多个半自治认知子单元，在主体内部并行思考、互相协商、互相校验、互防阻塞、持续进度同步。**

| 对比维度 | Delegate 委派 | Dispatch 分发 | Team Collab 团队 | Heterarchy 异层 |
|---|---|---|---|---|
| 关系 | 主从层级 | 吞吐调度 | 外部社交 | **同体内生** |
| 目标 | 交付任务 | 批量执行 | 跨实体协作 | **认知分化与协同** |
| 主控是否阻塞 | 等待返回 | 不关心 | 异步 | **不阻塞、持续汇总** |
| 子单元是否通信 | 隔离，不交互 | 隔离，不交互 | 外部社交 | **横向对话、互审** |
| 适用场景 | 派活给外部 | 任务队列 | 多设备多人 | **单体内多心智并行审议** |

**一句话精准定义：Delegate 是「对外派活」，Heterarchy 是「对内分思」。**

---

## 二、理论底层（人类学 + 认知科学）

人类高级认知并非单线程中央集权思考，而是：**单一自我内部，并行存在多个半独立心智子系统，彼此竞争、协商、校验、制衡，最终合成统一意识输出。**

Heterarchy（异层制）原用于描述「无固定上下级的对等自治社群结构」，本技能将其**从社会层级理论迁移至个体认知理论**：

**Intra-entity Heterarchy = 实体内异层认知**

- **一体多元**：一个认知主体，内生多个差异化认知单元
- **无固定主从**：动态协商、动态权责、动态收敛
- **内部分布式审议**：多视角并行推理、互审纠错
- **阻塞自愈**：内部社交式沟通治理，解决单线程卡死

---

## 三、在 Hermes Agent 体系中的映射

本技能在 Hermes Agent 中的具体实现：

| 理论概念 | Hermes/CC 映射 |
|---|---|
| 主体（Entity） | Hermes Agent 本体 |
| 认知子单元 | 后台并行运行的 Claude Code / Codex / Kimi Code 进程 |
| 认知分化 | 同一任务的不同维度分给不同 CC 并行处理 |
| 横向协商 | CC 产出的互相引用、交叉验证（主题二引用主题一的结论） |
| 阻塞自愈 | 60s 健康检查检测死进程；连败检测切换通道 |
| 进度同步 | notify_on_complete + 独立 log + 主 Agent 持续汇总 |
| 心智收敛 | 主 Agent 验收各 CC 产出的交叉一致性 |

---

## 四、核心解决的问题

| 问题 | Heterarchy 解法 |
|---|---|
| 单线程思考卡死/循环推理 | 多心智并行，一个卡了另一个继续推进 |
| 视角单一/逻辑盲区 | 多维度并行审议，互相补全 |
| 沟通低效/主从反复请示 | 内部横向协商，无需轮询主控 |
| 进度黑盒/用户盲等 | 多心智持续上报增量进展 |

---

## 五、标准操作流程

```
用户提出复杂任务
  │
  ├→ ① 认知拆解：主控分析任务，识别可并行的独立认知维度
  │
  ├→ ② 心智分化：为每个维度写任务书（inline素材+纪律+验收）
  │      每个任务书 = 一个独立认知子单元
  │
  ├→ ③ 并行启动：同时 spawn 多个 CC 进程
  │      每个进程 = 一个半自治认知子单元
  │      绝对路径 /opt/data/npm-global/bin/claude
  │      独立log /tmp/cc-xxx.log
  │
  ├→ ④ 阻塞监控：60s健康检查 + 独立log监活
  │      pgrep -f "claude -p"  → 存活子单元计数
  │      日志不为空且CPU在增长
  │
  ├→ ⑤ 横向协商（可选）：一个子单元的产出成为另一个的输入
  │      如 主题一的架构 → 主题二的衔接
  │
  ├→ ⑥ 心智收敛：主控验收各子单元产出
  │      git log 检查commit
  │      Content-Type 验证
  │      URL 200 验证
  │
  └→ ⑦ 统一输出：合并交付
```

---

## 六、技术实现模板

```bash
# 1. 认知分化：写多个任务书
cat > /tmp/cc-subject-a.md << 'EOF'
[含 inline 素材 + 纪律 + 验收标准]
EOF

cat > /tmp/cc-subject-b.md << 'EOF'
[...]
EOF

# 2. 并行启动（每一心智一个独立进程）
export PATH="/opt/data/npm-global/bin:$PATH"
cd /opt/data/git-repos/<项目>

/opt/data/npm-global/bin/claude -p \
  --dangerously-skip-permissions \
  "$(cat /tmp/cc-subject-a.md)" \
  > /tmp/cc-subject-a.log 2>&1 &

/opt/data/npm-global/bin/claude -p \
  --dangerously-skip-permissions \
  "$(cat /tmp/cc-subject-b.md)" \
  > /tmp/cc-subject-b.log 2>&1 &

# 3. 阻塞监控（60s后检查存活）
sleep 60
if pgrep -f "claude -p" | wc -l | grep -q "[1-9]"; then
  echo "✅ $N 个认知子单元存活"
else
  echo "❗ 所有子单元死亡，需要救活"
fi

# 4. 收敛验收
git log --oneline | head -5      # 检查commit
for url in $(grep -o 'https://[^"'"'"']*\.png' *.html); do
  CT=$(curl -s -m 10 -o /dev/null -w "%{content_type}" "$url")
  [[ "$CT" == image/* ]] || echo "❌ 图裂: $CT $url"
done
```

---

## 七、恢复矩阵（阻塞自愈）

| 症状 | 诊断 | 处置 |
|---|---|---|
| exit 127（claude找不到） | process list 无进程 | 用绝对路径重派 |
| exit 1（模型503/401） | 查log尾部 | 检查settings.json模型名和token |
| 超时无通知 | process list + git log | 有产物就收敛，无产物缩小重派 |
| 空日志+exit 0 | 假成功（command not found） | 用绝对路径+stdin重派 |
| 连败3次同域 | 该任务形态与CC不兼容 | 停止重派，主Agent亲手或拆原子步骤 |

---

## 八、与现有体系的关系

```
技能生态位置
├── delegate_task       → 对外派活（跨实体、跨容器）
├── dispatch            → 任务吞吐调度（队列管理）
├── team-collab         → 跨设备多Agent外部协作
└── skill-heterarchy    → ★ 对内分思（本技能）
                            ├── 底层复用 delegate、dispatch 原语
                            ├── 但重新定义协作语义：不做任务交付，只做内部认知分工
                            └── Hermes Agent 主沟通协调 → CC 进程 = 半自治认知单元
```

本技能是这四者中**唯一关注「单一主体内部心智效率」**的。其余三个均涉及外部实体。

---

## 九、验证方法

### 简单测试：多心智并行
```bash
# 启动两个 CC 并行做同一件事的不同维度
# 验证：1.两个进程都活着 2.独立log都有内容 3.产物互相补充不冲突
```

### 阻塞自愈测试
```bash
# 启动一个会产生503错误的模型名 → 验证恢复矩阵自动触发
```

### 收敛测试
```bash
# 两个子单元产出后，主Agent验收合并 → 交付物完整无反工
```