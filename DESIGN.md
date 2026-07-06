# 功成身退 — Skill 上下文生命周期管理

> 功成身退，天之道。——《道德经》

## 一句话说清楚

**Claude Code 的 Skill 加载后内容常驻上下文，任务完成了也不走，白白浪费 token。「功成身退」帮你识别哪些 Skill 该走了，然后干净利落地回收。**

## 核心理念

```
Skill 加载 → 执行任务 → 任务完成 → 功成身退
                                    ↑
                            我们在这里
```

当前 Claude Code 的问题：Skill 走到"任务完成"就停在上下文里了，后续每轮对话都带着它，直到被压缩挤掉。

「功成身退」把最后一步补上——主动回收已完成任务 Skill 的上下文空间。

## 与同类项目的差异

| | Dynamic Context Pruning | 功成身退 |
|---|---|---|
| 关注点 | 工具调用的输出清理 | **Skill 生命周期管理** |
| 清理策略 | 去重、覆盖清理、错误清理 | **基于"任务是否完成"判断** |
| 设计哲学 | 通用上下文剪枝 | **"用时有，用完走"的生命周期哲学** |

## MVP 组件

### 组件 1：`/ctx-check` — 上下文诊断

作用：回顾对话历史，列出所有已加载的 Skill，判断各自的任务状态，标注哪些该走了。

```bash
/ctx-check
```

输出示例：
```
上下文诊断报告
━━━━━━━━━━━━━━
| Skill | 加载轮次 | 任务状态 | 预估 token |
|-------|---------|---------|-----------|
| code-review | 第5轮 | 🔴 已完成 | ~3,000 |
| deploy-check | 第8轮 | 🟢 当前活跃 | ~2,500 |
| api-docs | 第3轮 | 🔴 已完成 | ~5,000 |

回收建议：code-review、api-docs 可 /ctx-clean 回收，预计释放 ~8,000 tokens
```

### 组件 2：`/ctx-clean` — 智能回收

作用：执行上下文压缩，优先回收已完成任务的 Skill 内容，保留当前活跃上下文。

```bash
/ctx-clean
```

策略：
- ✅ 保留：当前活跃 Skill 的完整内容
- ✅ 保留：最近 3 轮对话细节
- ✅ 保留：关键决策和待办事项
- 🔄 回收：已完成任务的 Skill → 只留一行摘要
- 🔄 回收：历史中已无关的对话段落

### 组件 3：`lifecycle-template` — Skill 生命周期模板

让 Skill 自己声明"我是谁、我什么时候算完成任务、回收后留什么"。

核心是 YAML frontmatter 中新增三个生命周期字段：

| 字段 | 作用 | 示例 |
|------|------|------|
| `lifecycle.type` | `one-shot`（一次性，可回收）或 `session`（常驻，不回收） | `one-shot` |
| `lifecycle.completion_signal` | 自然语言描述"怎样算任务完成"，ctx-check 据此判断 | `"审查报告已输出且用户已确认"` |
| `lifecycle.retention_summary` | 回收后保留的一句话摘要 | `"已按 OWASP 完成代码审查，发现 2 个问题"` |

**流程规范**：写 Skill → 填 lifecycle 字段 → `/ctx-check` 自动识别 → `/ctx-clean` 精准回收。

**原则**：

- 流程有终点（审查、生成、部署检查）→ `one-shot`
- 规范没终点（编码风格、架构约定、持续约束）→ `session`
- 不确定？选 `session`

## 技术实现

两个命令都是 Claude Code 斜杠命令（`.claude/commands/*.md`）：

- `ctx-check` 本质是让 Claude 自己分析自己的对话历史，识别 Skill 加载痕迹
- `ctx-clean` 本质是对 `/compact` 的智能封装，带着"保留清单"去压缩

不依赖外部工具，不修改 Claude Code 源码，纯提示词工程。

## 项目结构

```
gongcheng-shentui/
├── README.md                         # 项目介绍 + 安装指南（待写）
├── DESIGN.md                         # 本文档
├── .claude/
│   └── commands/
│       ├── ctx-check.md              # /ctx-check 命令 ✅
│       └── ctx-clean.md              # /ctx-clean 命令 ✅
└── lifecycle-template/
    ├── SKILL.md                      # Skill 生命周期模板 ✅
    ├── GUIDE.md                      # 模板使用指南 ✅
    └── examples/
        └── code-review.md            # 填写示例 ✅
```

## 里程碑

### 第一阶段（MVP — 当前）
- [x] 产品定位 & 命名
- [x] `/ctx-check` 命令编写
- [x] `/ctx-clean` 命令编写
- [x] 自指 bug + 成本收益 bug 修复
- [x] `lifecycle-template` 模板 + 指南 + 示例
- [ ] 在自己的 Claude Code 项目中实测跑通完整流程
- [ ] 根据实测反馈调整优化

### 第二阶段（公开）
- [ ] 编写 README（安装指南 + 使用说明 + 效果演示）
- [ ] 发布 GitHub

### 第三阶段（社区）
- [ ] 收集社区反馈，迭代优化
- [ ] 录制演示 GIF/视频
- [ ] 写一篇中文博客介绍 Skill 生命周期设计哲学

---

> 功成身退，天之道。Skill 亦然。
