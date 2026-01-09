# 组件方法定义 - 商品审核模块

## Controller 层方法

### ProductController

| 方法 | HTTP | 路径 | 描述 |
|------|------|------|------|
| `createProduct` | POST | `/api/products` | 创建商品 |
| `getProduct` | GET | `/api/products/{id}` | 获取商品详情 |
| `getMyProducts` | GET | `/api/products/my` | 获取我的商品列表 |
| `updateProduct` | PUT | `/api/products/{id}` | 更新商品信息 |

### AuditController

| 方法 | HTTP | 路径 | 描述 |
|------|------|------|------|
| `getPendingList` | GET | `/api/audits/pending` | 获取待审核列表 |
| `getReviewList` | GET | `/api/audits/review` | 获取待复审列表 |
| `submitAudit` | POST | `/api/audits/{productId}` | 提交审核结果 |
| `getAuditHistory` | GET | `/api/audits/history/{productId}` | 获取审核历史 |
| `getAuditStats` | GET | `/api/audits/stats` | 获取审核统计 |

### AuthController

| 方法 | HTTP | 路径 | 描述 |
|------|------|------|------|
| `login` | POST | `/api/auth/login` | 用户登录 |
| `logout` | POST | `/api/auth/logout` | 用户登出 |
| `getCurrentUser` | GET | `/api/auth/me` | 获取当前用户信息 |

---

## Service 层方法

### ProductService

```java
// 创建商品
Product createProduct(ProductCreateDTO dto, Long merchantId);

// 获取商品详情
Product getProductById(Long id);

// 获取商家的商品列表
Page<Product> getProductsByMerchant(Long merchantId, Pageable pageable);

// 更新商品（仅限"需修改"状态）
Product updateProduct(Long id, ProductUpdateDTO dto);

// 重新提交商品审核
void resubmitProduct(Long id);
```

### AuditService

```java
// 获取待初审商品列表
Page<Product> getPendingProducts(Pageable pageable);

// 获取待复审商品列表
Page<Product> getReviewProducts(Pageable pageable);

// 执行审核操作
void submitAudit(Long productId, AuditDTO dto, Long auditorId);

// 获取商品审核历史
List<AuditRecord> getAuditHistory(Long productId);

// 获取审核统计数据
AuditStatsDTO getAuditStats(LocalDate startDate, LocalDate endDate);
```

### UserService

```java
// 用户登录
LoginResponseDTO login(LoginRequestDTO dto);

// 用户登出
void logout(String token);

// 获取当前用户信息
UserDTO getCurrentUser(Long userId);

// 验证用户权限
boolean hasPermission(Long userId, String permission);
```

---

## Repository 层方法 (MyBatis-Plus Mapper)

### ProductMapper

```java
// 继承 BaseMapper<Product>，自动获得CRUD方法

// 自定义查询：按商家ID分页查询
IPage<Product> selectByMerchantId(IPage<Product> page, @Param("merchantId") Long merchantId);

// 自定义查询：按状态查询
IPage<Product> selectByStatus(IPage<Product> page, @Param("status") ProductStatus status);
```

### AuditMapper

```java
// 继承 BaseMapper<AuditRecord>

// 自定义查询：按商品ID查询审核历史
List<AuditRecord> selectByProductId(@Param("productId") Long productId);

// 自定义查询：统计审核数据
AuditStatsDTO selectStats(@Param("startDate") LocalDate startDate, @Param("endDate") LocalDate endDate);
```

### UserMapper

```java
// 继承 BaseMapper<User>

// 自定义查询：按用户名查询
User selectByUsername(@Param("username") String username);
```

---

**注**: 详细的业务规则和验证逻辑将在 CONSTRUCTION 阶段的功能设计中定义。
