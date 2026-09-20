---
name: astrbot-new-plugin
description: 按 W1ndys 已有 AstrBot 插件惯例 + 官方插件开发文档新建独立插件仓。用户说新建 AstrBot 插件、开一个 astrbot_plugin_、群功能插件脚手架时使用。
---

# 新建 AstrBot 插件

按官方插件开发文档落地独立仓库，再叠我们现仓已经钉住的分层、消息路径和权限。不要从群规仓整棵复制业务。

用中文和用户交流。缺插件名、缺「确定性群规 / NL 工具」类型时先问，不要猜一个名字开仓。

## 何时加载

用户说新建 AstrBot 插件、开一个 `astrbot_plugin_`、群功能插件脚手架、按现有插件惯例起仓时加载。

不要用本 skill 去改 AstrBot 核心，也不要把业务源码写进 `dev-cycle`。

## 冲突时听谁的

1. **官方**：API、目录名、`main.py` / `metadata.yaml`、存储路径、过滤器语义、热重载。
2. **现仓**：产品形态。一插件一仓、群规不经 LLM、开关默认关、不抽跨插件公共模块。

官方 API 全书不要抄进本 skill。写之前先打开对应文档，以仓库当前文本为准。

## 先读哪些官方文档

源码：[AstrBotDevs/AstrBot](https://github.com/AstrBotDevs/AstrBot) 的 `docs/zh/dev/star/`。线上入口：

- 新指南（主文档）：https://docs.astrbot.app/dev/star/plugin-new.html
- 最小实例：https://docs.astrbot.app/dev/star/guides/simple.html
- 消息事件 / 指令 / 过滤器 / `stop_event`：https://docs.astrbot.app/dev/star/guides/listen-message-event.html
- 发消息：https://docs.astrbot.app/dev/star/guides/send-message.html
- 插件配置：https://docs.astrbot.app/dev/star/guides/plugin-config.html
- 存储：https://docs.astrbot.app/dev/star/guides/storage.html
- 插件 Pages：https://docs.astrbot.app/dev/star/guides/plugin-pages.html
- 会话控制：https://docs.astrbot.app/dev/star/guides/session-control.html
- AI / llm_tool：https://docs.astrbot.app/dev/star/guides/ai.html
- 发布：https://docs.astrbot.app/dev/star/plugin-publish.html
- 官方模板：https://github.com/Soulter/helloworld

旧指南（v4.5.7 后停更，只作对照）：https://docs.astrbot.app/dev/star/plugin.html

## 官方已经写死、必须遵守

- 仓名：`astrbot_plugin_` 前缀、全小写、无空格、尽量短。
- 开发时 clone 进 `AstrBot/data/plugins/<插件名>`，改完在 WebUI 插件页热重载。
- 必须改 `metadata.yaml`（AstrBot 认这个）。可写 `display_name` / `short_desc` / `support_platforms` / `astrbot_version`（PEP 440，不要加 `v`）。
- 插件类继承 `Star`，写在 `main.py`。Handler 前两个参数必须是 `self, event`。
- 日志用 `from astrbot.api import logger`，不要用 `logging`。
- 大文件 / 业务库放 `data/plugin_data/{plugin_name}/`。用 `StarTools.get_data_dir()`（官方封装，等价于该路径）。KV（`put_kv_data`）只适合极少配置或临时数据，不当业务表。
- 不要用 `requests`，用 `aiohttp` / `httpx`。有第三方依赖才写 `requirements.txt`。
- `@filter.command` 不能带空格；需要「出现某词就回」必须自己用 `event_message_type` 解析。
- 业务命中后要拦 LLM：`event.stop_event()`。
- 事件钩子（`on_llm_request` 等）不能跟 command / `event_message_type` 叠在一起；钩子里发消息用 `event.send()`，不能 `yield`。
- `llm_tool` 把结果 `return str` 给模型；`yield event.plain_result` 会直接发给用户，模型拿到空结果。
- 密钥类配置在 `_conf_schema.json` 标 `"secret": true`。
- `llm_tool` 的 docstring 必须有 `Args:` 段，格式 `参数名(类型): 描述`。装饰器只解析注释，不读类型注解。

## 对照现仓，不要每次从群规仓抄

| 仓库 | 形态 | 钉住的惯例 |
|---|---|---|
| [astrbot_plugin_w1ndys_rules](https://github.com/w1ndys/astrbot_plugin_w1ndys_rules) | 确定性群规 | 群员走代码、不经 LLM；开关默认关；管理开/关/批量无唤醒前缀；单条 NL CRUD 要唤醒；`group_id` 只信会话 |
| [astrbot_plugin_prompt_ctf](https://github.com/w1ndys/astrbot_plugin_prompt_ctf) | 代码判胜负 | 学生/管理命令都走代码；群默认关；输赢不交给模型 |
| [astrbot_plugin_qq_agent](https://github.com/w1ndys/astrbot_plugin_qq_agent) | NL 群管 | 工具必须 `return str`；发模型前按权限摘工具 |
| [w1ndys-astrbot-plugins](https://github.com/w1ndys/w1ndys-astrbot-plugins) | 已归档 | **一插件一仓库**，不要回去 |

旧 skill 仓 [w1ndys/skills](https://github.com/w1ndys/skills) 已迁入本仓 `skills/`。新 skill 也写在这里，不要写回旧仓。

## 脚手架步骤

1. 问清插件短名、一句话职责、类型（见「两类插件」）。仓名 `astrbot_plugin_<name>`。
2. 新建**独立** GitHub 仓库，不要推进已归档 monorepo。
3. 按下面骨架落文件。插件类必须在 `main.py`。
4. 从本仓拷 `human-coding-contract` 到新仓 `.agents/skills/` 和 `.ohmyagent/skills/`，`AGENTS.md` 写明本仓默认启用，不必每次喊触发词。
5. 开发时把仓 clone 进 `AstrBot/data/plugins/<插件名>`。改完 WebUI 重载。
6. 做完在 `dev-cycle/projects/<repo-name>/` 登记，并更新根索引。周期说明按 `dev-cycle` skill，不要把业务源码写进那个仓。

```text
astrbot_plugin_<name>/
  entity/             固定值、数据形状
  data/               SQLite + 内存快照
  business/           权限、校验、匹配、文案
  main.py             过滤器 / 指令 / llm_tool，只取群号再调业务
  metadata.yaml
  _conf_schema.json   仅全局标量（开关、私钥名），不放业务表
  requirements.txt    有第三方依赖才写
  tests/
  AGENTS.md
  __init__.py
```

`_shared/` 只放**本插件内**重复底座（例如本仓自己的 SQLite connect、本仓自己的群开关表）。每个插件自己管功能开关，不抽跨插件公共模块。

## 分层

入口不直接查库。依赖单向：`main.py` → `business/` → `data/`。`entity/` 不依赖 AstrBot，也不访问数据库。

| 层 | 目录 | 职责 |
|---|---|---|
| 实体 | `entity/` | 固定值和数据形状 |
| 数据 | `data/`、可选 `_shared/` | SQLite 与内存快照 |
| 业务 | `business/` | 权限、校验、匹配、文案 |
| 入口 | `main.py` | 过滤器、指令、llm_tool；只取群号/QQ 并调用业务层 |

设计顺序：实体 → 数据 → 业务 → 入口。函数（不含空行和注释）≤ 50 行。每个文件、每个函数、每个 `if` 写短中文注释（判断目的和后果）。

## metadata 我们的默认值

```yaml
name: astrbot_plugin_<name>
display_name: <中文短名>
short_desc: <一句话>
desc: |
  <行为说明，面向使用者>
version: 0.1.0
author: w1ndys
repo: https://github.com/w1ndys/astrbot_plugin_<name>
astrbot_version: ">=4.13"
support_platforms:
  - aiocqhttp
```

## 消息路径

群员日常命中用 `@filter.event_message_type(GROUP_MESSAGE)`，命中后 `event.stop_event()`，不经 LLM。

斜杠开头的消息放过，不当本插件关键词/命令。

关键词 / 管理口令匹配用 `event.message_str`（纯文本）。记住：AstrBot 在唤醒检查阶段会把唤醒前缀从开头剥掉。关键词本身不能以唤醒前缀开头，否则永远匹配不到；写入时拦下并说明原因。

`@filter.command` 不能带空格。中文「某某 开」这类口令必须自己解析，不能注册成 command。

群默认关。管理员「某某 开 / 某某 关」无唤醒前缀；群员误发静默。开/关必须整条消息完全相等，避免把后面的闲聊当命令。

确定性 Handler 回用户用 `yield event.plain_result(...)`，然后停事件。

## 存储

```python
from pathlib import Path
from astrbot.api.star import StarTools

db_path = Path(StarTools.get_data_dir()) / "plugin.db"
```

热路径读内存快照；写的时候先落库再改内存，落库失败整条不算。表里查不到开关就当关闭。

`_conf_schema.json` 只放全局标量（判断准则、私钥、超时秒数）。业务表进 SQLite，不要塞进 WebUI schema。密钥项标 `"secret": true`，不要在日志、回报、提交信息里回显。

KV 存储不当业务表。

需要复杂表单或测试页再用 Pages：`pages/<page_name>/index.html`，脚本必须 `type=module` 才能等 AstrBot 注入 `window.AstrBotPluginPage`。后端用 `context.register_web_api` + `astrbot.api.web`。少量配置优先 `_conf_schema.json`。

## 权限

`group_id` / 发送者只信当前会话（`event.get_group_id()` / `event.get_sender_id()`），不信模型填的 ID。私聊没有群号就拒绝群功能。

管理员判定用 `event.is_admin()`；没有这个方法或抛异常，默认拒绝。

非管理员的工具列表**不清空**。权限统一在 `business/auth.py` 拦，工具返回拒绝文案，库可证明没有落库。

NL 写操作直接落库，不做二次确认。回报文案要把最终写入的关键值括起来，避免模型改写成口语时把值弄丢。

列表/查询走 tool 查库，不指望模型记住全表。

发模型前若需要按权限摘工具（群管类），在 `on_llm_request` 里 `req.func_tool.remove_tool(...)`。钩子不能 `yield`。

## 两类插件，不要混在一个仓

**确定性（群规 / 活动 / 判胜负）**

- 群员命中、开/关/批量、学生命令：代码 Handler，命中后停事件。
- 管理员单条增删改查可以挂 `llm_tool`，但要唤醒前缀才进 Agent。
- 输赢、是否命中、是否落库，不交给模型。

**NL 工具（群管）**

- 动作做成 `llm_tool`，必须 `return str`，不能 `yield`。
- 没权限的人在发模型前看不到这些工具（摘工具），工具内部仍要再鉴权。

一个仓只选一种主形态。群管沿用 `astrbot_plugin_qq_agent`，不要为同一套 OneBot 群管再开仓。

## 入口骨架（确定性）

```python
from pathlib import Path

from astrbot.api import logger
from astrbot.api.event import AstrMessageEvent, filter
from astrbot.api.star import Context, Star, StarTools


class Plugin(Star):
    def __init__(self, context: Context, config=None) -> None:
        super().__init__(context)
        self.config = config
        db_path = Path(StarTools.get_data_dir()) / "plugin.db"
        logger.info("[plugin] 业务库已载入：%s", db_path)

    @filter.event_message_type(filter.EventMessageType.GROUP_MESSAGE)
    async def on_group_message(self, event: AstrMessageEvent):
        group_id = event.get_group_id()
        if not group_id:
            return
        text = event.message_str
        if not text or text.startswith("/"):
            return
        # 交给 business：开/关、命中、文案。命中才 yield 并 stop_event。
```

`llm_tool` 只 `return str`：

```python
@filter.llm_tool(name="example_add")
async def tool_example_add(self, event: AstrMessageEvent, keyword: str) -> str:
    """给本群新增一条规则。只在管理员明确要求时调用。

    Args:
        keyword(string): 要写入的关键词
    """
    return await add_rule(event, event.get_group_id(), keyword)
```

## 工程

- 仓内默认启用 `human-coding-contract`（写进 `AGENTS.md`）。
- 提交前跑 `python3 -m unittest`。现仓还没有 ruff 配置时，不要假装跑过 ruff；官方建议 ruff，等仓里真有配置再跑。
- 测试用 `unittest` + 临时目录 SQLite；用 FakeEvent / FakeContext 冒充 `is_admin` 和唤醒前缀，不要连真的 AstrBot。
- 密钥、token、密码不进代码、注释、日志、diff、提交信息。
- `__init__.py` 只标明这是插件包，真正注册在 `main.py`。
- `.gitignore` 忽略 `__pycache__/`、`*.db`、虚拟环境；**不要**忽略 `.agents/` 和 `.ohmyagent/`。
- 提交信息中文，格式 `类型(内容): 中文描述`。未经用户明确允许不要 push。

## 不要写进本 skill / 不要写进新仓的

业务词库、密钥、部署主机路径、某个插件的表结构细节。那些留在各项目 README 和 WebUI。

## 验收清单

开出来的空插件必须同时满足：

- [ ] 独立仓库，名符合 `astrbot_plugin_`；不是 monorepo 子目录
- [ ] 有 `main.py`（插件类在此）和已改过的 `metadata.yaml`
- [ ] `entity/` / `data/` / `business/` / `main.py` 分层；入口不直接查库
- [ ] `StarTools.get_data_dir()` 下自建 SQLite 目录；热路径可先只有空快照
- [ ] 群开关默认关（没记录 = 关）
- [ ] `AGENTS.md` 默认启用 human-coding-contract
- [ ] `tests/` 里至少有一个能跑的桩（`python3 -m unittest`）
- [ ] `_conf_schema.json` 没有业务表；密钥项有 `secret: true`
- [ ] 没引入 `requests`；没抽跨插件公共包
- [ ] 已在 `dev-cycle/projects/<repo-name>/` 登记
- [ ] 没违反官方 `main.py` / `metadata.yaml` / `data/plugin_data/{plugin_name}/` 约定
