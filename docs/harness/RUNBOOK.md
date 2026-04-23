# RUNBOOK.md

# OpenClaw 排障与验收手册

## 1. 不回复时先查什么

### 1.1 先看网关
```bash
openclaw gateway status
```

### 1.2 再看进程
```bash
ps aux | grep openclaw | grep -v grep
ps aux | grep node | grep -v grep
```

### 1.3 再看当前模型状态
在会话里执行：
- `session_status`

重点看：
- 当前模型
- 当前 key 来源
- fallback 是否生效
- context 是否异常

---

## 2. 模型 / key 异常排查

## 生效位置优先级
重点检查：
1. `/root/.openclaw/agents/main/agent/models.json`
2. `/root/.openclaw/.env`
3. `/root/.openclaw/openclaw.json`

## 检查方法
### 2.1 看当前主模型 key
```bash
grep -A3 '"gpt":' /root/.openclaw/agents/main/agent/models.json | head -5
```

### 2.2 直接测 API
```bash
curl -s -X POST https://ai.td.ee/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <API_KEY>" \
  -d '{"model":"gpt-5.4","messages":[{"role":"user","content":"hi"}],"max_tokens":5}'
```

### 2.3 改完 key 后
- 重启 gateway
- 再查 `session_status`
- 再做一次直连 API 测试

---

## 3. Feishu 文档 / 云盘异常排查

## 3.1 文档写入失败先查
- doc_token 是否正确
- 是否有写权限
- 是 write 失败还是 append 失败

## 3.2 云盘移动失败先查
- folder_token 是否正确
- 目标文件夹是否已共享
- 文件类型是否匹配（docx / file / folder）

## 3.3 验证方式
- 新建临时文档
- append 一行测试文本
- list 目标 folder 确认文件存在

---

## 4. cron 异常排查

## 4.1 看配置文件
```bash
cat /root/.openclaw/cron/jobs.json
```

## 4.2 重点看这些字段
- `enabled`
- `schedule.expr`
- `payload.kind`
- `state.lastRunStatus`
- `state.lastError`
- `state.lastDeliveryStatus`

## 4.3 当前已知注意点
- `systemEvent` 更适合提醒类任务
- `agentTurn` 更适合生成文档/执行复杂动作
- 不要依赖未确认支持的变量占位符（如 `${TASK_DAY}`）

## 4.4 修 cron 的原则
- 提醒类 → 固定文本 + systemEvent
- 生成类 → agentTurn
- 需要落飞书 → 在 message 里明确 folder token / 文档动作
- 需要固定输出结构的学习/总结任务 → 直接写成模板化 agentTurn，不只做提醒

## 4.5 生成类 cron 模板要求
对于“每日学习”“总结生成”“固定结构输出”类 cron：
- 优先使用 `agentTurn`
- prompt 中必须写清输出结构
- prompt 中必须写清主题
- 如果需要写飞书，必须写清目标 folder / doc 与回链要求
- 完成汇报时必须只报结果，不重复输出无关过程

---

## 5. 验收顺序

每次做完配置修改后，按这个顺序验收：

1. 模型是否可调通
2. session_status 是否显示正确模型和 key 来源
3. Feishu doc 是否可写
4. Feishu drive 是否可读 / 可移动
5. cron 是否没有 error job
6. TODOS 是否反映当前真实待办
7. 回复前自检是否完成

### 5.1 回复前必须自检
在对用户输出“已完成”之前，必须逐项确认：
- 需求是否真的完成
- 是否遗漏显式要求
- 结果是否落到了指定位置
- 是否有验证结果 / 写入结果 / 命令结果作为证据
- 如果仍有未完成项，是否明确写出风险与下一步

只要任一项不能明确回答，就不能以“完成”状态回复。

---

## 6. 当前硬约束

### 6.1 任务结束前必须检查
- 是否真正完成需求
- 是否有验证证据
- 是否写到用户要求的位置

### 6.2 文档沉淀优先级
- 最终成果优先写飞书
- 本地文件默认只作为临时产物

### 6.3 多步骤任务
- 必须有 plan / todo
- 必须能说明当前进度

### 6.4 调研任务
- 尽量读原文
- 区分事实与判断
- 输出结构化结论

### 6.5 代码任务
- 修改后必须验证
- 汇报时必须说明验证结果
