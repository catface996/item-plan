# 设计模式 - 前端应用

## 概述

本文档定义前端应用的设计模式和最佳实践。

---

## 1. 组件设计模式

### 1.1 容器/展示组件模式

将组件分为容器组件（处理逻辑）和展示组件（纯UI）。

```typescript
// 展示组件：只负责渲染
interface ProductCardProps {
  product: Product;
  onEdit: () => void;
}

const ProductCard: FC<ProductCardProps> = ({ product, onEdit }) => (
  <Card>
    <h3>{product.name}</h3>
    <p>{product.description}</p>
    <Button onClick={onEdit}>编辑</Button>
  </Card>
);

// 容器组件：处理数据和逻辑
const ProductCardContainer: FC<{ productId: number }> = ({ productId }) => {
  const product = useProductStore((s) => s.products.find((p) => p.id === productId));
  const navigate = useNavigate();
  
  const handleEdit = () => navigate(`/products/${productId}/edit`);
  
  if (!product) return null;
  return <ProductCard product={product} onEdit={handleEdit} />;
};
```

### 1.2 复合组件模式

用于构建灵活的组件 API。

```typescript
// 复合组件示例：AuditPanel
const AuditPanel = ({ children }) => {
  return <div className="audit-panel">{children}</div>;
};

AuditPanel.Header = ({ title }) => <div className="panel-header">{title}</div>;
AuditPanel.ProductInfo = ({ product }) => <ProductInfo product={product} />;
AuditPanel.History = ({ records }) => <AuditTimeline records={records} />;
AuditPanel.Form = ({ onSubmit }) => <AuditForm onSubmit={onSubmit} />;

// 使用
<AuditPanel>
  <AuditPanel.Header title="审核商品" />
  <AuditPanel.ProductInfo product={product} />
  <AuditPanel.History records={records} />
  <AuditPanel.Form onSubmit={handleSubmit} />
</AuditPanel>
```

### 1.3 高阶组件模式 (HOC)

用于权限控制等横切关注点。

```typescript
// 权限控制 HOC
function withAuth<P extends object>(
  WrappedComponent: ComponentType<P>,
  allowedRoles: UserRole[]
) {
  return function AuthenticatedComponent(props: P) {
    const { isAuthenticated, user } = useUserStore();
    
    if (!isAuthenticated) {
      return <Navigate to="/login" />;
    }
    
    if (!allowedRoles.includes(user?.role)) {
      return <Navigate to="/" />;
    }
    
    return <WrappedComponent {...props} />;
  };
}

// 使用
const ProtectedProductList = withAuth(ProductList, ['MERCHANT']);
```

---

## 2. 状态管理模式

### 2.1 Store 切片模式

按功能域拆分 Store。

```typescript
// 每个 Store 只管理一个功能域
const useUserStore = create<UserState>(...);      // 用户认证
const useProductStore = create<ProductState>(...); // 商品管理
const useAuditStore = create<AuditState>(...);     // 审核管理
```

### 2.2 选择器模式

使用选择器避免不必要的重渲染。

```typescript
// ❌ 不推荐：订阅整个 store
const { products, loading } = useProductStore();

// ✅ 推荐：使用选择器只订阅需要的状态
const products = useProductStore((state) => state.products);
const loading = useProductStore((state) => state.loading);

// ✅ 更好：使用 shallow 比较
import { shallow } from 'zustand/shallow';

const { products, loading } = useProductStore(
  (state) => ({ products: state.products, loading: state.loading }),
  shallow
);
```

### 2.3 异步操作模式

统一的异步操作处理。

```typescript
// Store 中的异步操作模式
const useProductStore = create<ProductState>((set, get) => ({
  products: [],
  loading: false,
  error: null,

  fetchProducts: async (params) => {
    set({ loading: true, error: null });
    try {
      const data = await productApi.getMyProducts(params);
      set({ products: data.data, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },
}));
```

---

## 3. 路由设计模式

### 3.1 路由守卫模式

```typescript
// components/ProtectedRoute.tsx
interface ProtectedRouteProps {
  children: React.ReactNode;
  allowedRoles?: UserRole[];
}

export const ProtectedRoute: FC<ProtectedRouteProps> = ({ 
  children, 
  allowedRoles 
}) => {
  const { isAuthenticated, user } = useUserStore();
  const location = useLocation();

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (allowedRoles && user && !allowedRoles.includes(user.role)) {
    return <Navigate to="/" replace />;
  }

  return <>{children}</>;
};
```

### 3.2 懒加载路由模式

```typescript
// router.tsx
import { lazy, Suspense } from 'react';

const ProductList = lazy(() => import('@/pages/product/ProductList'));
const AuditPanel = lazy(() => import('@/pages/audit/AuditPanel'));

const router = createBrowserRouter([
  {
    path: '/products',
    element: (
      <Suspense fallback={<LoadingSpinner />}>
        <ProtectedRoute allowedRoles={['MERCHANT']}>
          <ProductList />
        </ProtectedRoute>
      </Suspense>
    ),
  },
]);
```

---

## 4. 表单处理模式

### 4.1 受控表单模式

使用 Ant Design Form 的受控模式。

```typescript
const ProductForm: FC = () => {
  const [form] = Form.useForm();
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (values: ProductCreateDTO) => {
    setLoading(true);
    try {
      await productApi.createProduct(values);
      message.success('创建成功');
      navigate('/products');
    } catch (error) {
      handleApiError(error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <Form form={form} onFinish={handleSubmit} layout="vertical">
      <Form.Item
        name="name"
        label="商品名称"
        rules={[
          { required: true, message: '请输入商品名称' },
          { max: 100, message: '最多100个字符' },
        ]}
      >
        <Input />
      </Form.Item>
      {/* ... */}
      <Button type="primary" htmlType="submit" loading={loading}>
        提交
      </Button>
    </Form>
  );
};
```

### 4.2 表单验证模式

```typescript
// 验证规则集中管理
export const productRules = {
  name: [
    { required: true, message: '请输入商品名称' },
    { min: 1, max: 100, message: '长度1-100个字符' },
  ],
  description: [
    { required: true, message: '请输入商品描述' },
    { min: 1, max: 1000, message: '长度1-1000个字符' },
  ],
  price: [
    { required: true, message: '请输入商品价格' },
    { type: 'number', min: 0.01, max: 999999.99, message: '价格范围0.01-999999.99' },
  ],
};

// 使用
<Form.Item name="name" label="商品名称" rules={productRules.name}>
  <Input />
</Form.Item>
```

---

## 5. 错误处理模式

### 5.1 全局错误边界

```typescript
// components/ErrorBoundary.tsx
class ErrorBoundary extends Component<Props, State> {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught:', error, errorInfo);
    // 可以上报错误到监控系统
  }

  render() {
    if (this.state.hasError) {
      return (
        <Result
          status="error"
          title="页面出错了"
          subTitle="请刷新页面重试"
          extra={<Button onClick={() => window.location.reload()}>刷新</Button>}
        />
      );
    }
    return this.props.children;
  }
}
```

### 5.2 API 错误处理

```typescript
// utils/errorHandler.ts
import { message } from 'antd';

export const handleApiError = (error: unknown) => {
  if (error instanceof Error) {
    message.error(error.message);
  } else {
    message.error('操作失败，请重试');
  }
};

// 使用 try-catch 包装
const handleSubmit = async () => {
  try {
    await someApiCall();
    message.success('操作成功');
  } catch (error) {
    handleApiError(error);
  }
};
```

---

## 6. 性能优化模式

### 6.1 Memo 优化

```typescript
// 使用 React.memo 避免不必要的重渲染
const ProductCard = memo<ProductCardProps>(({ product, onEdit }) => {
  return (
    <Card>
      <h3>{product.name}</h3>
      <Button onClick={onEdit}>编辑</Button>
    </Card>
  );
});

// 使用 useMemo 缓存计算结果
const filteredProducts = useMemo(
  () => products.filter((p) => p.status === status),
  [products, status]
);

// 使用 useCallback 缓存回调函数
const handleEdit = useCallback((id: number) => {
  navigate(`/products/${id}/edit`);
}, [navigate]);
```

### 6.2 虚拟列表模式

对于大数据量列表，使用虚拟滚动。

```typescript
// 使用 Ant Design Table 的虚拟滚动
<Table
  virtual
  scroll={{ y: 400 }}
  columns={columns}
  dataSource={data}
/>
```

---

## 设计模式追溯

| 模式 | NFR需求 | 应用场景 |
|------|---------|----------|
| 容器/展示组件 | 可维护性 | 所有业务组件 |
| Store 切片 | 可维护性 | 状态管理 |
| 选择器模式 | 性能 | Store 订阅 |
| 路由守卫 | 安全 | 权限控制 |
| 懒加载路由 | 性能 | 路由配置 |
| 错误边界 | 可用性 | 全局错误处理 |
| Memo 优化 | 性能 | 列表渲染 |
