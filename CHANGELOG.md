# Changelog

## v1.2 (2026-07-06)

### 改进
- **门控优化**：`/ctx-check` 门控从"对话 >20轮 或 Skill >3个"改为 **Skill ≥3个直接诊断**，不等变慢
- **续接提醒**：新增 `-c` 续接模式提示——显式用户消息少但实际轮次可能多
- **生命周期声明优先**：已声明 `lifecycle.completion_signal` 的 Skill 优先以声明为准，不再纯靠 AI 推断

### 文档
- **STRATEGY.md** — 新增！实战分层策略（0~4 层操作指南）
- **README** — 更新项目结构，新增策略入口
- **测试报告** — 收入仓库

### 修复
- 表格对齐修复
- 测试报告脱敏（移除服务器 IP）

## v1.1 (2026-07-06)

### 改进
- `ctx-check.md` 第〇步门控增加 **Skill 加载数**判断维度
- `ctx-clean.md` 明确要求先执行 `/ctx-check`

### 文档
- README 许可从 `GPL-3.0` 改为完整名称 `GNU General Public License v3.0`

## v1.0 (2026-07-04)

### 初始版本
- `/ctx-check` — 上下文诊断命令
- `/ctx-clean` — 智能回收命令
- `lifecycle-template/` — Skill 生命周期模板 + 指南 + 示例
- README、DESIGN 文档
