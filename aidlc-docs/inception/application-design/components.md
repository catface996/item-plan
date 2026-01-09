# 组件定义 - 商品审核模块

## 架构概述

采用前后端分离架构：
- **后端**: Spring Boot + MyBatis-Plus，分层架构 (Controller → Service → Repository)
- **前端**: React，按功能模块划分
- **通信**: RESTful API

---

## 后端组件

### 1. 商品模块 (Product Module)

| 属性 | 描述 |
|------|------|
| 包名 | `com.example.review.product` |
| 职责 | 商品信息的创建、查询、修改 |
| 主要实体 | Product |

**组件结构**:
- `ProductController` - 商品相关API端点
- `ProductService` - 商品业务逻辑
- `ProductMapper` - 商品数据访问

### 2. 审核模块 (Audit Module)

| 属性 | 描述 |
|------|------|
| 包名 | `com.example.review.audit` |
| 职责 | 审核流程管理、审核记录 |
| 主要实体 | AuditRecord |

**组件结构**:
- `AuditController` - 审核相关API端点
- `AuditService` - 审核业务逻辑
- `AuditMapper` - 审核数据访问

### 3. 用户模块 (User Module)

| 属性 | 描述 |
|------|------|
| 包名 | `com.example.review.user` |
| 职责 | 用户认证、授权、角色管理 |
| 主要实体 | User, Role |

**组件结构**:
- `AuthController` - 认证相关API端点
- `UserService` - 用户业务逻辑
- `UserMapper` - 用户数据访问

### 4. 公共模块 (Common Module)

| 属性 | 描述 |
|------|------|
| 包名 | `com.example.review.common` |
| 职责 | 通用工具、异常处理、响应封装 |

**组件结构**:
- `GlobalExceptionHandler` - 全局异常处理
- `ApiResponse` - 统一响应格式
- `BaseEntity` - 基础实体类

---

## 前端组件

### 1. 商品模块 (product/)

| 组件 | 职责 |
|------|------|
| `ProductList` | 商品列表展示 |
| `ProductForm` | 商品提交/编辑表单 |
| `ProductDetail` | 商品详情展示 |

### 2. 审核模块 (audit/)

| 组件 | 职责 |
|------|------|
| `PendingList` | 待审核商品列表 |
| `AuditPanel` | 审核操作面板 |
| `AuditHistory` | 审核历史记录 |
| `AuditStats` | 审核统计报表 |

### 3. 用户模块 (user/)

| 组件 | 职责 |
|------|------|
| `Login` | 登录页面 |
| `UserProfile` | 用户信息 |

### 4. 公共组件 (common/)

| 组件 | 职责 |
|------|------|
| `Layout` | 页面布局 |
| `Navbar` | 导航栏 |
| `Table` | 通用表格 |
| `Modal` | 通用弹窗 |

---

## 组件接口

### 后端API接口规范

| 前缀 | 模块 | 说明 |
|------|------|------|
| `/api/auth` | 用户模块 | 认证相关 |
| `/api/products` | 商品模块 | 商品CRUD |
| `/api/audits` | 审核模块 | 审核操作 |
