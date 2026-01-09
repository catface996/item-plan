# 组件设计 - 前端应用

## 组件分类

| 类别 | 目录 | 描述 |
|------|------|------|
| 布局组件 | `components/Layout/` | 页面整体布局 |
| 通用组件 | `components/common/` | 可复用的基础组件 |
| 业务组件 | `components/business/` | 业务相关的复合组件 |

---

## 1. 布局组件

### 1.1 AppLayout

**路径**: `components/Layout/AppLayout.tsx`

**职责**: 应用整体布局，包含Header、Sider、Content

**Props**:
```typescript
interface AppLayoutProps {
  children: React.ReactNode;
}
```

**结构**:
```tsx
<Layout>
  <Header>
    <Logo />
    <UserDropdown />
  </Header>
  <Layout>
    <Sider>
      <SideMenu />
    </Sider>
    <Content>
      {children}
    </Content>
  </Layout>
</Layout>
```

### 1.2 SideMenu

**路径**: `components/Layout/SideMenu.tsx`

**职责**: 侧边导航菜单，根据用户角色显示不同菜单项

**Props**:
```typescript
interface SideMenuProps {
  role: UserRole;
  currentPath: string;
}
```

### 1.3 UserDropdown

**路径**: `components/Layout/UserDropdown.tsx`

**职责**: 用户信息下拉菜单，显示用户名和登出选项

**Props**:
```typescript
interface UserDropdownProps {
  username: string;
  onLogout: () => void;
}
```

---

## 2. 通用组件

### 2.1 PageHeader

**路径**: `components/common/PageHeader.tsx`

**职责**: 页面标题栏，支持标题、副标题、操作按钮

**Props**:
```typescript
interface PageHeaderProps {
  title: string;
  subtitle?: string;
  extra?: React.ReactNode;  // 右侧操作区
  onBack?: () => void;      // 返回按钮
}
```

**使用示例**:
```tsx
<PageHeader 
  title="我的商品" 
  extra={<Button type="primary">新建商品</Button>}
/>
```

### 2.2 StatusTag

**路径**: `components/common/StatusTag.tsx`

**职责**: 商品状态标签，统一状态显示样式

**Props**:
```typescript
interface StatusTagProps {
  status: ProductStatus;
}
```

**状态映射**:
```typescript
const statusConfig = {
  PENDING: { color: 'blue', text: '待审核' },
  FIRST_PASS: { color: 'cyan', text: '初审通过' },
  APPROVED: { color: 'green', text: '已通过' },
  REJECTED: { color: 'red', text: '已拒绝' },
  NEED_MODIFY: { color: 'orange', text: '需修改' },
};
```

### 2.3 LoadingSpinner

**路径**: `components/common/LoadingSpinner.tsx`

**职责**: 加载状态显示

**Props**:
```typescript
interface LoadingSpinnerProps {
  tip?: string;
  fullScreen?: boolean;
}
```

### 2.4 EmptyState

**路径**: `components/common/EmptyState.tsx`

**职责**: 空数据状态显示

**Props**:
```typescript
interface EmptyStateProps {
  description?: string;
  action?: React.ReactNode;
}
```

### 2.5 ConfirmModal

**路径**: `components/common/ConfirmModal.tsx`

**职责**: 确认对话框

**Props**:
```typescript
interface ConfirmModalProps {
  title: string;
  content: string;
  onConfirm: () => void;
  onCancel: () => void;
  visible: boolean;
  confirmLoading?: boolean;
}
```

---

## 3. 业务组件

### 3.1 ProductTable

**路径**: `components/business/ProductTable.tsx`

**职责**: 商品列表表格，支持不同场景的列配置

**Props**:
```typescript
interface ProductTableProps {
  data: Product[];
  loading: boolean;
  pagination: {
    current: number;
    pageSize: number;
    total: number;
  };
  onPageChange: (page: number, pageSize: number) => void;
  columns: 'merchant' | 'reviewer' | 'senior_reviewer';
  onAction: (action: string, product: Product) => void;
}
```

**列配置**:
```typescript
// merchant 模式：商家查看自己的商品
const merchantColumns = ['name', 'price', 'status', 'createdAt', 'actions'];

// reviewer 模式：审核员查看待审核商品
const reviewerColumns = ['name', 'merchant', 'price', 'createdAt', 'actions'];

// senior_reviewer 模式：高级审核员查看待复审商品
const seniorColumns = ['name', 'merchant', 'price', 'firstAuditTime', 'firstAuditor', 'actions'];
```

### 3.2 ProductInfo

**路径**: `components/business/ProductInfo.tsx`

**职责**: 商品信息展示卡片

**Props**:
```typescript
interface ProductInfoProps {
  product: Product;
  showMerchant?: boolean;
}
```

**展示内容**:
- 商品名称
- 商品描述
- 商品价格
- 商家信息（可选）
- 创建/更新时间

### 3.3 AuditTimeline

**路径**: `components/business/AuditTimeline.tsx`

**职责**: 审核历史时间线

**Props**:
```typescript
interface AuditTimelineProps {
  records: AuditRecord[];
}
```

**展示内容**:
- 审核时间
- 审核员姓名
- 审核结果（带颜色标识）
- 审核意见

### 3.4 AuditForm

**路径**: `components/business/AuditForm.tsx`

**职责**: 审核操作表单

**Props**:
```typescript
interface AuditFormProps {
  productId: number;
  onSubmit: (data: AuditSubmitDTO) => Promise<void>;
  loading: boolean;
}
```

**表单字段**:
```typescript
interface AuditSubmitDTO {
  result: 'PASS' | 'REJECT' | 'NEED_MODIFY';
  comment?: string;
}
```

**验证规则**:
- result: 必选
- comment: 当 result 为 REJECT 或 NEED_MODIFY 时必填

### 3.5 AuditFeedback

**路径**: `components/business/AuditFeedback.tsx`

**职责**: 显示最近一次审核反馈（用于商品编辑页）

**Props**:
```typescript
interface AuditFeedbackProps {
  record: AuditRecord;
}
```

### 3.6 StatCard

**路径**: `components/business/StatCard.tsx`

**职责**: 统计数据卡片

**Props**:
```typescript
interface StatCardProps {
  title: string;
  value: number;
  icon?: React.ReactNode;
  color?: string;
}
```

---

## 组件依赖关系

```
AppLayout
├── SideMenu
└── UserDropdown

ProductList (页面)
├── PageHeader
├── ProductTable
│   └── StatusTag
└── Pagination

ProductForm (页面)
├── PageHeader
├── Form (Ant Design)
└── AuditFeedback

ProductDetail (页面)
├── PageHeader
├── ProductInfo
│   └── StatusTag
└── AuditTimeline

AuditPanel (页面)
├── PageHeader
├── ProductInfo
├── AuditTimeline
└── AuditForm

AuditStats (页面)
├── PageHeader
├── DateRangePicker
└── StatCard (x4)
```
