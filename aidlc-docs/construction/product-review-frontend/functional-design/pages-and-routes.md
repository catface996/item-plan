# 页面与路由设计 - 前端应用

## 路由结构

| 路由 | 页面 | 权限 | 描述 |
|------|------|------|------|
| `/login` | Login | 公开 | 登录页 |
| `/` | Dashboard | 登录用户 | 首页/仪表盘 |
| `/products` | ProductList | MERCHANT | 我的商品列表 |
| `/products/new` | ProductForm | MERCHANT | 新建商品 |
| `/products/:id` | ProductDetail | MERCHANT | 商品详情 |
| `/products/:id/edit` | ProductForm | MERCHANT | 编辑商品 |
| `/audit/pending` | PendingList | REVIEWER | 待初审列表 |
| `/audit/review` | ReviewList | SENIOR_REVIEWER | 待复审列表 |
| `/audit/:productId` | AuditPanel | REVIEWER, SENIOR_REVIEWER | 审核操作页 |
| `/audit/history/:productId` | AuditHistory | REVIEWER, SENIOR_REVIEWER | 审核历史 |
| `/audit/stats` | AuditStats | SENIOR_REVIEWER | 审核统计 |

---

## 页面详细设计

### 1. Login 登录页

**路由**: `/login`

**功能**:
- 用户名密码输入
- 登录验证
- 登录成功跳转到首页

**组件结构**:
```
Login
├── LoginForm
│   ├── Input (用户名)
│   ├── Input.Password (密码)
│   └── Button (登录)
└── ErrorMessage
```

**交互逻辑**:
- 输入验证：用户名非空，密码非空
- 登录失败显示错误提示
- 登录成功保存Token，跳转首页

---

### 2. Dashboard 首页

**路由**: `/`

**功能**:
- 根据角色显示不同内容
- 快捷入口导航

**组件结构**:
```
Dashboard
├── WelcomeCard (欢迎信息)
├── QuickActions (快捷操作)
│   ├── [MERCHANT] 新建商品、我的商品
│   ├── [REVIEWER] 待审核列表
│   └── [SENIOR_REVIEWER] 待复审列表、审核统计
└── RecentActivity (最近动态，可选)
```

---

### 3. ProductList 商品列表页

**路由**: `/products`
**权限**: MERCHANT

**功能**:
- 显示当前商家的所有商品
- 按状态筛选
- 分页查询

**组件结构**:
```
ProductList
├── PageHeader (页面标题 + 新建按钮)
├── FilterBar
│   └── Select (状态筛选)
├── ProductTable
│   ├── Column: 商品名称
│   ├── Column: 价格
│   ├── Column: 状态 (Tag)
│   ├── Column: 创建时间
│   └── Column: 操作 (查看/编辑)
└── Pagination
```

**状态标签颜色**:
| 状态 | 颜色 | 文本 |
|------|------|------|
| PENDING | blue | 待审核 |
| FIRST_PASS | cyan | 初审通过 |
| APPROVED | green | 已通过 |
| REJECTED | red | 已拒绝 |
| NEED_MODIFY | orange | 需修改 |

---

### 4. ProductForm 商品表单页

**路由**: `/products/new` (新建) 或 `/products/:id/edit` (编辑)
**权限**: MERCHANT

**功能**:
- 新建：填写商品信息并提交
- 编辑：修改被退回的商品

**组件结构**:
```
ProductForm
├── PageHeader (新建商品 / 编辑商品)
├── Form
│   ├── Form.Item: 商品名称 (Input)
│   ├── Form.Item: 商品描述 (TextArea)
│   └── Form.Item: 商品价格 (InputNumber)
├── [编辑模式] AuditFeedback (显示审核意见)
└── ButtonGroup
    ├── Button (取消)
    └── Button (提交)
```

**验证规则**:
| 字段 | 规则 |
|------|------|
| name | 必填，1-100字符 |
| description | 必填，1-1000字符 |
| price | 必填，0.01-999999.99 |

---

### 5. ProductDetail 商品详情页

**路由**: `/products/:id`
**权限**: MERCHANT

**功能**:
- 显示商品完整信息
- 显示审核历史
- 提供编辑入口（仅需修改状态）

**组件结构**:
```
ProductDetail
├── PageHeader (商品名称 + 状态Tag)
├── Descriptions (商品信息)
│   ├── 商品名称
│   ├── 商品描述
│   ├── 商品价格
│   ├── 创建时间
│   └── 更新时间
├── AuditTimeline (审核历史时间线)
└── [NEED_MODIFY] Button (去修改)
```

---

### 6. PendingList 待初审列表

**路由**: `/audit/pending`
**权限**: REVIEWER

**功能**:
- 显示所有待初审商品
- 分页查询
- 进入审核操作

**组件结构**:
```
PendingList
├── PageHeader (待初审商品)
├── ProductTable
│   ├── Column: 商品名称
│   ├── Column: 商家
│   ├── Column: 价格
│   ├── Column: 提交时间
│   └── Column: 操作 (去审核)
└── Pagination
```

---

### 7. ReviewList 待复审列表

**路由**: `/audit/review`
**权限**: SENIOR_REVIEWER

**功能**:
- 显示所有待复审商品（初审通过）
- 分页查询
- 进入审核操作

**组件结构**:
```
ReviewList
├── PageHeader (待复审商品)
├── ProductTable
│   ├── Column: 商品名称
│   ├── Column: 商家
│   ├── Column: 价格
│   ├── Column: 初审时间
│   ├── Column: 初审员
│   └── Column: 操作 (去审核)
└── Pagination
```

---

### 8. AuditPanel 审核操作页

**路由**: `/audit/:productId`
**权限**: REVIEWER, SENIOR_REVIEWER

**功能**:
- 显示商品详细信息
- 显示历史审核记录
- 执行审核操作

**组件结构**:
```
AuditPanel
├── PageHeader (审核商品)
├── ProductInfo (商品信息卡片)
│   ├── 商品名称
│   ├── 商品描述
│   ├── 商品价格
│   └── 商家信息
├── AuditHistory (历史审核记录)
├── Divider
└── AuditForm (审核表单)
    ├── Radio.Group (审核结果)
    │   ├── 通过
    │   ├── 拒绝
    │   └── 需修改
    ├── TextArea (审核意见)
    └── Button (提交审核)
```

**交互逻辑**:
- 选择"拒绝"或"需修改"时，审核意见必填
- 提交成功后返回列表页

---

### 9. AuditHistory 审核历史页

**路由**: `/audit/history/:productId`
**权限**: REVIEWER, SENIOR_REVIEWER

**功能**:
- 显示商品完整审核历史

**组件结构**:
```
AuditHistory
├── PageHeader (审核历史)
├── ProductSummary (商品摘要)
└── Timeline (审核时间线)
    └── Timeline.Item
        ├── 审核时间
        ├── 审核员
        ├── 审核结果
        └── 审核意见
```

---

### 10. AuditStats 审核统计页

**路由**: `/audit/stats`
**权限**: SENIOR_REVIEWER

**功能**:
- 显示审核统计数据
- 支持日期范围筛选

**组件结构**:
```
AuditStats
├── PageHeader (审核统计)
├── DateRangePicker (日期筛选)
├── StatCards (统计卡片)
│   ├── Card: 总审核数
│   ├── Card: 通过数
│   ├── Card: 拒绝数
│   └── Card: 需修改数
└── Charts (可选，图表展示)
    └── PieChart (审核结果分布)
```

---

## 布局结构

### 整体布局

```
┌─────────────────────────────────────────────────┐
│                    Header                        │
│  Logo          Navigation          UserInfo     │
├─────────┬───────────────────────────────────────┤
│         │                                       │
│  Sider  │              Content                  │
│  Menu   │                                       │
│         │                                       │
│         │                                       │
└─────────┴───────────────────────────────────────┘
```

### 侧边菜单（按角色）

**MERCHANT**:
- 首页
- 我的商品
- 新建商品

**REVIEWER**:
- 首页
- 待审核列表
- 审核历史

**SENIOR_REVIEWER**:
- 首页
- 待复审列表
- 审核统计
- 审核历史
