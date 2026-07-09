---
name: auto-research
description: Use when implementing tasks where design choices, algorithm details, API usage, or best practices are uncertain — automatically research via ChatGPT (Playwright) or web search, present options, and wait for user approval before coding
---

# 自动研究 + 用户决策流程

## 核心规则

遇到不确定的问题时，先自己研究，再让用户拍板：

1. **自动搜索** — 使用 Playwright（ChatGPT）或 WebSearch/WebFetch 调研
2. **整理呈现** — 提炼为简洁方案建议（含来源），展示给用户
3. **等待裁决** — 用户说"行"后，才开始写代码

## 触发词

| 输入 | 行为 |
|------|------|
| `?` | **一键反馈给 GPT** — 把当前对话上下文（含 Claude 上一条输出）自动拿去问 ChatGPT，获取反馈/验证/改进建议 |
| `??` | 搜索引擎调研 — WebSearch/WebFetch 查资料 |

### `?` 的上下文拼接规则

用户输入 `?` 时，将以下内容拼接成 ChatGPT prompt：

1. 用户最近一条有意义的问题/指令
2. Claude 最近一条输出（摘要 + 关键结论）
3. 追问："请评估上述方案/输出的正确性、完整性和可改进之处"

然后打开 ChatGPT → 输入 → 抓回答 → 呈现。

## Playwright 浏览器自动化流程

配置：`@playwright/mcp@latest --extension --browser msedge`，通过浏览器扩展连接用户 Edge。

1. Edge 保持运行，Playwright 通过扩展直连，自动携带登录态和 Cookie
2. 导航、输入、抓取内容 — 用户可在 Edge 窗口实时看到
3. 完毕后整理回答呈现，等用户裁决

ChatGPT 输入框是 ProseMirror `div#prompt-textarea[contenteditable]`，用 `page.keyboard.type()` 填入文字，`data-testid="send-button"` 提交。

## 什么算"不确定"

- 多个可行方案，各有取舍
- 算法/API 细节记不清
- 不知道某个库的用法
- 架构/设计层面的选择

## 什么不算

- 语法拼写、简单 API 调用（直接写）
- 用户已明确指定做法
- 纯粹的单行修改

## 原则

**宁可多搜一次，不要瞎写一段。用户拍板后再动手。**
