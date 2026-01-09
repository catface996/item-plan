# 组件依赖关系 - 商品审核模块

## 后端组件依赖图

```
┌─────────────────────────────────────────────────────────────┐
│                      Controller 层                          │
├─────────────────┬─────────────────┬─────────────────────────┤
│ ProductController│ AuditController │    AuthController       │
└────────┬────────┴────────┬────────┴───────────┬─────────────┘
         │                 │                    │
         ▼                 ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│                       Service 层                            │
├─────────────────┬─────────────────┬─────────────────────────┤
│  ProductService │  AuditService   │     UserService         │
└────────┬────────┴────────┬────────┴───────────┬─────────────┘
         │                 │                    │
         │    ┌────────────┼────────────┐       │
         │    │            │            │       │
         ▼    ▼            ▼            ▼       ▼
┌─────────────────────────────────────────────────────────────┐
│                      Repository 层                          │
├─────────────────┬─────────────────┬─────────────────────────┤
│  ProductMapper  │   AuditMapper   │      UserMapper         │
└────────┬────────┴────────┬────────┴───────────┬─────────────┘
         │                 │                    │
         └─────────────────┼────────────────────┘
                           ▼
                    ┌─────────────┐
                    │    MySQL    │
                    └─────────────┘
```

---

## 依赖矩阵

| 组件 | 依赖 | 依赖类型 |
|------|------|----------|
| ProductController | ProductService | 直接依赖 |
| AuditController | AuditService | 直接依赖 |
| AuthController | UserService | 直接依赖 |
| ProductService | ProductMapper | 直接依赖 |
| AuditService | AuditMapper, ProductMapper | 直接依赖 |
| UserService | UserMapper | 直接依赖 |

---

## 前端组件依赖

```
┌─────────────────────────────────────────────────────────────┐
│                         App                                 │
└─────────────────────────────┬───────────────────────────────┘
                              │
         ┌────────────────────┼────────────────────┐
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  商品模块        │  │   审核模块       │  │   用户模块       │
│  (product/)     │  │   (audit/)      │  │   (user/)       │
├─────────────────┤  ├─────────────────┤  ├─────────────────┤
│ ProductList     │  │ PendingList     │  │ Login           │
│ ProductForm     │  │ AuditPanel      │  │ UserProfile     │
│ ProductDetail   │  │ AuditHistory    │  └─────────────────┘
└────────┬────────┘  │ AuditStats      │
         │           └────────┬────────┘
         │                    │
         └────────────────────┼────────────────────┐
                              │                    │
                              ▼                    ▼
                    ┌─────────────────┐  ┌─────────────────┐
                    │   公共组件       │  │    API服务       │
                    │   (common/)     │  │   (services/)   │
                    ├─────────────────┤  ├─────────────────┤
                    │ Layout          │  │ productApi      │
                    │ Navbar          │  │ auditApi        │
                    │ Table           │  │ authApi         │
                    │ Modal           │  └─────────────────┘
                    └─────────────────┘
```

---

## 通信模式

### 前后端通信

| 模式 | 描述 |
|------|------|
| 协议 | HTTP/HTTPS |
| 格式 | JSON |
| 认证 | JWT Bearer Token |
| 错误处理 | 统一错误响应格式 |

### API响应格式

```json
{
  "code": 200,
  "message": "success",
  "data": { ... }
}
```

### 错误响应格式

```json
{
  "code": 400,
  "message": "错误描述",
  "data": null
}
```

---

## 数据流向

### 商品提交流程

```
React Form → productApi.create() → POST /api/products
                                          │
                                          ▼
                                   ProductController
                                          │
                                          ▼
                                   ProductService
                                          │
                                          ▼
                                   ProductMapper
                                          │
                                          ▼
                                       MySQL
```

### 审核操作流程

```
React AuditPanel → auditApi.submit() → POST /api/audits/{id}
                                              │
                                              ▼
                                       AuditController
                                              │
                                              ▼
                                       AuditService
                                              │
                              ┌───────────────┼───────────────┐
                              ▼               ▼               ▼
                        AuditMapper    ProductMapper    (状态更新)
                              │               │
                              └───────┬───────┘
                                      ▼
                                   MySQL
```
