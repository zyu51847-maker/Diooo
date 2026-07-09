# Auto-Research Plugin

让 Claude Code 在遇到不确定问题时，自动通过 ChatGPT（Playwright 浏览器）或搜索引擎调研，整理方案后等你裁决再动手。

## 触发词

| 输入 | 行为 |
|------|------|
| `?` | 确认去问 ChatGPT |
| `??` | 搜索引擎调研 |

## 前置条件

### 1. 安装 Playwright 浏览器扩展

在 Edge 中打开以下链接，点击 **"添加到 Edge"**：

```
https://chromewebstore.google.com/detail/playwright-extension/mmlmfjhmonkocbjadbfplnigmagldckm
```

### 2. 安装本插件

```bash
# 从本地目录安装
claude plugins install ./auto-research-plugin

# 或从 GitHub 安装（发布后）
claude plugins install auto-research@your-marketplace
```

### 3. 批准 MCP Server

首次使用时 Claude Code 会提示批准 Playwright MCP server，批准一次后全局生效。

## 工作流示例

```
用户: PyTorch 中怎么自定义 ADMM 优化器？方案 A 还是 B？
Claude: 建议问一下 GPT 对比，输入 ? 确认
用户: ?
Claude: [自动打开 ChatGPT → 输入问题 → 抓取回答 → 呈现方案]
Claude: GPT 建议方案 A，原因是...。要继续实现吗？
用户: 行
Claude: [开始写代码]
```

## 工作原理

- **Playwright MCP** 通过浏览器扩展直达用户 Edge，自动携带登录态
- ChatGPT 页面通过 ProseMirror 编辑器交互
- 回答自动提取呈现，用户全程可见浏览器操作

## 许可

MIT
