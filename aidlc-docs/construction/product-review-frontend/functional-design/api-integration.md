# API 集成设计 - 前端应用

## API 模块结构

```
src/api/
├── index.ts          # 统一导出
├── request.ts        # Axios 实例配置
├── auth.ts           # 认证相关 API
├── product.ts        # 商品相关 API
└── audit.ts          # 审核相关 API
```

---

## 1. Request 配置

**路径**: `api/request.ts`

```typescript
import axios from 'axios';
import { useUserStore } from '@/store/userStore';

const request = axios.create({
  baseURL: '/api',
  timeout: 10000,
});

// 请求拦截器：添加 Token
request.interceptors.request.use((config) => {
  const token = useUserStore.getState().token;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// 响应拦截器：统一错误处理
request.interceptors.response.use(
  (response) => response.data,
  (error) => {
    const { response } = error;
    
    if (response?.status === 401) {
      // Token 过期，清除登录状态
      useUserStore.getState().logout();
      window.location.href = '/login';
    }
    
    // 提取错误信息
    const message = response?.data?.message || '请求失败';
    return Promise.reject(new Error(message));
  }
);

export default request;
```

---

## 2. Auth API

**路径**: `api/auth.ts`

```typescript
import request from './request';

export interface LoginRequest {
  username: string;
  password: string;
}

export interface LoginResponse {
  token: string;
  user: {
    id: number;
    username: string;
    role: string;
  };
}

export interface UserInfo {
  id: number;
  username: string;
  role: string;
}

export const authApi = {
  // POST /api/auth/login
  login: (data: LoginRequest): Promise<LoginResponse> => {
    return request.post('/auth/login', data);
  },

  // POST /api/auth/logout
  logout: (): Promise<void> => {
    return request.post('/auth/logout');
  },

  // GET /api/auth/me
  getCurrentUser: (): Promise<UserInfo> => {
    return request.get('/auth/me');
  },
};
```

---

## 3. Product API

**路径**: `api/product.ts`

```typescript
import request from './request';

export interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
  status: ProductStatus;
  merchantId: number;
  createdAt: string;
  updatedAt: string;
}

export type ProductStatus = 
  | 'PENDING' 
  | 'FIRST_PASS' 
  | 'APPROVED' 
  | 'REJECTED' 
  | 'NEED_MODIFY';

export interface ProductCreateDTO {
  name: string;
  description: string;
  price: number;
}

export interface ProductUpdateDTO {
  name: string;
  description: string;
  price: number;
}

export interface PageResponse<T> {
  data: T[];
  total: number;
  page: number;
  pageSize: number;
}

export interface ProductListParams {
  page?: number;
  pageSize?: number;
  status?: ProductStatus;
}

export const productApi = {
  // POST /api/products
  createProduct: (data: ProductCreateDTO): Promise<Product> => {
    return request.post('/products', data);
  },

  // GET /api/products/{id}
  getProduct: (id: number): Promise<Product> => {
    return request.get(`/products/${id}`);
  },

  // GET /api/products/my
  getMyProducts: (params: ProductListParams): Promise<PageResponse<Product>> => {
    return request.get('/products/my', { params });
  },

  // PUT /api/products/{id}
  updateProduct: (id: number, data: ProductUpdateDTO): Promise<Product> => {
    return request.put(`/products/${id}`, data);
  },
};
```

---

## 4. Audit API

**路径**: `api/audit.ts`

```typescript
import request from './request';
import { Product, PageResponse } from './product';

export interface AuditRecord {
  id: number;
  productId: number;
  auditorId: number;
  auditorName: string;
  result: AuditResult;
  comment: string | null;
  createdAt: string;
}

export type AuditResult = 'PASS' | 'REJECT' | 'NEED_MODIFY';

export interface AuditSubmitDTO {
  result: AuditResult;
  comment?: string;
}

export interface AuditStats {
  total: number;
  passed: number;
  rejected: number;
  needModify: number;
}

export interface AuditListParams {
  page?: number;
  pageSize?: number;
}

export interface AuditStatsParams {
  startDate: string;  // YYYY-MM-DD
  endDate: string;    // YYYY-MM-DD
}

export const auditApi = {
  // GET /api/audits/pending
  getPendingList: (params: AuditListParams): Promise<PageResponse<Product>> => {
    return request.get('/audits/pending', { params });
  },

  // GET /api/audits/review
  getReviewList: (params: AuditListParams): Promise<PageResponse<Product>> => {
    return request.get('/audits/review', { params });
  },

  // POST /api/audits/{productId}
  submitAudit: (productId: number, data: AuditSubmitDTO): Promise<void> => {
    return request.post(`/audits/${productId}`, data);
  },

  // GET /api/audits/history/{productId}
  getAuditHistory: (productId: number): Promise<AuditRecord[]> => {
    return request.get(`/audits/history/${productId}`);
  },

  // GET /api/audits/stats
  getAuditStats: (params: AuditStatsParams): Promise<AuditStats> => {
    return request.get('/audits/stats', { params });
  },
};
```

---

## 5. 统一导出

**路径**: `api/index.ts`

```typescript
export * from './auth';
export * from './product';
export * from './audit';

export { authApi } from './auth';
export { productApi } from './product';
export { auditApi } from './audit';
```

---

## API 与页面映射

| 页面 | 调用的 API | 说明 |
|------|-----------|------|
| Login | `authApi.login` | 用户登录 |
| Dashboard | `authApi.getCurrentUser` | 获取当前用户 |
| ProductList | `productApi.getMyProducts` | 获取商家商品列表 |
| ProductForm (新建) | `productApi.createProduct` | 创建商品 |
| ProductForm (编辑) | `productApi.getProduct`, `productApi.updateProduct` | 获取并更新商品 |
| ProductDetail | `productApi.getProduct`, `auditApi.getAuditHistory` | 商品详情和审核历史 |
| PendingList | `auditApi.getPendingList` | 待初审列表 |
| ReviewList | `auditApi.getReviewList` | 待复审列表 |
| AuditPanel | `productApi.getProduct`, `auditApi.getAuditHistory`, `auditApi.submitAudit` | 审核操作 |
| AuditHistory | `auditApi.getAuditHistory` | 审核历史 |
| AuditStats | `auditApi.getAuditStats` | 审核统计 |

---

## 错误处理

### 错误码映射

```typescript
// utils/errorHandler.ts
import { message } from 'antd';

const errorMessages: Record<string, string> = {
  PRODUCT_NOT_FOUND: '商品不存在',
  INVALID_STATUS: '商品状态不允许此操作',
  PERMISSION_DENIED: '无权限执行此操作',
  VALIDATION_ERROR: '数据验证失败',
  DUPLICATE_AUDIT: '不能重复审核同一商品',
  INTERNAL_ERROR: '系统内部错误，请稍后重试',
};

export const handleApiError = (error: Error) => {
  const msg = errorMessages[error.message] || error.message || '操作失败';
  message.error(msg);
};
```

### 使用示例

```typescript
// 在组件中使用
const handleSubmit = async (values) => {
  try {
    await productApi.createProduct(values);
    message.success('商品创建成功');
    navigate('/products');
  } catch (error) {
    handleApiError(error);
  }
};
```
