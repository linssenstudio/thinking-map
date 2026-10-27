# Thinking Map

把一段话变成一张拿得出手的图。

你只用说清楚系统里有什么、谁跟谁连着，Thinking Map 会把它渲染成一张可交互的单文件 HTML 图——深色/浅色双主题、缩放平移、搜索聚焦、路径追踪，还能一键导出 PNG / SVG / WebM。默认皮肤是暖纸配墨色的画廊质感。

## 五种图

| 类型 | 画什么 |
|---|---|
| `architecture` | 系统架构、组件、云与安全边界 |
| `workflow` | 业务流程、审批、CI/CD、操作手册 |
| `sequence` | API 调用链、请求生命周期 |
| `dataflow` | 数据管道、ETL、血缘 |
| `lifecycle` | 状态机、重试与终态 |

也支持直接粘贴 Mermaid（flowchart / sequenceDiagram / stateDiagram）转换美化。

## 用法

它是一份 AI 技能（Skill）：把整个目录放进你的技能库，AI 会按 `SKILL.md` 的流程工作——先写结构化 JSON，再校验、再渲染交付，每一步都有机器检查兜底。

零 npm 依赖，本机有 Node ≥ 18 就能跑：

```bash
node bin/thinking-map.mjs doctor        # 自检
node bin/thinking-map.mjs demo out/     # 生成一套示例
node bin/thinking-map.mjs render architecture examples/web-app.architecture.json out.html
```

`examples/` 里有五类图的 JSON 规格和渲染好的 HTML，打开就能看效果。

## 设计说明

- **暖纸 × 墨色**：默认 classic 主题是重新设计的——浅色是暖白纸面，深色是暖炭墨底，七个组件色取自青、松、檀、金、朱、柿、灰一组东方色调。
- **图面等宽、界面无衬线**：图内文字保持等宽字体保证排版稳定，工具栏、标题、卡片用无衬线字体。
- 另有三个可选视觉预设：`signal-flow`（信号流动）、`blueprint`（蓝图）、`editorial`（出版物）。

## 致谢与许可

本项目基于 [tt-a1i/archify](https://github.com/tt-a1i/archify)（MIT）深度定制而来，上游又基于 Cocoon-AI/architecture-diagram-generator（MIT）。感谢原作者把工作做得这么扎实。

MIT License，详见 [LICENSE](LICENSE)。品牌图标等第三方素材的署名见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。
