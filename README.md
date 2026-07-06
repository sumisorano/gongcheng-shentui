# 功成身退

> 功成身退，天之道。——《道德经》

**Claude Code Skill 上下文生命周期管理。让用完的 Skill 体面离开，不赖着占 token。**

> 🔧 **Skill lifecycle management for Claude Code.**  
> Keep your context lean. Retire completed skills, reclaim ~20k tokens.  
> `/ctx-check` · `/ctx-clean` · Skill lifecycle templates  
> [English](README.md) | [中文](README.md)

---

## 解决了什么问题

Claude Code 的 Skill 激活后，SKILL.md 全文会进入对话历史。任务完成后它也不走——每轮都跟着传给大模型，白白浪费 token。

长对话 + 多 Skill 场景下尤其严重：第 5 轮加载 code-review、第 8 轮加载 deploy-check、第 12 轮加载 api-docs……三个 Skill 加起来一两万 token，可能比对话本身还大，而它们早就完成任务了。

**功成身退就是来给 Skill 画句号的。**

---

## 设计哲学

Anthropic 为 Skills 设计了三层 **Progressive Disclosure（渐进式披露）**：

| 层     | 内容                              | 加载时机         |
| ------ | --------------------------------- | ---------------- |
| 第一层 | frontmatter（name + description） | 始终扫描         |
| 第二层 | SKILL.md 正文                     | 触发时加载       |
| 第三层 | references / scripts              | 正文引用时加载   |

三层一层比一层重，一层比一层懒。这是 Skill **"怎么进来"** 的设计。

**功成身退是第四层——"怎么出去"：**

| 层     | 内容                            | 回收时机                             |
| ------ | ------------------------------- | ------------------------------------ |
| 第四层 | 已完成任务的 SKILL.md 正文      | ctx-clean 回收全文，只留一行摘要      |

进来是渐进的，出去也是渐进的。**完整的 Skill 生命周期 = 渐进式披露 × 功成身退。**

---

## 快速开始

### 安装

把两个命令文件拷到你的 Claude Code 项目里：

```bash
cp -r .claude/commands/ 你的项目/.claude/commands/
```

### 使用

```bash
# 第一步：诊断
/ctx-check
```

根据诊断结果：

- 🟢 "上下文很清爽" → 不用管，继续干活
- 🔴 列出了一堆已完成 Skill → 继续第二步：

```bash
# 第二步：回收
/ctx-clean
```

### 让 Skill 声明自己的生命周期

用 `lifecycle-template/SKILL.md` 模板写新 Skill，加上三个字段：

```yaml
lifecycle:
  type: one-shot                # 一次性任务 / session 常驻
  completion_signal: "..."      # 什么标志表示任务完成了
  retention_summary: "..."      # 回收后保留的一句话摘要
```

详细用法见 [`lifecycle-template/GUIDE.md`](lifecycle-template/GUIDE.md)。

---

## 工作原理

| 命令         | 本质       | 做什么                                                          |
| ------------ | ---------- | --------------------------------------------------------------- |
| `/ctx-check` | 上下文诊断 | 让 Claude 分析自己的对话历史，识别已加载 Skill 和任务状态       |
| `/ctx-clean` | 智能回收   | 对 `/compact` 的封装——带着保留清单压缩，优先回收已完成 Skill    |

两个命令都是纯提示词，不依赖外部工具，不改 Claude Code 源码。

### 前置轻量判断

`/ctx-check` 不会每次都做完整诊断。对话 < 10 轮 + Skill < 2 个时直接跳过——"上下文还很清爽，暂不需要功成身退。"

### 安全原则

- 🛡️ 当前活跃的 Skill 绝不回收
- 🛡️ 不确定是否已完成的，宁可保留
- 🛡️ 回收的是内容不是记忆——已回收的 Skill 留一行摘要

---

## 实战策略

光有工具不够，还要知道**什么时候该做什么**。

[`docs/STRATEGY.md`](docs/STRATEGY.md) 是一份实操指南，回答三个问题：

| 问题                           | 答案                         |
| ------------------------------ | ---------------------------- |
| 什么时候跑 `/ctx-check`？      | 加载 **≥ 3 个 Skill** 时     |
| 什么时候跑 `/ctx-clean`？      | `/ctx-check` 建议回收时      |
| 几十轮后还慢怎么办？           | 新会话 + 上下文摘要          |

从第 0 层（预防）到第 4 层（环境优化），分层递进，拿着就能用。

---

## 项目结构

```
gongcheng-shentui/
├── README.md                         # 本文档（快速开始）
├── CHANGELOG.md                      # 版本记录
├── DESIGN.md                         # 设计文档（为什么这么设计）
├── CONTRIBUTING.md                   # 贡献指南
├── CODE_OF_CONDUCT.md                # 行为准则
├── SECURITY.md                       # 安全策略
├── LICENSE                           # GPL-3.0
├── docs/
│   └── STRATEGY.md                   # 实战策略（什么时候该做什么）
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md             # Bug 报告模板
│   │   └── feature_request.md        # 功能建议模板
│   └── PULL_REQUEST_TEMPLATE.md      # PR 模板
├── .claude/
│   └── commands/
│       ├── ctx-check.md              # /ctx-check 命令
│       └── ctx-clean.md              # /ctx-clean 命令
├── test-reports/
│   └── 功成身退-压力测试报告.md       # 实测数据（5 技能压力测试）
└── lifecycle-template/
    ├── SKILL.md                      # 生命周期模板（复制即用）
    ├── GUIDE.md                      # 模板使用指南
    └── examples/
        └── code-review.md            # 填写示例
```

---

## 与同类项目的差异

| 维度     | Dynamic Context Pruning | 功成身退                     |
| -------- | ----------------------- | ---------------------------- |
| 关注点   | 通用上下文剪枝           | **Skill 生命周期管理**       |
| 清理策略 | 去重、覆盖清理、错误清理  | **基于任务是否完成判断**     |
| 设计哲学 | 上下文优化               | **用时有，用完走的生命周期哲学** |

---

## 路线图

- [x] 产品定位 & 命名
- [x] `/ctx-check` + `/ctx-clean` 命令
- [x] Skill 生命周期模板 + 使用指南
- [x] 实测验证 — 5 技能压力测试通过 ✅（释放 ~20k tokens）
- [ ] 录制演示 GIF

---

## 许可

GPL-3.0

---

> 功成身退，天之道。Skill 亦然。
