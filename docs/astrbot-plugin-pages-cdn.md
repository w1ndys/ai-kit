# AstrBot 插件 Pages：打成一份 IIFE

AstrBot 把插件页放进受限 iframe：`sandbox="allow-scripts allow-forms allow-downloads"`，**没有** `allow-same-origin` / `allow-top-navigation`。静态资源靠短期 `asset_token`。宿主只稳定改写：

- HTML 的 `src` / `href`
- JS 里带空格的 `from "./x.js"`（压缩后的 `from"./x"` **对不上**）

所以运行时不要再走 CDN，也不要 ESM 拆成多文件互相 import。

参考实现：[astrbot_plugin_qq_group_daily_analysis](https://github.com/SXP-Simon/astrbot_plugin_qq_group_daily_analysis) 的 `dashboard/vite.config.ts`。

## 正确做法

源码放 `dashboard/`。Vite 打到 `pages/<page>/`：

- `base: "./"`
- `format: "iife"` + `inlineDynamicImports: true`
- 构建后把 `type="module"` 改成 `<script defer src="./assets/index.js">`
- 去掉 `crossorigin`

产物只有：

```html
<script defer src="./assets/index.js"></script>
<link rel="stylesheet" href="./assets/style.css" />
```

AstrBot 给这两条打上 `asset_token`。iframe 一次加载，不跟子模块、不出网。

改前端后执行 `cd dashboard && npm run build`，把 `pages/<page>/` 一并提交。运行时不需要 Node。

## 多页用页内 Tab，不要多条宿主路由

侧栏只链到每个插件的**第一页**。iframe 改不了顶层 hash。

有多块业务时：一个 `pages/<page>/`（例如 `console`），页顶 antd `Tabs`，`onChange` 只改 React state。不要给每个 Tab 单独建目录。

## 踩过的坑（不要再用）

### 1. esm.sh importmap

`antd@5.24.0?external=react,react-dom` 会让图标向 **AstrBot 站点根** 要 `/@ant-design/colors` 的 `blue`，页面空白。就算加上 `?bundle`，国内打开 esm.sh 也很慢。

### 2. 把 React/antd 拆进 `vendor/` 再 ESM import

`antd.js` 里是 `from"./react.js"`。AstrBot 正则要空格，token 打不上。浏览器相对解析还会丢掉 query，请求 `/vendor/react.js` **401**。

### 3. `../nav.js`、多目录互引

Pages 只提供 `pages/<页名>/` 下的文件。`../nav.js` 打到 `/api/plugin/page/content/<插件>/nav.js`，401。

### 4. 点 Tab 改 `window.top.location.hash`

iframe 没有顶层跳转权限，赋值抛错，看起来像点不动。

### 5. 被扩展日志带偏

这些不是插件故障：`Vue分析` / `COSE` / `Missing required param "pluginId"`；`cloud.astrbot.app` 公告 CORS。
