# 防串群配置规范

## 问题背景

多个飞书群同时使用定时任务时，消息会发到错误的群（串群）。

**根因**：不指定 `delivery.to` 时，OpenClaw 默认发到「最近活跃（last）」的会话，导致任务发到错误的群。

> 关键概念：`sessionKey` ≠ 投递目标。`sessionKey` 只表示任务在哪个会话里创建/绑定，真正决定发到哪里的是 `delivery.to`。

## 正确配置 vs 错误配置

| 配置项 | ❌ 会串群 | ✅ 不串群 |
|--------|-----------|-----------|
| `sessionTarget` | `"session"` 或 `"main"` | `"isolated"` |
| `delivery.to` | 不填 或 `"last"` | 完整群ID（`oc_xxx...`） |
| `delivery.channel` | 不填 | `"feishu"` |

## 标准防串群配置片段

```json
"sessionTarget": "isolated",
"delivery": {
  "mode": "announce",
  "channel": "feishu",
  "to": "<在目标群发 /status 获取的群ID>"
}
```

## 获取群 ID

在目标群里发送 `/status`，返回信息中找 session ID，格式为 `group:oc_xxx...`，取 `oc_xxx...` 部分即为群ID。

**创建任务前，Agent 应主动向用户确认群ID，不能凭记忆填写。**

## 验证配置是否正确

```bash
cat ~/.openclaw/cron/jobs.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
for job in data['jobs']:
    if job.get('enabled', True):
        d = job.get('delivery', {})
        ok = job.get('sessionTarget') == 'isolated' and d.get('to')
        status = '✅' if ok else '❌'
        print(f\"{status} {job.get('name')}: sessionTarget={job.get('sessionTarget')}, delivery.to={d.get('to')}\")
"
```

所有有效任务应输出 ✅。
