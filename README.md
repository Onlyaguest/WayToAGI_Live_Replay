# 🎙️ Live Replay Engine — 共学直播复盘术

> 直播刚结束，5 分钟出精华文档、金句卡片、社群推送。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 它做什么

直播结束后，把 STT（语音转文字）/ 飞书妙记原始材料自动转化为：

1. **精华文档** — 去口水话，提炼核心观点（飞书文档）
2. **金句卡片** — 嘉宾金句结构化存储（飞书 Base）
3. **社群推送** — 精华摘要发到群里（飞书群消息）
4. **知识卡片** — 知识点入库，串联 Study Reviver（飞书 Base）
5. **下期预告** — 基于观众提问生成下期话题（飞书文档）

## 快速开始

```bash
# 安装 lark-cli (https://github.com/larksuite/cli)
npm install -g @anthropic-ai/lark-cli

# 认证（需要 docs/base/im/wiki/vc 权限）
lark-cli auth login --domain docs,base,im,wiki,vc
```

### OpenClaw

```bash
git clone https://github.com/Onlyaguest/WayToAGI_Live_Replay.git ~/.openclaw/skills/lark-live-replay
```

### Claude Code / Trae / Codex

```bash
git clone https://github.com/Onlyaguest/WayToAGI_Live_Replay.git
# 打开项目，Agent 自动读取 SKILL.md
```

## 常用指令

```plaintext
这是今晚共学直播的妙记链接：{链接}。帮我生成精华文档，
提取金句写入 Base，然后在共学群发一条精华摘要。
```

```plaintext
从最近 5 期直播的金句表里，找适合发小红书的金句，
配上背景说明，生成 10 条待发布内容。
```

```plaintext
分析本期直播的观众提问，找出没充分回答的问题，
生成下期直播话题建议和预告文案。
```

## 与 Study Reviver 的关系

| | Study Reviver | Live Replay Engine |
|---|---|---|
| 输入 | 已有文档（冷数据） | 直播 STT（热数据） |
| 时机 | 事后沉淀 | 趁热打铁 |
| 目的 | 让旧内容活过来 | 让新内容快速扩散 |
| 串联 | 知识卡片 Base 共享 | 直播知识点自动写入 Study Reviver |

两个 Skill 组合 = **直播产出 → 即时传播 → 长期沉淀 → 学习复用**。

## 技术架构

```
lark-cli 命令组合：
├── lark-minutes  — 获取妙记（STT、总结、章节）
├── lark-vc       — 获取会议记录和纪要
├── lark-doc      — 生成精华文档、下期预告
├── lark-base     — 金句卡片、知识卡片存储
├── lark-im       — 社群推送
└── lark-drive    — 可选：下载妙记音视频
```

## 依赖

- [lark-cli](https://github.com/larksuite/cli) >= 1.0.5
- 飞书应用权限：docs, base, im, wiki, vc, minutes
- 任意本地 Agent（OpenClaw、Claude Code、Trae、Codex 等）

## License

MIT
