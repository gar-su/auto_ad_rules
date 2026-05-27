# 控损规则创建 — 高保真原型设计文档

## 概述

- **文件**: `index.html` (单文件，零依赖)
- **行数**: ~1580
- **技术**: HTML5 + 内联 CSS + 原生 JS (事件委托)
- **用途**: FB 广告控损规则创建表单的高保真交互原型

## 页面区块树

```
创建[FB]规则控损 (container)
│
├── header: 返回链接 + 页面标题
│
├── [S1] 基本信息 (section-title)
│   ├── 名称: text input #name
│   └── 状态: .status-group > .status-btn × 2 (生效|停用) 互斥
│
├── [S2] 控损对象 (section-title) ← 父级分组
│   ├── 时区: select (零时区)
│   │
│   ├── 账户范围: .form-item
│   │   ├── .tab-group > .tab-btn × 3 (全部|部分|排除) 互斥
│   │   ├── .search-bar > input + search-btn
│   │   └── .table-container
│   │       ├── .table-wrapper > table (候选: 账户名称, 账户ID)
│   │       └── .selected-wrapper > table (已选: 账户名称, 账户ID, 操作)
│   │           └── .clear-all > a (清空全部)
│   │
│   ├── app类型: select (NetShort)
│   ├── 语种: .tag-group > .tag × N (English, 简体中文)
│   ├── 业务类型: select (全部)
│   ├── 渠道号名称: .tag-group > .tag
│   │
│   ├── 短剧范围: .form-item
│   │   ├── .tab-group > .tab-btn × 3 (全部|部分|排除) 互斥
│   │   ├── .search-bar > input + search-btn
│   │   └── .table-container
│   │       ├── .table-wrapper > table (候选: 剧本名称, 短剧名称, 语种)
│   │       └── .selected-wrapper > table (已选: 剧本名称, 语种, 操作)
│   │
│   ├── 用户范围: .form-item
│   │   ├── .tab-group > .tab-btn × 2 (部门|人员) 互斥
│   │   └── .table-container
│   │       ├── .table-wrapper > checkbox tree
│   │       │   ├── #all-dept (全选)
│   │       │   ├── #dept-root (投放部门) ← 父级
│   │       │   │   └── #dept1-1~7 (海外投放二~七部)
│   │       │   └── #dept2-1~6 (海外素材一~六部) ← 根级
│   │       └── .selected-wrapper > 已选卡片列表 + 清空 + 计数器
│   │
│   └── 推广链接: .form-item
│       ├── .tab-group > .tab-btn × 3 (全部|部分|排除) 互斥
│       ├── .search-bar > input + search-btn
│       ├── .batch-area > textarea + search-btn (默认隐藏)
│       ├── .batch-toggle (批量搜索 ↔ 普通搜索)
│       └── .table-container
│           ├── .table-wrapper > table (候选: 链接名称, 链接ID)
│           └── .selected-wrapper > table (已选: 链接名称, 链接ID, 操作)
│
├── [S3] 控损规则 (section-title)
│   ├── 生效时间: .time-range (开始 → 结束) + 添加时间范围
│   │
│   ├── 触发条件: .trigger-condition
│   │   ├── .condition-top (自然日 + 说明文字)
│   │   ├── .condition-row × N (AND 组内行)
│   │   │   └── [指标 select] [时间 input] [单位 select] [运算符 select] [阈值 input] [+/- 按钮]
│   │   ├── .or-line (或者 分隔符)
│   │   └── .add-or-btn (+ 或者)
│   │
│   ├── 炸量信号: select (是|否)
│   ├── 触发操作: .action-group > .action-btn × 4 互斥
│   ├── 警告方式: radio (飞书)
│   ├── 警告频率: input[60] + 分钟提醒一次
│   └── 警告范围: checkbox × 2 (本人, 上级)
│
└── .footer-buttons
    ├── 取消 (.cancel-btn)
    └── 保存 (.save-btn)
```

## 交互映射

| ID | 选择器 | 事件 | 行为 |
|----|--------|------|------|
| I1 | `.status-group` | click `.status-btn` | 同组内互斥切换 `active` |
| I2 | `.tab-group` | click `.tab-btn` | 同组内互斥切换 `active` |
| I3 | `.trigger-condition` | click `.btn-add` | 克隆当前行插入其后，清空 input 值 |
| I4 | `.trigger-condition` | click `.btn-del` / `.delete-text` | 删除当前行；若为该 OR 组最后一行，同时删除 `or-line` 分隔符 |
| I5 | `.trigger-condition` | click `.add-or-btn` | 在其前插入 `or-line` + 一个空条件行 |
| I6 | `.action-group` | click `.action-btn` | 同组内互斥切换 `active` |
| I7 | `.table-container` | change `input[checkbox]` (左侧) | checked: 复制行数据到右侧已选列表；unchecked: 从右侧移除 |
| I8 | `.selected-wrapper` | click `.delete-btn` | 删除右侧行，同步取消左侧 checkbox |
| I9 | `.selected-wrapper` | click `.clear-all a` | 清空右侧全部，取消所有左侧 checkbox |
| I10 | `#all-dept` | change | 全选/取消所有部门 checkbox |
| I11 | `#dept-root` | change | 全选/取消 dept1-1~7 |
| I12 | `#dept1-*` `#dept2-*` | change | 更新父级/全选半选态，重建右侧卡片列表 |
| I13 | 右侧部门卡片 `button` | click | 取消对应 checkbox，更新树和卡片 |
| I14 | 右侧部门 `a` (清空) | click | 取消所有部门 checkbox |
| I15 | `.batch-toggle` | click | 切换搜索栏/textarea 显隐，切换文字"批量搜索"↔"普通搜索" |
| I16 | `.save-btn` | click | 收集全表单数据 → `console.log(JSON)` |

## 保存输出数据结构

```json
{
  "name": "string",
  "status": "生效|停用",
  "effectiveStart": "string",
  "effectiveEnd": "string",
  "triggerConditions": [
    [
      {
        "metric": "CPI|老达标率|...",
        "timeRange": "string",
        "timeUnit": "分钟|小时",
        "operator": ">|<|=|>=|<=",
        "value": "string",
        "isPercent": "bool"
      }
    ]
  ],
  "explosionSignal": "是|否",
  "triggerAction": "发送警告|关停计划|开启计划|调整预算",
  "warningMethod": "飞书",
  "warningFrequency": "string",
  "warningScope": ["本人", "上级"],
  "timezone": "string",
  "accounts": [{"name": "string", "id": "string"}],
  "appType": "string",
  "languages": ["string"],
  "businessType": "string",
  "channelName": "string",
  "dramas": [{"scriptName": "string", "language": "string"}],
  "departments": ["string"],
  "promotionLinks": [{"name": "string", "id": "string"}]
}
```

## CSS 类名速查

| 类名 | 用途 |
|------|------|
| `.container` | 页面容器 max-width:1400px |
| `.section-title` | 区块标题 (蓝色左边框 + 16px 加粗) |
| `.form-item` | 表单项行 (flex, label 100px + 内容区 flex:1) |
| `.form-label` | 标签 (100px 宽, 14px) |
| `.form-label span` | 必填红星 |
| `.form-content` | 表单内容区 (flex:1) |
| `.input-box` | 通用输入框/下拉 (max-width:350px) |
| `.status-group` / `.status-btn` | 状态互斥按钮组 |
| `.tab-group` / `.tab-btn` | Tab 互斥按钮组 |
| `.search-bar` / `.search-input` / `.search-btn` | 搜索栏 |
| `.batch-toggle` / `.batch-area` / `.batch-textarea` | 批量搜索 (仅推广链接) |
| `.table-container` / `.table-wrapper` / `.selected-wrapper` | 双栏选表布局 |
| `.row-selected` | 左侧表格选中行高亮 (#e6f7ff) |
| `.delete-btn` | 红色删除按钮 |
| `.clear-all` | 清空全部链接 (右对齐) |
| `.pagination` / `.page-btn` / `.page-size` | 分页组件 |
| `.tag-group` / `.tag` / `.tag .close` | 标签组件 |
| `.empty-state` | 空态占位 (暂无数据) |
| `.time-range` / `.time-input` / `.time-add` / `.time-tip` | 时间范围选择 |
| `.trigger-condition` / `.condition-top` / `.condition-row` | 触发条件区域 |
| `.trigger-select` | 触发条件下拉 (min-width:120px) |
| `.select-box` | 条件行指标下拉 (min-width:180px) |
| `.operator-select` | 条件行运算符/单位下拉 |
| `.value-input` | 条件行阈值输入 |
| `.percent-input` / `.percent-symbol` | 百分比输入组 |
| `.btn-add` / `.btn-del` | 条件行 +/- 圆形按钮 |
| `.or-line` | "或者"分隔线 |
| `.add-or-btn` | "+ 或者"按钮 |
| `.delete-text` | "删除"文字链接 |
| `.action-group` / `.action-btn` | 触发操作按钮组 |
| `.radio-group` / `.radio-item` | 警告方式单选 |
| `.frequency-input` | 警告频率输入 |
| `.checkbox-group` / `.checkbox-item` | 警告范围复选 |
| `.footer-buttons` / `.footer-btn` | 底部操作栏 |
| `.cancel-btn` | 取消按钮 (白底灰边) |
| `.save-btn` | 保存按钮 (蓝底白字) |

## 设计 Token (硬编码值，待提取为 CSS 变量)

```
--color-primary: #1677ff
--color-danger:  #ff4d4f
--color-border:  #d9d9d9
--color-bg:      #f5f7fa
--color-warning: #ffc107       (仅部门已选卡片)
--color-selected:#e6f7ff       (表格行选中)
--color-text:    #333
--color-text-secondary: #666
--color-text-disabled: #999

--font-size:     14px
--font-size-lg:  16px

--spacing-xs:    4px
--spacing-sm:    8px
--spacing-md:    16px
--spacing-lg:    20px
--spacing-xl:    24px
--spacing-xxl:   30px

--border-radius: 4px
```

## 已知限制 & 扩展点

| 类别 | 现状 | 建议扩展方向 |
|------|------|-------------|
| 搜索 | 按钮无逻辑 | 前端过滤表格行 (按名称/ID match) |
| 分页 | 纯静态展示 | 前端分页切片 + 页码点击事件 |
| 表单校验 | 无 | required 字段空值检查 + 红色边框提示 |
| 取消按钮 | 无逻辑 | 清空表单 / 回退上一页 |
| 保存按钮 | console.log | 替换为 fetch POST |
| 弹窗确认 | 无 | 取消/删除时 confirm modal |
| 响应式 | 无 | @media breakpoint 折叠双栏为上下布局 |
| 部门"人员"Tab | 无内容 | 人员搜索 + 选择器 |
| 调整预算 | 按钮已展示 | 点击后展开预算调整表单 |
| 炸量信号 ⓘ | 纯文字 | tooltip/popover |
| 已选部门卡片色 | 黄色 #ffc107 | 统一为蓝色主题 |
