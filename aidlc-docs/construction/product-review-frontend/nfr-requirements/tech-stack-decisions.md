# 技术栈决策 - 前端应用

## 技术栈概览

| 类别 | 技术 | 版本 | 选型理由 |
|------|------|------|----------|
| 框架 | React | 18.x | 组件化、生态丰富、团队熟悉 |
| 语言 | TypeScript | 5.x | 类型安全、开发体验好 |
| 构建 | Vite | 5.x | 快速构建、HMR 体验好 |
| UI库 | Ant Design | 5.x | 企业级组件、功能完善 |
| 状态管理 | Zustand | 4.x | 轻量简洁、学习成本低 |
| 路由 | React Router | 6.x | 官方推荐、功能完整 |
| HTTP | Axios | 1.x | 拦截器、错误处理方便 |

---

## 核心依赖

### 生产依赖

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.20.0",
    "antd": "^5.12.0",
    "@ant-design/icons": "^5.2.0",
    "axios": "^1.6.0",
    "zustand": "^4.4.0",
    "dayjs": "^1.11.0"
  }
}
```

### 开发依赖

```json
{
  "devDependencies": {
    "typescript": "^5.3.0",
    "vite": "^5.0.0",
    "@vitejs/plugin-react": "^4.2.0",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "eslint": "^8.55.0",
    "eslint-plugin-react": "^7.33.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "@typescript-eslint/eslint-plugin": "^6.13.0",
    "@typescript-eslint/parser": "^6.13.0",
    "prettier": "^3.1.0",
    "vitest": "^1.0.0",
    "@testing-library/react": "^14.1.0"
  }
}
```

---

## 项目配置

### Vite 配置

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
      },
    },
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom', 'react-router-dom'],
          antd: ['antd', '@ant-design/icons'],
        },
      },
    },
  },
});
```

### TypeScript 配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### ESLint 配置

```javascript
// .eslintrc.cjs
module.exports = {
  root: true,
  env: { browser: true, es2020: true },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
  ],
  ignorePatterns: ['dist', '.eslintrc.cjs'],
  parser: '@typescript-eslint/parser',
  plugins: ['react-refresh'],
  rules: {
    'react-refresh/only-export-components': [
      'warn',
      { allowConstantExport: true },
    ],
  },
};
```

---

## 目录结构

```
product-review-frontend/
├── public/
│   └── favicon.ico
├── src/
│   ├── api/                    # API 接口
│   │   ├── index.ts
│   │   ├── request.ts
│   │   ├── auth.ts
│   │   ├── product.ts
│   │   └── audit.ts
│   ├── components/             # 组件
│   │   ├── Layout/
│   │   │   ├── AppLayout.tsx
│   │   │   ├── SideMenu.tsx
│   │   │   └── UserDropdown.tsx
│   │   ├── common/
│   │   │   ├── PageHeader.tsx
│   │   │   ├── StatusTag.tsx
│   │   │   ├── LoadingSpinner.tsx
│   │   │   └── EmptyState.tsx
│   │   └── business/
│   │       ├── ProductTable.tsx
│   │       ├── ProductInfo.tsx
│   │       ├── AuditTimeline.tsx
│   │       ├── AuditForm.tsx
│   │       └── StatCard.tsx
│   ├── pages/                  # 页面
│   │   ├── Login.tsx
│   │   ├── Dashboard.tsx
│   │   ├── product/
│   │   │   ├── ProductList.tsx
│   │   │   ├── ProductForm.tsx
│   │   │   └── ProductDetail.tsx
│   │   └── audit/
│   │       ├── PendingList.tsx
│   │       ├── ReviewList.tsx
│   │       ├── AuditPanel.tsx
│   │       ├── AuditHistory.tsx
│   │       └── AuditStats.tsx
│   ├── store/                  # 状态管理
│   │   ├── index.ts
│   │   ├── userStore.ts
│   │   ├── productStore.ts
│   │   └── auditStore.ts
│   ├── types/                  # 类型定义
│   │   └── index.ts
│   ├── utils/                  # 工具函数
│   │   ├── errorHandler.ts
│   │   └── constants.ts
│   ├── App.tsx
│   ├── main.tsx
│   └── router.tsx
├── index.html
├── package.json
├── tsconfig.json
├── tsconfig.node.json
├── vite.config.ts
├── .eslintrc.cjs
├── .prettierrc
└── README.md
```

---

## 技术决策记录

| 决策ID | 决策内容 | 理由 | 日期 |
|--------|----------|------|------|
| FE-TD-001 | 使用 Vite 而非 CRA | 构建速度快，开发体验好 | 2026-01-09 |
| FE-TD-002 | 使用 Zustand 而非 Redux | 轻量简洁，学习成本低 | 2026-01-09 |
| FE-TD-003 | 使用 Ant Design 5 | 企业级组件库，功能完善 | 2026-01-09 |
| FE-TD-004 | 使用 TypeScript strict | 类型安全，减少运行时错误 | 2026-01-09 |
| FE-TD-005 | 使用 Axios 而非 fetch | 拦截器方便，错误处理统一 | 2026-01-09 |
| FE-TD-006 | 按路由代码分割 | 减少首屏加载体积 | 2026-01-09 |

---

## 开发规范

### 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 组件文件 | PascalCase | `ProductList.tsx` |
| 工具文件 | camelCase | `errorHandler.ts` |
| 组件名 | PascalCase | `ProductList` |
| 函数名 | camelCase | `handleSubmit` |
| 常量 | UPPER_SNAKE_CASE | `API_BASE_URL` |
| 类型/接口 | PascalCase | `ProductStatus` |

### 组件规范

```typescript
// 推荐的组件结构
import { FC } from 'react';

interface ProductListProps {
  status?: ProductStatus;
}

export const ProductList: FC<ProductListProps> = ({ status }) => {
  // hooks
  const { products, loading, fetchMyProducts } = useProductStore();
  
  // effects
  useEffect(() => {
    fetchMyProducts(1, status);
  }, [status]);
  
  // handlers
  const handlePageChange = (page: number) => {
    fetchMyProducts(page, status);
  };
  
  // render
  return (
    <div>
      {/* ... */}
    </div>
  );
};
```
