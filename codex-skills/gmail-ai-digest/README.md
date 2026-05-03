# Gmail AI Digest Codex Skill

This folder contains a Codex skill for generating a Gmail unread-message digest by whitelisted sources, grouping it into `finance`, `news`, `research`, and `other`, and writing the result to the Notion database `gmail信件重點摘要`.

## Install On Another Computer

Run this on the other computer:

```bash
mkdir -p ~/.codex/skills/gmail-ai-digest
curl -L https://raw.githubusercontent.com/alien4817/awesome-chatgpt-prompts/main/codex-skills/gmail-ai-digest/SKILL.md -o ~/.codex/skills/gmail-ai-digest/SKILL.md
```

Restart Codex after installing the skill so it can be discovered.

## Requirements

- Codex desktop or Codex environment that supports local skills.
- Gmail connector installed and authenticated.
- Notion connector installed and authenticated if you want automatic Notion writes.
- Access to the Notion database `gmail信件重點摘要`.

## Usage

Ask Codex something like:

```text
使用 gmail-ai-digest 幫我整理今天 Gmail 未讀重要信件，並寫入 Notion。
```

The skill defaults to the Asia/Taipei timezone and today's Taipei date window.
