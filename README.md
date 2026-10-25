# dsh-plugin-smooth-stream

简体中文 | [English](README.en.md)

> **这是 [SpookySandwich/dsh-plugin-smooth-stream](https://github.com/SpookySandwich/dsh-plugin-smooth-stream) 的二改版（fork）。**
> 修复了在 DSH 0.2 上每条消息渲染两遍、思考块无法实时展开、思考变长后页面不跟随滚动三个问题，
> 并新增「自动展开当前步骤」。改动详见 [本版改动](#本版改动)。原项目版权归原作者，遵循 MIT。

[![version](https://img.shields.io/badge/version-1.5.0-4b8dff)](https://github.com/DeerFishSheep/dsh-plugin-smooth-stream)
[![CI](https://github.com/DeerFishSheep/dsh-plugin-smooth-stream/actions/workflows/ci.yml/badge.svg)](https://github.com/DeerFishSheep/dsh-plugin-smooth-stream/actions/workflows/ci.yml)
[![license](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![dsh](https://img.shields.io/badge/dsh-0.2.0--rc.2%20~%200.2.1--rc.3-4b8dff)](https://github.com/deepseek-ai/deepseek-harness)
[![stars](https://img.shields.io/github/stars/DeerFishSheep/dsh-plugin-smooth-stream?style=flat&label=stars)](https://github.com/DeerFishSheep/dsh-plugin-smooth-stream/stargazers)

把逐字抖动的流式输出，换成按段落分批、淡入呈现的阅读体验。流式期间平滑跟随滚动，思考块显示实时单行摘要。

**左：DSH 原生渲染 · 右：Smooth Stream** —— 同一个问题，同时开始：

![对比演示](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/demo.gif)

## 安装

从本仓库直接安装（推荐，一条命令）：

```bash
dsh plugin --profile desktop add github:DeerFishSheep/dsh-plugin-smooth-stream
```

没有 `dsh` 命令行时，用包管理器直接装进 profile：

```bash
cd ~/.dsh/profiles/desktop && pnpm add github:DeerFishSheep/dsh-plugin-smooth-stream
```

> `desktop` 换成你实际的 profile 名，用 `ls ~/.dsh/profiles/` 查看。
> 安装需要能访问 GitHub；本包与 npm 上的同名包互相覆盖，直接装即可，不用先卸载。

打开任意会话即可生效。设置入口：设置面板 → **Smooth Stream**。装完请重启或刷新 DSH。

## 本版改动

相对上游 `1.3.0`，本版 `1.5.0` 修了三个问题、加了两个功能：

| 项目 | 说明 |
|---|---|
| 消息渲染两遍 | DSH 0.2 会把一条 assistant 消息拆成「思考」和「正文」两次渲染，本插件原先两次都画全量内容。现已按 `groupPart` 过滤，与宿主行为一致（对应上游 issue #2） |
| 思考块显示不出来 | 思考块原先要攒满 1000 字才渲染第一段，短思考全程不可见；阈值已归零，整段无换行也不会卡住 |
| 思考块误判已结束 | 原先以「数组最后一个元素」判断当前步骤，消息尾部若带空块或工具调用，思考块会刚出现就收起；现改为「最后一个有内容的块」 |
| **自动展开当前步骤**（新增） | 正在输出的步骤自动展开，开始下一步时自动收起；手动点击优先，直到该步骤角色变化。默认开启，可在设置里关闭 |
| **思考收起时机**（新增） | 可切换「下一段思考开始时」或「下一步开始时」收起思考。默认前者，正文输出期间思考仍保持展开 |
| 滚动跟随 | 原先只有一行 `scroll-behavior: smooth`，没有任何代码真正滚动，思考变长就会滑到输入框下面。现用 `ResizeObserver` 监测高度增长并贴底，往上翻看历史时不会被打扰 |

## 它改变了什么

- **按段落呈现** — 回复不再一个字一个字地蹦，而是攒满一段，整段淡入。断句时自动避开没写完的代码块和表格，Markdown 不会渲染到一半。
- **平滑滚动** — 流式输出时页面匀速滑向底部，长回复也不会猛地一跳；往上滚就交还给你，滚回底部自动继续跟随。
- **思考摘要** — 模型思考时，折叠行里实时滚动最新一句，两端渐隐过渡；回合结束时下划线缓缓收起，示意"想完了"。
- **Markdown 与公式** — 正文走 DSH 自带的渲染器：代码高亮、复制按钮、表格、KaTeX 公式都和原生一模一样。

## 设置

设置面板中的 **Smooth Stream** 分区，改动即时生效并持久保存：

| 设置项 | 默认 | 说明 |
|---|---|---|
| 启用 Smooth Stream | 开 | 关闭后即时恢复 DSH 内置渲染，方便对比 |
| 入场动画 | 淡入 | 淡入 / 上升 / 浮现 / 拂过 / 聚焦 / 映亮 / 晕开 / 洇染 |
| 动画时长 | 520 ms | 入场动画时长（120–1200 ms） |
| 分批大小 | 500 字符 | 攒多少文字呈现一批；越小越"实时"，越大越安静 |
| 平滑滚动跟随 | 开 | 流式期间的平滑滚动跟随 |
| 自动展开当前步骤 | 开 | 正在输出的思考步骤自动展开，开始下一步时自动收起 |
| 思考收起时机 | 下一段思考开始时 | 可选「下一段思考开始时」或「下一步开始时」。前者让思考在正文输出期间仍然可读；后者一旦进入下一步（正文、工具调用或命令）就收起 |

带实时预览，可一键恢复默认。设置界面跟随 DSH 的显示语言。

## 动画风格

八种入场动画，默认节奏各自按风格调校；下面全部用同一段回复录制，便于直接对比。

| 淡入 Fade | 上升 Rise |
|---|---|
| ![fade](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/fade.gif) | ![rise](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/rise.gif) |
| **浮现 Dissolve** | **拂过 Wipe** |
| ![dissolve](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/dissolve.gif) | ![wipe](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/wipe.gif) |
| **聚焦 Focus** | **映亮 Glow** |
| ![focus](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/focus.gif) | ![glow](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/glow.gif) |
| **晕开 Iris** | **洇染 Soak** |
| ![iris](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/iris.gif) | ![soak](https://raw.githubusercontent.com/DeerFishSheep/dsh-plugin-smooth-stream/main/assets/variants/soak.gif) |

## 兼容性

版本 `1.5.0`（本仓库）：修复 DSH 0.2 下的双重渲染，新增「自动展开当前步骤」与「思考收起时机」，并让滚动真正跟随内容增长。渲染仍注册在 `conversation.chat.node` 与 `settings.section`。已在 DSH `0.2.1-rc.3` 上验证。

版本 `1.3.0`（上游）：可在官方 dsh `0.2.0-rc.2` 上加载。

版本 `1.2.0`：更新客户端渲染服务依赖与已验证的 DSH 版本范围；现有渲染行为通过新版宿主验证。

声明兼容范围为 `>=0.2.0-rc.2 <0.3.0-0`；官方 dsh `0.2.0-rc.2` 满足该范围。旧版 DSH 请保留上一插件版本。[验证记录](.github/reviews/dsh-0.1.5.md)。

如需固定版本，可从 [GitHub Release](https://github.com/DeerFishSheep/dsh-plugin-smooth-stream/releases) 下载 `.tgz`，再执行 `dsh plugin --profile desktop add ./dsh-plugin-smooth-stream-1.5.0.tgz`；也可在依赖里写 `github:DeerFishSheep/dsh-plugin-smooth-stream#<commit>`。

运行 `npm ci`、`npm test` 和 `npm run check:package` 可验证构建及发布包。更新后请重启 DSH。


- 使用宿主的客户端服务、Markdown 和图片接口；自动化测试覆盖加载与组件渲染。
- 本插件接管对话视图的 assistant 消息渲染（`conversation.chat.node` 的 `assistant-step` 槽位），与其他同样接管该渲染的插件互斥——同装时只有一个生效。
- 遵循 `prefers-reduced-motion`：系统开启减少动态效果时，所有动画与平滑滚动自动关闭。
- 渲染出错时自动降级为纯文本显示，不影响会话其余部分。

## License

[MIT](LICENSE)
