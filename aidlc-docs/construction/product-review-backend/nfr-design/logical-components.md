# 逻辑组件设计 - 后端服务

## 概述

本文档定义后端服务的逻辑组件和基础设施元素。

---

## 1. 安全组件

### 1.1 JwtUtil

**职责**: JWT Token 生成和验证

```java
@Component
public class JwtUtil {
    private String secret = "your-secret-key";
    private long expiration = 86400000; // 24小时
    
    public String generateToken(User user);
    public Long getUserIdFromToken(String token);
    public String getRoleFromToken(String token);
    public boolean validateToken(String token);
}
```

### 1.2 JwtAuthenticationFilter

**职责**: 请求认证过滤

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    // 从请求头获取Token
    // 验证Token有效性
    // 设置SecurityContext
}
```

### 1.3 SecurityConfig

**职责**: 安全配置

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    // 配置认证过滤器
    // 配置权限规则
    // 配置CORS
}
```

---

## 2. 数据访问组件

### 2.1 MyBatis-Plus 配置

```java
@Configuration
@MapperScan("com.example.review.*.mapper")
public class MyBatisPlusConfig {
    
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 分页插件
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        // 乐观锁插件
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
}
```

### 2.2 数据源配置

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/product_review?useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
```

---

## 3. 公共组件

### 3.1 ApiResponse

**职责**: 统一响应封装

```java
@Data
@AllArgsConstructor
public class ApiResponse<T> {
    private int code;
    private String message;
    private T data;
}
```

### 3.2 GlobalExceptionHandler

**职责**: 全局异常处理

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    // 处理业务异常
    // 处理参数校验异常
    // 处理系统异常
}
```

### 3.3 BusinessException

**职责**: 业务异常定义

```java
@Getter
public class BusinessException extends RuntimeException {
    private int code;
    private String message;
    
    public BusinessException(int code, String message) {
        super(message);
        this.code = code;
        this.message = message;
    }
}
```

---

## 4. 基础实体组件

### 4.1 BaseEntity

**职责**: 基础实体类

```java
@Data
public abstract class BaseEntity {
    @TableId(type = IdType.AUTO)
    private Long id;
    
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdAt;
    
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updatedAt;
}
```

### 4.2 MetaObjectHandler

**职责**: 自动填充处理

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createdAt", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "updatedAt", LocalDateTime.class, LocalDateTime.now());
    }
    
    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updatedAt", LocalDateTime.class, LocalDateTime.now());
    }
}
```

---

## 5. 跨域配置组件

### 5.1 CorsConfig

**职责**: CORS跨域配置

```java
@Configuration
public class CorsConfig {
    
    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.addAllowedOrigin("http://localhost:3000");
        config.addAllowedMethod("*");
        config.addAllowedHeader("*");
        config.setAllowCredentials(true);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }
}
```

---

## 6. API文档组件

### 6.1 OpenAPI配置

```java
@Configuration
public class OpenApiConfig {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("商品审核系统 API")
                .version("1.0")
                .description("商品审核模块 RESTful API 文档"));
    }
}
```

---

## 组件依赖图

```
┌─────────────────────────────────────────────────────────────┐
│                    Spring Boot Application                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │  SecurityConfig │  │   CorsConfig    │                   │
│  └────────┬────────┘  └─────────────────┘                   │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │ JwtAuthFilter   │  │    JwtUtil      │                   │
│  └────────┬────────┘  └─────────────────┘                   │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────────────────────────┐                │
│  │            Controllers                   │               │
│  │  (Product, Audit, Auth)                  │               │
│  └────────────────────┬────────────────────┘                │
│                       │                                     │
│                       ▼                                     │
│  ┌─────────────────────────────────────────┐                │
│  │             Services                     │               │
│  │  (Product, Audit, User)                  │               │
│  └────────────────────┬────────────────────┘                │
│                       │                                     │
│                       ▼                                     │
│  ┌─────────────────────────────────────────┐                │
│  │    MyBatis-Plus Mappers                  │               │
│  │  (Product, Audit, User)                  │               │
│  └────────────────────┬────────────────────┘                │
│                       │                                     │
│                       ▼                                     │
│  ┌─────────────────────────────────────────┐                │
│  │           MySQL Database                 │               │
│  └─────────────────────────────────────────┘                │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │GlobalException  │  │  ApiResponse    │                   │
│  │   Handler       │  │                 │                   │
│  └─────────────────┘  └─────────────────┘                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```
