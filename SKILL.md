---
name: zcode-session-manager
description: |
  检索、列举与快速切换 ZCode 历史会话（Session）。
  当用户询问“列出会话”、“会话列表”、“历史对话有哪些”、“查看会话标题”、“切换到某会话”、“读取某会话记录”时调用此技能。
  该技能提供秒级查询本地会话数据库提取所有会话 ID、标题(Title)、工作区与时间的能力，并配合 ReadSessionContext 工具实现无缝上下文接续。
license: MIT
metadata:
  version: "1.2.0"
---

# ZCode Session Manager (zcode-session-manager)（ZCode 历史会话管理与切换技能）

本技能用于在 ZCode 中秒级检索历史会话列表（包含会话 ID、标题、所属工作区、智能相对时间），并指导 Agent 快速无缝地读取并切换到指定会话。

---

## 核心特性

- ⚡ **极致性能**：启用 256MB SQLite 内存映射（I/O mmap）与只读安全模式，响应低于 20ms，零锁库冲突。
- 🎨 **终端丝滑排版**：自适应终端列宽，集成 CJK 全角中文字符宽度精确对齐算法，超长标题自动平滑截断，绝不折行爆屏。
- 🕒 **人性化时序**：支持相对时间智能计算（“刚刚”、“X分钟前”、“X小时前”、“昨天 15:30”），一目了然定位最近会话。
- 📋 **极速提取**：支持 `-c / --copy <序号>` 直接输出指定会话 ID，实现一键复制与切换。

---

## 核心能力与工作流

### 1. 秒级列出/检索历史会话

系统中的会话元数据直接存储在本地 SQLite 数据库中：`~/.zcode/cli/db/db.sqlite` 的 `session` 表。

#### 推荐方式：直接执行随附的高性能查询脚本
本技能内置了预编译的查询脚本，无需重复探索数据库路径：

```bash
# 1. 默认极速浏览 (最新 25 条，终端自适应高亮 + 相对时间)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py

# 2. 仅看今天活跃会话
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -t

# 3. 关键字模糊搜索（按标题或 Session ID 过滤，命中词高亮）
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -s "重构"

# 4. 快捷提取 ID：直接输出第 1 个会话的 ID (适合快速复制或命令串联)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -c 1

# 5. 指定显示数量
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py -n 50

# 6. 输出标准 Markdown 表格格式 (适合文档记录或 Agent 文本引用)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py --md

# 7. 输出标准 JSON 格式 (适合程序自动化处理)
python ~/.zcode/skills/zcode-session-manager/scripts/list_sessions.py --json
```

#### 脚本输出示例（终端自适应卡片视图）
```text
⚡ ZCode 会话管理器 (共 5 条会话)
#     会话标题 (Title)                                                更新时间      工作区            会话 ID (Session ID)
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
1     后端 API 性能优化与重构                                         刚刚          backend-service   sess_a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d
2     前端组件库规范设计                                              40分钟前      frontend-web      sess_b2c3d4e5-f6a7-4b8c-9d0e-1f2a3b4c5d6e
3     微服务链路追踪排查                                              昨天 17:41    backend-service   sess_c3d4e5f6-a7b8-4c9d-0e1f-2a3b4c5d6e7f
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
💡 切换提示: 复制上方任意 Session ID，在对话中输入 '切到该会话' 即可无缝继承上下文。
```

---

### 2. 切换并读取指定会话上下文

当用户确认需要“切换到某个会话”或“读取某个会话”的内容时，**严禁自行手动逐行读取 log/rollout 文件**，直接使用内置的 `ReadSessionContext` 工具：

#### 调用规范：
1. **工作交接 / 继续上一次工作**：
   ```json
   {
     "sessionId": "sess_xxxx-xxxx-xxxx",
     "strategy": "handoff",
     "query": "获取上一会话的完整背景、讨论内容、已完成的工作以及后续待办交接信息"
   }
   ```
2. **特定问题查询 / 局部背景参考**：
   ```json
   {
     "sessionId": "sess_xxxx-xxxx-xxxx",
     "strategy": "relevant",
     "query": "用户关于<某个主题>的具体要求和结论是什么"
   }
   ```

---

## 原理与数据库结构（备忘）
- **数据库路径**：`~/.zcode/cli/db/db.sqlite`
- **关键表**：`session`
  - `id`: 会话唯一标识（如 `sess_a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d`）
  - `title`: 会话标题（如 `后端服务架构调优`）
  - `directory`: 会话所在的工作目录绝对路径
  - `time_updated`: 最近更新时间（毫秒时间戳）
  - `time_created`: 创建时间（毫秒时间戳）
- **查询模式**：开启 `PRAGMA query_only=ON`、`PRAGMA mmap_size=268435456` 与 `file:path?mode=ro` 只读 URI，亚毫秒级无感返回，绝对无锁冲突。
