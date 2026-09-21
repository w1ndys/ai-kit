# AstrBot 解析出视频，群里却只看到简介

AstrBot + NapCat 分两个容器部署时，链接解析插件能把 B 站视频下下来，但发到群里只剩标题和简介。看起来像解析器没做发原视频，其实是跨容器读文件失败后的文本兜底。

## 现象

群里丢 B 站小程序卡片、短链或 BV 号，机器人回复只有文本描述，没有视频。

容易误判成：

- 小程序卡片没解析到链接
- 插件配置没开「发原视频」
- 没登录 B 站，下不到高画质

这几条都不是这次的原因。插件本来就会下载 mp4，再用 `Video` 发出去。没有单独的「发不发原视频」开关。

## 日志

AstrBot 侧能看到下载成功，紧接着发送失败：

```text
Merged BV....mp4, 大小: 2.18 MB
发送解析结果失败： ENOENT: no such file or directory
open '/AstrBot/data/plugin_data/astrbot_plugin_parser/cache/BV....mp4'
```

发送器失败后走文本兜底，把标题和简介发到群里。所以群里「只有描述」。

## 原因

两个容器文件系统是分开的。

| 容器 | 挂载 | 能不能看到这份 mp4 |
| --- | --- | --- |
| astrbot | 宿主机 data → `/AstrBot/data` | 能。文件就写在这里 |
| napcat | 只有 QQ 数据、config、plugins | 不能 |

AstrBot 把**自己容器内的路径**交给 NapCat。NapCat 在自己容器里按同一路径去 `open`，找不到文件，报 `ENOENT`。

路径必须一字不差。挂到 NapCat 的 `/app/...` 没用，因为它打开的是 `/AstrBot/data/...`。

这和入口形式无关。小程序卡片、短链、BV 号最后都走同一条发视频路径，都会踩这个坑。

## 处理

给 NapCat 只读挂上同一份 AstrBot data，容器内路径对齐：

```yaml
# napcat docker-compose.yml
volumes:
  - ./data/QQ:/app/.config/QQ
  - ./config:/app/napcat/config
  - ./plugins:/app/napcat/plugins
  - /opt/astrbot/data:/AstrBot/data:ro
```

注意：

- 容器内目标必须是 `/AstrBot/data`，不能改名
- `:ro` 够用，NapCat 只读文件
- 改的是 volume，`docker restart` 不够，要用 `docker compose up -d` 重建
- 重建后 QQ 会短暂重连

验证：

```bash
docker exec napcat ls /AstrBot/data/plugin_data/astrbot_plugin_parser/cache
```

能列出 mp4 之后，再丢一条 B 站链接，应直接出视频，不再只剩简介。

## 不是这次的问题

日志里如果还有「哔哩哔哩凭证缺少 SESSDATA」，只影响高画质和 AI 总结。已经 `Merged ... mp4` 说明下载成功，缺 cookie 解释不了「只发简介」。
