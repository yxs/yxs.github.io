+++
title = 'Keep the Thinking, Automate the Rest'
date = '2026-04-02T12:31:38-06:00'
summary = ""
tags = []
categories = []
showToc = true
TocOpen = false
slug = "keep-the-thinking-automate-the-rest"
+++

# 优化每日工作流程

从期望结果倒推需求和流程。

## 需求是什么？

方向： post train infra + post train

1. GRPO RL with rollout
2. FSDP2/HSDP distributed platform
3. E2E multi-stage pipeline at scale
4. Triton kernel
5. reward modeling, PRM

最大化每天用于思考的时间，deep dive project、system design、presentation、选方向、planning。
把执行侧尽可能地交给 AI 去做，收集上下文、跑实验、debug、结果验证，最小化琐碎的交互。

## 每天都在做什么？用到了哪些东西，它们是怎么组织起来的

1. deep dive 一个topic，一比如设计一个 GRPO training loop，debug it。基于一个较好的资料/教程去拓展。要对重点 topic 达到了如指掌的程度。
2. take a task，不管是开源的 verl，还是内部的 post train pipeline、 training platform -> 和 Claude 讨论方案、读相关 repo、PR、issue ->  Claude 写代码、在 pod 跑实验、看结果、debug、迭代 -> 产出结果 -> final review、提交 PR/文档

## 瓶颈在哪里？应该怎么优化？

> 缺乏有效上下文，方案质量不够，导致反复迭代

先明确需要哪些上下文，然后写 plan。

1. 每个 repo 一份持久化的架构分析，整体资源情况，定期更新，放入本地 repo 的 CLAUDE.md
2. 每个 task 一份聚焦分析，模块、调用链，issue/PR 及 maintainer 意见，资源配置、依赖版本、模型数据
3. 对于相关issue/PR 的上下文收集，必须读的有 正文，core contributor 的评论，引用的 link
4. 人 + GPT Codex review plan
5. 确定验收标准，给出 Claude 可 100% 自动化执行的方案，kick off

claude config:

`~/.claude/CLAUDE.md` 个人风格偏好 like deeply、aggressive、no emotional，pod 配置，`ralph-loop`

`~/.claude/settings.json` 开放除了 git push 的所有权限

`~/.claude/projects/.../memory` 自行管理

`CLAUDE.md` per repo，定期更新的持久化架构分析

各自一个 PROGRESS.md track 进度: 

- `Obsidian/XY/1.RE/train`，工作项目
- `Obsidian/XY/1.RE/opensource`
- `Obsidian/XY/1.RE/interview` find a topic, then deep dive in `Obsidian/XY/1.RE/mlsys`

checklist:

1. 写 plan 之前，上下文是否充分？
2. 完 plan 之后，review 之前， plan 是否可 100% 自动执行？

> 本地和 pod 之间的 debug 循环需要自动化

pod config、kubectl exec 等等放入 `~/.claude/CLAUDE.md`，引导 Claude 自行操作

- `ralph-loop`
- `cursor-agent --print --mode plan --model gpt-5.3-codex` review the work 
- Glean MCP when the task involves internal context

`~/.claude/CLAUDE.md` 中明确强制使用

每步写 checkpoint，`～/.claude/task-state.md`，session 断开后可恢复

轮询间隔默认 60 sec

> Claude 执行前对于方案的确信度缺乏评判，偶尔会尝试 trick 的方式去解决

Confidence gate — 执行前必须回答三个问题：

1. 能解释 WHY this works，不只是 WHAT it does？
2. 如果失败，能预测错误是什么？
3. 选这个方案是因为正确，还是因为最容易试？

任何一个答不上来，停下来补上下文：Glean（无限 token，遇事不决就问）、cursor-agent second opinion、读更多代码/issue/PR、写 probe code 验证假设。

Error handling：读完整报错 → 诊断 → 3 次同方法失败 → 自己换方向继续 → ralph-loop 15 次迭代耗尽才人工介入。

Mistake recording — 犯错后记录教训：
- Repo 相关 → repo CLAUDE.md
- 通用行为 → memory（feedback type，Claude 自主管理）

> 可用开发环境消耗时间长

不需要自定义 Image。挑选合适的 base image 启动后，由 Claude 解决依赖和环境配置。

可用 images 通过 Glean 查询，根据 task 需求（CUDA 版本、PyTorch、verl/sglang）推荐最合适的 image 和版本。

## 使用方式

我来定义问题、给出初步上下文 -> Claude 生成 plan -> 我来 review、判断 -> Claude 执行 -> 我 review 结果
0. per repo 维护一个 `CLAUDE.md`

1. 我选 task — 从 opensource/train/interview 等方向选一个具体任务，给 Claude task + 基础上下文
2. 等 Claude 完成上下文收集和 plan — Claude 读 repo、issue/PR、Glean，写 plan，跑 cursor-agent review
3. Review plan
4. Kick off - 我说"开始"，Claude 自动执行（ralph-loop）
5. 看结果

只参与 plan + review


## 配置示例


`~/.claude/settings.json`

```json
{
  "permissions": {
    "defaultMode": "bypassPermissions", // may override by managed policy
    "skipDangerousModePermissionPrompt": true,
    "allow": [
      "Read",
      "Edit",
      "Write",
      "Glob",
      "Grep",
      "WebFetch",
      "Bash(*)",
      "Skill(*)"
    ],
    "deny": [
      "Bash(git push *)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(git push*)",
        "hooks": [
          {
            "type": "command",
            "command": "exit 2"
          }
        ]
      },
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "CLAUDE_PLUGIN_ROOT=/Users/sye/code/everything-claude-code node /Users/sye/code/everything-claude-code/scripts/hooks/run-with-flags.js pre:config-protection scripts/hooks/config-protection.js standard,strict",
            "timeout": 5
          }
        ]
      },
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "CLAUDE_PLUGIN_ROOT=/Users/sye/code/everything-claude-code node /Users/sye/code/everything-claude-code/scripts/hooks/run-with-flags.js pre:mcp-health-check scripts/hooks/mcp-health-check.js standard,strict"
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "CLAUDE_PLUGIN_ROOT=/Users/sye/code/everything-claude-code node /Users/sye/code/everything-claude-code/scripts/hooks/session-start-bootstrap.js"
          }
        ]
      }
    ],
    "PreCompact": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "CLAUDE_PLUGIN_ROOT=/Users/sye/code/everything-claude-code node /Users/sye/code/everything-claude-code/scripts/hooks/pre-compact.js"
          }
        ]
      }
    ],
    "PostToolUseFailure": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "CLAUDE_PLUGIN_ROOT=/Users/sye/code/everything-claude-code node /Users/sye/code/everything-claude-code/scripts/hooks/run-with-flags.js post:mcp-health-check scripts/hooks/mcp-health-check.js standard,strict"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "CLAUDE_PLUGIN_ROOT=/Users/sye/code/everything-claude-code node /Users/sye/code/everything-claude-code/scripts/hooks/run-with-flags.js stop:session-end scripts/hooks/session-end.js minimal,standard,strict",
            "timeout": 30
          }
        ]
      }
    ]
  },
  "env": {
    "DISABLE_FEEDBACK_SURVEY": "1",
    "DISABLE_COST_WARNINGS": "1"
  },
  "model": "opus[1m]",
  "statusLine": {
    "type": "command",
    "command": "input=$(cat); current_dir=$(echo \"$input\" | jq -r '.workspace.current_dir'); dir_name=$(basename \"$current_dir\"); git_info=\"\"; if git -C \"$current_dir\" rev-parse --git-dir > /dev/null 2>&1; then branch=$(git -C
 \"$current_dir\" branch --show-current 2>/dev/null || git -C \"$current_dir\" rev-parse --short HEAD 2>/dev/null); if [ -n \"$branch\" ]; then if git -C \"$current_dir\" --no-optional-locks diff --quiet 2>/dev/null && git -C
\"$current_dir\" --no-optional-locks diff --cached --quiet 2>/dev/null; then git_info=$(printf \" \\033[1;34mgit:(\\033[0;31m%s\\033[1;34m)\\033[0m\" \"$branch\"); else git_info=$(printf \" \\033[1;34mgit:(\\033[0;31m%s\\033[1;34m)
\\033[0;33m✗\\033[0m\" \"$branch\"); fi; fi; fi; printf \"\\033[1;32m➜\\033[0m  \\033[0;36m%s\\033[0m%s\" \"$dir_name\" \"$git_info\""
  },
  "enabledPlugins": {
    "context7@claude-plugins-official": true,
    "code-review@claude-plugins-official": true,
    "superpowers@claude-plugins-official": true,
    "claude-md-management@claude-plugins-official": true,
    "playwright@claude-plugins-official": true,
    "ralph-loop@claude-plugins-official": true,
    "pr-review-toolkit@claude-plugins-official": true,
    // more internal plugins
  },
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp",
        "--browser",
        "chrome",
        "--user-data-dir",
        "/Users/sye/Library/Application Support/Google/Chrome-Playwright",
        "--shared-browser-context"
      ]
    }
  }
}
```

`~/.claude/CLAUDE.md`, JUST a minimal sample

```markdown
## Preferences
- Think deeply. Understand before responding.
- Be direct. Challenge weak ideas.
- Quality over speed. State confidence level explicitly.
- Diagnose before fixing. Probe code to verify hypothesis.

## Planning Gates

1. **Context gate** (before writing plan): read target code + call chain, read related issues/PRs, confirm resources exist.
2. **Auto-execution gate** (before kick-off): every step has input/output, zero human judgment mid-run, verification = concrete command, failure path defined.

## Checkpoint Rule

Update `.claude/task-state.md` after every step. New session → resume from checkpoint. No exceptions.

## Execution

- **Confidence gate**: WHY does this work? Can I predict the failure mode? Correct vs easy? Any "no" → stop, gather context.
- Verify each step before moving on. Diagnosis before retry. Max 3 retries then switch approach.
- Record mistakes: repo-specific → repo CLAUDE.md, general → persistent memory.
```