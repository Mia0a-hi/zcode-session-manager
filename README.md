# ZCode Session Manager (zcode-session-manager)
### ZCode 历史会话极速检索与跨端无缝接续扩展

专为 ZCode / AI Coding Agent 设计的高性能本地会话管理扩展技能。
秒级读取 SQLite 会话元数据，支持全角 CJK 自适应终端对齐、智能相对时间计算（刚刚/X分钟前/昨天）、今日过滤与一键复制会话 ID，指导 Agent 无缝接续跨端、跨会话工作上下文。

---

## 📱 诞生背景与痛点破解：跨端无缝接续（电脑桌面 ⇄ 移动端 IM）

### 1. 真实业务痛点
随着企业将 ZCode / AI Coding Agent 接入即时通讯软件（如**企业微信、飞书、钉钉、Slack 等移动端 IM**），一个高频且令人抓狂的体验断层随之出现：
- **跨设备上下文黑洞**：工程师在办公室电脑桌面端用 ZCode 调试了一下午代码、梳理了复杂的重构架构或排查方案。合上电脑离开工位后（在通勤途中、会议室或居家），拿出手机在企业微信/钉钉里想继续跟同一个 AI 推进任务。
- **对话割裂与“失忆”**：移动端 IM 接入的每次对话往往是一个全新的会话实例，**手机端根本无法读取电脑本地桌面端的历史上下文**。AI 宛如初见，对几分钟前在电脑上讨论的设计细节一无所知。
- **极高的重复录入成本**：在手机巴掌大的屏幕上打字繁琐费时，工程师被迫重新组织长篇大论向 AI 复述背景、粘贴代码片段，导致移动办公协同体验极其痛苦。
- **本地数据库壁垒**：ZCode 桌面端的全量会话历史保存在本地 SQLite 中，无法直接被移动端 IM 实例感知。

---

### 2. 破局方案：`zcode-session-manager` 作为“跨端记忆中继器”

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

---

## 🚀 核心特性与亮点

- ⚡ **内核级极速响应**：启用 256MB SQLite 内存映射（I/O mmap）与只读安全事务，全表毫秒级无感返回（< 20ms），绝不锁库冲突。
- 🎨 **终端丝滑对齐**：自适应当前终端列宽，内置标准库 CJK 全角中文字符对齐算法，超长标题平滑智能截断，**彻底告别折行爆屏**。
- 🕒 **人性化时序**：支持智能相对时间计算（“刚刚”、“40分钟前”、“昨天 17:41”），一眼掌握最近会话活跃状态。
- 📋 **一键提取与复制**：提供 `-c / --copy <序号>`，直接打印对应会话 ID，方便在手机端或 CLI 中一键输入“切到该会话”。
- 🔒 **纯离线与脱敏安全**：纯本地操作，零外部依赖，示例完全通用化脱敏。

---

## 📂 目录结构

```text
zcode-session-manager/
├── SKILL.md                 # 技能定义与规范文档（供 AI 加载与决策使用）
├── README.md                # 技能说明与使用指南（本文件）
└── scripts/
    └── list_sessions.py     # 极速丝滑的 SQLite 查询引擎（纯标准库实现）
```

---

## 🤖 极简交付：让 AI 帮您一键安装（强烈推荐）

无需繁琐地手动复制或翻找隐藏目录，只需将 `zcode-session-manager` 目录放在任意工作区，直接将以下提示词发送给您的 **ZCode Agent**，AI 将全自动完成部署与测试：

### 提示词模板 1：全局用户安装（推荐，所有项目通用）
```text
请帮我安装当前目录下的 zcode-session-manager 技能：将其复制部署到我的全局技能目录（~/.zcode/skills/zcode-session-manager/），确保包含 SKILL.md 与 scripts/list_sessions.py，并测试运行 list_sessions.py -n 3 验证安装成功。
```

### 提示词模板 2：当前工程专属安装（随项目代码管理）
```text
请帮我把当前目录下的 zcode-session-manager 部署到当前工作区的 .zcode/skills/zcode-session-manager/ 目录中，并验证文件完整性。
```

> **AI 自动化执行流**：Agent 收到指令后，会自动创建对应目标目录、复制必要文件、校验脚本执行权限并进行健康检查（Smoke Test），整个过程 2 秒内完成。

---

## 🛠️ 手动安装指南

### 方式 1：全局用户级安装（所有工作区通用，推荐）
直接将 `zcode-session-manager` 目录复制至当前用户主目录下的 `.zcode/skills/`：
- **Windows**: `%USERPROFILE%\.zcode\skills\zcode-session-manager\`
- **Linux / macOS**: `~/.zcode/skills/zcode-session-manager/`

### 方式 2：工作区专属安装
直接将 `zcode-session-manager` 目录复制至目标工程根目录下的 `.zcode/skills/`：
```bash
./.zcode/skills/zcode-session-manager/
```

---

## 💻 命令行速查 (CLI Cheat Sheet)

```bash
# 1. 默认极速浏览 (最新 25 条，终端自适应高亮 + 相对时间)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py

# 2. 仅看今天活跃会话
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -t

# 3. 关键字模糊搜索（按标题或 Session ID 过滤，命中词高亮）
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -s "重构"

# 4. 快捷提取 ID：直接打印第 1 个会话的 ID (适合快速复制或命令管道)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -c 1

# 5. 指定拉取数量 (如最新 50 条)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -n 50

# 6. 输出标准 Markdown 表格 (适合直接复制给大模型或写入文档)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py --md

# 7. 输出标准 JSON 格式 (适合工具链与程序自动化)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py --json

# 8. 禁用色彩高亮 (输出纯文本)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py --no-color
```

---

## 🤖 与 AI 对话中的自然语言触发

在手机 IM 或桌面 ZCode 对话中，您可随时通过自然语言调用：
- `“看看刚才电脑在聊什么”` / `“列出今天下午的会话”`
- `“帮我查找关于 <关键词> 的会话”`
- `“接续第 1 个会话继续工作”`
- `“读取会话 sess_xxx 的上下文背景”`
