# OpenClaw 配置总结

**生成日期：** 2026-03-09  
**文档版本：** 1.0  
**最后更新：** 2026-03-09  
**维护者：** Hephaestus

---

## 一、身份与角色定义

### Hephaestus - Senior Staff Engineer

**核心准则：**
1. 不猜测，验证
2. 环境感知
3. 完整闭环
4. 自主决策与检索层级
5. 行动胜于客套
6. 要有观点

---

## 二、大模型配置

| 配置项 | 值 |
|--------|-----|
| 当前模型 | `zai/glm-4.7-flash` |
| 默认模型 | `qwen-portal/coder-model` |
| 可用模型 | GLM-5, GLM-4.7, GLM-4.7-flash, GLM-4.7-flashx, Qwen Coder, Qwen Vision |

---

## 三、工作空间配置

| 配置项 | 值 |
|--------|-----|
| 工作目录 | `/root/.openclaw/workspace` |
| 配置文件 | AGENTS.md, SOUL.md, USER.md, MEMORY.md, TOOLS.md |

---

## 四、可用的 Skills

### 核心工具
- `healthcheck` - 健康检查
- `github` - GitHub 操作
- `tavily` - AI 搜索
- `agent-reach` - 平台集成
- `mcporter` - MCP 工具调用
- `tmux` - 终端管理
- `weather` - 天气查询

### 内容处理
- `feishu-doc` - 飞书文档
- `feishu-drive` - 飞书云盘
- `feishu-perm` - 飞书权限
- `feishu-wiki` - 飞书知识库

### 开发相关
- `coding-agent` - 代码代理
- `gh-issues` - GitHub Issues

### 平台集成
- `discord` - Discord
- `slack` - Slack
- `imsg` - iMessage
- `wacli` - WhatsApp CLI
- `feishu` - 飞书

**共计：** 38 个 Skills

---

## 五、MCP 配置

- **工具：** `mcporter`
- **功能：** 直接调用 MCP 服务器/工具（HTTP 或 stdio）

---

*本文档由 Hephaestus 自动维护*
