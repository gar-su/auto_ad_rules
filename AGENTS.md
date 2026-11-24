# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 项目概述

FB/TikTok 广告控损规则创建表单的高保真交互原型。单文件、零依赖。

## 开发

```bash
open index.html   # 浏览器预览，所有交互在浏览器控制台验证
```

无构建/测试/lint。修改后刷新浏览器即可验证。

## 架构

`index.html` 单文件三层结构：**内联 CSS → HTML 表单 → 原生 JS**，约 1930 行。

### 页面区块（3 个 section）

1. **S1 基本信息** — 名称、状态（生效/停用）
2. **S2 控损对象** — 时区、账户范围（双栏选表）、app类型、语种、业务类型、渠道号、短剧范围（双栏选表+搜索维度）、用户范围（部门树）、推广链接（双栏选表+批量搜索）
3. **S3 控损规则** — 生效时间、触发条件（AND/OR 行组）、触发操作、警告方式/频率/范围

### JS 架构（事件委托 + 配置驱动）

```
APP_TYPE_CONFIG        ← fb/nt/tt 差异化数据（label, metrics, showChannel, accounts, links, dramas）
MINI_PROGRAMS          ← TikTok 小程序列表（含独立 links）
render*() 函数         ← 数据驱动渲染表格/指标/小程序/短剧列表
onAppTypeChange()      ← app类型切换调度器
事件委托               ← 7 大交互模块都用委托模式
```

- **App 类型切换**：`APP_TYPE_CONFIG` 对象按 `fb`/`nt`/`tt` 键组织差异化数据，`onAppTypeChange()` 统一调度页面标题、指标选项、渠道号显隐、小程序选择区、表格数据联动。TikTok 模式下小程序选择单独维护 `MINI_PROGRAMS` 数组，监测链接取已选小程序 links 并集。NetToon 与 NetShort 共用 `[FB]` 标题、有渠道号、无小程序选择，但指标/账户/链接/短剧均为独立数据源。
- **双栏选表**（账户/短剧/推广链接）：左表 checkbox → 复制行到右侧已选列表 + `row-selected` 高亮；右侧删除 → 同步取消左侧 checkbox；清空全部。三个模块共用同一套事件委托逻辑（遍历 `.table-container`）。
- **部门树**：全选 → 父级 → 子级三级联动 + 半选态（`indeterminate`）；右侧卡片列表动态重建。
- **触发条件**：AND 行组内 `+/-` 增减行，`+ 或者` 插入 OR 分隔线+新行。指标 select 统一 `.metric-select` class，切 app 类型时批量更新。
- **批量搜索**（推广链接+短剧范围）：普通模式 ↔ 批量模式切换，textarea ↔ input 显隐，textarea 第一行内容带回。
- **保存**：`console.log(JSON)` 输出完整表单数据 + `alert` 提示。

### 关键 CSS 类名

| 类名 | 用途 |
|------|------|
| `.table-container` / `.table-wrapper` / `.selected-wrapper` | 双栏选表布局 |
| `.condition-row` / `.or-line` / `.add-or-btn` | 触发条件 AND/OR 结构 |
| `.search-dim` / `.batch-toggle` / `.batch-area` | 搜索维度 + 批量搜索 |
| `.mp-trigger-text` / `.mp-dropdown-item` / `.mp-checkbox` | TikTok 小程序多选 |
| `.row-selected` | 表格行选中高亮 |

### 已知限制（原型阶段）

搜索/分页无真实逻辑；表单无校验；无响应式；部门人员 Tab 无内容；预算调整未展开。这些在 DESIGN.md 中有扩展建议。

## 需求文档

- `DESIGN.md` — 架构参考：区块树、交互映射表、CSS 类名速查、设计 Token、数据结构
- `REQUIREMENT-搜索维度.md` — 短剧范围搜索维度选择器 + 批量搜索需求
- `REQUIREMENT-批量搜索.md` — 推广链接批量搜索需求
- `REQUIREMENT-TikTok小程序.md` — TikTok 小程序 App 类型完整需求（F1-F9）
- `REQUIREMENT-NetToon.md` — NetToon App 类型新增需求

需求文档是规格说明，`index.html` 是实现。修改前先对需求文档确认预期行为。
