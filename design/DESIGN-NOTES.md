# dsh-subagent-max · 设计决策说明

> 静态高保真原型：`design/dsh-subagent-max.html`（双击浏览器直接打开，右上角可切亮/暗色）。

## 一、视觉基准（先读官方再设计）
- 逐字复刻 `@deepseek-ai/dsh-client-ui-theme` 的 token 表：`design-platform.css`（原色/alias/specific 全套 + 亮暗两态）、`base.css`（`--dsw-font-family`、`--ds-font-family-code`、动效曲线）、`gradient-shadow-text.css`（`--dsw-shadow-lv1/2/3` + 完整 `--dsw-font-*` 字号表）。
- 组件形态对齐官方源码：
  - **StateDot**（primitives）：done/warning/error = 圆点（外圈 10% 透明度 + 中心点）；**ongoing = 蓝色 8 格追光动画**（`--dsw-static-deepseek-450`）。
  - **SubagentCatalogAction**（ui-subagent）：trigger 无边框 12px、menu 336px/圆角 12px/阴影 lv3/内边距 4px、行 = 状态点 + 标题 + summary + 右侧 token/时长。
  - **ToolRow + toolview**（ui-tool + primitives）：折叠行 24px = [图标 16] [工具名 14px secondary] · [摘要 14px tertiary 省略]；运行中带 2.6s 横向扫光；展开体 = ioCard（12px/18px 代码）+ 终端/读取/diff 卡片（圆角 12px、行号 gutter、+/− 用 success/error 色）。

## 二、信息密度怎么做（核心）
1. **去前缀 label**：所有字段用「位置 + 图标 + 单位 + 字号/颜色层级」表达，零「xx：」。token → `12.4K tok`、tps → `18.2 t/s`、step → `12 steps`、时长 → `⏱ 2m42s`、创建 → `📅 14:32`、模型 → 等宽字体浅色、会话 id → 等宽 10px 最弱灰并截断。
2. **卡片三行栅格**：第 1 行 状态点 + 标题（13px/500 省略）+ mode pill + 子代理计数 + 打开小窗；第 2 行 指标条（上下文进度条 + token/tps/step 数值）；第 3 行 元信息（模型 · 时长 · 创建 + 右侧会话 id）。信息一次扫完，层级靠字号与颜色（primary/secondary/tertiary/caption）。
3. **上下文占比用进度条**：96×4px 细条 + 百分比，不用文字 label；接近上限可切 warn 色。
4. **弹窗正文更紧凑**：正文 14px/22px（`--dsw-font-s-14`，比主会话 16px/28px 低一档）；工具卡内容 12px/18px（`--dsw-font-markdown-code-block-small`），对齐官方 ToolRow 内终端覆盖字号——满足「不比主会话大、更紧凑」。

## 三、字号 / 间距体系
| 用途 | 规格 | token |
|---|---|---|
| 弹窗正文（流式） | 14/22 | `--dsw-font-s-14` |
| 弹窗标题（切换器） | 13/20 500 | `--dsw-font-xs-strong-13` |
| 工具卡标题/摘要 | 13/18 | `--dsw-font-xs-13` |
| 工具卡代码/终端/读取 | 12/18 等宽 | `--dsw-font-markdown-code-block-small` |
| 卡片标题 | 13/20 500 | `--dsw-font-xs-strong-13` |
| 卡片指标/元信息 | 11/16 | `--dsw-font-xxxs-11` |
| 会话 id | 10/14 等宽 | `--ds-font-family-code` |

间距走 4px 节奏：menu 内边距 4px、卡片内边距 9–12px、行 gap 4/6/8px。圆角：菜单/卡片 12px、弹窗 14px、行 hover 8px、pill 999px。阴影：弹窗/菜单 lv3、卡片 hover lv2。

## 四、状态色语义（圆点）
- **running**：蓝追光动画 `--dsw-static-deepseek-450`（官方 ongoing）。
- **done**：绿 `--dsw-alias-state-success-primary`。
- **idle**：灰实心 `--dsw-static-neutral-bluish-400`（已加载、回合间）。
- **ready**：灰空心 `--dsw-static-neutral-bluish-300`（仅存储、未加载）。
- **failed**：红 `--dsw-alias-state-error-primary`。
- **warning**：琥珀 `--dsw-alias-state-warn-primary`。

## 五、token 映射备注
- 你提到的 `--dsw-alias-accent` / `--dsw-alias-danger` / `--dsw-font-mono` 在官方并不存在，已映射到真实 token：accent→`--dsw-alias-state-business-primary`（deepseek-500/400）、danger→`--dsw-alias-state-error-primary`、mono→`--ds-font-family-code`（或 `--dsw-font-markdown-code-block`）。
- 所有 `var(--dsw-*, fallback)` 均带亮色 fallback，保证直接打开也能看。
