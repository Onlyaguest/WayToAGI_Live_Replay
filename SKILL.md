---
name: lark-live-replay
version: 1.0.0
description: "共学直播复盘术：直播刚结束，把 STT/妙记原始材料自动转化为精华文档、金句卡片、社群推送、知识卡片入库。当用户需要处理直播/会议的语音转文字材料、生成直播精华回顾、提取金句、做社群二次传播、生成下期预告时使用。"
metadata:
  requires:
    bins: ["lark-cli"]
---

# Live Replay Engine — 共学直播复盘术

> **前置条件：** 需要 lark-cli 已安装并认证（`lark-cli auth login --domain docs,base,im,wiki,vc`）。

## 概述

直播刚结束，趁热打铁。把 STT（语音转文字）原始材料自动转化为多种可传播的内容资产：

1. **精华文档** — 去口水话，提炼核心观点，生成结构化飞书文档
2. **金句卡片** — 提取嘉宾金句，写入飞书 Base，可做社交媒体素材
3. **社群推送** — 生成直播精华摘要，发到飞书群做二次传播
4. **知识卡片** — 提取知识点写入 Study Reviver Base（上下游串联）
5. **下期预告** — 基于本期内容生成下期话题建议和预告模板

## 核心流程

```
直播 STT / 飞书妙记                    多形态输出
┌──────────────┐    AI清洗     ┌──────────────────────┐
│ 原始 STT 文本 │ ──────────→ │ 1. 精华文档（飞书文档）  │
│ 或飞书妙记链接 │  去口水话    │ 2. 金句卡片（飞书 Base） │
│              │  断句分段    │ 3. 社群推送（飞书群消息） │
│              │  识别说话人  │ 4. 知识卡片（Base 入库）  │
│              │             │ 5. 下期预告（飞书文档）   │
└──────────────┘             └──────────────────────┘
```

## 第一步：获取直播原始材料

### 方式 A：从飞书妙记获取

```bash
# 如果直播用的飞书会议，直接拉妙记
lark-cli minutes +get --minute-token <minute_token>
# 返回：标题、时长、AI总结、待办、章节、逐字稿
```

### 方式 B：从飞书会议记录获取

```bash
# 搜索最近的会议
lark-cli vc +search --query "共学直播" --start "2026-04-13" --end "2026-04-14"

# 获取会议纪要产物
lark-cli vc +notes --meeting-id <meeting_id>
# 返回：总结、待办、章节
```

### 方式 C：本地 STT 文件

```bash
# 如果 STT 是本地文件（如 .txt/.srt），直接读取
cat ~/Downloads/live-stt-20260413.txt
```

## 第二步：AI 清洗与结构化

AI 对原始 STT 做以下处理：
- **去口水话**：删除"嗯"、"那个"、"就是说"等口头禅
- **断句分段**：按话题切分段落，加小标题
- **识别说话人**：区分主持人、嘉宾、观众提问
- **提炼核心观点**：每个段落提取 1-2 个核心观点

## 第三步：多形态输出

### 输出 1：精华文档

```bash
# AI 生成精华文档 markdown，写入飞书
lark-cli docs +create \
  --title "共学直播精华回顾：{主题} — {日期}" \
  --folder <folder_token> \
  --markdown @/tmp/live-recap.md
```

精华文档结构：
- 直播信息（主题、嘉宾、时长、日期）
- 核心观点（3-5 个，每个带引用原话）
- 精彩问答（观众提问 + 嘉宾回答精华）
- 行动建议（听完这期可以做什么）
- 相关资源链接

### 输出 2：金句卡片

```bash
# 创建金句表（首次）
lark-cli base +table-create \
  --base-token <base_token> \
  --name "直播金句" \
  --fields '[
    {"type":"text","name":"金句内容"},
    {"type":"text","name":"说话人"},
    {"type":"text","name":"直播主题"},
    {"type":"datetime","name":"直播日期"},
    {"type":"text","name":"上下文"},
    {"type":"select","name":"适合平台","multiple":true,"options":[
      {"name":"朋友圈"},{"name":"小红书"},{"name":"Twitter"},{"name":"公众号"}
    ]}
  ]'

# 写入金句
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <table_id> \
  --json '{
    "金句内容": "AI 真正带来的不是更聪明的一次回答，而是把 judgment 外化成可复用的工作流。",
    "说话人": "AJ",
    "直播主题": "Agent 协作新范式",
    "直播日期": "2026-04-13 00:00:00",
    "上下文": "讨论 AI 对组织架构的影响时提出",
    "适合平台": ["朋友圈","Twitter"]
  }'
```

### 输出 3：社群推送

```bash
# 生成精华摘要发到群里
lark-cli im +messages-send \
  --chat-id <chat_id> \
  --type text \
  --content "🎙️ 刚刚直播精华回顾\n\n主题：{主题}\n嘉宾：{嘉宾}\n\n💡 3个核心观点：\n1. {观点1}\n2. {观点2}\n3. {观点3}\n\n📖 完整精华文档：{链接}\n🔥 下期预告：{预告}\n\n错过直播的同学看这篇就够了！"
```

### 输出 4：知识卡片入库（串联 Study Reviver）

```bash
# 提取知识点写入 Study Reviver 的知识卡片 Base
lark-cli base +record-upsert \
  --base-token <study_reviver_base_token> \
  --table-id <knowledge_cards_table_id> \
  --json '{
    "知识点标题": "{从直播中提取的知识点}",
    "知识点内容": "{3-5句话解释}",
    "主题": "{分类}",
    "难度": "{入门/进阶/高级}",
    "来源文档": "共学直播 {日期} {链接}",
    "关键人物": "{说话人}"
  }'
```

### 输出 5：下期预告

```bash
# 基于本期内容生成下期话题建议
lark-cli docs +create \
  --title "下期共学直播预告 — {建议主题}" \
  --folder <folder_token> \
  --markdown @/tmp/next-preview.md
```

## Use Cases

### Use Case 1：直播刚结束，5 分钟出精华

**场景：** 晚 8 点共学直播刚结束，运营想趁热在群里发精华回顾。

**操作：**
```plaintext
这是今晚共学直播的妙记链接：{链接}。帮我生成精华文档发到飞书，
提取 5 个金句写入 Base，然后在共学群发一条精华摘要。
```

**效果：** 5 分钟内完成，以前运营要花 2 小时整理。

### Use Case 2：金句素材库积累

**场景：** 社区想在社交媒体做传播，需要嘉宾金句做素材。

**操作：**
```plaintext
从最近 5 期直播的金句表里，找适合发小红书的金句，帮我配上背景说明，生成 10 条待发布内容。
```

**效果：** 金句不再散落在各期纪要里，结构化存储后随时可用。

### Use Case 3：系列直播知识沉淀

**场景：** "Agent 系列"做了 8 期直播，想把所有知识点串起来。

**操作：**
```plaintext
从知识卡片 Base 里找所有来源是"共学直播"的卡片，按时间排序，
生成一篇"Agent 系列直播知识图谱"。
```

**效果：** 8 期直播的知识点变成一份有脉络的学习资料，串联 Study Reviver 的学习路径。

### Use Case 4：下期直播策划

**场景：** 本期讨论了 RAG，观众提了很多关于向量数据库的问题没来得及展开。

**操作：**
```plaintext
分析本期直播的观众提问，找出没有充分回答的问题，生成下期直播的话题建议和预告文案。
```

**效果：** 下期话题从观众真实需求中来，不是运营拍脑袋。

## 技术架构

```
lark-cli 命令组合：
├── lark-minutes  — 获取妙记（STT、总结、章节）
├── lark-vc       — 获取会议记录和纪要
├── lark-doc      — 生成精华文档、下期预告
├── lark-base     — 金句卡片、知识卡片存储
├── lark-im       — 社群推送
└── lark-drive    — 可选：下载妙记音视频文件
```

跨 5 个飞书域编排。与 Study Reviver 通过知识卡片 Base 串联。

## 与 Study Reviver 的关系

| | Study Reviver | Live Replay Engine |
|---|---|---|
| 输入 | 已有的文档（冷数据） | 刚结束的直播 STT（热数据） |
| 时机 | 事后沉淀 | 趁热打铁 |
| 目的 | 让旧内容活过来 | 让新内容快速扩散 |
| 串联 | 知识卡片 Base 是共享的 | 直播知识点自动写入 Study Reviver Base |

两个 Skill 组合 = 社区知识的完整生命周期：**直播产出 → 即时传播 → 长期沉淀 → 学习复用**。
