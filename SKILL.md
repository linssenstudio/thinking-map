---
name: thinking-map
description: 把系统描述变成精致的可交互图——架构、工作流、时序、数据流（data-flow）、生命周期五类图（architecture / workflow / sequence / dataflow / lifecycle），输出自带深色/浅色主题的单文件 HTML，内联 SVG，支持缩放、搜索、聚焦、路径追踪和 PNG/JPEG/WebP/SVG/WebM 导出。接受自然语言描述或直接粘贴 Mermaid（flowchart、sequenceDiagram、stateDiagram）转换美化。当用户要画系统架构图、基础设施、云/网络拓扑、技术工作流、API 调用时序、请求生命周期、数据管道、ETL/ELT、数据血缘、状态机，或想转换美化 Mermaid 时使用。
license: MIT
metadata:
  version: "1.0.0"
  author: Sen
  based_on: tt-a1i/archify (MIT, v2.17)
---

# Thinking Map

从一份小小的结构化 JSON 规格，生成一张自包含、可交互的 HTML 图。默认静态输出；只有用户明确要演示或汇报时才开启动效。

## 快速创作路径

普通画图需求就走这条路径。除非用户问到查看器的高级功能，否则不要去读 Viewer Runtime 参考文档。

1. 从需求里选一个类型：`architecture`、`workflow`、`sequence`、`dataflow` 或 `lifecycle`。
2. 只读三份材料：`schemas/` 里对应的那一个 schema、`schemas/common.schema.json`、`examples/` 里对应的一个 JSON 示例。别的先不读。新创作意味着新的稳定 ID、新的领域措辞和新的布局；示例只用来看字段形状，不要抄它的事实内容。新工作流用 `schema_version: 2` 和它那可读的布局约定；只有要保留旧工作流的固定几何时才用 `schema_version: 1`。涉及真实产品品牌时，跑 `node bin/thinking-map.mjs brands "<名称>" --json`；只有遇到陌生品牌且用户提供了官网 URL 时才去读 `references/brand-marks.md`。
3. 产物优先：下一步工具动作必须是写出候选 JSON。先看渲染器内部实现之前，先把候选写出来。不要在文字里推演精确坐标。起手式：一条清晰的主路径、少量短支线、稀疏的标签、最多 12 个主节点。除非用户明确要高密度的 `standard` 地图，否则把 `meta.quality_profile` 设为 `"showcase"`。先用自动路由和自动标签。不要在诊断要求之前加 `via`、`channelX`、`channelY` 或 `labelAt`；每轮修复最多只动一个已确诊的几何控制项。
4. 每次改完候选都要校验，交付前也必须再校验一次：

   ```bash
   node bin/thinking-map.mjs validate <类型> <候选.json> --quality showcase --json
   ```

   只含 4 项产物检查的收据只是基础校验，永远不算 showcase 验收。showcase 通过必须报告全部 9 项产物检查，且 0 个构图错误、0 个警告。如果候选漏写或拼错了 `meta.quality_profile` 字段，先修字段再谈几何。工作流 v2 的几何诊断跑 `node bin/thinking-map.mjs validate workflow <候选.json> --layout-json`，用那份稳定的编译器收据；求解器内部参数不是创作控制项。最终校验通过后候选即冻结：之后不许再改。
5. 交付 HTML 时，`deliver` 是最终验收命令：

   ```bash
   node bin/thinking-map.mjs deliver <类型> <候选.json> <输出.html> --quality showcase --json
   ```

   退出码非零就绝不能说成成功。交付失败会保留上一份输出，所以别对那个路径跑 `visual-check`：它检查的是过期的旧产物，不是失败的候选。校验失败时，只改诊断指出的 `subject`，核对 `evidence`，从 `supportedFixes` 里选修法，然后重跑。只要客观错误数在创新低，就继续针对性修正。连续两轮没有刷新最优错误数就停下，如实报告未解决的诊断。

工作流提示：新工作流用 schema v2；旧源需要固定几何时保留 schema v1。保持语义化的边标签，按编译器诊断行动。规范的布局、钉住、迁移和收据约定见 [`renderers/workflow/README.md`](renderers/workflow/README.md#layout-contracts)。

生命周期提示：阶段列 `0..4` 在主轨道上；事件/终态列 `N`（`0..2` 范围内）与主列 `N + 2` 正下方对齐。可恢复状态用 `type: "failure"` 加一条回到活动状态的真实转移。

## 类型路由

| 类型 | 适用场景 |
|---|---|
| `architecture` | 组件、服务、云/安全边界、基础设施 |
| `workflow` | 流程、审批节点、工具调用、操作手册、CI/CD |
| `sequence` | API 调用链、请求生命周期、异步追踪、返回 |
| `dataflow` | 数据管道、ETL/ELT、血缘、治理、消费方 |
| `lifecycle` | 状态/状态转移、重试、等待与终态 |

拿不准时跑 `node bin/thinking-map.mjs guide "<场景描述>" --json`。场景证明示例是结构参考，不是要照抄的事实。

## Mermaid 输入

读 Mermaid 拿拓扑和语义，然后写一份全新的 Thinking Map JSON；不要机械照搬 Mermaid 的样式。

- `flowchart` / `graph` → `workflow`；组件地图则用 `architecture`。
- `sequenceDiagram` → `sequence`；参与者变成语义参与者，箭头变成消息。
- `stateDiagram` → `lifecycle`；状态和转移保留语义，不保留 Mermaid 风格。

## 创作铁律

- 一条明显的主路径；支线从最近的主路径节点分出。先删低价值的边，再考虑加路由控制。
- 默认不写 `meta.visual_preset`，每张图都以 `classic` 打开，无论解析出的色彩模式是深是浅。色彩模式和视觉预设相互独立：切换深色/浅色必须保持当前预设。只有用户明确要那种视觉风格时才设 `signal-flow`、`blueprint` 或 `editorial`。
- 默认不写 `meta.subtitle`。永远不要编一个复述标题、节点或卡片的副标题；只有用户明确要时，才写一句简短的支撑说明。
- 独立桌面查看器默认是首屏作品，不是浅条带。只生成一份响应式产物，面向笔记本和外接显示器——绝不做设备专用 HTML 或另一套拓扑。查看器只可根据实时视口高度调整外层阅读宽度；必须保持作者设定的 SVG/viewBox、比例、语义几何和正常文档流。在宽大桌面上，用足够的纵向节奏让图面板和必要的结论卡片作为一个平衡整体占满屏幕；运行时缩放救不了被过度压缩的 Y 布局或过小的显式 `meta.viewBox`。交付前，在 1440×900, 1600×1000, and 1920×1080 打开真实 HTML；为大桌面准备的构图再加查 2048×1320。每个尺寸都要求 `document.documentElement.scrollWidth <= window.innerWidth` 且 `scrollHeight <= window.innerHeight`，同时肉眼确认图在最大视口下仍清晰可读、纵向平衡。修溢出的顺序：先删真正冗余的内容或压缩间距，再考虑缩小节点、标签或主面板。如果最大视口在查看器宽度上限处仍有一条显眼的下部空带，按此例重排 Y 坐标并增大 viewBox 高度；不要加填充文案或装饰卡片。绝不用 `overflow: hidden`、裁剪内容、图内滚动条、拉伸 SVG 高度或缩小字号来伪造通过。窄屏/移动布局在容纳需要时可以纵向滚动。
- `meta.legend` 默认省略，用诚实的 `auto`。需要时只用 `mode: auto|all|hidden` 和渲染器支持的 `entries.<kind>.label|visible`；图例标签永远不改语义。
- 作者语言由用户明确选择决定；否则跟随请求或对话的主导语言。`meta.locale` 只控制渲染器拥有的查看器 UI：对应支持的主语言用 `"en"` 或 `"zh-CN"`。其他语言一律省略 `meta.locale`，并明确告知固定查看器 UI 和 `<html lang>` 回退英文。渲染器永远不翻译创作内容。详见 `references/authoring-contract.md`。
- 精确保留产品名、代码标识符、命令、协议、API 路径和环境名。它们可以在本地化文案里保持英文，但不能以此为由让周围说明文字也用另一种语言。
- 品牌标识是可选项，且必须显式。节点指代真实产品时，把内置规范 ID 写进 `brand`。没有预设匹配且用户提供了官方 HTTP(S) URL 时，先跑 `node bin/thinking-map.mjs brands capture "<url>" --json`，再写入返回的带摘要钉定的 `brand` 对象。渲染和校验永远不执行未钉定的抓取。其余情况省略 `brand`。永远别从"数据库"这种模糊角色推断品牌，也永远别让徽标取代语义 `type`、标签或关系事实。
- 时序图默认省略 `meta.column_fit`，用稳定的 `fixed` 布局。当宽 viewBox 会留出大片横向空白、或有意义的参与者标签放不进固定框时，设为 `"spread"`；先尝试 `spread`，再考虑缩短语义标签。
- 组件类型有 `frontend`、`backend`、`database`、`cloud`、`security`、`messagebus`、`external`；变体有 `default`、`emphasis`、`security`、`dashed`。
- 关系标签是语义数据。碰撞时按序处理：移动标签、调整路由或间距、在保留语义的前提下缩短措辞。只有当措辞已被两端节点完全隐含、且不含协议、动作、方向、同步/异步行为或跨边界机制时才能省略。保留每个有意义的标签；删标签不是几何修复。如果一条关系因端点完全隐含而天生无标签，说明措辞冗余的原因；这是语义创作决策，不是几何修复。
- 默认省略 `meta.engineering_profile`。区域、集群和安全边界的措辞本身不启用它。只有当用户明确要生产部署拓扑、所有权交接或故障关闭式部署评审、且源事实已知时，才启用 `deployment-ownership`。启用后，绝不能为了过校验而移除工程档案；修事实或如实报告诊断。
- 间距指净间隙，不是中心距。关系标签的净间隙必须大于其量得的遮罩宽度；遵循保标签的修复顺序。
- 自动路由拥有自己的端点侧边。侧边是方向契约：第一段和最后一段必须与该侧边垂直地离开/进入。
- 自动端口展开是架构、工作流、数据流、生命周期的默认渲染器行为。单条关系和显式 `via`、`channelX`、`channelY`、`labelAt` 或非 `auto` 路由不参与。邻近的平行端口走外侧桥，避免自动路由产生小于 8px 的段或小于 16px 的内部转角。架构图另外让无遮挡的相对自动端口（`left`/`right` 或 `top`/`bottom`）在偏移小于 16px 且两端口都保留角部余量时共用一条轴。如果恰好只有一个端点被展开，只有未共享的端点可以移到该轴上；两个端点都被展开时保持外侧桥，让竞争端口保持区分。
- 永远不接受：边穿过无关的不透明节点、含糊的共享走廊、关系标签遮住另一条路由。

只有需要字段枚举、间距数学、几何修复规则、仓库证据或特定模式的放置规则时，才读 `references/authoring-contract.md`。

## 交付

修复阶段用 `validate`，最终验收用一次 `deliver`。交付会把规格的确切字节冻结为同目录的私有快照，渲染并检查该快照，原子化地提交 HTML，并报告规格与产物两份 SHA-256 和字节数。这是确定性的产物证据；它不会在浏览器里运行查看器。

交付后，在不修改、不重渲染受信 HTML 的前提下收集有边界的桌面证据：

```bash
node bin/thinking-map.mjs visual-check <输出.html> --json
```

`visual-check` 对交付的确切 HTML 收集自动化浏览器证据，不修改也不重渲染。它的机器可读测量和截图不等于认可观感层面的打磨。规范的收据字段、覆盖范围、旁车文件、退出行为和补充人工记录要求见 `references/delivery-contract.md`。

三个声明分开讲：`deliver` 证明确定性产物检查，`visual-check` 证明真实浏览器里的有边界行为，观感评审则需要真实的人或具备图像能力的评审者。浏览器证据和观感评审要独立汇报。无约束的一瞥只能支撑观感评审；记录补充性人工浏览器操作或处理环境故障时，遵循规范的交付契约。

用户要立刻本地预览时才加 `--open`。桌面创作循环中的可选命令：

```bash
node bin/thinking-map.mjs preview <类型> <输入>.json <输出>.html --quality showcase
```

默认绝不启动 preview。使用 preview、仓库证据、导出收据、视觉评审或提交后打开时，读 `references/delivery-contract.md`。

## 查看器可选能力

生成的 HTML 自带主题切换、平移缩放、搜索、聚焦、关系追踪、语义视图、演示和如实的导出。这些是读者能力，不是额外的创作工作。`meta.animation: "trace"` 需要显式开启；`meta.views` 可选，最多五个精选章节。

只有当用户明确问到分享卡、路由/可达卡、动效、导览故事、深链、演示、搜索/聚焦或其他查看器运行功能时，才读 `references/viewer-runtime.md`。

## 安装与兜底

技能包内无需安装任何依赖。自检：

```bash
node bin/thinking-map.mjs doctor
node bin/thinking-map.mjs demo <输出目录>
```

没有 shell 环境时，手工把架构 SVG 放进 `assets/template.html`，用 CSS 语义类而不是内联颜色，并遵循 `references/delivery-contract.md` 的视觉评审契约。

## 输出

返回：检查过的 HTML 路径、图类型、校验摘要、规格/产物收据、浏览器证据状态、如实的视觉评审状态。退出码非零不许声称成功，没做过的视觉检查不许声称做过。
