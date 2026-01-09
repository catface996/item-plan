# 工作单元定义 - 商品审核模块

## 概述

本项目采用前后端分离架构，拆分为两个独立的工作单元：
- **后端工作单元**: Spring Boot API 服务
- **前端工作单元**: React Web 应用

---

## 工作单元列表

| 单元ID | 单元名称 | 技术栈 | 部署方式 |
|--------|----------|--------|----------|
| Unit-001 | product-review-backend | Spring Boot + MyBatis-Plus + MySQL | JAR包部署 |
| Unit-002 | product-review-frontend | React + TypeScript + Vite | 静态资源部署 |

---

## Unit-001: 后端服务 (product-review-backend)

### 基本信息

| 属性 | 值 |
|------|-----|
| 单元名称 | product-review-backend |
| 单元类型 | RESTful API 服务 |
| 技术栈 | Java 17 + Spring Boot 3.x + MyBatis-Plus |
| 数据库 | MySQL 8.0 |
| 部署方式 | JAR包，本地服务器 |
| 端口 | 8080 |

### 职责范围
- 提供 RESTful API 接口
- 商品数据管理
- 审核流程处理
- 用户认证授权 (JWT)
- 数据持久化

### 包含模块

| 模块 | 包名 | 职责 |
|------|------|------|
| common | com.example.review.common | 公共配置、异常处理、响应封装 |
| user | com.example.review.user | 用户认证、授权、JWT |
| product | com.example.review.product | 商品CRUD、状态管理 |
| audit | com.example.review.audit | 审核流程、审核记录 |

### 项目结构

```
product-review-backend/
├── src/main/java/com/example/review/
│   ├── ProductReviewApplication.java
│   ├── common/
│   │   ├── config/
│   │   ├── exception/
│   │   ├── response/
│   │   └── util/
│   ├── user/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── mapper/
│   │   ├── entity/
│   │   └── dto/
│   ├── product/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── mapper/
│   │   ├── entity/
│   │   └── dto/
│   └── audit/
│       ├── controller/
│       ├── service/
│       ├── mapper/
│       ├── entity/
│       └── dto/
├── src/main/resources/
│   ├── application.yml
│   └── mapper/
├── src/test/java/
└── pom.xml
```

---

## Unit-002: 前端应用 (product-review-frontend)

### 基本信息

| 属性 | 值 |
|------|-----|
| 单元名称 | product-review-frontend |
| 单元类型 | SPA Web 应用 |
| 技术栈 | React 18 + TypeScript + Vite |
| UI框架 | Ant Design 5 |
| 状态管理 | Zustand |
| 部署方式 | 静态资源，Nginx |
| 端口 | 3000 (开发) / 80 (生产) |

### 职责范围
- 用户界面展示
- 用户交互处理
- API 调用封装
- 路由管理
- 状态管理

### 包含模块

| 模块 | 目录 | 职责 |
|------|------|------|
| api | src/api/ | API 接口封装 |
| components | src/components/ | 公共组件 |
| pages/product | src/pages/product/ | 商品相关页面 |
| pages/audit | src/pages/audit/ | 审核相关页面 |
| pages/user | src/pages/user/ | 用户相关页面 |
| store | src/store/ | 状态管理 |
| utils | src/utils/ | 工具函数 |

### 项目结构

```
product-review-frontend/
├── src/
│   ├── api/
│   │   ├── index.ts
│   │   ├── product.ts
│   │   ├── audit.ts
│   │   └── auth.ts
│   ├── components/
│   │   ├── Layout/
│   │   └── common/
│   ├── pages/
│   │   ├── product/
│   │   │   ├── ProductList.tsx
│   │   │   ├── ProductForm.tsx
│   │   │   └── ProductDetail.tsx
│   │   ├── audit/
│   │   │   ├── PendingList.tsx
│   │   │   ├── ReviewList.tsx
│   │   │   ├── AuditPanel.tsx
│   │   │   ├── AuditHistory.tsx
│   │   │   └── AuditStats.tsx
│   │   └── user/
│   │       ├── Login.tsx
│   │       └── UserProfile.tsx
│   ├── store/
│   │   ├── index.ts
│   │   └── userStore.ts
│   ├── utils/
│   │   └── request.ts
│   ├── types/
│   │   └── index.ts
│   ├── App.tsx
│   ├── main.tsx
│   └── router.tsx
├── public/
├── package.json
├── tsconfig.json
└── vite.config.ts
```

---

## 开发顺序建议

| 顺序 | 工作单元 | 说明 |
|------|----------|------|
| 1 | product-review-backend | 先完成后端API，提供接口 |
| 2 | product-review-frontend | 后端完成后开发前端，调用API |

**并行开发策略**:
- 后端先完成 API 文档 (Swagger)
- 前端可基于 API 文档并行开发
- 使用 Mock 数据进行前端开发
