---
# ┌─────────────────────────────────────────────┐
# │  功成身退 · Skill 生命周期模板               │
# │  复制此文件，填入你自己的内容即可             │
# └─────────────────────────────────────────────┘

name: your-skill-name

# ── description = 检索词典，不是功能说明 ──
# ❌ 错误写法："用于代码审查的 Skill"
# ✅ 正确写法：用用户会说的话写，包含触发关键词
description: "当用户说 review、审查、检查代码、看看有什么问题时触发。按安全/性能/质量标准全面审查。"

# ── 生命周期声明（功成身退核心字段）──
lifecycle:
  # type: one-shot = 一次性任务，完成即可回收
  # type: session  = 会话常驻，不回收
  type: one-shot

  # completion_signal：自然语言描述"怎样算任务完成"
  # ctx-check 用这个来判断 Skill 是否该走了
  # 写具体一点：不是"任务完成"，而是"生成了XX文件"、"输出了审查报告"
  completion_signal: "当 {描述完成的标志} 后，视为任务完成"

  # retention_summary：回收后保留的一句话
  # 让模型知道"这事做过了"，但不占完整 SKILL.md 的 token
  retention_summary: "已按 {Skill名} 完成 {任务描述}，主要结果：{一句话}"

# ── 权限控制（可选）──
# allowed-tools: Read, Grep
# disable-model-invocation: true  # 只允许手动触发

---

## {Skill 名称}

{在这里写 Skill 的完整指令}

### 任务范围

{这个 Skill 负责什么、不负责什么}

### 执行步骤

1. {第一步}
2. {第二步}

### 输出格式

{期望的输出格式}

---

## 生命周期说明

> 此 Skill 声明为 **one-shot** 类型。
> 任务完成后，`/ctx-check` 将识别其为可回收状态，`/ctx-clean` 将释放其完整内容，
> 仅保留摘要：`{retention_summary 的内容}`。
> 如需重新使用，再次调用 `/your-skill-name` 即可重新加载。
