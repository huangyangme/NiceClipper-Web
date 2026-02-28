# NiceClipper – AI_CONTEXT.md（Kindle 标注整理工具）

Last Updated: 2026-02-24

---

## 0. 项目定位

NiceClipper 是一个 Web App，用来解析 Kindle 的 `My Clippings.txt`，进行结构化建模、去重处理，并导出为 Markdown。

定位：

- 本地优先
- 结构清晰
- 去重可靠
- Markdown 导出质量高
- 不做云同步平台

## 1. 数据模型设计

### Book

职责：

- 聚合同一本书的所有 ClipItem
- 提供排序、去重后条目集合
- 提供导出接口（按书生成 Markdown）

不负责：

- 原始文本解析
- StoreKit
- UI 状态

---

### ClipItem

核心字段建议：

- id
- bookTitle
- type（highlight / note / bookmark）
- location（原始）
- page（可选）
- createdAt（可选）
- text
- normalizedText（用于去重）

原则：

- ClipItem 是最小可持久实体
- 去重逻辑必须基于稳定字段

---

## 3. 解析层（ClippingsParser）

文件：

- `ClippingsParser.swift`

职责：

- 接收完整 `My Clippings.txt`
- 分块（按 =========）
- 解析出 Book + ClipItem
- 对异常块标记并跳过

原则：

- 解析错误不能导致全局崩溃
- 必须保持幂等（同文件 → 同结果）

---

## 4. 去重逻辑（目前在 ContentView+CopyAndDedupe）

当前结构：

- 去重逻辑与复制逻辑在扩展文件中
- 使用 TextNormalize 进行文本标准化

去重键建议：

- bookTitle（normalized）
- type
- location（normalized）
- normalizedText

原则：

- 同一书 + 同一位置 + 同一内容 = 重复
- 不做模糊相似度算法（避免不可解释结果）

---

## 5. 文本标准化（TextNormalize）

文件：

- `TextNormalize.swift`

职责：

- trim
- 统一换行
- 去除多余空格
- 可选：全角半角统一

原则：

- 标准化必须 deterministic
- 不要做语义改写

---

## 6. UI 架构原则

采用 Sidebar + Detail 结构：

- SidebarView：书列表
- BookDetailView：条目列表
- ClipRowView：单条展示

扩展拆分：

- Search 单独扩展
- Localization 单独扩展
- Copy + Dedupe 单独扩展

优点：

- ContentView 不膨胀
- 功能横向拆分清晰

---

## 7. IAP 架构

核心文件：

- `ProStore.swift`
- `PaywallView.swift`

原则：

- Pro 仅影响功能解锁
- 不影响解析与去重质量
- 使用 StoreKit 2

---

## 8. 当前技术边界

- 不做数据库持久化（无 CoreData）
- 不做云同步
- 不做复杂模板系统
- 不做 AI 处理

---

## 9. 工程哲学

- 解析稳定 > UI 花哨
- 去重可解释 > 智能猜测
- 拆分清晰 > 巨型文件
- 功能用扩展横向拆开
- 不预设未来需求

---

## 10. 未来可能的重构方向（暂不做）

- 将去重逻辑从 View 层抽离到独立 Deduper
- 将 Markdown 导出抽象为 Exporter
- 支持多导出模板
- 引入 Library 模式（可选）

当前阶段：

> 维持简单、稳定、可读。
