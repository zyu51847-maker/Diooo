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
| `?` | **反馈给 GPT** — 把当前对话上下文（含 Claude 上一条输出）自动拿去问 ChatGPT，获取反馈/验证/改进建议 |
| `? + 问题` | **直接问 GPT** — 把问题拿去问 ChatGPT，抓回答整理方案，等用户说"行"再写代码 |
| `??` | **根据上下文搜索** — 自动提取当前讨论中的问题，用搜索引擎调研 |
| `?? + 问题` | **指定搜索** — 用指定问题搜索引擎调研，整理方案后呈现 |

### 工作流

**`? + 问题` 直接问 GPT：**
```
用户: ? PyTorch 中自定义 ADMM 优化器怎么设计
Claude: [自动打开 ChatGPT → 输入问题 → 抓取回答 → 整理方案]
Claude: GPT 建议方案 A，原因是 X Y Z。要继续实现吗？
用户: 行
Claude: [开始写代码]
```

**`?` 单独使用（反馈模式）：**
```
Claude: [输出了某个方案/代码]
用户: ?
Claude: [把上面的方案拿去问 GPT 验证 → 呈现反馈]
```

**`??` 根据上下文搜索：**
```
Claude: [讨论中涉及 ADMM 收敛条件]
用户: ??
Claude: [自动提取"ADMM 收敛条件" → WebSearch → 整理方案 → 呈现]
```

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

### 性能优化

| 优化 | 做法 | 效果 |
|------|------|------|
| **JS 注入文字** | `page.locator('#prompt-textarea').fill(text)` 替代 `keyboard.type()` | ~15s → ~0.1s |
| **复用标签页** | 首次导航后保持页面，后续 `?` 直接复用已加载的 SPA | 节省 ~3-5s |

### 交互 API

```javascript
// 填入文字（瞬间注入，比 keyboard.type() 快 100 倍）
await page.locator('div#prompt-textarea').fill(prompt);

// 点击发送
await page.locator('[data-testid="send-button"]').click();

// 等待回复（观察 DOM 变化或固定等待）
await page.waitForSelector('[data-message-author-role="assistant"]');

// 提取最后一条回复
const response = await page.evaluate(() => {
  const turns = document.querySelectorAll('[data-message-author-role="assistant"]');
  return turns[turns.length - 1]?.textContent;
});
```

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
