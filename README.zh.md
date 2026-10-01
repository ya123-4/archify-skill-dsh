# `archify-skill-dsh`

一个 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 插件包（bundle），把
**Archify 3.0.1** 文件系统 Skill（[上游](https://github.com/tt-a1i/archify)，MIT，作者 tt-a1i）
作为可安装插件提供。

Archify 能生成精致、经过校验的**架构图 / 工作流图 / 时序图 / 数据流图 / 生命周期图**——输出为
可交互的独立 HTML，内联 SVG，支持深/浅主题、可选轨迹动画，并可导出 PNG/JPEG/WebP/SVG/WebM。
它接受自然语言需求或粘贴的 Mermaid 输入；当图表需要反映真实代码时，还能读取仓库证据。

## 这个插件是什么

纯 Skill 提供：它在 DSH 中插入一个文件系统 Skill provider（`archify-plugin`），其唯一根目录就是
包内的 `skills/` 目录。**不注册**任何原生渲染/校验/交付工具，没有 Web 客户端，没有遥测，不处理
凭据，没有后台服务，也没有 `prepare`/`install`/`postinstall` 钩子。

| 路径 | 用途 |
|---|---|
| `cordis.patch.yml` | bundle 层：插入 `@deepseek-ai/dsh-skill-filesystem` provider，`includeDefaultRoots: false`，`bundledSkillDir` 指向本包的 `skills/` |
| `lib/index.js` | 同一 skill 根目录的包内解析辅助函数（有文档说明） |
| `skills/archify/` | 内置的、未改动的 Archify 3.0.1 发布载荷（106 个文件） |
| `package.json` | bundle 清单（`dsh.bundle.patch`）、npm `files` 白名单 |

## 为什么不用 `@tt-a1i/archify-dsh`

已发布的适配器 `@tt-a1i/archify-dsh@0.1.0` 内置的是 Skill **2.14.0**；未发布的 `0.2.0` 内置的是
**2.17.0-dev.1** 开发快照。本包携带的是**已发布的稳定版 Skill 3.0.1**——取自官方 tag `v3.0.1`
的 `archify.zip` 发布产物，别无其他。

## 安装

### 用发布包（tarball）安装

1. 从 [Releases](../../releases) 下载 `archify-skill-dsh-*.tgz`，或在本仓库根目录执行
   `npm pack` 自行构建。
2. 在 DeepSeek Harness 界面中：侧边栏 → **插件（Plugins）** → **添加插件（Add plugin）**。
3. 粘贴该 `.tgz` 的绝对路径（对话框支持包名、Git 地址、tarball 或绝对本地路径）。
4. **安装（Install）**。HMR 会把新 bundle 应用到正在运行的组合中，无需重启。

> **请先删除用户级 Skill。** 如果 `~/.dsh/skills/archify/` 仍然存在，同名 Skill `archify`
> 会由两个 provider 同时提供（rank 400 用户根与 rank 600 内置根）。两者只保留其一。

### 从本仓库安装

在仓库根目录执行 `npm pack` 会生成同样的 tarball，之后按上面的步骤通过界面安装。

## 已验证行为

实测环境：DSH 44.0.0 / Node 24.21.0，desktop profile：

| 检查项 | 结果 |
|---|---|
| bundle 安装并热生效（无需重启） | 通过 |
| provider 发现包内 Skill | 通过 — 目录名为 `archify`，基础目录指向 `…/node_modules/archify-skill-dsh/skills/archify` |
| 包内载荷完整性 | 通过 — 106/106 个文件与已发布的 3.0.1 载荷逐文件 SHA256 一致 |
| 从安装位置运行 `archify doctor` | 通过 — 全部检查通过 |
| `archify demo <dir>` | 通过 |
| `archify finalize architecture … --quality showcase --json` | 通过 — `validate`、`deliver`、`check`、`browser-check` 全部 `pass` |

### `browser-check` 需要 Chromium 内核浏览器

`finalize` 与 `browser-check` 会驱动真实的 Chromium 浏览器。找不到浏览器时，`browser-check`
闸门会报告 `skipped`（退出码 2），并给出 `viewer/chrome-unavailable` 警告；其余闸门仍然通过。
Chromium 内核的 Microsoft Edge 可用：

```powershell
$env:ARCHIFY_CHROME = 'C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe'
```

该变量必须在 DSH 宿主进程启动时所在的环境里设置。缺少浏览器是环境限制，不是插件缺陷。

## 卸载

插件页 → 对应 bundle 的卡片 → **卸载（Uninstall）**。profile 依赖与 bundle 层会被移除，基础
profile 仍可正常使用。

## 更新内置 Skill

`skills/archify/` 是上游发布的逐字快照。跟随新版本：

1. 从上游 Release 下载官方 `archify.zip`，用其内容替换 `skills/archify/`（zip 根目录下已经是
   `archify/` skill 目录）。
2. 把包的 `version` 提升为与 Skill 版本一致。
3. 验证：`npm run check`（运行 `archify doctor`）；可选地重跑上表中的 `finalize` 证据流程。
4. `npm pack` 并安装新的 tarball。

## 安全说明

`cordis.patch.yml` 是 DSH 宿主消费的配置。其中 `bundledSkillDir` 的值是一个 `!!js` 表达式，会在
宿主进程中、代理沙箱之外求值——请把它当作宿主加载的代码看待。该表达式只使用 Node 内置的 `path`
与 `module` 辅助函数来解析本包自身的 `skills` 目录；它不发起网络请求、不读取凭据、不创建进程，
也不注册额外的权限路径。

## 许可证与第三方声明

本插件包以 MIT 许可发布。内置 Skill 版权归 tt-a1i
（[Archify](https://github.com/tt-a1i/archify)，MIT），基于
Cocoon-AI/architecture-diagram-generator（MIT）。可选的品牌标识矢量数据遵循 Simple Icons 的
许可，详见
[`skills/archify/THIRD_PARTY_NOTICES.md`](skills/archify/THIRD_PARTY_NOTICES.md)。
