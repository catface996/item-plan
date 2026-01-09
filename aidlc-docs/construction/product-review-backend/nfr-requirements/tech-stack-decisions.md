# 技术栈决策 - 商品审核模块

## 技术栈概览

| 层级 | 技术选型 | 版本 | 选型理由 |
|------|----------|------|----------|
| 后端框架 | Spring Boot | 3.x | 企业级标准，生态完善 |
| 数据访问 | MyBatis-Plus | 3.5.x | 简化开发，灵活SQL |
| 数据库 | MySQL | 8.0 | 稳定可靠，团队熟悉 |
| 前端框架 | React | 18.x | 组件化开发，生态丰富 |
| 构建工具 | Vite | 5.x | 快速构建，开发体验好 |
| 部署环境 | 本地服务器 | - | 企业内部系统 |

---

## 后端技术栈详情

### 核心框架

| 组件 | 技术 | 用途 |
|------|------|------|
| Web框架 | Spring Boot 3.x | RESTful API |
| 安全框架 | Spring Security | 认证授权 |
| ORM框架 | MyBatis-Plus | 数据访问 |
| 连接池 | HikariCP | 数据库连接管理 |

### 工具库

| 组件 | 技术 | 用途 |
|------|------|------|
| JSON处理 | Jackson | 序列化/反序列化 |
| 参数校验 | Hibernate Validator | 数据验证 |
| 日志框架 | SLF4J + Logback | 日志记录 |
| API文档 | SpringDoc OpenAPI | Swagger文档 |
| JWT | jjwt | Token生成验证 |
| 工具类 | Hutool | 通用工具 |

### 测试框架

| 组件 | 技术 | 用途 |
|------|------|------|
| 单元测试 | JUnit 5 | 单元测试 |
| Mock框架 | Mockito | 模拟对象 |
| 集成测试 | Spring Boot Test | 集成测试 |

---

## 前端技术栈详情

### 核心框架

| 组件 | 技术 | 用途 |
|------|------|------|
| UI框架 | React 18 | 组件化开发 |
| 状态管理 | Zustand | 轻量状态管理 |
| 路由 | React Router 6 | 页面路由 |
| HTTP客户端 | Axios | API请求 |

### UI组件库

| 组件 | 技术 | 用途 |
|------|------|------|
| 组件库 | Ant Design 5 | UI组件 |
| 图标 | @ant-design/icons | 图标库 |
| 表单 | Ant Design Form | 表单处理 |
| 表格 | Ant Design Table | 数据表格 |

### 开发工具

| 组件 | 技术 | 用途 |
|------|------|------|
| 构建工具 | Vite | 项目构建 |
| 类型检查 | TypeScript | 类型安全 |
| 代码规范 | ESLint + Prettier | 代码质量 |

---

## 数据库设计

### MySQL配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 字符集 | utf8mb4 | 支持emoji |
| 排序规则 | utf8mb4_unicode_ci | 通用排序 |
| 存储引擎 | InnoDB | 事务支持 |

### 连接池配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 最大连接数 | 20 | maximum-pool-size |
| 最小空闲 | 5 | minimum-idle |
| 连接超时 | 30秒 | connection-timeout |
| 空闲超时 | 10分钟 | idle-timeout |

---

## 安全实现方案

### JWT认证

```
Token结构:
- Header: 算法类型 (HS256)
- Payload: 用户ID, 角色, 过期时间
- Signature: 签名验证

Token有效期: 24小时
刷新机制: 暂不实现
```

### 密码加密

```
算法: BCrypt
强度: 10 (默认)
```

### CORS配置

```
允许来源: 前端域名
允许方法: GET, POST, PUT, DELETE
允许头: Authorization, Content-Type
```

---

## 项目结构

### 后端项目结构

```
product-review-backend/
├── src/main/java/com/example/review/
│   ├── common/           # 公共模块
│   │   ├── config/       # 配置类
│   │   ├── exception/    # 异常处理
│   │   ├── response/     # 响应封装
│   │   └── util/         # 工具类
│   ├── product/          # 商品模块
│   │   ├── controller/
│   │   ├── service/
│   │   ├── mapper/
│   │   └── entity/
│   ├── audit/            # 审核模块
│   │   ├── controller/
│   │   ├── service/
│   │   ├── mapper/
│   │   └── entity/
│   └── user/             # 用户模块
│       ├── controller/
│       ├── service/
│       ├── mapper/
│       └── entity/
├── src/main/resources/
│   ├── application.yml
│   └── mapper/           # MyBatis XML
└── pom.xml
```

### 前端项目结构

```
product-review-frontend/
├── src/
│   ├── api/              # API接口
│   ├── components/       # 公共组件
│   ├── pages/            # 页面组件
│   │   ├── product/      # 商品模块
│   │   ├── audit/        # 审核模块
│   │   └── user/         # 用户模块
│   ├── store/            # 状态管理
│   ├── utils/            # 工具函数
│   ├── App.tsx
│   └── main.tsx
├── package.json
├── tsconfig.json
└── vite.config.ts
```

---

## 技术决策记录

| 决策ID | 决策内容 | 理由 | 日期 |
|--------|----------|------|------|
| TD-001 | 使用MyBatis-Plus而非JPA | 灵活SQL控制，团队熟悉 | 2026-01-08 |
| TD-002 | 使用Zustand而非Redux | 轻量简洁，学习成本低 | 2026-01-08 |
| TD-003 | 使用Vite而非CRA | 构建速度快，开发体验好 | 2026-01-08 |
| TD-004 | 使用Ant Design | 企业级组件库，功能完善 | 2026-01-08 |
| TD-005 | JWT认证而非Session | 前后端分离，无状态 | 2026-01-08 |
