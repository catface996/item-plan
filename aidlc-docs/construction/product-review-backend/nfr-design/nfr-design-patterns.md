# NFR设计模式 - 后端服务

## 概述

本文档定义后端服务的非功能性设计模式，确保系统满足性能、安全、可用性等需求。

---

## 1. 安全设计模式

### 1.1 JWT认证模式

```
┌─────────┐     ┌─────────────┐     ┌─────────────┐
│  Client │────▶│ AuthFilter  │────▶│ Controller  │
└─────────┘     └──────┬──────┘     └─────────────┘
                       │
                       ▼
                ┌─────────────┐
                │  JwtUtil    │
                │ (验证Token) │
                └─────────────┘
```

**实现要点**:
- 使用 Spring Security Filter Chain
- JWT Token 存储用户ID和角色
- Token 有效期 24 小时
- 无状态认证，不使用 Session

### 1.2 RBAC权限模式

```java
// 注解式权限控制
@PreAuthorize("hasRole('MERCHANT')")
public void createProduct() { ... }

@PreAuthorize("hasRole('REVIEWER')")
public void submitAudit() { ... }

@PreAuthorize("hasRole('SENIOR_REVIEWER')")
public void submitReview() { ... }
```

**角色权限映射**:
| 角色 | 权限 |
|------|------|
| MERCHANT | product:create, product:read:own, product:update:own |
| REVIEWER | product:read:pending, audit:create:first |
| SENIOR_REVIEWER | product:read:review, audit:create:final, audit:stats |

---

## 2. 性能设计模式

### 2.1 数据库连接池模式

```yaml
# HikariCP 配置
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
```

### 2.2 分页查询模式

```java
// MyBatis-Plus 分页
public IPage<Product> getProducts(int page, int size) {
    Page<Product> pageParam = new Page<>(page, size);
    return productMapper.selectPage(pageParam, queryWrapper);
}
```

**分页规则**:
- 默认每页 10 条
- 最大每页 100 条
- 返回总数和总页数

### 2.3 索引优化模式

```sql
-- 商品表索引
CREATE INDEX idx_product_merchant ON product(merchant_id);
CREATE INDEX idx_product_status ON product(status);
CREATE INDEX idx_product_created ON product(created_at);

-- 审核记录表索引
CREATE INDEX idx_audit_product ON audit_record(product_id);
CREATE INDEX idx_audit_auditor ON audit_record(auditor_id);
CREATE INDEX idx_audit_created ON audit_record(created_at);
```

---

## 3. 可靠性设计模式

### 3.1 统一异常处理模式

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(BusinessException.class)
    public ApiResponse<?> handleBusiness(BusinessException e) {
        return ApiResponse.error(e.getCode(), e.getMessage());
    }
    
    @ExceptionHandler(Exception.class)
    public ApiResponse<?> handleException(Exception e) {
        log.error("系统异常", e);
        return ApiResponse.error(500, "系统内部错误");
    }
}
```

### 3.2 事务管理模式

```java
@Transactional(rollbackFor = Exception.class)
public void submitAudit(Long productId, AuditDTO dto) {
    // 1. 更新商品状态
    productMapper.updateStatus(productId, newStatus);
    // 2. 创建审核记录
    auditMapper.insert(auditRecord);
    // 事务保证原子性
}
```

### 3.3 乐观锁模式

```java
@Data
public class Product {
    @Version
    private Integer version;
}

// MyBatis-Plus 自动处理乐观锁
// UPDATE product SET status=?, version=version+1 
// WHERE id=? AND version=?
```

---

## 4. 可观测性设计模式

### 4.1 日志记录模式

```java
// 使用 SLF4J + Logback
@Slf4j
public class ProductService {
    
    public Product createProduct(ProductCreateDTO dto) {
        log.info("创建商品: merchantId={}, name={}", 
                 dto.getMerchantId(), dto.getName());
        // ...
        log.info("商品创建成功: productId={}", product.getId());
        return product;
    }
}
```

**日志级别规范**:
| 级别 | 使用场景 |
|------|----------|
| ERROR | 系统错误、异常 |
| WARN | 警告、潜在问题 |
| INFO | 关键业务操作 |
| DEBUG | 调试信息 |

### 4.2 操作审计模式

```java
@Aspect
@Component
public class AuditLogAspect {
    
    @Around("@annotation(auditLog)")
    public Object logOperation(ProceedingJoinPoint pjp, AuditLog auditLog) {
        // 记录操作前
        String operation = auditLog.value();
        Long userId = getCurrentUserId();
        
        Object result = pjp.proceed();
        
        // 记录操作后
        saveAuditLog(userId, operation, result);
        return result;
    }
}
```

---

## 5. API设计模式

### 5.1 统一响应格式

```java
@Data
public class ApiResponse<T> {
    private int code;
    private String message;
    private T data;
    
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(200, "success", data);
    }
    
    public static <T> ApiResponse<T> error(int code, String message) {
        return new ApiResponse<>(code, message, null);
    }
}
```

### 5.2 参数校验模式

```java
@Data
public class ProductCreateDTO {
    @NotBlank(message = "商品名称不能为空")
    @Size(min = 1, max = 100, message = "商品名称长度1-100")
    private String name;
    
    @NotBlank(message = "商品描述不能为空")
    @Size(min = 1, max = 1000, message = "商品描述长度1-1000")
    private String description;
    
    @NotNull(message = "商品价格不能为空")
    @DecimalMin(value = "0.01", message = "价格最小0.01")
    @DecimalMax(value = "999999.99", message = "价格最大999999.99")
    private BigDecimal price;
}
```

---

## 设计模式追溯

| 模式 | NFR需求 | 实现方式 |
|------|---------|----------|
| JWT认证 | NFR-004 | Spring Security + jjwt |
| RBAC权限 | NFR-005 | @PreAuthorize注解 |
| 连接池 | NFR-003 | HikariCP |
| 分页查询 | NFR-002 | MyBatis-Plus Page |
| 统一异常 | NFR-007 | @RestControllerAdvice |
| 事务管理 | NFR-007 | @Transactional |
| 乐观锁 | CC-002 | @Version |
| 日志记录 | NFR-006 | SLF4J + Logback |
| 操作审计 | NFR-006 | AOP切面 |
