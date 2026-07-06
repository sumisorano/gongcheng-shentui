# 参与贡献

感谢你考虑为功成身退做贡献！🎉

## 项目定位

功成身退是 **Claude Code Skill 上下文生命周期管理工具**。核心是两个命令文件（`.claude/commands/`）外加一套模板。

**重要：** 功成身退不依赖外部工具、不改 Claude Code 源码、纯提示词工程。任何贡献都应保持这一原则。

## 贡献方式

### 🐛 报告 Bug

1. 先搜一下 [Issues](https://github.com/sumisorano/ctx-lifecycle/issues) 有没有人报过
2. 如果没找到，用 [Bug Report 模板](https://github.com/sumisorano/ctx-lifecycle/issues/new?template=bug_report.md) 提交
3. 描述清楚：
   - Claude Code 版本（`claude --version`）
   - 你的对话场景（加载了几个 Skill、多少轮）
   - 实际输出 vs 预期输出
   - 如果可以，贴一张截图

### 💡 提功能建议

用 [Feature Request 模板](https://github.com/sumisorano/ctx-lifecycle/issues/new?template=feature_request.md) 提交。

特别感兴趣的方向：
- 跨平台适配方案（Hermes / OpenClaw 等）
- 新的 Skill 生命周期类型
- 更好的诊断报告格式
- 演示 GIF / 录屏

### 📝 改进文档

文档和代码一样重要。如果你发现 README / DESIGN / STRATEGY 有不清楚的地方，直接提 PR。

### 🔧 提交代码

1. Fork 本仓库
2. 创建功能分支：`git checkout -b feat/my-feature`
3. 提交改动
4. 确保 .claude/commands/ 中的命令文件在 Claude Code 中能正常运行
5. 推送到你的 Fork：`git push origin feat/my-feature`
6. 提交 Pull Request

## 开发指南

### 本地调试

```bash
# 1. 把命令文件拷到你的 Claude Code 项目里
cp -r .claude/commands/ 你的项目/.claude/commands/

# 2. 加载几个 Skill，跑几轮对话
claude

# 3. 运行命令
/ctx-check
/ctx-clean
```

### 修改命令

两个命令文件在 `.claude/commands/` 目录下：

- `ctx-check.md` — 上下文诊断命令
- `ctx-clean.md` — 智能回收命令

修改后，在同目录下用 Claude Code 实测验证。

### 模板修改

`lifecycle-template/` 目录下的模板和指南用于指导 Skill 作者声明生命周期。修改时注意保持示例代码和执行逻辑的一致性。

## 代码风格

- Markdown 表格保持列对齐（`| --- | --- |` 格式）
- 中文和英文/代码之间不加空格（"GitHub 上" 而不是 "GitHub 上"）
- 命令用法示例用 `bash` 代码块
- 输入/输出示例用 `text` 代码块

## 许可

提交代码即表示你同意你的贡献在 [GPL-3.0](LICENSE) 许可下发布。

---

再次感谢！🎉
