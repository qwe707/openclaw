# ARCHITECTURE.md - OpenClaw 运行架构

这是 xuan 当前 OpenClaw 的正式运行架构。

目标不是画概念图，而是约束后续配置、执行、排障与扩展方式。

## 一、6 层主架构（System Architecture）

### 1. Channel 接入层
负责外部消息进入 OpenClaw。

当前已接入：
- Feishu DM

对应配置：
- `channels.feishu.enabled = true`
- `session.dmScope = per-channel-peer`

职责：
- 接收用户消息
- 维持会话边界
- 绑定 reply / direct chat 上下文

---

### 2. Gateway / Transport 层
负责本地网关、连接控制与节点访问边界。

当前配置：
- `gateway.mode = local`
- `gateway.bind = loopback`
- `gateway.auth.mode = token`
- `gateway.tailscale.mode = off`

职责：
- 本机网关暴露
- token 鉴权
- 节点能力限制
- 作为 channels / nodes / browser 等能力的底层入口

当前安全边界：
- 已禁用高风险 node 命令：
  - `camera.snap`
  - `camera.clip`
  - `screen.record`
  - `calendar.add`
  - `contacts.add`
  - `reminders.add`

---

### 3. Agent Orchestration 层
负责会话策略、子代理并发、命令调用方式与执行编排。

当前配置：
- `agents.defaults.model.primary = gpt/gpt-5.4`
- `agents.defaults.maxConcurrent = 4`
- `agents.defaults.subagents.maxConcurrent = 8`
- `commands.native = auto`
- `commands.nativeSkills = auto`
- `commands.restart = true`

职责：
- 主代理默认模型选择
- 子代理扩展与 ACP / subagent 编排
- 技能调用调度
- 命令执行策略

默认约定：
- 主模型优先使用 `gpt/gpt-5.4`
- 长任务优先使用 sub-agent / ACP
- 能用一等工具时，不绕 CLI

---

### 4. Model Provider 层
负责模型供应商、路由别名与推理能力。

当前 provider：
- `gpt`
- `zai`
- `qwen-portal`
- `github-copilot`（以 auth profile 形式存在）

当前默认模型：
- `gpt/gpt-5.4`

当前别名：
- `GPT-5.4`
- `GLM`
- `qwen`

职责：
- 统一模型入口
- 多 provider 兼容
- alias 映射
- 推理能力与上下文窗口控制

架构决策：
- 日常主路径统一走 `gpt/gpt-5.4`
- `zai/glm-5` 作为高推理备选
- `qwen-portal/coder-model` 作为替代编码模型

---

### 5. Memory / Knowledge 层
负责长期记忆、短期日志、用户画像与运行约束。

当前文件：
- `SOUL.md`
- `USER.md`
- `MEMORY.md`
- `memory/YYYY-MM-DD.md`
- `TOOLS.md`
- `ARCHITECTURE.md`
- `AGENTS.md`

职责：
- 人设与工作风格
- 用户偏好
- 长短期记忆分层
- 本地操作约束
- 经验沉淀

运行规则：
- 主会话必须读取 `SOUL.md` / `USER.md` / `MEMORY.md`
- 当任务涉及历史决策、偏好、待办，先查 memory
- 重要决定写入 `MEMORY.md`
- 架构变更写入 `ARCHITECTURE.md`

---

### 6. Skills / Tools Execution 层
负责把意图变成真实动作。

当前主要执行能力：
- 文件：`read` / `write` / `edit`
- 系统：`exec` / `process`
- 网页：`web_search` / `web_fetch` / `browser`
- 消息：`message`
- 文档：`feishu_doc` / `feishu_wiki` / `feishu_drive`
- 记忆：`memory_search` / `memory_get`
- 编排：`sessions_spawn` / `subagents`

职责：
- 调用一等工具
- 做真实修改
- 做验证闭环
- 在可行时避免让用户手工执行

执行原则：
- 能验证就验证
- 能直接落地就不只给草案
- 能用工具就不用空想

---

## 二、5 层工作流架构（Execution Workflow）

这是面向“怎么工作”的 5 层，不是系统组件图。

### 1. 输入层
来源：Feishu / 其他 channel / 系统消息 / 心跳 / 子代理反馈

### 2. 记忆与上下文层
来源：
- `SOUL.md`
- `USER.md`
- `MEMORY.md`
- `memory/*.md`
- 当前仓库 / 当前配置

### 3. 决策层
内容：
- 判断是否需要 skill
- 判断是否需要 sub-agent
- 判断是否需要改配置、查文档、跑命令
- 判断是否要直接执行还是先确认风险
- 用 `TASK_ENTRY.md` 做任务入口分类
- 用 `PROMPT_MODULES.md` 选择需要注入的 prompt 模块

### 3.5 轻量运行时约束层
内容：
- 用 `PROMPT_MODULES.md` 提供可复用 prompt 组件
- 用 `TASK_ENTRY.md` 固定任务入口分类
- 用 `COMPLETION_GATE.md` 统一决定能否按“完成”出闸
- 用 `FIXED_TASKS.md` 把已有 cron / 固定任务正式映射回主流程体系
- 用 `RUNTIME_BINDING.md` 定义任务与 prompt 模块、闸门、failsafe 的绑定关系

职责：
- 在不改底层源码的前提下，把规范从文档层推进到准运行时层
- 减少临场发挥
- 降低遗漏、重复失败、虚假完成的概率
- 让已有固定任务不再是孤立配置，而是有入口、有模板、有闸门的正式流程
- 为后续真正的 prompt 插桩 / middleware 拦截提供绑定基础

### 4. 执行层
内容：
- 改文件
- 调工具
- 跑命令
- 写文档
- 更新记忆

### 5. 验证与输出层
内容：
- `openclaw status`
- 运行结果检查
- 会话模型确认
- 向用户输出“已做什么 + 结果是什么 + 还差什么”

---

## 三、当前实例映射（xuan 的 OpenClaw）

### 已完成落地
- 6 层主架构：已定义并文档化
- 5 层工作流架构：已定义并文档化
- 默认主模型：`gpt/gpt-5.4`
- 主 channel：Feishu DM
- Gateway：本地 loopback + token
- Memory 机制：已启用并纳入主会话规则
- 运行手册：`RUNBOOK.md`
- Bootstrap / Checklist 可执行规范：`HARNESS_SPEC.md`
- 任务模板库：`TASK_TEMPLATE.md`
- 文档落地规则：`DOC_WORKFLOW.md`
- 轻量运行时约束层：`PROMPT_MODULES.md` / `TASK_ENTRY.md` / `COMPLETION_GATE.md`
- 固定任务接入规则：`FIXED_TASKS.md`
- 运行时模块绑定规则：`RUNTIME_BINDING.md`
- 第三层自动化 hook 规格层：`HOOK_PLAN.md` / `PROMPT_INJECTION_SPEC.md` / `MIDDLEWARE_SPEC.md` / `LOOP_DETECTION_SPEC.md` / `TRACE_SUMMARIZER_SPEC.md` / `ROUTING_ENGINE_SPEC.md`
- 固定任务级 runtime harness 主链：每日 Hangfire 学习任务 / 每周飞书总结任务 已完成真实 `cron + isolated agentTurn` 接入
- 固定任务级 Completion Gate：已真实接入并完成结果验证
- Feishu cron delivery 修复：已通过显式 `delivery.to` 绑定恢复自动投递，并增加 `heartbeat` 漂移 guard
- 通用运行轨迹状态文件：`TRACE_RUNTIME_STATE.md`
- 通用 Routing / Budgeting 状态文件：`ROUTING_RUNTIME_STATE.md`

### 当前不是问题、无需再改
- 默认模型无需再切换
- Feishu channel 已工作
- Gateway 可用
- 插件可用

### 后续扩展建议
1. 增加 `ROUTING.md`
   - 规定什么任务用 GPT、什么任务用 GLM、什么任务交给 ACP
2. 增加 `RUNBOOK.md`
   - 规定排障顺序、重启顺序、状态检查命令
3. 增加 `SKILLS.md` 或自定义 skills
   - 把常用业务动作封装成稳定流程
4. 增加 `HOOK_PLAN.md`
   - 定义第三层自动化 hook 的阶段计划：prompt 插桩、middleware 拦截、loop detection、trace summarizer、routing engine
5. 增加 `PROMPT_INJECTION_SPEC.md`
   - 定义 Prompt 插桩的对象优先级、模块注入顺序、强制/条件注入规则与验收标准
6. 增加 `MIDDLEWARE_SPEC.md` / `LOOP_DETECTION_SPEC.md` / `TRACE_SUMMARIZER_SPEC.md` / `ROUTING_ENGINE_SPEC.md`
   - 把第三层自动化 hook 的 middleware、循环阻断、轨迹摘要、路由引擎继续拆到规格层
7. 增加 `IMPLEMENTATION_QUEUE.md`
   - 把第三层从规格层推进到真实实现排程层，明确实施顺序、依赖关系与每步验收标准
8. 增加 `CRON_AGENTTURN_INJECTION_IMPL.md`
   - 把 Queue 1 从“排程层”推进到实现级接入说明，先处理 cron `agentTurn` 的 Prompt Injection
9. 增加 `OUTPUT_COMPLETION_GATE_IMPL.md`
   - 把 Queue 2 从“排程层”推进到实现级接入说明，先处理输出前 Completion Gate 的强制过闸
10. 增加 `FAILSAFE_LOOP_IMPL.md`
   - 把 Queue 3 从“排程层”推进到实现级接入说明，先处理失败兜底与循环阻断
11. 增加 `TRACE_SUMMARIZER_IMPL.md`
   - 把 Queue 4 从“排程层”推进到实现级接入说明，先处理复杂任务完成后的最小轨迹摘要
12. 增加 `ROUTING_ENGINE_IMPL.md`
   - 把 Queue 5 从“排程层”推进到实现级接入说明，先处理任务路径选择
13. 增加 `FINAL_ACCEPTANCE.md`
   - 定义第三层实现级文件补齐后的最后串联验收标准
14. 增加 `RUNTIME_ENTRYPOINTS.md`
   - 定义 Queue 1 / Queue 2 真实进入 runtime implementation 时，最小可改接入点与最小改造范围

---

## 四、强制执行规则

后续对 xuan 的 OpenClaw 做任何配置或结构调整时：

1. 必须说明影响的是哪一层
2. 必须说明是“系统 6 层”还是“执行 5 层”
3. 必须给出验证结果，不能只给草案
4. 架构变化必须同步更新本文件

---

## 五、当前结论

xuan 的 OpenClaw：
- 现在已经具备可运行的 6 层主架构定义
- 现在已经具备可执行的 5 层工作流定义
- 这次不是口头说明，而是已经写入 workspace 并正式生效
