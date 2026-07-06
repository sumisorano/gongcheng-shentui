---
name: code-review
description: "当用户说 review、审查代码、帮我看看这个文件、检查 PR、有没有安全问题、代码质量怎么样时触发。按 OWASP Top 10 + 性能 + 代码质量三维度全面审查。"

lifecycle:
  type: one-shot
  completion_signal: "审查报告已输出（含 Critical/Major/Minor 分级问题清单），且用户已确认收到或已开始修改"
  retention_summary: "已按 OWASP Top 10 + 团队规范完成代码审查，发现 N 个问题（其中 Critical: X, Major: Y, Minor: Z），已在后续对话中修复"

allowed-tools: Read, Grep, Glob
---

## 代码审查

按以下标准审查 $ARGUMENTS 指定的代码（未指定则审查当前修改的文件）：

### 安全检查（OWASP Top 10）

1. SQL 注入、XSS、CSRF 等常见漏洞
2. 密钥和敏感信息是否硬编码
3. 权限验证是否完整
4. 文件上传是否有安全校验

### 性能检查

1. 是否有 N+1 查询模式
2. 循环中是否有不必要的 IO 操作
3. 大数据量场景的内存占用

### 代码质量

1. 函数是否过长（超过 50 行建议拆分）
2. 变量命名是否清晰
3. 是否有重复代码可提取
4. 错误处理是否完整

### 输出格式

按严重程度排列：

```
🚨 Critical（阻断合并）
- 问题描述 | 位置 | 修复建议

⚠️ Warning（应该修复）
- 问题描述 | 位置 | 修复建议

💡 Suggestion（锦上添花）
- 问题描述 | 位置 | 修复建议
```

---

> 此 Skill 声明为 one-shot。审查报告输出后即可被功成身退回收。
> 如需再次审查，重新调用 `/code-review` 即可。
