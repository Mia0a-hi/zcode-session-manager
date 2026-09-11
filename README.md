<div align="center">

# ⚡ ZCode Session Manager (`zcode-session-manager`)

**让手机 IM 随时无缝接续电脑桌面端 AI 完整会话的高性能跨端记忆中继器**  
*An ultra-fast (<20ms) session manager & cross-device context relay for ZCode / AI Coding Agents.*

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](LICENSE)
[![Python 3.8+](https://img.shields.io/badge/Python-3.8+-3776AB.svg?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Latency: <20ms](https://img.shields.io/badge/Speed-%3C20ms%20(mmap)-brightgreen?style=flat-square&logo=lightning)](https://github.com/)
[![Zero Dependency](https://img.shields.io/badge/Dependencies-Standard%20Library%20Only-orange?style=flat-square)](https://github.com/)
[![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey?style=flat-square)](https://github.com/)

[🇨🇳 中文文档](#-中文文档) • [🇬🇧 English Documentation](#-english-documentation) • [🤖 AI 一键安装](#-极简交付让-ai-帮您一键安装--ai-assisted-installation) • [💻 命令行速查](#-命令行速查--cli-cheat-sheet)

</div>

---

## 🇨🇳 中文文档

### 📱 诞生背景与痛点破解：跨端无缝接续（电脑桌面 ⇄ 移动端 IM）

#### 1. 真实业务痛点
随着企业将 ZCode / AI Coding Agent 接入即时通讯软件（如**企业微信、飞书、钉钉、Slack 等移动端 IM**），一个高频且令人抓狂的体验断层随之出现：
- **跨设备上下文黑洞**：在办公室电脑桌面端用 ZCode 调试了一下午代码、梳理了复杂的重构架构或排查方案。合上电脑离开工位后（通勤路上、会议室或居家），拿出手机在企业微信/钉钉里想继续跟同一个 AI 推进任务。
- **对话割裂与“失忆”**：移动端 IM 接入的每次对话往往是一个全新的会话实例，**手机端根本无法读取电脑本地桌面端的历史上下文**。AI 宛如初见，对几分钟前在电脑上讨论的设计细节一无所知。
- **极高的重复录入成本**：在手机巴掌大的屏幕上打字繁琐费时，工程师被迫重新组织长篇大论向 AI 复述背景、粘贴代码片段，导致移动办公协同体验极其痛苦。
- **本地数据库壁垒**：ZCode 桌面端的全量会话历史保存在本地 SQLite 中，无法直接被移动端 IM 实例感知。

#### 2. 破局方案：`zcode-session-manager` 作为“跨端记忆中继器”

通过 `zcode-session-manager`，手机端与电脑端之间的记忆鸿沟被彻底填平：

```text
 ┌──────────────────────────────────────┐          ┌──────────────────────────────────────┐
 │          PC / Mac 电脑桌面端         │          │         手机移动端 (企微/飞书/钉钉)    │
 │  · 运行 ZCode 完成复杂长链任务开发   │          │  · 随时随地拿出手机想继续推进任务   │
 │  · 产生丰富的本地 Session 上下文     │          │  · 痛点：默认无法读取电脑上聊了什么  │
 └──────────────────┬───────────────────┘          └──────────────────┬───────────────────┘
                    │                                                 │
                    │ 本地 SQLite 存储                                │ 通过 IM 发起请求
                    ▼                                                 ▼
 ┌────────────────────────────────────────────────────────────────────────────────────────┐
 │                      zcode-session-manager (跨端记忆中继调度)                         │
 │                                                                                        │
 │  1. 手机端唤起：发一条 “看看刚才电脑在聊啥” 或 “列出今天下午的会话”                  │
 │  2. 极速检索：19ms 内存映射秒级提取桌面端历史会话列表及清晰摘要                       │
 │  3. 无缝接续：回复 “接续第1个会话” 或指定 Session ID                                  │
 │  4. 记忆注入：调用 ReadSessionContext (Handoff 策略) 提取上个会话的完整任务与交接点   │
 └──────────────────────────────────────────┬─────────────────────────────────────────────┘
                                            │
                                            ▼
                        ┌──────────────────────────────────────┐
                        │      手机端瞬间唤醒记忆，丝滑继续聊！ │
                        │  · 无需打字重复复述背景             │
                        │  · 完美承接电脑端未竟事项与决策逻辑 │
                        └──────────────────────────────────────┘
```

### 🚀 核心特性

- ⚡ **内核级极速响应**：启用 256MB SQLite 内存映射（I/O mmap）与只读安全事务，全表毫秒级无感返回（< 20ms），绝不锁库冲突。
- 🎨 **终端丝滑对齐**：自适应当前终端列宽，内置标准库 CJK 全角中文字符对齐算法，超长标题平滑智能截断，**彻底告别折行爆屏**。
- 🕒 **人性化时序**：支持智能相对时间计算（“刚刚”、“40分钟前”、“昨天 17:41”），一眼掌握最近会话活跃状态。
- 📋 **一键提取与复制**：提供 `-c / --copy <序号>`，直接打印对应会话 ID，方便在手机端或 CLI 中一键输入“切到该会话”。
- 🔒 **纯离线与脱敏安全**：纯本地操作，零外部依赖，示例完全通用化脱敏。

---

## 🇬🇧 English Documentation

### 📱 Motivation: Seamless Context Handoff (Desktop ⇄ Mobile IM)

#### 1. The Real-World Pain Point
As enterprises integrate ZCode / AI Coding Agents into mobile Instant Messengers (e.g., **WeChat Work, Feishu/Lark, DingTalk, Slack**), developers face a frustrating context gap:
- **The Cross-Device Amnesia**: You spend hours debugging complex logic and designing architectures with ZCode on your desktop workstation. When you step away from your desk (commuting, in a meeting, or at home), you pull out your phone hoping to continue with the AI.
- **Disconnected Context**: The mobile IM starts an isolated, blank conversation. **The mobile client cannot access your desktop machine's local SQLite database**. The AI acts like a stranger, having zero memory of the code or decisions made minutes ago.
- **Painful Re-Typing**: Typing out long context explanations and pasting code snippets on a mobile keyboard is tedious and inefficient.

#### 2. The Solution: `zcode-session-manager` as Context Relay
`zcode-session-manager` bridges this gap effortlessly:
1. **Query on Mobile**: Send a simple prompt in your mobile IM: *"What was I working on my desktop?"* or *"List today's sessions"*.
2. **Sub-20ms Retrieval**: Uses SQLite memory-mapped I/O (mmap) to query desktop session metadata in milliseconds.
3. **Seamless Handoff**: Reply *"Continue from session #1"* or provide the Session ID.
4. **Context Injection**: Calls `ReadSessionContext` (using the `handoff` strategy) to inject previous tasks, decisions, and state into the mobile session. **Pick up exactly where you left off without typing a single background paragraph!**

### 🌟 Key Highlights

| Feature / 特性 | Description (中文) | Description (English) |
|---|---|---|
| **⚡ Sub-20ms Latency** | 启用 256MB SQLite 内存映射 (mmap)，纯内存零拷贝极速直读 | Powered by 256MB SQLite mmap I/O for zero-copy memory reads |
| **🎨 CJK-Aware Terminal UI** | 内置 CJK 全角字符宽度算法，自适应终端列宽，超长标题绝不折行爆屏 | Native CJK wide-character alignment prevents terminal line wrapping |
| **🕒 Humanized Timestamps** | 自动呈现“刚刚”、“5分钟前”、“昨天 17:41”，一目了然定位最近会话 | Displays intuitive relative times ("just now", "10m ago", "yesterday") |
| **📋 Instant Session Copy** | `-c / --copy <INDEX>` 一键打印对应 Session ID，极速复制切换 | Quick-extract session ID for instant terminal or pipe switching |
| **🔒 Zero Dependency & Safe** | 纯 Python 标准库驱动，无第三方依赖，离线脱敏无外泄风险 | Pure Python standard library with zero external dependencies |

---

## 🤖 极简交付：让 AI 帮您一键安装 / AI-Assisted Installation

无需手动解压或繁琐地翻找隐藏目录，只需将 `zcode-session-manager` 目录放在任意工作区，直接将以下提示词发送给您的 **ZCode Agent**，AI 将全自动完成部署与测试：

### 提示词模板 1：全局用户安装（推荐，所有项目通用）/ Global User Install
```text
请帮我安装当前目录下的 zcode-session-manager 技能：将其复制部署到我的全局技能目录（~/.zcode/skills/zcode-session-manager/），确保包含 SKILL.md 与 scripts/list_sessions.py，并测试运行 list_sessions.py -n 3 验证安装成功。
```
*English Prompt:*
```text
Please install the zcode-session-manager skill from the current directory: copy it to my global user skills directory (~/.zcode/skills/zcode-session-manager/), ensure SKILL.md and scripts/list_sessions.py are in place, and run list_sessions.py -n 3 to verify the installation.
```

### 提示词模板 2：当前工程专属安装 / Workspace-Only Install
```text
请帮我把当前目录下的 zcode-session-manager 部署到当前工作区的 .zcode/skills/zcode-session-manager/ 目录中，并验证文件完整性。
```

---

## 💻 命令行速查 / CLI Cheat Sheet

```bash
# 1. 默认极速浏览 (最新 25 条，终端自适应高亮 + 相对时间)
# Default view: latest 25 sessions with adaptive columns & relative timestamps
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py

# 2. 仅看今天活跃会话 / Filter today's active sessions only
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -t

# 3. 关键字模糊搜索（按标题或 Session ID 过滤，命中词高亮）
# Fuzzy keyword search (filters by title or session ID with highlights)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -s "重构"

# 4. 快捷提取 ID：直接打印第 1 个会话的 ID (适合快速复制或命令管道)
# Quick copy: print the Session ID of index 1 (ideal for copy/paste or piping)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -c 1

# 5. 指定拉取数量 (如最新 50 条) / Fetch specified count
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -n 50

# 6. 输出标准 Markdown 表格 (适合直接复制给大模型或写入文档)
# Output standard Markdown table format
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py --md

# 7. 输出标准 JSON 格式 (适合工具链与程序自动化)
# Output standard JSON format
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py --json

# 8. 禁用色彩高亮 (输出纯文本) / Disable ANSI colors
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py --no-color
```

---

## 🤖 自然语言交互范式 / Natural Language Interaction

在手机 IM 或电脑桌面端 ZCode 中，随时使用自然语言唤醒：
- `“看看刚才电脑在聊什么”` / *"What was I working on my desktop?"*
- `“列出今天下午的活跃会话”` / *"List active sessions from this afternoon"*
- `“帮我查找关于 <关键词> 的会话”` / *"Search sessions related to <keyword>"*
- `“接续第 1 个会话继续工作”` / *"Continue from session #1"*
- `“读取会话 sess_xxx 的上下文背景”` / *"Read context from sess_xxx"*

---

## 📄 开源许可证 / License

本项目采用 [MIT License](LICENSE) 授权。
Distributed under the MIT License.
