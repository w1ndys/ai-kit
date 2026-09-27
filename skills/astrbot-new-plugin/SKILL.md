---
name: astrbot-new-plugin
description: 按官方插件开发文档和 W1ndys 现仓惯例新建独立 AstrBot 插件。配置优先走 WebUI。用户说新建 AstrBot 插件、开一个 astrbot_plugin_、插件页面或 WebUI 配置时使用。
---

# 新建 AstrBot 插件

按官方插件开发文档落地独立仓库，再叠本 skill 钉住的分层和 WebUI 配置。不要从群规仓整棵复制业务，也不要默认做群消息命令。

用中文和用户交流。缺插件短名时先问，不要猜一个名字开仓。

## 何时加载

用户说新建 AstrBot 插件、开一个 `astrbot_plugin_`、按现有插件惯例起仓、给插件做 WebUI 配置时加载。

不要用本 skill 去改 AstrBot 核心，也不要把业务源码写进 `dev-cycle`。

## 冲突时听谁的

1. **官方文档**：API、目录名、`main.py` / `metadata.yaml`、存储路径、Pages、配置 schema、过滤器语义、热重载。写之前打开对应文档，以文档当前文本为准。不要把官方全书抄进本 skill，也不要凭记忆写 API。
2. **本 skill**：产品形态。一插件一仓、配置优先 WebUI、开关默认关、不抽跨插件公共模块。群消息命令不是新插件的默认入口。

## 官方开发文档

源码目录：[AstrBotDevs/AstrBot](https://github.com/AstrBotDevs/AstrBot) 的 `docs/zh/dev/star/`。

写插件前先打开和这次改动相关的篇目：

- 总览：https://docs.astrbot.app/dev/star/plugin-new.html
- 最小实例：https://docs.astrbot.app/dev/star/guides/simple.html
- 插件配置（少量标量优先这个）：https://docs.astrbot.app/dev/star/guides/plugin-config.html
- 插件 Pages（列表增删改查优先这个）：https://docs.astrbot.app/dev/star/guides/plugin-pages.html
- 存储：https://docs.astrbot.app/dev/star/guides/storage.html
- 发消息，含主动推送和 `unified_msg_origin`：https://docs.astrbot.app/dev/star/guides/send-message.html
- 消息事件：https://docs.astrbot.app/dev/star/guides/listen-message-event.html
- 会话控制：https://docs.astrbot.app/dev/star/guides/session-control.html
- AI / llm_tool：https://docs.astrbot.app/dev/star/guides/ai.html
- 发布：https://docs.astrbot.app/dev/star/plugin-publish.html
- 官方模板：https://github.com/Soulter/helloworld

旧指南（v4.5.7 后停更，只作对照）：https://docs.astrbot.app/dev/star/plugin.html

## 官方已经写死、必须遵守

- 仓名：`astrbot_plugin_` 前缀、全小写、无空格、尽量短。
- 开发时 clone 进 `AstrBot/data/plugins/<插件名>`，改完在 WebUI 插件页热重载。
- 必须改 `metadata.yaml`（AstrBot 认这个）。可写 `display_name` / `short_desc` / `support_platforms` / `astrbot_version`（PEP 440，不要加 `v`）。
- 插件类继承 `Star`，写在 `main.py`。消息 Handler 前两个参数必须是 `self, event`。
- 日志用 `from astrbot.api import logger`，不要用 `logging`。
- 大文件 / 业务库放 `data/plugin_data/{plugin_name}/`。用 `StarTools.get_data_dir()`。KV（`put_kv_data`）只适合极少临时数据，不当业务表。
- 不要用 `requests`，用 `aiohttp` / `httpx`。有第三方依赖才写 `requirements.txt`。
- 插件页面放 `pages/<page_name>/index.html`。脚本必须 `type="module"`，通过 `window.AstrBotPluginPage` 调后端。后端用 `context.register_web_api(route, handler, methods, desc)`，路由必须带插件名前缀。页面 endpoint 不带插件名。请求和响应用 `astrbot.api.web` 的 `request` / `json_response` / `error_response`，不要把 Quart 原始对象当新插件的公共 API。
- 主动推送用 `event.unified_msg_origin` 记下的会话串，或官方格式 `platform_id:GroupMessage:group_id`，再 `context.send_message(umo, MessageChain)`。平台 ID 含冒号时不要自己拼。
- 密钥类配置在 `_conf_schema.json` 标 `"secret": true`。
- 只有产品本身必须接消息时才写消息 Handler。`@filter.command` 不能带空格。事件钩子不能跟 command / `event_message_type` 叠在一起；钩子里发消息用 `event.send()`，不能 `yield`。
- `llm_tool` 把结果 `return str` 给模型。docstring 必须有 `Args:` 段，格式 `参数名(类型): 描述`。

## 配置优先 WebUI

新插件的配置和业务数据不要做成群口令。

| 数据 | 放哪 | 文档 |
|---|---|---|
| 超时、平台 ID、开关默认值这类全局标量 | `_conf_schema.json` | [插件配置](https://docs.astrbot.app/dev/star/guides/plugin-config.html) |
| 群列表、规则表、需要增删改查的业务行 | SQLite + `pages/<page_name>/` | [插件 Pages](https://docs.astrbot.app/dev/star/guides/plugin-pages.html) |
| 密钥 | `_conf_schema.json`，标 `secret: true` | 同上 |

`_conf_schema.json` 不放业务表。表进 SQLite。热路径读内存快照；写的时候先落库再改内存，落库失败整条不算。表里没有记录就当关闭。

群消息命令只在用户明确要求「在群里用口令操作」时才加。旧群规仓的「某某 开 / 某某 关」不要复制到新插件。新插件默认没有这类口令。

## 对照现仓，不要每次从群规仓抄

| 仓库 | 形态 | 怎么对待 |
|---|---|---|
| [astrbot_plugin_w1ndys_rules](https://github.com/w1ndys/astrbot_plugin_w1ndys_rules) | 旧群规，群口令开关 | 只作历史对照。新插件不要抄它的开/关口令 |
| [astrbot_plugin_prompt_ctf](https://github.com/w1ndys/astrbot_plugin_prompt_ctf) | 代码判胜负 | 输赢不交给模型。配置仍优先 WebUI |
| [astrbot_plugin_qq_agent](https://github.com/w1ndys/astrbot_plugin_qq_agent) | NL 群管 | 群管沿用这个仓，不要为同一套 OneBot 群管再开仓 |
| [astrbot_plugin_codex_reset](https://github.com/w1ndys/astrbot_plugin_codex_reset) | WebUI 维护开启群，后台推送 | 新插件配置页按这个方向做 |
| [w1ndys-astrbot-plugins](https://github.com/w1ndys/w1ndys-astrbot-plugins) | 已归档 | **一插件一仓库**，不要回去 |

旧 skill 仓 [w1ndys/skills](https://github.com/w1ndys/skills) 已迁入本仓 `skills/`。新 skill 也写在这里，不要写回旧仓。

## 脚手架步骤

1. 问清插件短名和一句话职责。仓名 `astrbot_plugin_<name>`。
2. 新建**独立** GitHub 仓库，不要推进已归档 monorepo。
3. 按下面骨架落文件。插件类必须在 `main.py`。配置走 WebUI，不先写群口令。
4. 从本仓拷 `human-coding-contract` 到新仓 `.agents/skills/` 和 `.ohmyagent/skills/`，`AGENTS.md` 写明本仓默认启用，不必每次喊触发词。
5. 开发时把仓 clone 进 `AstrBot/data/plugins/<插件名>`。改完 WebUI 重载。
6. 做完在 `dev-cycle/projects/<repo-name>/` 登记，并更新根索引。周期说明按 `dev-cycle` skill，不要把业务源码写进那个仓。

```text
astrbot_plugin_<name>/
  entity/             固定值、数据形状
  data/               SQLite + 内存快照
  business/           校验、匹配、文案
  main.py             注册 WebUI API，必要时才接消息
  pages/<page_name>/  有业务表时的增删改查页
  metadata.yaml
  _conf_schema.json   仅全局标量，不放业务表
  requirements.txt    有第三方依赖才写
  tests/
  AGENTS.md
  __init__.py
```

`_shared/` 只放**本插件内**重复底座（例如本仓自己的 SQLite connect）。每个插件自己管自己的表，不抽跨插件公共模块。

## 分层

入口不直接查库。依赖单向：`main.py` → `business/` → `data/`。`entity/` 不依赖 AstrBot，也不访问数据库。

| 层 | 目录 | 职责 |
|---|---|---|
| 实体 | `entity/` | 固定值和数据形状 |
| 数据 | `data/`、可选 `_shared/` | SQLite 与内存快照 |
| 业务 | `business/` | 校验、匹配、文案 |
| 入口 | `main.py`、`pages/` | WebUI API 和页面。只有必须接消息时才写 Handler |

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

## 入口骨架（WebUI）

```python
from astrbot.api.star import Context, Star
from astrbot.api.web import error_response, json_response, request

PLUGIN_NAME = "astrbot_plugin_<name>"


class Plugin(Star):
    def __init__(self, context: Context, config=None) -> None:
        super().__init__(context)
        self.config = config
        context.register_web_api(
            f"/{PLUGIN_NAME}/items",
            self.page_list_items,
            ["GET"],
            "列出业务行",
        )

    async def page_list_items(self):
        return json_response({"items": []})
```

页面：

```html
<script type="module" src="./app.js"></script>
```

```javascript
const bridge = window.AstrBotPluginPage;
await bridge.ready();
const result = await bridge.apiGet("items");
```

主动推送见[发消息](https://docs.astrbot.app/dev/star/guides/send-message.html)。先有 `unified_msg_origin` 或可拼的平台 ID，再 `context.send_message`。

## 消息和模型

产品必须接群消息时才写 `@filter.event_message_type`。命中后的业务回复用 `yield event.plain_result(...)`，然后 `event.stop_event()`。斜杠开头的消息放过。

`group_id` / 发送者只信当前会话，不信模型填的 ID。管理员判定用 `event.is_admin()`；没有这个方法或抛异常，默认拒绝。

`llm_tool` 只在用户明确要求自然语言工具时加，并且必须 `return str`。群管不要新开仓。

## 工程

- 仓内默认启用 `human-coding-contract`（写进 `AGENTS.md`）。
- 提交前跑 `python3 -m unittest`。现仓还没有 ruff 配置时，不要假装跑过 ruff。
- 测试用 `unittest` + 临时目录 SQLite。不要连真的 AstrBot。
- 密钥、token、密码不进代码、注释、日志、diff、提交信息。
- `__init__.py` 只标明这是插件包，真正注册在 `main.py`。
- `.gitignore` 忽略 `__pycache__/`、`*.db`、虚拟环境；**不要**忽略 `.agents/` 和 `.ohmyagent/`。
- 提交信息中文，格式 `类型(内容): 中文描述`。未经用户明确允许不要 push。

## 不要写进本 skill / 不要写进新仓的

业务词库、密钥、部署主机路径、某个插件的表结构细节。那些留在各项目 README 和 WebUI。

## 验收清单

开出来的插件必须同时满足：

- [ ] 独立仓库，名符合 `astrbot_plugin_`；不是 monorepo 子目录
- [ ] 有 `main.py`（插件类在此）和已改过的 `metadata.yaml`
- [ ] `entity/` / `data/` / `business/` / `main.py` 分层；入口不直接查库
- [ ] 配置优先 WebUI：标量在 `_conf_schema.json`，业务行在 Pages，而不是群口令
- [ ] `StarTools.get_data_dir()` 下自建 SQLite；没记录等于关
- [ ] `AGENTS.md` 默认启用 human-coding-contract
- [ ] `tests/` 里至少有一个能跑的桩（`python3 -m unittest`）
- [ ] `_conf_schema.json` 没有业务表；密钥项有 `secret: true`
- [ ] 没引入 `requests`；没抽跨插件公共包
- [ ] 写过的 API 能在上面的官方文档里对上
- [ ] 已在 `dev-cycle/projects/<repo-name>/` 登记
- [ ] 没违反官方 `main.py` / `metadata.yaml` / `data/plugin_data/{plugin_name}/` / Pages 约定
