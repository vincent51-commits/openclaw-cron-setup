---
name: openclaw-cron-setup
description: 创建、修改、查询和排查 OpenClaw 定时任务（Cron Job）。当用户需要：(1) 新增定时播报/提醒任务；(2) 修改现有 Cron 任务的时间、目标群、prompt；(3) 停用/删除 Cron 任务；(4) 查看当前 Cron 任务列表；(5) 排查串群、delivery 失败等 Cron 问题时使用。关键词：定时任务、cron、播报、每天、每小时、串群、定时。
---

# OpenClaw Cron 任务配置指南

## 核心文件

- 任务定义：`~/.openclaw/cron/jobs.json`
- 任务状态：`~/.openclaw/cron/jobs-state.json`

## 查看任务列表

列出指定 agentId 的任务（只列当前 Agent 的，不含其他 agent 和系统内置任务）：

```bash
cat ~/.openclaw/cron/jobs.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
for job in data['jobs']:
    if job.get('agentId') == '<当前agentId>':
        print(job['id'][:8], job.get('name'), job.get('enabled'))
"
```

> 查看所有任务（含系统内置）去掉 `agentId` 过滤条件即可。

## 新建任务

### 第一步：确认目标群ID

**必须在创建任务前确认群ID**，不要凭记忆填写。

- 让用户在目标群里发 `/status`，从返回的 session ID（格式 `group:oc_xxx...`）提取群ID
- 或请用户直接告知群ID
- 如果用户未提供，**主动询问**：「请问这个任务要发到哪个群？可以在目标群里发 `/status` 查看群ID」

### 第二步：编辑 jobs.json

直接编辑 `jobs.json`，在 `jobs` 数组中添加新对象。**必须使用防串群配置**：

```json
{
  "id": "<uuid4>",
  "agentId": "<你的agentId>",  // 从 runtime context 的 agent=xxx 字段获取，Agent 自己知道
  "name": "任务名称",
  "enabled": true,
  "createdAtMs": <毫秒时间戳>,
  "schedule": {
    "kind": "cron",
    "expr": "0 10 * * *",
    "tz": "Asia/Shanghai",
    "staggerMs": 0
  },
  "sessionTarget": "isolated",
  "wakeMode": "now",
  "payload": {
    "kind": "agentTurn",
    "message": "<详细的任务 prompt>",
    "timeoutSeconds": 300,
    "lightContext": true
  },
  "delivery": {
    "mode": "announce",
    "channel": "feishu",
    "to": "<完整群ID，如 oc_235eaaaa...>"
  },
  "state": {}
}
```

生成 UUID：`python3 -c "import uuid; print(uuid.uuid4())"`
生成时间戳：`python3 -c "import time; print(int(time.time()*1000))"`

## 防串群配置要点（必读）

详见 [references/anti-cross-group.md](references/anti-cross-group.md)

**核心三点：**
1. `sessionTarget` = `"isolated"`（不能是 `"session"` 或 `"main"`）
2. `delivery.to` = 完整群ID（不能省略，不能用 `"last"`，**创建前必须向用户确认**）
3. `agentId` = 当前 Agent 的 agentId（从系统 runtime 信息获取）

## 停用/删除任务

**停用**（保留配置，不再执行）：将 `"enabled": false`

**删除**：从 `jobs` 数组中移除整个对象

## Prompt 写法要点

- 详细说明要查什么数据、用哪张表
- 结尾加输出格式说明（如「用 Markdown 表格输出」）
- 如果任务可能无需输出（如预警类），加 `"如无异常直接回复 NO_REPLY"`
- 对于数据查询任务，建议加 `"lightContext": true` 降低 token 消耗

## 配置修改后

OpenClaw 会热加载 `jobs.json`，无需重启 gateway。可用以下命令确认：

```bash
cat ~/.openclaw/cron/jobs-state.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
for id, state in data.get('jobs', {}).items():
    print(id[:8], state.get('lastRunStatus'), state.get('consecutiveErrors', 0), 'errors')
"
```

## 排查问题

详见 [references/troubleshooting.md](references/troubleshooting.md)
