# 状态管理设计 - 前端应用

## 状态管理方案

采用 **Zustand** 作为状态管理库，轻量且易于使用。

---

## Store 结构

| Store | 职责 | 持久化 |
|-------|------|--------|
| userStore | 用户认证状态 | ✅ localStorage |
| productStore | 商品数据状态 | ❌ |
| auditStore | 审核数据状态 | ❌ |

---

## 1. UserStore

**路径**: `store/userStore.ts`

**职责**: 管理用户认证状态

```typescript
interface UserState {
  // 状态
  token: string | null;
  user: User | null;
  isAuthenticated: boolean;
  
  // 操作
  login: (username: string, password: string) => Promise<void>;
  logout: () => void;
  fetchCurrentUser: () => Promise<void>;
}

interface User {
  id: number;
  username: string;
  role: 'MERCHANT' | 'REVIEWER' | 'SENIOR_REVIEWER';
}
```

**实现**:
```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useUserStore = create<UserState>()(
  persist(
    (set, get) => ({
      token: null,
      user: null,
      isAuthenticated: false,

      login: async (username, password) => {
        const response = await authApi.login({ username, password });
        set({
          token: response.token,
          user: response.user,
          isAuthenticated: true,
        });
      },

      logout: () => {
        set({
          token: null,
          user: null,
          isAuthenticated: false,
        });
      },

      fetchCurrentUser: async () => {
        const user = await authApi.getCurrentUser();
        set({ user });
      },
    }),
    {
      name: 'user-storage',
      partialize: (state) => ({ token: state.token }),
    }
  )
);
```

---

## 2. ProductStore

**路径**: `store/productStore.ts`

**职责**: 管理商品列表和详情数据

```typescript
interface ProductState {
  // 列表状态
  products: Product[];
  loading: boolean;
  pagination: {
    current: number;
    pageSize: number;
    total: number;
  };
  filter: {
    status?: ProductStatus;
  };

  // 详情状态
  currentProduct: Product | null;
  detailLoading: boolean;

  // 操作
  fetchMyProducts: (page?: number, status?: ProductStatus) => Promise<void>;
  fetchProductDetail: (id: number) => Promise<void>;
  createProduct: (data: ProductCreateDTO) => Promise<void>;
  updateProduct: (id: number, data: ProductUpdateDTO) => Promise<void>;
  clearCurrentProduct: () => void;
}

interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
  status: ProductStatus;
  merchantId: number;
  createdAt: string;
  updatedAt: string;
}

type ProductStatus = 'PENDING' | 'FIRST_PASS' | 'APPROVED' | 'REJECTED' | 'NEED_MODIFY';
```

**实现**:
```typescript
export const useProductStore = create<ProductState>((set, get) => ({
  products: [],
  loading: false,
  pagination: { current: 1, pageSize: 10, total: 0 },
  filter: {},
  currentProduct: null,
  detailLoading: false,

  fetchMyProducts: async (page = 1, status) => {
    set({ loading: true });
    try {
      const response = await productApi.getMyProducts({ page, status });
      set({
        products: response.data,
        pagination: {
          current: page,
          pageSize: 10,
          total: response.total,
        },
        filter: { status },
      });
    } finally {
      set({ loading: false });
    }
  },

  fetchProductDetail: async (id) => {
    set({ detailLoading: true });
    try {
      const product = await productApi.getProduct(id);
      set({ currentProduct: product });
    } finally {
      set({ detailLoading: false });
    }
  },

  createProduct: async (data) => {
    await productApi.createProduct(data);
  },

  updateProduct: async (id, data) => {
    await productApi.updateProduct(id, data);
  },

  clearCurrentProduct: () => {
    set({ currentProduct: null });
  },
}));
```

---

## 3. AuditStore

**路径**: `store/auditStore.ts`

**职责**: 管理审核相关数据

```typescript
interface AuditState {
  // 待审核列表
  pendingProducts: Product[];
  pendingLoading: boolean;
  pendingPagination: Pagination;

  // 待复审列表
  reviewProducts: Product[];
  reviewLoading: boolean;
  reviewPagination: Pagination;

  // 审核历史
  auditHistory: AuditRecord[];
  historyLoading: boolean;

  // 统计数据
  stats: AuditStats | null;
  statsLoading: boolean;

  // 操作
  fetchPendingProducts: (page?: number) => Promise<void>;
  fetchReviewProducts: (page?: number) => Promise<void>;
  fetchAuditHistory: (productId: number) => Promise<void>;
  submitAudit: (productId: number, data: AuditSubmitDTO) => Promise<void>;
  fetchAuditStats: (startDate: string, endDate: string) => Promise<void>;
}

interface AuditRecord {
  id: number;
  productId: number;
  auditorId: number;
  auditorName: string;
  result: 'PASS' | 'REJECT' | 'NEED_MODIFY';
  comment: string | null;
  createdAt: string;
}

interface AuditStats {
  total: number;
  passed: number;
  rejected: number;
  needModify: number;
}

interface Pagination {
  current: number;
  pageSize: number;
  total: number;
}
```

**实现**:
```typescript
export const useAuditStore = create<AuditState>((set) => ({
  pendingProducts: [],
  pendingLoading: false,
  pendingPagination: { current: 1, pageSize: 10, total: 0 },

  reviewProducts: [],
  reviewLoading: false,
  reviewPagination: { current: 1, pageSize: 10, total: 0 },

  auditHistory: [],
  historyLoading: false,

  stats: null,
  statsLoading: false,

  fetchPendingProducts: async (page = 1) => {
    set({ pendingLoading: true });
    try {
      const response = await auditApi.getPendingList({ page });
      set({
        pendingProducts: response.data,
        pendingPagination: { current: page, pageSize: 10, total: response.total },
      });
    } finally {
      set({ pendingLoading: false });
    }
  },

  fetchReviewProducts: async (page = 1) => {
    set({ reviewLoading: true });
    try {
      const response = await auditApi.getReviewList({ page });
      set({
        reviewProducts: response.data,
        reviewPagination: { current: page, pageSize: 10, total: response.total },
      });
    } finally {
      set({ reviewLoading: false });
    }
  },

  fetchAuditHistory: async (productId) => {
    set({ historyLoading: true });
    try {
      const records = await auditApi.getAuditHistory(productId);
      set({ auditHistory: records });
    } finally {
      set({ historyLoading: false });
    }
  },

  submitAudit: async (productId, data) => {
    await auditApi.submitAudit(productId, data);
  },

  fetchAuditStats: async (startDate, endDate) => {
    set({ statsLoading: true });
    try {
      const stats = await auditApi.getAuditStats({ startDate, endDate });
      set({ stats });
    } finally {
      set({ statsLoading: false });
    }
  },
}));
```

---

## 数据流图

```
┌─────────────────────────────────────────────────────────────┐
│                        Components                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ProductList          AuditPanel          AuditStats       │
│       │                    │                   │            │
│       ▼                    ▼                   ▼            │
│  ┌─────────┐         ┌─────────┐         ┌─────────┐       │
│  │ product │         │  audit  │         │  audit  │       │
│  │  Store  │         │  Store  │         │  Store  │       │
│  └────┬────┘         └────┬────┘         └────┬────┘       │
│       │                   │                   │            │
│       ▼                   ▼                   ▼            │
│  ┌─────────────────────────────────────────────────┐       │
│  │                   API Layer                      │       │
│  │  productApi      auditApi       authApi          │       │
│  └─────────────────────────────────────────────────┘       │
│                           │                                 │
└───────────────────────────┼─────────────────────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │  Backend API  │
                    └───────────────┘
```

---

## 权限控制

### 路由守卫

```typescript
// router.tsx
const ProtectedRoute = ({ children, allowedRoles }) => {
  const { isAuthenticated, user } = useUserStore();
  
  if (!isAuthenticated) {
    return <Navigate to="/login" />;
  }
  
  if (allowedRoles && !allowedRoles.includes(user?.role)) {
    return <Navigate to="/" />;
  }
  
  return children;
};
```

### 使用示例

```tsx
<Route 
  path="/products" 
  element={
    <ProtectedRoute allowedRoles={['MERCHANT']}>
      <ProductList />
    </ProtectedRoute>
  } 
/>

<Route 
  path="/audit/pending" 
  element={
    <ProtectedRoute allowedRoles={['REVIEWER']}>
      <PendingList />
    </ProtectedRoute>
  } 
/>
```
