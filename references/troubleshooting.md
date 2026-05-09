# Cron 任务排查手册

## 常见错误及解法

### 1. 串群（消息发到错误的群）

**症状**：A 群的任务发到了 B 群，A 群没收到。

**根因**：`sessionTarget` 不是 `isolated`，或 `delivery.to` 未指定。

**解法**：
- 将 `sessionTarget` 改为 `"isolated"`
- 在 `delivery` 中显式指定 `"to": "oc_xxx..."` 完整群ID

详见 [anti-cross-group.md](anti-cross-group.md)

---

### 2. Delivery Mismatch 错误

**错误信息**：
```
Delivering to Feishu requires target <chatId|user:openId|chat:chatId>;
the agent used the message tool, but OpenClaw could not verify that
message matched the cron delivery target
```

**根因**：Agent 在 payload 的 prompt 里用 `message` 工具发送了消息，但发送的目标和 `delivery.to` 不一致，或 Agent 直接回复而非用 delivery 投递。

**解法**：
- 确保 `delivery.mode` = `"announce"`，这会让 OpenClaw 捕获 Agent 的输出并投递到 `delivery.to`
- Agent 的 prompt 里**不要**写「发到群 xxx」这类指令，让 delivery 配置负责路由
- 如果任务可能无需输出，在 prompt 末尾加：`如无需输出，直接回复 NO_REPLY`

---

### 3. 任务执行超时

**错误信息**：`cron: job execution timed out`

**解法**：
- 增大 `payload.timeoutSeconds`（数据查询任务建议 300，复杂任务可到 600）
- 简化 prompt，减少 Agent 需要做的工作量
- 加 `"lightContext": true` 减少 context 加载时间

---

### 4. 连续错误（consecutiveErrors > 0）

查看具体错误：

```bash
cat ~/.openclaw/cron/jobs-state.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
for id, state in data.get('jobs', {}).items():
    errs = state.get('consecutiveErrors', 0)
    if errs > 0:
        print(f'ID: {id[:8]}')
        print(f'  连续错误: {errs}')
        print(f'  错误信息: {state.get(\"lastError\")}')
        print(f'  错误原因: {state.get(\"lastErrorReason\")}')
        print()
"
```

根据 `lastError` 内容对应上面的解法处理。

---

### 5. 任务配置修改后不生效

OpenClaw 会热加载 `jobs.json`，通常无需重启。如果不生效：

```bash
openclaw gateway restart
```

---

## 健康检查清单

对每个有效任务，确认以下字段：

- [ ] `enabled` = `true`
- [ ] `sessionTarget` = `"isolated"`
- [ ] `delivery.mode` = `"announce"`
- [ ] `delivery.channel` = `"feishu"`
- [ ] `delivery.to` = 完整群ID（非 `null`、非 `"last"`）
- [ ] `payload.timeoutSeconds` >= 120（数据任务建议 300）
- [ ] `consecutiveErrors` = 0（在 jobs-state.json 中查看）
