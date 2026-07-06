# 安全策略

## 报告安全漏洞

功成身退是纯提示词工具，**不执行代码，不联网，不访问文件系统**。因此安全风险极低。

但在以下情况请及时报告：

- `/ctx-check` 或 `/ctx-clean` 有意外行为（如错误地回收了活跃 Skill 的内容）
- 命令提示词中包含误导性或有害指令
- 其他你认为构成安全问题的任何情况

### 报告方式

请直接在 GitHub 上提交 [Issue](https://github.com/sumisorano/ctx-lifecycle/issues)。

如果是敏感安全问题（不适合公开讨论），可以在 Issue 中注明 `[SECURITY]`，我们会优先处理。
