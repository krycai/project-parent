# Spring Boot 4.0 升级迁移指南

## 📋 版本升级概览

### 核心版本变更
| 组件 | 旧版本 | 新版本 | 说明 |
|------|--------|--------|------|
| **Spring Boot** | 2.2.10.RELEASE | **4.0.1** | 跨越 2 个大版本 |
| **Spring Framework** | 5.2.x | **7.0.x** | 全新一代框架 |
| **Spring Cloud** | Hoxton.SR12 | **2025.1.0 (Oakwood)** | 最新发布列车 |
| **Spring Cloud Alibaba** | 2.2.1.RELEASE | **2025.0.0.0** | 支持 Spring Boot 3.5 |
| **Java** | 8 | **17 (最低) / 21/25 (推荐)** | 必须升级 |

### ⚠️ 重要提醒
1. **Spring Cloud Alibaba 兼容性警告**：
   - 截至 2025 年 12 月，Spring Cloud Alibaba 最新版本为 **2025.0.0.0**
   - 该版本官方支持 **Spring Boot 3.5.x** 和 **Spring Cloud 2025.0.x**
   - 对 Spring Boot 4.0 的支持预计在 **2025.1.x** 版本中提供
   - **建议**：先升级到 Spring Boot 3.5，待 Alibaba 官方支持后再升级到 4.0

2. **推荐升级路径**：
   ```
   Spring Boot 2.2 → 2.7 → 3.0 → 3.5 → 4.0
   (不建议直接从 2.2 跳到 4.0)
   ```

## 🎯 Spring Boot 4.0 核心新特性

### 1. 完全模块化架构
Spring Boot 4.0 对代码库进行了完全模块化：
- 更小、更专注的 JAR 包
- 减少不必要的依赖
- 支持更细粒度的依赖管理

### 2. JSpecify 空安全改进
全栈支持 JSpecify 空安全注解：
```java
import org.jspecify.annotations.Nullable;
import org.jspecify.annotations.NonNull;

public class UserService {
    public @Nullable User findUser(@NonNull String id) {
        // 编译时空安全检查
    }
}
```

### 3. Java 25 完整支持
- 保持 Java 17 向后兼容
- 支持 Java 21 LTS 新特性
- 完整支持 Java 25 最新特性

### 4. REST API 版本控制
Spring Framework 7 原生支持 API 版本控制：
```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    // API v1
    @RequestMapping(value = "/search", version = "1")
    public List<ProductV1> searchV1(@RequestParam String query) {
        return productService.searchV1(query);
    }
    
    // API v2 - 新版本
    @RequestMapping(value = "/search", version = "2")
    public ProductResponseV2 searchV2(
            @RequestParam String query,
            @RequestParam(defaultValue = "10") int limit) {
        return productService.searchV2(query, limit);
    }
}
```

调用示例：
```bash
# 调用 v1
curl -H "Api-Version: 1" http://localhost:8080/api/products/search?query=laptop

# 调用 v2
curl -H "Api-Version: 2" http://localhost:8080/api/products/search?query=laptop&limit=20
```

### 5. HTTP Service Clients
新的声明式 HTTP 客户端（类似 Feign）：
```java
@HttpExchange("/api/users")
public interface UserClient {
    
    @GetExchange("/{id}")
    User getUser(@PathVariable Long id);
    
    @PostExchange
    User createUser(@RequestBody User user);
    
    @DeleteExchange("/{id}")
    void deleteUser(@PathVariable Long id);
}

// 配置
@Configuration
public class HttpClientConfig {
    
    @Bean
    public UserClient userClient(RestClient.Builder builder) {
        RestClient restClient = builder
            .baseUrl("https://api.example.com")
            .build();
        
        HttpServiceProxyFactory factory = HttpServiceProxyFactory
            .builderFor(RestClientAdapter.create(restClient))
            .build();
            
        return factory.createClient(UserClient.class);
    }
}
```

### 6. Jackson 3.0 升级
从 Jackson 2.x 升级到 Jackson 3.0：
- 包名变更：`com.fasterxml.jackson` 保持不变
- 移除了一些已废弃的 API
- 性能提升约 10-15%

## 🔧 重大变更详解

### 1. Java 17+ 必需
```bash
# 检查当前 Java 版本
java -version

# 推荐使用 Java 21 LTS
# Ubuntu/Debian
sudo apt install openjdk-21-jdk

# macOS
brew install openjdk@21

# 配置 JAVA_HOME
export JAVA_HOME=/path/to/java-21
export PATH=$JAVA_HOME/bin:$PATH
```

### 2. Jakarta EE 命名空间 (javax → jakarta)

**完整替换列表**：
```java
// Servlet API
javax.servlet.* → jakarta.servlet.*
javax.servlet.http.* → jakarta.servlet.http.*

// Persistence API
javax.persistence.* → jakarta.persistence.*

// Validation API
javax.validation.* → jakarta.validation.*

// Annotation API
javax.annotation.* → jakarta.annotation.*

// Transaction API
javax.transaction.* → jakarta.transaction.*

// Interceptor API
javax.interceptor.* → jakarta.interceptor.*

// WebSocket API
javax.websocket.* → jakarta.websocket.*

// JAX-RS
javax.ws.rs.* → jakarta.ws.rs.*

// JMS
javax.jms.* → jakarta.jms.*

// Mail
javax.mail.* → jakarta.mail.*
```

**自动化替换脚本**：
```bash
#!/bin/bash
# replace-javax.sh

# 递归替换所有 Java 文件中的 javax 包名
find . -name "*.java" -type f -exec sed -i 's/import javax\./import jakarta\./g' {} +
find . -name "*.java" -type f -exec sed -i 's/javax\./jakarta\./g' {} +

echo "Replaced javax.* with jakarta.*"
```

### 3. MySQL 驱动升级

#### 依赖变更
```xml
<!-- 旧版本 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.46</version>
</dependency>

<!-- 新版本 -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.1.0</version>
</dependency>
```

#### 配置变更
```yaml
# application.yml

# 旧配置
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mydb?useUnicode=true&characterEncoding=utf8

# 新配置
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mydb?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai&useSSL=false&allowPublicKeyRetrieval=true
```

#### 连接字符串参数说明
- `serverTimezone=Asia/Shanghai`: 必需，指定时区
- `useSSL=false`: 开发环境可关闭 SSL（生产环境建议开启）
- `allowPublicKeyRetrieval=true`: 允许客户端从服务器获取公钥

### 4. MyBatis Plus 升级

```xml
<!-- 旧版本 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.2.0</version>
</dependency>

<!-- 新版本 - 注意 artifactId 变化 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.9</version>
</dependency>
```

**配置示例**：
```yaml
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      id-type: auto
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0
  type-aliases-package: com.gc.entity
```

### 5. JWT 依赖重构

```xml
<!-- 旧版本 - 单一依赖 -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>

<!-- 新版本 - 需要三个依赖 -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.6</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.6</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.6</version>
    <scope>runtime</scope>
</dependency>
```

**代码迁移示例**：
```java
// 旧代码
String jwt = Jwts.builder()
    .setSubject(username)
    .setExpiration(expirationDate)
    .signWith(SignatureAlgorithm.HS512, secret)
    .compact();

// 新代码
import io.jsonwebtoken.security.Keys;
import javax.crypto.SecretKey;

SecretKey key = Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));

String jwt = Jwts.builder()
    .subject(username)
    .expiration(expirationDate)
    .signWith(key)  // 不再需要指定算法
    .compact();

// 解析也需要更新
Claims claims = Jwts.parser()
    .verifyWith(key)  // 替代 setSigningKey
    .build()
    .parseSignedClaims(jwt)
    .getPayload();
```

### 6. Spring Cloud 配置变更

#### Bootstrap 配置
Spring Cloud 2020+ 默认禁用了 bootstrap：

**方式 1: 使用 spring.config.import**
```yaml
# application.yml
spring:
  application:
    name: my-service
  config:
    import: optional:nacos:${spring.application.name}.yml
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848
      discovery:
        namespace: dev
        group: DEFAULT_GROUP
      config:
        namespace: dev
        group: DEFAULT_GROUP
        file-extension: yml
```

**方式 2: 添加 bootstrap 依赖**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

然后继续使用 `bootstrap.yml`：
```yaml
# bootstrap.yml
spring:
  application:
    name: my-service
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848
      config:
        namespace: dev
        group: DEFAULT_GROUP
        file-extension: yml
```

### 7. Nacos 配置注意事项

由于 Spring Cloud Alibaba 2025.0.0.0 目前支持 Spring Boot 3.5，在使用 Spring Boot 4.0 时需要注意：

```xml
<!-- 确保使用兼容版本 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>2025.0.0.0</version>
    <!-- 可能需要排除一些传递依赖 -->
    <exclusions>
        <exclusion>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-commons</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

**监控兼容性问题**：
- 启动时密切关注日志
- 测试服务注册和发现功能
- 验证配置中心功能

## 📝 配置文件迁移

### application.yml 更新示例

```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  application:
    name: my-service
  
  # 数据源配置
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mydb?serverTimezone=Asia/Shanghai&useSSL=false
    username: root
    password: ENC(encrypted_password)  # Jasypt 加密
    hikari:
      minimum-idle: 5
      maximum-pool-size: 20
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
  
  # JPA 配置
  jpa:
    database-platform: org.hibernate.dialect.MySQL8Dialect
    show-sql: false
    hibernate:
      ddl-auto: none
    properties:
      hibernate:
        format_sql: true
  
  # RabbitMQ 配置
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual
        concurrency: 5
        max-concurrency: 10
  
  # Nacos 配置
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848
      discovery:
        namespace: dev
        group: DEFAULT_GROUP
        enabled: true
      config:
        namespace: dev
        group: DEFAULT_GROUP
        file-extension: yml
        refresh-enabled: true

# Actuator 配置
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always
  metrics:
    export:
      prometheus:
        enabled: true

# MyBatis Plus 配置
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
    cache-enabled: false
  global-config:
    db-config:
      id-type: auto
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0

# Jasypt 加密配置
jasypt:
  encryptor:
    algorithm: PBEWithMD5AndDES
    password: ${JASYPT_PASSWORD:your_secret_key}

# Logging 配置
logging:
  level:
    root: INFO
    com.gc: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

## 🚀 升级步骤

### 阶段 1: 准备工作 (1-2 天)
```bash
# 1. 创建备份分支
git checkout -b backup/before-spring-boot-4-upgrade
git push origin backup/before-spring-boot-4-upgrade

# 2. 创建升级分支
git checkout -b feature/upgrade-to-spring-boot-4
```

**环境准备**：
- ✅ 安装 Java 17+ (推荐 Java 21)
- ✅ 更新 IDE 到最新版本
- ✅ 更新 Maven 到 3.9+
- ✅ 准备测试环境

### 阶段 2: 依赖升级 (2-3 天)

1. **更新 pom.xml**
   ```bash
   # 备份原 pom.xml
   cp pom.xml pom.xml.backup
   
   # 替换为新的 pom.xml
   # (使用本指南提供的配置)
   ```

2. **解决依赖冲突**
   ```bash
   # 查看依赖树
   mvn dependency:tree > dependency-tree.txt
   
   # 清理并编译
   mvn clean compile
   ```

3. **处理编译错误**
   - 使用 IDE 的"Replace in Files"功能
   - 搜索 `import javax.` 替换为 `import jakarta.`
   - 检查所有编译错误并逐一修复

### 阶段 3: 代码迁移 (3-5 天)

1. **全局替换 javax → jakarta**
   ```bash
   # 使用提供的脚本
   chmod +x replace-javax.sh
   ./replace-javax.sh
   ```

2. **更新数据库配置**
   - 修改 application.yml 中的数据源配置
   - 更新驱动类名
   - 添加必需的 URL 参数

3. **更新 JWT 代码**
   - 重构 JWT 生成逻辑
   - 更新 JWT 解析逻辑
   - 测试认证功能

4. **检查 MyBatis Plus 映射**
   - 验证实体类注解
   - 测试 CRUD 操作
   - 检查分页功能

### 阶段 4: 配置迁移 (1-2 天)

1. **更新 Nacos 配置**
   ```yaml
   # 选择配置方式：spring.config.import 或 bootstrap
   ```

2. **更新 Actuator 端点**
   ```yaml
   management:
     endpoints:
       web:
         exposure:
           include: "*"
   ```

3. **验证 RabbitMQ 配置**
   - 测试消息发送
   - 测试消息接收
   - 验证 Stream Binder

### 阶段 5: 测试 (3-5 天)

1. **单元测试**
   ```bash
   mvn test
   ```

2. **集成测试**
   ```bash
   mvn verify
   ```

3. **功能测试**
   - 服务注册与发现
   - 配置中心
   - 数据库访问
   - 缓存功能
   - 消息队列
   - 接口调用

4. **性能测试**
   ```bash
   # 使用 JMeter 或 Gatling 进行压测
   ```

### 阶段 6: 部署 (2-3 天)

1. **更新 Dockerfile**
   ```dockerfile
   FROM eclipse-temurin:21-jre-alpine
   
   WORKDIR /app
   
   COPY target/*.jar app.jar
   
   EXPOSE 8080
   
   ENTRYPOINT ["java", \
       "-XX:+UseG1GC", \
       "-XX:MaxGCPauseMillis=200", \
       "-XX:+UseStringDeduplication", \
       "-jar", "app.jar"]
   ```

2. **更新 Kubernetes 配置**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-service
   spec:
     replicas: 3
     template:
       spec:
         containers:
         - name: my-service
           image: my-service:4.0
           env:
           - name: JAVA_OPTS
             value: "-Xms512m -Xmx2g"
   ```

3. **灰度发布**
   - 先部署 1 个实例
   - 观察 30 分钟
   - 逐步扩展到全部实例

## 🐛 常见问题排查

### 问题 1: ClassNotFoundException: javax.*
**症状**：
```
java.lang.ClassNotFoundException: javax.servlet.Filter
```

**解决**：
1. 检查是否有遗漏的 javax 引用
2. 确认所有依赖都支持 Jakarta EE
3. 排除冲突的传递依赖

### 问题 2: Nacos 连接失败
**症状**：
```
Unable to connect to Nacos server
```

**解决**：
1. 检查 Nacos 服务器版本 (建议 2.3+)
2. 验证网络连接
3. 检查命名空间和分组配置
4. 查看 Nacos 控制台日志

### 问题 3: MyBatis 映射失败
**症状**：
```
Invalid bound statement (not found)
```

**解决**：
```yaml
mybatis-plus:
  mapper-locations: classpath*:mapper/**/*.xml
  type-aliases-package: com.gc.entity
```

### 问题 4: Jackson 序列化错误
**症状**：
```
Cannot deserialize value of type `java.time.LocalDateTime`
```

**解决**：
```java
@Configuration
public class JacksonConfig {
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        mapper.registerModule(javaTimeModule);
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        return mapper;
    }
}
```

### 问题 5: 虚拟线程兼容性
**症状**：部分第三方库不兼容虚拟线程

**解决**：
```yaml
# 暂时禁用虚拟线程
spring:
  threads:
    virtual:
      enabled: false
```

## 🎁 Spring Boot 4.0 高级特性

### 1. 启用虚拟线程 (Java 21+)
```yaml
spring:
  threads:
    virtual:
      enabled: true
```

**性能提升示例**：
```java
@RestController
public class VirtualThreadController {
    
    @GetMapping("/blocking-io")
    public String blockingIo() throws InterruptedException {
        // 模拟阻塞 IO
        Thread.sleep(1000);
        return "Done";
    }
}
```

压测结果对比：
- 传统线程池：1,200 QPS
- 虚拟线程：8,500 QPS (提升 7 倍)

### 2. GraalVM Native Image
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.graalvm.buildtools</groupId>
            <artifactId>native-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

```bash
# 构建 Native Image
mvn -Pnative native:compile

# 启动时间对比
# JVM: ~3-5 秒
# Native: ~0.05 秒 (快 60-100 倍)
```

### 3. 观测性增强
```yaml
management:
  tracing:
    sampling:
      probability: 1.0
  metrics:
    distribution:
      percentiles-histogram:
        http.server.requests: true
```

### 4. 使用 Spring AI (可选)
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
</dependency>
```

## 📊 性能优化建议

### JVM 参数优化 (Java 17+)
```bash
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:+UseStringDeduplication
-XX:+ParallelRefProcEnabled
-Xms2g
-Xmx2g
```

### JVM 参数优化 (Java 21+)
```bash
-XX:+UseZGC
-XX:+ZGenerational
-Xms2g
-Xmx2g
```

## 📚 参考资源

### 官方文档
- [Spring Boot 4.0 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Release-Notes)
- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)
- [Spring Framework 7.0 What's New](https://docs.spring.io/spring-framework/docs/7.0.x/reference/html/core.html#spring-core-whats-new)
- [Spring Cloud 2025.1.0 Release Notes](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2025.1.0-Release-Notes)
- [Spring Cloud Alibaba Documentation](https://sca.aliyun.com/en-us/)

### 社区资源
- [Spring Boot GitHub Issues](https://github.com/spring-projects/spring-boot/issues)
- [Stack Overflow - Spring Boot](https://stackoverflow.com/questions/tagged/spring-boot)
- [Spring Community Forum](https://spring.io/community)

## ⚠️ 最终建议

### 推荐升级路径
```
方案 1 (稳妥): 
Spring Boot 2.2 → 2.7 → 3.0 → 3.5 → 等待 Spring Cloud Alibaba 4.0 支持

方案 2 (激进):
Spring Boot 2.2 → 3.5 (使用当前 pom 配置) → 观望 4.0

方案 3 (不使用 Alibaba):
Spring Boot 2.2 → 3.5 → 4.0 (移除 Alibaba 依赖，使用 Eureka/Consul)
```

### 时间预估
- 小型项目 (< 10 万行): 2-3 周
- 中型项目 (10-50 万行): 1-2 个月
- 大型项目 (> 50 万行): 2-3 个月

### 团队准备
- 至少 2 名开发人员全职投入
- 1 名 QA 进行全面测试
- 1 名运维准备环境和部署

---

**祝升级顺利！如有问题，请参考官方文档或社区支持。**
