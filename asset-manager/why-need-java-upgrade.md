# Java 升级内部培训手册 (Java 8 → Java 21)

> 📋 **适用场景**：Spring Boot 应用从 Java 8 升级到 Java 21 LTS  
> 🎯 **目标受众**：研发团队、架构师、测试与运维同学  
> ⏱️ **培训时长**：2-4 小时（可按项目规模调整）  
> 🎓 **培训目标**：理解升级原因、掌握标准路径、识别常见风险、能够独立完成验证

### 培训使用说明

本手册面向**内部培训、技术传递和项目复盘**场景，重点回答四个问题：

1. **为什么要从 Java 8 升级到 Java 21**
2. **为什么升级路径是 8 → 17 → 21，而不是一步到位**
3. **Spring Boot 3.x 与 Jakarta 迁移分别解决什么问题**
4. **如何在真实项目中完成升级、验证并控制风险**

---

## 📌 一、为什么要升级？

### 培训重点：升级的业务与技术动因

1. **安全问题** 🔒
    - Java 8 在 2019 年后已不再提供免费公开更新
    - 长期停留在旧版本会带来安全补丁缺失与合规风险
    - 对企业系统而言，继续停留在旧运行时会放大基础设施风险

2. **性能提升** ⚡
    - Java 21 在吞吐、GC、容器支持等方面相比 Java 8 有明显提升
    - 内存利用率和运行效率通常更优
    - 对高并发、容器化和云部署场景更友好

3. **新特性** 🚀
    - Java 8 到 Java 21 跨越了 **13 年**的语言与运行时演进
    - 可以使用更现代的语言能力和更成熟的运行时优化
    - 也为 Spring Boot 3.x、现代中间件和云原生部署提供了基础条件

---

## 📅 二、Java 版本演进速览（培训导入）

### Java 8 (2014年发布)
- 🌟 **经典版本**，曾经的行业标准
- 引入了 Lambda 表达式（那个箭头 `->` 的写法）
- 引入了 Stream API（处理集合数据更方便）
- ⚠️ **2019年停止免费商用更新**

### Java 11 (2018年发布)
- 第一个"长期支持版本"（LTS）
- 删除了很多老旧的模块
- 性能提升 10-15%

### Java 17 (2021年发布)
- 第二个重要的 LTS 版本
- 密封类、Records（像 C# 的 record）
- Spring Boot 3.x 的**最低要求版本**

### Java 21 (2023年发布) ⭐
- 🎯 **当前最新的 LTS 版本**
- ✅ 官方支持到 **2031年**（至少 8 年维护期）
- 虚拟线程（Virtual Threads）- 处理高并发超强
- 模式匹配、记录模式 - 代码更简洁
- 性能和内存优化达到新高度

**小贴士**：LTS = Long Term Support（长期支持），意味着会持续获得安全更新和 bug 修复。

### ❓为什么这次升级要先验证 Java 17，再最终切到 Java 21？

在培训和项目实施中，这是最常见的问题之一：**既然目标是 Java 21，为什么不一步到位？**

标准答案是：**理论上可以直接升级，但对于从 Java 8 迁移的存量项目，分两步实施更稳妥。**

这次升级其实不是单纯换一个 JDK，而是同时涉及三类变化：

1. **JDK 版本跨越很大**：Java 8 → Java 21
2. **框架升级**：Spring Boot 2.x → 3.x
3. **企业规范迁移**：`javax.*` → `jakarta.*`

如果一步直接切到 Java 21，一旦编译失败或测试失败，就很难快速判断问题到底来自：

- JDK 21 本身的兼容性
- Spring Boot 3.x 的升级影响
- Jakarta 命名空间变更
- 某些旧依赖或 Maven 插件版本过老

因此，内部推荐的实施顺序是：

- **先在 Java 17 上完成 Spring Boot 3 + Jakarta 迁移**
- **确认编译和测试通过后，再切到 Java 21**

### 为什么 Java 17 特别适合作为“过渡版本”？

因为 Java 17 是：

- ✅ **LTS 版本**
- ✅ **Spring Boot 3.x / Spring Framework 6 的最低基线**
- ✅ **大量第三方库优先适配的版本**

这意味着：

- 先用 Java 17，可以先把**框架迁移问题**清理干净
- 再切到 Java 21，剩下的问题通常就只和**JDK 21 本身**或少量依赖兼容性有关
- 这样定位更快、风险更低、回退也更简单

### 培训结论

> **先验证 Java 17，不是因为 Java 21 不能用，而是为了把“框架迁移问题”和“JDK 升级问题”拆开处理。**

对于已经是 Spring Boot 3.x、依赖也较新的项目，**可以直接上 Java 21**；但对于像本项目这种从 **Java 8 老项目现代化升级** 的场景，**先 17、后 21** 是更安全、也更适合团队协作与排障的企业实践。

---

## 🔍 三、Java 8 vs Java 21 核心差异速览

| 对比项 | Java 8 (2014) | Java 21 (2023) | 实际影响 |
|--------|---------------|----------------|----------|
| **支持状态** | ❌ 已停止免费更新 | ✅ 支持到 2031年 | 安全和合规 |
| **性能** | 基准线 | 🚀 快 20-40% | 响应速度、成本 |
| **内存占用** | 基准线 | 📉 减少 10-20% | 服务器成本 |
| **虚拟线程** | ❌ 不支持 | ✅ 原生支持 | 高并发场景性能翻倍 |
| **语法糖** | Lambda、Stream | ✨ + Records、模式匹配、文本块 | 代码量减少 30% |
| **垃圾回收器** | G1GC（较老） | 🎯 ZGC、G1GC（优化版） | 停顿时间从 100ms → 1ms |
| **容器支持** | 😐 一般 | ✅ 原生优化 | Docker 环境省资源 |

---

## 🛠️ 四、升级步骤（内部标准流程）

### 前提条件检查 ✅

```bash
# 1. 确认是 Maven 或 Gradle 项目
ls pom.xml          # Maven 项目
# 或
ls build.gradle     # Gradle 项目

# 2. 确认 Git 已初始化
git status

# 3. 确认当前 Java 版本
java -version       # 应该看到 1.8.x
```

---

### 🚦 标准升级流程（6 步）

#### **第 1 步：环境准备**（5-10 分钟）

```bash
# 安装 JDK 17（中间版本，更安全）
# 方法 1：使用 SDKMAN（推荐）
sdk install java 17.0.16-tem

# 方法 2：从官网下载
# https://adoptium.net/temurin/releases/?version=17

# 安装 JDK 21（目标版本）
sdk install java 21.0.8-tem
```

**为什么要装两个版本？**
- Java 17 是"过渡版本"，先测试兼容性
- Java 21 是最终目标
- 就像装修房子，先刷底漆再刷面漆

---

#### **第 2 步：建立基线**（10-15 分钟）

```bash
# 运行当前版本的编译和测试，记录结果
./mvnw clean compile test-compile
./mvnw test

# 记录测试通过率，比如：10个测试，9个通过 = 90%
# 这是后面验证的"及格线"
```

**📝 记录模板**：
```
编译状态：✅ 成功 / ❌ 失败
测试通过：___ / ___ (___%)
```

---

#### **第 3 步：升级 Spring Boot + Jakarta 命名空间**（30-60 分钟）

这是**整个升级过程中最关键的一步**，也是培训中应重点讲解的部分。

##### 3.1 修改 `pom.xml`（根目录）

```xml
<!-- 之前 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.18</version>
</parent>

<!-- 改成 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.3</version>
</parent>

<!-- 同时修改 Java 版本 -->
<properties>
    <java.version>17</java.version>  <!-- 先改成 17 -->
</properties>
```

##### 3.2 批量替换命名空间（重要！）

**为什么必须替换？**
- Spring Boot 3.x 使用了新的 Jakarta EE 标准
- 所有 `javax.*` 的包名改成了 `jakarta.*`
- 这是 Spring Boot 3.x / Spring Framework 6 的兼容性前提

**搜索替换清单**（在整个项目中执行）：

| 查找 | 替换为 | 用途 |
|------|--------|------|
| `import javax.persistence.` | `import jakarta.persistence.` | JPA 注解 |
| `import javax.annotation.` | `import jakarta.annotation.` | 注解（如 @PostConstruct） |
| `import javax.servlet.` | `import jakarta.servlet.` | Web 相关 |
| `import javax.validation.` | `import jakarta.validation.` | 数据验证 |
| `import javax.inject.` | `import jakarta.inject.` | 依赖注入 |

**操作方法**：
- IDE：使用"在路径中替换"功能（Ctrl+Shift+H / Cmd+Shift+H）
- 命令行：`find . -name "*.java" -exec sed -i 's/javax.persistence/jakarta.persistence/g' {} +`

##### 3.3 处理常见编译错误

**错误 1**：`WebMvcConfigurerAdapter` 找不到
```java
// ❌ 旧写法（已删除）
public class WebConfig extends WebMvcConfigurerAdapter {
    ...
}

// ✅ 新写法
public class WebConfig implements WebMvcConfigurer {
    ...
}
```

**错误 2**：`HandlerInterceptorAdapter` 找不到
```java
// ❌ 旧写法（已删除）
public class MyInterceptor extends HandlerInterceptorAdapter {
    ...
}

// ✅ 新写法
public class MyInterceptor implements HandlerInterceptor {
    ...
}
```

##### 3.4 编译验证

```bash
# 使用 JDK 17 编译
export JAVA_HOME=/path/to/jdk-17
./mvnw clean test-compile

# 目标：编译成功
# 说明：此阶段若存在少量测试失败，可在最终验证阶段统一修复
```

---

#### **第 4 步：验证 Java 17**（5 分钟）

```bash
# 确认 pom.xml 中已经设置
<java.version>17</java.version>

# 重新编译验证
export JAVA_HOME=/path/to/jdk-17
./mvnw clean compile

# 看到类似输出就对了：
# [INFO] Compiling 15 source files with javac [debug release 17]
```

---

#### **第 5 步：升级到 Java 21**（10 分钟）

```xml
<!-- 修改 pom.xml -->
<properties>
    <java.version>21</java.version>  <!-- 17 改成 21 -->
</properties>
```

```bash
# 使用 JDK 21 编译
export JAVA_HOME=/path/to/jdk-21
./mvnw clean compile

# 再次验证编译成功
```

---

#### **第 6 步：终极验证**（30-60 分钟）

```bash
# 完整的编译 + 测试
export JAVA_HOME=/path/to/jdk-21
./mvnw clean test

# 📊 检查结果
```

**成功标准**：
- ✅ 编译成功（主代码 + 测试代码都能编译）
- ✅ 测试通过率 ≥ 基线（比如基线 90%，现在至少 90%）
- ✅ 最好达到 100% 通过

**如果测试失败怎么办？**

1. **查看错误日志**，常见问题：
   - `ClassNotFoundException`：检查是否漏掉了 jakarta 命名空间替换
   - `NoSuchMethodError`：Spring Boot 3.x 删除了某些旧 API，需要找替代方案
   - Mockito 相关错误：升级 Mockito 依赖版本

2. **逐个修复**：
   ```bash
   # 只运行失败的测试
   ./mvnw test -Dtest=FailedTestClassName
   ```

3. **提交代码**：
   ```bash
   git add .
   git commit -m "Step 6: Final Validation - Compile: SUCCESS | Tests: 4/4 passed"
   ```

---

## 🧰 五、使用的工具清单

### 核心工具

| 工具 | 版本 | 用途 | 获取方式 |
|------|------|------|----------|
| **JDK 17** | 17.0.16+ | 中间版本（测试兼容性） | [Adoptium](https://adoptium.net) |
| **JDK 21** | 21.0.8+ | 目标版本 | [Adoptium](https://adoptium.net) |
| **Maven** | 3.8+ | 构建工具 | 项目内置 Maven Wrapper (./mvnw) |
| **Spring Boot** | 3.2.3+ | 框架升级 | pom.xml 依赖 |
| **Git** | 2.0+ | 版本控制 | 系统自带 |

### 辅助工具（可选但推荐）

| 工具 | 用途 | 命令/链接 |
|------|------|-----------|
| **SDKMAN!** | JDK 版本管理 | `curl -s "https://get.sdkman.io" \| bash` |
| **IntelliJ IDEA** | IDE 支持 | 内置 Jakarta 迁移助手 |
| **VS Code** | 轻量编辑器 | + Java Extension Pack |
| **OpenRewrite** | 自动化迁移 | [https://docs.openrewrite.org](https://docs.openrewrite.org) |

---

## ⚠️ 六、常见问题与解决方案

### 问题 1：编译时提示"找不到符号"

**现象**：
```
[ERROR] cannot find symbol: class Entity
```

**原因**：`javax.persistence.Entity` 改成了 `jakarta.persistence.Entity`

**解决**：
```bash
# 全局搜索替换
find . -name "*.java" -exec grep -l "javax.persistence" {} \; \
  | xargs sed -i 's/javax\.persistence/jakarta.persistence/g'
```

---

### 问题 2：Spring Boot 应用启动失败

**现象**：
```
org.springframework.beans.factory.BeanCreationException
```

**常见原因**：
1. `application.properties` 配置项改名
2. 自定义的 `WebMvcConfigurer` 写法过时

**解决**：
- 查看 [Spring Boot 3.0 迁移指南](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)
- 检查是否有使用 `@Configuration` + `WebMvcConfigurerAdapter`（已删除）

---

### 问题 3：测试失败 - Mockito 无法模拟

**现象**：
```
Mockito cannot mock/spy because final class
```

**原因**：Java 8 的 Mockito 版本太老

**解决**：Spring Boot 3.2.3 会自动升级 Mockito 到 5.x，支持 final 类模拟

---

### 问题 4：Hibernate 查询报错

**现象**：
```
org.hibernate.QueryException: could not resolve property
```

**原因**：Hibernate 6.x 的查询语法更严格

**解决**：
- 检查 HQL/JPQL 查询中的属性名是否正确
- 确保实体类的 `@Entity` 注解已改为 `jakarta.persistence.Entity`

---

## 📋 七、升级检查清单

建议在内部培训、实施演练或项目执行时，按清单逐项勾选：

### 升级前检查
- [ ] 备份当前代码（`git commit` 或创建分支）
- [ ] 记录当前 Java 版本：`java -version`
- [ ] 运行基线测试，记录通过率
- [ ] 确认 Maven Wrapper 可用：`./mvnw --version`

### 升级中检查
- [ ] 安装 JDK 17 和 JDK 21
- [ ] 修改 `pom.xml` 中的 Spring Boot 版本
- [ ] 批量替换 `javax.*` → `jakarta.*`
- [ ] 修复编译错误（主要是 Jakarta 相关）
- [ ] 使用 JDK 17 编译成功
- [ ] 修改 `java.version` 为 21
- [ ] 使用 JDK 21 编译成功

### 升级后验证
- [ ] 运行完整测试套件
- [ ] 测试通过率 ≥ 基线
- [ ] 检查日志中无严重警告
- [ ] 本地启动应用，访问主要功能
- [ ] 提交代码到 Git

---

## 🎓 八、内部培训演示脚本（15 分钟快速版）

### 适合培训讲解的简化流程

```bash
# === 准备阶段（2 分钟）===
cd your-project
git checkout -b upgrade-java21

# === 安装 JDK（3 分钟）===
sdk install java 17.0.16-tem
sdk install java 21.0.8-tem

# === 基线测试（2 分钟）===
./mvnw clean test
# 记录：X 个测试通过

# === 核心升级（5 分钟）===
# 1. 手动修改 pom.xml：
#    - spring-boot: 2.7.18 → 3.2.3
#    - java.version: 8 → 17

# 2. 全局替换（在 IDE 中执行）：
#    - javax.persistence → jakarta.persistence
#    - javax.annotation → jakarta.annotation
#    - javax.servlet → jakarta.servlet

# 3. 编译验证
export JAVA_HOME=$HOME/.sdkman/candidates/java/17.0.16-tem
./mvnw clean compile

# === 最终升级（1 分钟）===
# 修改 pom.xml：java.version: 17 → 21
export JAVA_HOME=$HOME/.sdkman/candidates/java/21.0.8-tem
./mvnw clean test

# === 展示结果（2 分钟）===
java -version  # 显示 Java 21
cat pom.xml | grep java.version  # 显示 <java.version>21</java.version>
```

---

## 💡 九、给内部团队的实施建议

### ✅ 推荐做法

1. **先在测试环境试** - 不要直接在生产环境升级
2. **多模块项目要一起升** - 不能只升级一个模块
3. **使用版本控制** - 每一步都提交 Git
4. **保留旧版本 JDK** - 万一要回滚

### ⚠️ 实施注意事项

1. **升级时机**：选择业务低峰期，留出 1-2 天的验证时间
2. **测试覆盖**：升级后至少测试主要业务流程
3. **依赖检查**：如果有私有依赖库，确保它们支持 Java 21
4. **容器镜像**：Docker 镜像也要更新基础镜像（如 `eclipse-temurin:21-jre`）

### 🎯 成功标准

- ✅ 编译零错误
- ✅ 测试通过率 ≥ 升级前
- ✅ 应用能正常启动
- ✅ 核心业务功能可正常使用
- ✅ 性能无明显下降（理想情况下应有提升）

---

## 📚 十、参考资料

### 官方文档
- [Spring Boot 3.0 迁移指南](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)
- [Jakarta EE 9 规范](https://jakarta.ee/specifications/)
- [Java 21 新特性](https://openjdk.org/projects/jdk/21/)

### 实用工具
- [JDK 下载](https://adoptium.net/temurin/releases/)
- [SDKMAN!](https://sdkman.io/) - JDK 版本管理
- [OpenRewrite](https://docs.openrewrite.org/recipes/java/migrate/upgradetospringboot3) - 自动化迁移工具

### 本次升级详细记录
- 📋 完整计划：`.github/java-upgrade/20260305015027/plan.md`
- 📊 执行进度：`.github/java-upgrade/20260305015027/progress.md`
- 📝 升级总结：`.github/java-upgrade/20260305015027/summary.md`

---

## 🙋 FAQ（内部培训常见问答）

**Q1：升级会影响现有功能吗？**  
A：如果按步骤操作，功能不会变。我们只升级运行环境，代码逻辑不变。

**Q2：升级需要多久？**  
A：小项目 2-4 小时，大项目 1-2 天（含测试）。

**Q3：可以跳过中间版本直接到 Java 21 吗？**  
A：理论可以，但不建议。先到 Java 17 更安全，能及早发现问题。

**Q4：升级失败怎么回滚？**  
A：`git reset --hard <升级前的commit>`，秒级恢复。

**Q5：为什么不升级到 Java 22 或更新版本？**  
A：Java 21 是 LTS（长期支持版），22 不是。就像 Windows 11 和 Windows Server 的区别。

**Q6：升级后性能一定会提升吗？**  
A：大概率会，但取决于应用类型。I/O 密集型提升明显，纯计算型提升有限。

---

**✅ 培训建议：在正式项目升级前，先用一套测试环境完整走一遍本文流程。**

---

*最后更新：2026-03-05*  
*基于实际项目：asset-manager (Session 20260305015027)*
