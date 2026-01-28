# Spring Boot 3.5 升级迁移指南

## 一、版本升级概览

### 核心版本变更
- **Spring Boot**: 2.2.10.RELEASE → 3.5.0
- **Spring Cloud**: Hoxton.SR12 → 2024.0.0
- **Spring Cloud Alibaba**: 2.2.1.RELEASE → 2023.0.3.2
- **Java**: 8 → 17 (最低要求)

## 二、重大变更说明

### 1. Java 版本要求
- Spring Boot 3.x 最低要求 **Java 17**
- 建议使用 Java 21 LTS 以获得最佳性能
- 需要更新所有构建服务器和生产环境的 JDK 版本

### 2. Jakarta EE 命名空间迁移 (重要!)
Spring Boot 3.x 从 `javax.*` 迁移到 `jakarta.*`

#### 需要全局替换的包名:
```java
// 替换前 (javax)
import javax.servlet.*;
import javax.persistence.*;
import javax.validation.*;
import javax.interceptor.*;

// 替换后 (jakarta)
import jakarta.servlet.*;
import jakarta.persistence.*;
import jakarta.validation.*;
import jakarta.interceptor.*;
```

#### 常见替换列表:
| 旧包名 (javax) | 新包名 (jakarta) |
|---------------|-----------------|
| javax.servlet.* | jakarta.servlet.* |
| javax.persistence.* | jakarta.persistence.* |
| javax.validation.* | jakarta.validation.* |
| javax.interceptor.* | jakarta.interceptor.* |
| javax.annotation.* | jakarta.annotation.* |
| javax.transaction.* | jakarta.transaction.* |
| javax.ws.rs.* | jakarta.ws.rs.* |

### 3. 数据库驱动变更

#### MySQL 驱动
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
    <version>8.0.33</version>
</dependency>
```

#### 连接字符串更新
```yaml
# 旧配置
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db?useUnicode=true&characterEncoding=utf8

# 新配置
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
```

### 4. MyBatis Plus 升级

```xml
<!-- 旧版本 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.2.0</version>
</dependency>

<!-- 新版本 - 注意 artifact 名称变化 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.9</version>
</dependency>
```

### 5. JWT 依赖变更

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

### 6. Apache CXF 移除
原配置中包含的 CXF 依赖需要升级到兼容 Jakarta EE 的版本:

```xml
<!-- 如果继续使用 CXF，需要升级到 4.x -->
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
    <version>4.0.5</version>
</dependency>
```

### 7. Spring Cloud 配置变更

#### Nacos 配置文件位置变更
```yaml
# 旧配置 (bootstrap.yml)
spring:
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848

# 新配置 (application.yml)
# 需要添加以下依赖启用 bootstrap
spring:
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
  config:
    import: optional:nacos:application.yml
```

或添加 bootstrap 支持:
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

### 8. RabbitMQ 依赖简化

```xml
<!-- 旧配置 -->
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
</dependency>

<!-- 新配置 - 使用 starter -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 9. Git Commit ID Plugin 更新

```xml
<!-- 旧版本 -->
<plugin>
    <groupId>pl.project13.maven</groupId>
    <artifactId>git-commit-id-plugin</artifactId>
</plugin>

<!-- 新版本 - groupId 变更 -->
<plugin>
    <groupId>io.github.git-commit-id</groupId>
    <artifactId>git-commit-id-maven-plugin</artifactId>
</plugin>
```

## 三、配置文件迁移

### 1. application.yml 需要调整的配置

```yaml
# 旧配置
spring:
  jackson:
    serialization:
      write-dates-as-timestamps: false

# 新配置 (某些属性路径可能变化)
spring:
  jackson:
    serialization:
      write-dates-as-timestamps: false
      
# 日志配置
logging:
  pattern:
    # 使用新的日志变量格式
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
```

### 2. Actuator 端点配置

```yaml
# 建议的 actuator 配置
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always
```

## 四、代码修改建议

### 1. 使用 Records (Java 17+ 特性)

```java
// 旧写法
public class UserDTO {
    private Long id;
    private String name;
    // getters, setters, equals, hashCode, toString
}

// 新写法 (推荐)
public record UserDTO(Long id, String name) {}
```

### 2. Pattern Matching for instanceof

```java
// 旧写法
if (obj instanceof String) {
    String str = (String) obj;
    // use str
}

// 新写法
if (obj instanceof String str) {
    // 直接使用 str
}
```

### 3. Switch 表达式

```java
// 旧写法
String result;
switch (day) {
    case MONDAY:
    case FRIDAY:
        result = "Work";
        break;
    default:
        result = "Rest";
}

// 新写法
String result = switch (day) {
    case MONDAY, FRIDAY -> "Work";
    default -> "Rest";
};
```

## 五、升级步骤建议

### 步骤 1: 准备工作
1. ✅ 备份当前项目代码
2. ✅ 升级开发环境到 Java 17+
3. ✅ 更新 IDE 到最新版本
4. ✅ 创建新的 feature 分支

### 步骤 2: 更新依赖
1. ✅ 更新 pom.xml 到新版本
2. ✅ 执行 `mvn clean compile` 查看编译错误
3. ✅ 解决所有依赖冲突

### 步骤 3: 代码迁移
1. ✅ 全局替换 `javax.*` 为 `jakarta.*`
2. ✅ 更新数据库驱动配置
3. ✅ 修改 JWT 相关代码
4. ✅ 更新 MyBatis Plus 相关注解

### 步骤 4: 配置文件更新
1. ✅ 更新 application.yml / application.properties
2. ✅ 更新 bootstrap.yml (如果使用)
3. ✅ 检查 Nacos 配置

### 步骤 5: 测试
1. ✅ 单元测试
2. ✅ 集成测试
3. ✅ 端到端测试
4. ✅ 性能测试

### 步骤 6: 部署
1. ✅ 更新 Docker 镜像 (使用 Java 17 基础镜像)
2. ✅ 更新 Kubernetes 配置
3. ✅ 灰度发布
4. ✅ 全量发布

## 六、常见问题排查

### 问题 1: NoClassDefFoundError javax.*
**原因**: 未完全替换为 jakarta.*
**解决**: 全局搜索并替换所有 javax 引用

### 问题 2: MySQL 连接失败
**原因**: 驱动类名或 URL 格式错误
**解决**: 使用新的驱动类名和 URL 格式

### 问题 3: MyBatis Plus 启动失败
**原因**: 使用了旧版本的 starter
**解决**: 使用 `mybatis-plus-spring-boot3-starter`

### 问题 4: Nacos 配置无法加载
**原因**: Spring Cloud 2020+ 移除了 bootstrap 默认支持
**解决**: 添加 bootstrap starter 或使用 spring.config.import

### 问题 5: JWT 解析失败
**原因**: JJWT 0.12+ API 有重大变更
**解决**: 更新 JWT 创建和解析代码

## 七、性能优化建议

1. **启用虚拟线程** (Java 21+)
```yaml
spring:
  threads:
    virtual:
      enabled: true
```

2. **使用 GraalVM Native Image** (可选)
   - 显著减少启动时间和内存占用
   - 需要额外配置和测试

3. **优化 JVM 参数**
```bash
# Java 17+ 推荐参数
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:+UseStringDeduplication
```

## 八、参考资源

- [Spring Boot 3.5 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.5-Release-Notes)
- [Spring Cloud 2024.0 Release Notes](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2024.0-Release-Notes)
- [Jakarta EE Migration Guide](https://jakarta.ee/specifications/platform/9/jakarta-platform-spec-9.html)
- [Spring Cloud Alibaba 文档](https://sca.aliyun.com/)

## 九、联系支持

如遇到问题，可以:
1. 查看 Spring Boot 官方文档
2. 搜索 GitHub Issues
3. 咨询团队技术专家
4. 参考社区最佳实践

---

**注意**: 本升级涉及重大版本变更，建议在测试环境充分测试后再上生产环境。
