# OpenClaw Cron Scheduled Task Configuration Guide
Let your Agent set up scheduled tasks correctly and eliminate cross-group delivery issues for good.

## What problem does this solve?
When multiple Feishu groups use OpenClaw scheduled tasks, messages are often sent to the wrong group—tasks intended for Group A end up in Group B, or fail to deliver entirely. This skill includes battle-tested configuration rules to help your Agent get the pairing right the first time.

## What’s included
📋 Complete task creation workflow: Step-by-step instructions from retrieving the group ID to writing to jobs.json
🛡️ Anti-cross-group configuration standards: Standard implementation of isolated mode + explicit delivery.to
🔧 Troubleshooting handbook: Root causes and fixes for common issues like cross-group delivery, delivery mismatch, timeouts, and repeated errors
✅ Health check checklist: Verify whether existing task configurations are correct

## Use cases
Adding new scheduled broadcasts/reminders
Modifying task execution times, target groups, or prompts
Disabling or deleting tasks
Troubleshooting tasks that fail to run or send messages to the wrong group

## Compatibility
OpenClaw 2026.04.24+
Feishu channel
