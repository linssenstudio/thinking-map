# Thinking Map

把一段系统描述，变成一张清晰、精致、可以交互的单文件 HTML 图。

Thinking Map 使用结构化 JSON 描述节点与关系，支持 **架构、工作流、时序、数据流、生命周期** 五类图。生成的 HTML 内联 SVG、样式和查看器脚本，无需部署网站，也不用把数据上传到在线画图服务；下载后直接交给浏览器打开即可。

它既可以作为独立的命令行工具使用，也可以作为一份 AI Skill，让 AI 按 `SKILL.md` 的流程完成“理解需求 → 编写 JSON → 校验 → 渲染 → 交付”。

## 核心能力

- **五种图共用一个入口**：同一条 CLI，按类型选择对应渲染器。
- **单文件、可离线分享**：HTML 自带图面和交互，不依赖运行时服务。
- **可交互阅读**：深色/浅色主题、缩放平移、搜索聚焦、关系追踪、语义视图和可选导览章节。
- **多种导出格式**：在查看器中导出 PNG、JPEG、WebP、SVG；开启动效后还可录制 WebM。
- **默认静态，动效可选**：只有显式设置 `meta.animation: "trace"` 才启用路径动效。
- **机器校验**：JSON Schema 检查字段结构，渲染器继续检查重叠、碰撞、路由和可读性。
- **可验证交付**：`deliver` 在检查通过后原子替换 HTML，并报告规格与产物的 SHA-256。
- **运行时零 npm 依赖**：日常使用只需要 Node.js；开发依赖不参与普通渲染。
- **支持 Mermaid 转换**：通过 AI Skill 读取 Mermaid 拓扑并重新创作，不机械复制原样式。

## 五种图怎么选

| 类型 | 适合表达 | JSON 的主要结构 | 内置示例 |
|---|---|---|---|
| `architecture` | 系统架构、组件、云资源、网络与安全边界 | `components`、`boundaries`、`connections` | [JSON](examples/web-app.architecture.json) · [HTML](examples/web-app-rendered.html) |
| `workflow` | 业务流程、审批、工具调用、CI/CD、操作手册 | `lanes`、`nodes`、`edges`，可配 `phases`、`groups` | [JSON](examples/agent-tool-call.workflow.json) · [HTML](examples/workflow-agent-tool-call-rendered.html) |
| `sequence` | API 调用链、请求与响应、异步消息、执行时序 | `participants`、`messages`，可配 `segments`、`activations` | [JSON](examples/cache-miss-request.sequence.json) · [HTML](examples/sequence-cache-miss-request.html) |
| `dataflow` | 数据管道、ETL/ELT、事件流、数据血缘与治理 | `stages`、`nodes`、`flows` | [JSON](examples/product-analytics.dataflow.json) · [HTML](examples/dataflow-product-analytics.html) |
| `lifecycle` | 状态机、等待、重试、回滚与终态 | `lanes`、`states`、`transitions` | [JSON](examples/agent-run.lifecycle.json) · [HTML](examples/lifecycle-agent-run.html) |

不知道选哪一种时，可以把场景交给内置向导：

```bash
node bin/thinking-map.mjs guide "用户下单后经过支付、库存和发货" --json
```

> GitHub 页面中的 HTML 链接用于查看文件。要体验交互，请下载对应 HTML，或克隆仓库后在本地用浏览器打开。

## 环境要求

- Node.js **18 或更高版本**。
- 一个现代浏览器，用于打开生成的 HTML。
- 可选：Chrome 或 Chromium，用于 `visual-check` 自动浏览器检查。

普通渲染不需要执行 `npm install`，也不需要联网。

## 快速开始

### 1. 获取项目并自检

```bash
git clone https://github.com/linssenstudio/thinking-map.git
cd thinking-map
node bin/thinking-map.mjs doctor
```

`doctor` 会检查 Node 版本、核心模板、五类渲染器、Schema、校验器、预览和视觉检查运行时。看到 `ThinkingMap is ready.` 即表示基础环境正常。

### 2. 生成一个演示图

```bash
node bin/thinking-map.mjs demo output
```

演示文件会写入：

```text
output/thinking-map-demo.html
```

用浏览器打开这个文件即可体验主题切换、缩放、搜索、聚焦和导出。

### 3. 渲染内置架构图

```bash
mkdir -p output
node bin/thinking-map.mjs render \
  architecture \
  examples/web-app.architecture.json \
  output/web-app.html
```

`render` 适合快速查看结果。正式交付建议使用后文的 `validate` 和 `deliver`。

### 4. 重新生成五类内置示例

```bash
node bin/thinking-map.mjs examples
```

该命令会根据 `examples/` 中的 JSON 重新生成五份配套 HTML。

## 创建自己的第一张图

新建 `my-architecture.json`，粘贴以下内容：

```json
{
  "schema_version": 1,
  "diagram_type": "architecture",
  "meta": {
    "title": "Web 请求架构",
    "locale": "zh-CN",
    "quality_profile": "showcase"
  },
  "components": [
    {
      "id": "browser",
      "type": "external",
      "label": "浏览器",
      "sublabel": "用户入口",
      "pos": [40, 180],
      "size": [140, 64]
    },
    {
      "id": "api",
      "type": "backend",
      "label": "API 服务",
      "sublabel": "业务逻辑",
      "pos": [300, 180],
      "size": [140, 64]
    },
    {
      "id": "database",
      "type": "database",
      "label": "数据库",
      "sublabel": "持久化存储",
      "pos": [560, 180],
      "size": [140, 64]
    }
  ],
  "boundaries": [],
  "connections": [
    {
      "id": "browser-to-api",
      "from": "browser",
      "to": "api",
      "label": "HTTPS",
      "variant": "emphasis"
    },
    {
      "id": "api-to-database",
      "from": "api",
      "to": "database",
      "label": "SQL"
    }
  ]
}
```

先校验，再正式交付：

```bash
node bin/thinking-map.mjs validate \
  architecture \
  my-architecture.json \
  --quality showcase \
  --json

node bin/thinking-map.mjs deliver \
  architecture \
  my-architecture.json \
  output/my-architecture.html \
  --quality showcase \
  --json
```

最后用浏览器打开 `output/my-architecture.html`。

### 这段 JSON 里的关键字段

| 字段 | 含义 |
|---|---|
| `schema_version` | 规格版本。除工作流支持 v1/v2 外，其余类型当前使用 `1`。 |
| `diagram_type` | 图类型，必须与 CLI 中的类型一致。 |
| `meta.title` | 必填标题，同时用于查看器和文档标题。 |
| `components` | 架构组件；`id` 必须稳定且唯一，关系通过 `id` 引用节点。 |
| `pos` / `size` | 架构组件的位置与尺寸，单位为 SVG 坐标。 |
| `connections` | 有方向的组件关系，`from` 和 `to` 必须指向已有组件。 |
| `variant` | 视觉语义变体，如 `default`、`emphasis`、`security`、`dashed`。 |

所有 Schema 都严格检查字段；拼错字段名不会被静默忽略，而会报错。不同图的完整字段定义见 [`schemas/`](schemas/README.md)。

## 常用 `meta` 配置

以下配置适用于多数场景；具体可用字段仍以对应 Schema 为准。

| 字段 | 可选值 / 结构 | 作用 |
|---|---|---|
| `title` | 字符串 | 图的标题，必填。 |
| `output` | 文件名 | 未显式传入输出路径时使用的默认输出名。 |
| `quality_profile` | `standard` / `showcase` | 构图质量标准；重要交付推荐 `showcase`。 |
| `visual_preset` | `classic` / `signal-flow` / `blueprint` / `editorial` | 视觉预设，默认 `classic`。只改变样式，不改变语义和几何。 |
| `animation` | `none` / `trace` | 默认静态；`trace` 开启由读者控制的路径动效。 |
| `locale` | `en` / `zh-CN` | 设置查看器固定 UI 和文档语言，不会翻译作者填写的内容。 |
| `legend` | `auto` / `all` / `hidden` 及条目覆盖 | 控制图例展示和标签。 |
| `views` | 最多五个章节 | 用稳定节点 ID 定义导览焦点和说明。 |
| `viewBox` | `[width, height]` | 显式指定 SVG 画布；不同渲染器的默认推导方式略有差异。 |

### 四种视觉预设

- **`classic`**：默认暖纸与墨色风格，适合大多数架构和文档场景。
- **`signal-flow`**：强调信号和路径流动，适合演示与动效表达。
- **`blueprint`**：高对比工程蓝图风格，适合技术评审。
- **`editorial`**：偏出版物排版，适合设计评审和说明文档。

预设与深色/浅色主题相互独立：切换主题不会丢失当前预设。

## CLI 完整用法

查看内置帮助：

```bash
node bin/thinking-map.mjs --help
```

### 日常命令

| 命令 | 用途 |
|---|---|
| `doctor` | 检查运行环境和核心文件。 |
| `demo [目录]` | 生成一份架构演示 HTML。 |
| `examples` | 重新渲染五类内置示例。 |
| `guide [场景] [--json]` | 根据场景推荐图类型和创作路径。 |
| `inspect <类型> <JSON>` | 查看输入规格的解析摘要。 |
| `validate <类型> <JSON>` | 校验结构、语义与构图，不写 HTML。 |
| `render <类型> <JSON> [HTML]` | 快速渲染一个 HTML。 |
| `deliver <类型> <JSON> [HTML]` | 完整检查后原子交付 HTML。 |
| `preview <类型> <JSON> [HTML]` | 启动本地实时预览，适合桌面创作循环。 |
| `check <HTML>` | 检查已有 HTML 产物。 |
| `visual-check <HTML> [--json]` | 用 Chrome/Chromium 收集桌面尺寸、主题与截图证据。 |

### 进阶命令

| 命令 | 用途 |
|---|---|
| `compare architecture <旧JSON> <新JSON> [HTML]` | 生成架构变更对比图和收据。 |
| `migrate workflow <旧JSON> <新JSON> --to-schema 2` | 将工作流规格迁移到 v2。 |
| `brands [名称或域名] [--json]` | 查询内置品牌图标 ID。 |
| `brands capture <官网URL> [--json]` | 显式采集并固定外部品牌图标摘要。 |

### 常用选项

- `--quality standard|showcase`：选择构图检查等级。
- `--json`：输出机器可读 JSON，便于自动化和 AI 工具处理。
- `--layout-json`：输出工作流布局诊断收据。
- `--open`：交付成功后尝试用系统默认程序打开 HTML。
- `--no-open`：预览时只打印本地地址，不自动打开浏览器。
- `--repo-root <路径>`：架构图启用仓库证据时，指定本地 Git 仓库根目录。

## `render`、`deliver`、`preview` 有什么区别

### 快速渲染：`render`

```bash
node bin/thinking-map.mjs render architecture input.json output.html
```

适合初次生成和快速查看。它需要明确的输入类型与 JSON 文件，输出路径必须以 `.html` 结尾。

### 正式交付：`validate` + `deliver`

```bash
node bin/thinking-map.mjs validate architecture input.json --quality showcase --json
node bin/thinking-map.mjs deliver architecture input.json output.html --quality showcase --json
```

`deliver` 会冻结本次输入的确切字节，在私有临时位置完成渲染和检查，全部通过后才替换目标 HTML。收据包含输入规格和 HTML 的 SHA-256 与字节数。

如果交付失败，命令退出码非零，已有的上一份可信 HTML 会保留。此时应修复 JSON 后重新交付，不要把旧文件当成本次结果。

`showcase` 正式通过要求全部 **9 项产物检查**完成，并且 **0 个构图错误、0 个警告**。

### 实时预览：`preview`

```bash
node bin/thinking-map.mjs preview architecture input.json output.html --quality showcase
```

`preview` 监听一个明确的输入文件。只有新版本通过交付检查后，页面和磁盘上的 HTML 才更新；无效或写到一半的输入不会覆盖上一版。按 `Ctrl-C` 停止预览。

它适合本地桌面创作，不用于 CI、远程分享或无人值守任务。

## 浏览器检查与视觉验收

交付成功后，可以检查同一个 HTML：

```bash
node bin/thinking-map.mjs visual-check output.html --json
```

该命令通过 Chrome/Chromium 检查多个桌面视口，并在 HTML 旁生成截图、联系表和 JSON 收据。未安装可用浏览器时，自动检查会跳过；这不影响已经成功的确定性交付。

请注意三个结论彼此独立：

1. `deliver` 证明确定性产物检查和字节身份。
2. `visual-check` 证明自动浏览器测量与截图结果。
3. 观感是否精致仍需人工或具备图像能力的评审者实际查看。

自动检查通过不等于人工视觉评审通过。

## 生成的 HTML 怎么阅读与导出

查看器已经内置以下能力，无需额外创作另一份文件：

- 深色/浅色主题切换。
- 缩放、平移和适配视图。
- 按标签或稳定 ID 搜索节点。
- 聚焦节点，查看其上游、下游和语义信息。
- 沿有向关系探测路径，不根据几何位置猜测连接。
- 使用可选章节进行导览和演示。
- 导出完整图面的 PNG、JPEG、WebP 或双主题 SVG。
- 在启用 `trace` 后录制 WebM。
- 生成适合 README、发布页或社交分享的 1200×630 分享卡。

导出的是完整图面，不会把搜索框、焦点浮层、工具栏等临时查看器状态带入正式图片。

## 作为 AI Skill 使用

将整个 `thinking-map` 目录放入你的 AI 宿主技能库，并保留 `SKILL.md` 及其配套目录。不要只复制 `bin/`：Schema、模板、渲染器、示例和检查脚本都属于运行所需内容。

可以直接这样提需求：

```text
用 Thinking Map 画一张订单系统架构图。
包括 Web、API、Redis、PostgreSQL 和异步 Worker，
标出 HTTPS、缓存读取和任务入队关系，中文标签，正式交付。
```

AI 应按以下流程工作：

1. 判断五类图中的哪一种最合适。
2. 阅读对应 Schema 和一个同类型示例。
3. 创建新的稳定节点 ID 和 JSON，不照抄示例事实。
4. 每轮修改后运行 `validate`。
5. 冻结候选后运行 `deliver`。
6. 如实区分交付检查、浏览器证据和视觉评审结果。

完整创作与交付约束见 [`SKILL.md`](SKILL.md)。

## Mermaid 转换

支持把 Mermaid 的拓扑和语义作为输入，再重新编写 Thinking Map JSON：

| Mermaid 类型 | 推荐转换目标 |
|---|---|
| `flowchart` / `graph` | `workflow`；组件地图也可选择 `architecture` |
| `sequenceDiagram` | `sequence` |
| `stateDiagram` | `lifecycle` |

转换会保留节点、关系和语义，但不会机械照搬 Mermaid 的样式与布局。

**注意：这是 AI Skill 的创作流程，CLI 不直接读取 `.mmd` 文件。** 普通命令行输入仍是 Thinking Map JSON。

## 品牌图标

节点可以显式引用真实产品品牌，但不会根据“数据库”“云服务”等模糊角色自动猜测品牌。

```bash
# 查询内置品牌
node bin/thinking-map.mjs brands "PostgreSQL" --json

# 用户提供官网 URL 时，显式采集未知品牌
node bin/thinking-map.mjs brands capture "https://example.com" --json
```

优先使用查询返回的内置规范 ID。外部采集结果带 SHA-256 摘要固定；普通 `render` 和 `validate` 不会临时抓取未固定的网络资源。

## 工作流 v1 与 v2

- 新工作流推荐使用 `schema_version: 2`，通过泳道、列和 `mainPath` 获得更可读的布局约定。
- 需要保留旧图固定几何时继续使用 v1。
- 可显式迁移：

```bash
node bin/thinking-map.mjs migrate workflow old.json new.json --to-schema 2 --json
```

布局、迁移和收据约定见 [`renderers/workflow/README.md`](renderers/workflow/README.md)。

## 项目结构

```text
thinking-map/
├── bin/                    # CLI、预览、打开产物与视觉检查入口
├── assets/                 # 单文件 HTML 模板和字体许可
├── renderers/              # 五类渲染器及共享布局、校验和国际化逻辑
├── delta/                  # 架构变更对比
├── schemas/                # JSON Schema 与 Schema 说明
├── examples/               # 示例 JSON、预渲染 HTML 和对比示例
├── scripts/                # 示例渲染、品牌与校验器生成脚本
├── references/             # 创作、查看器、品牌与交付契约
├── recipes/                # 场景创作参考
├── migrations/             # 工作流版本迁移
├── test/                   # 测试与基准文件
├── SKILL.md                # AI Skill 工作流
└── package.json            # 包信息与开发依赖
```

日常修改图时通常只需要关注 `schemas/`、`examples/` 和 CLI。修改渲染器或交付行为前，再阅读 `references/` 中的详细契约。

## 常见问题

### 提示字段不被允许

Schema 使用严格校验，未知字段会直接失败。检查字段拼写、层级和对应图类型，不要通过添加无关配置绕过错误。

### 提示节点或关系引用不存在

关系的 `from`、`to` 以及 `meta.views[].focus` 都引用稳定 ID。先检查 ID 是否拼写一致、是否重复，以及是否放进了当前类型要求的节点集合。

### JSON 合法，但仍然校验失败

JSON Schema 只负责字段形状；节点重叠、标签碰撞、路由穿过无关节点等问题由渲染器继续诊断。根据错误中的路径、`subject`、`evidence` 和可用修复建议调整源 JSON。

### 输出路径报错

CLI 的 HTML 输出路径必须以 `.html` 结尾，架构对比收据必须以 `.json` 结尾。类型不匹配会在写入前失败。

### `deliver` 失败但旧 HTML 还在

这是刻意设计：失败不会破坏上一份可信产物。旧 HTML 不代表当前候选已通过，修复源文件并重新运行 `deliver`。

### `visual-check` 没有运行

确认系统存在可用的 Chrome/Chromium。普通渲染和交付不需要浏览器；只有自动浏览器证据采集需要它。

### 中文内容变了，但界面仍是英文

`meta.locale` 控制查看器固定 UI，不会自动翻译节点和标签。将作者内容直接写成所需语言，并显式设置 `locale: "zh-CN"`。

## 许可与致谢

本项目基于 [tt-a1i/archify](https://github.com/tt-a1i/archify)（MIT）深度定制而来，上游又基于 Cocoon-AI/architecture-diagram-generator（MIT）。感谢原作者及开源社区。

项目使用 [MIT License](LICENSE)。品牌图标、字体等第三方素材的署名与许可说明见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。
