# 领域实体定义 - 商品审核模块

## 实体概览

| 实体 | 描述 | 主要属性 |
|------|------|----------|
| Product | 商品 | 名称、描述、价格、状态 |
| User | 用户 | 用户名、密码、角色 |
| AuditRecord | 审核记录 | 商品ID、审核人、结果、意见 |

---

## 1. Product (商品实体)

### 属性定义

| 属性 | 类型 | 约束 | 描述 |
|------|------|------|------|
| id | Long | PK, 自增 | 商品ID |
| name | String | 非空, 长度1-100 | 商品名称 |
| description | String | 非空, 长度1-1000 | 商品描述 |
| price | BigDecimal | 非空, 0.01-999999.99 | 商品价格 |
| status | ProductStatus | 非空 | 审核状态 |
| merchantId | Long | FK, 非空 | 商家ID |
| createdAt | LocalDateTime | 非空 | 创建时间 |
| updatedAt | LocalDateTime | 非空 | 更新时间 |

### 状态枚举 (ProductStatus)

| 值 | 描述 | 允许的下一状态 |
|------|------|----------------|
| PENDING | 待审核 | FIRST_PASS, REJECTED, NEED_MODIFY |
| FIRST_PASS | 初审通过 | APPROVED, REJECTED, NEED_MODIFY |
| APPROVED | 通过 | - (终态) |
| REJECTED | 拒绝 | - (终态) |
| NEED_MODIFY | 需修改 | PENDING |

---

## 2. User (用户实体)

### 属性定义

| 属性 | 类型 | 约束 | 描述 |
|------|------|------|------|
| id | Long | PK, 自增 | 用户ID |
| username | String | 非空, 唯一, 长度3-50 | 用户名 |
| password | String | 非空 | 密码(加密存储) |
| role | UserRole | 非空 | 用户角色 |
| createdAt | LocalDateTime | 非空 | 创建时间 |

### 角色枚举 (UserRole)

| 值 | 描述 | 权限 |
|------|------|------|
| MERCHANT | 商家 | 提交商品、查看自己的商品、修改被退回商品 |
| REVIEWER | 普通审核员 | 查看待审核商品、执行初审 |
| SENIOR_REVIEWER | 高级审核员 | 查看待复审商品、执行复审、查看统计 |

---

## 3. AuditRecord (审核记录实体)

### 属性定义

| 属性 | 类型 | 约束 | 描述 |
|------|------|------|------|
| id | Long | PK, 自增 | 记录ID |
| productId | Long | FK, 非空 | 商品ID |
| auditorId | Long | FK, 非空 | 审核人ID |
| result | AuditResult | 非空 | 审核结果 |
| comment | String | 条件必填, 长度0-500 | 审核意见 |
| createdAt | LocalDateTime | 非空 | 审核时间 |

### 审核结果枚举 (AuditResult)

| 值 | 描述 | 意见要求 |
|------|------|----------|
| PASS | 通过 | 可选 |
| REJECT | 拒绝 | 必填 |
| NEED_MODIFY | 需修改 | 必填 |

---

## 实体关系图

```
┌─────────────┐       ┌─────────────┐
│    User     │       │   Product   │
├─────────────┤       ├─────────────┤
│ id (PK)     │       │ id (PK)     │
│ username    │◄──────│ merchantId  │
│ password    │  1:N  │ name        │
│ role        │       │ description │
│ createdAt   │       │ price       │
└─────────────┘       │ status      │
      │               │ createdAt   │
      │               │ updatedAt   │
      │               └──────┬──────┘
      │                      │
      │ 1:N                  │ 1:N
      │                      │
      │               ┌──────▼──────┐
      │               │ AuditRecord │
      │               ├─────────────┤
      └──────────────►│ id (PK)     │
                      │ productId   │
                      │ auditorId   │
                      │ result      │
                      │ comment     │
                      │ createdAt   │
                      └─────────────┘
```

---

## 数据库表设计

### product 表

```sql
CREATE TABLE product (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description VARCHAR(1000) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING',
    merchant_id BIGINT NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (merchant_id) REFERENCES user(id)
);
```

### user 表

```sql
CREATE TABLE user (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### audit_record 表

```sql
CREATE TABLE audit_record (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    product_id BIGINT NOT NULL,
    auditor_id BIGINT NOT NULL,
    result VARCHAR(20) NOT NULL,
    comment VARCHAR(500),
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES product(id),
    FOREIGN KEY (auditor_id) REFERENCES user(id)
);
```
