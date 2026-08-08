---
title: "第二、三周周考复盘：从 Spring Boot 基础到 AOP，越往后越要抠细节"
description: "连续两周周考复盘：从 Web 前端三剑客、Spring Boot 入门到 Maven 多模块、MyBatis 进阶、Spring AOP，整理易错点和知识盲点。"
pubDate: 2026-08-08
tags:
  - Java
  - Spring Boot
  - MyBatis
  - Spring AOP
  - 周考复盘
  - 学习记录
---

两周的周考连在一起做下来，一个明显的感觉是：**第一周考的是"有没有印象"，第二周开始考"理解准不准"，第三周直接考"会不会踩坑"**。

第一周 68 分那篇复盘里，我主要在补"知道 vs 不知道"的差距；这两周做下来，更多的问题出在"以为自己懂了，但细节一问就露馅"。这篇把两周的知识点、易错点放在一起整理，既是复盘，也是以后复习的索引。

---

## 整体感受：难度曲线是这样的

| 周次 | 主题范围 | 最大的感受 |
|------|---------|-----------|
| 第一周 | Java 基础 + 面向对象 + 集合 + Stream + Lambda | 概念多，但只要背了就有分 |
| 第二周 | Web 前端基础 + Spring Boot 入门 + MyBatis 基础 + SQL + AI 提示词 | 开始串起来了，分层架构、RESTful 这些要理解 |
| 第三周 | Git + Maven 多模块 + MyBatis 多表 + 全局异常 + Spring AOP | 坑变多了，自调用失效、动态代理区别、`<where>` 标签细节 |

两周连起来看，知识从"一个个孤立的点"变成了"一条完整的链路"：前端发请求 → Controller 接收 → Service 处理业务 → Mapper 查数据库 → 再一路返回。到了第三周，又在这条链路上加了切面、异常处理、分页、多表联查这些"增强器"。

下面按模块整理易错点。

---

## 一、Web 前端基础：看起来简单，细节容易丢分

第二周开头几道题考的是前端基础，属于"送分题"，但真要全对也不容易。

### 易错点 1：CSS 选择器权重排序

口诀必须记死：

> **行内（1000）> id（100）> 类 / 伪类 / 属性（10）> 元素 / 伪元素（1）> \*（0）**

容易搞混的是"行内样式"和"id 选择器"谁更高——行内样式（style 属性）权重最高，因为它直接写在元素上。

### 易错点 2：var / let / const 别再用 var 了

现代 JS 的推荐是：**能用 const 就用 const，需要重新赋值再用 let，永远别用 var**。

原因很实在：var 有变量提升、没有块级作用域，容易出奇怪的 bug。let/const 有块级作用域，更安全。

### 易错点 3：B/S 与 C/S 架构的区别

这是问答题第一题，必须能清晰对比：

| 对比点 | B/S 架构 | C/S 架构 |
|--------|---------|---------|
| 客户端 | 浏览器，不用安装 | 需下载安装专用客户端 |
| 更新 | 服务器更新即可，用户无感 | 用户要重新下载安装 |
| 跨平台 | 有浏览器就能用 | 不同 OS 要不同版本 |
| 典型例子 | 淘宝、微博、B 站 | 微信桌面版、QQ、王者荣耀 |

> 记不住的时候想：B 是 Browser（浏览器），C 是 Client（客户端软件），这样就不会搞混了。

---

## 二、Maven：scope 和多模块是重灾区

Maven 看起来就是个 pom.xml，但知识点藏得很深。

### 易错点 1：四种依赖范围 scope

| scope | 编译 | 测试 | 运行 | 打包 | 典型例子 |
|-------|-----|-----|-----|-----|---------|
| compile（默认）| ✅ | ✅ | ✅ | ✅ | spring-boot-starter-web |
| test | ❌ | ✅ | ❌ | ❌ | junit |
| provided | ✅ | ✅ | ❌ | ❌ | servlet-api（Tomcat 自带）|
| runtime | ❌ | ✅ | ✅ | ✅ | MySQL 驱动（编译时不需要，运行时才用）|

最容易记错的是 **provided** 和 **runtime**：provided 是"编译测试都有，但运行时服务器会提供，不用打包"；runtime 是"编译不需要，但运行时必须有"。

### 易错点 2：dependencyManagement vs dependencies（第三周重点）

这是第三周单选题第 7 题，非常容易搞混：

- **父工程 `<dependencies>`**：声明的依赖会**自动传递**给所有子模块，子模块不用再写。
- **父工程 `<dependencyManagement>`**：只**统一管理版本号**，不会自动添加依赖。子模块想用还得自己写（但不用写 version）。

一句话区分：**dependencies 是"自动送你"，dependencyManagement 是"我把价格标好了，你要买自己来拿"。**

### 易错点 3：父工程的 packaging

Maven 多模块项目中，父工程的 `<packaging>` 必须是 **pom**，不是 jar 也不是 war。因为父工程本身不写代码，只是用来聚合子模块和管理版本。

---

## 三、Spring Boot 核心：IOC/DI 必须讲清楚

### 易错点 1：@SpringBootApplication 是个组合注解

启动类上的 `@SpringBootApplication` 不是单一注解，它包含三个：

- `@Configuration`：标记配置类
- `@EnableAutoConfiguration`：开启自动配置
- `@ComponentScan`：组件扫描

考试时如果问"Spring Boot 启动类注解是什么"，答 `@SpringBootApplication` 就对了；如果问"它由什么组成"，要能说出这三个。

### 易错点 2：IOC 和 DI 不是一回事（问答题高频）

这是第二周问答题，也是最容易"说不清楚"的题。整理成三段式：

**1. 什么是 IOC（控制反转）？**

传统开发：对象自己 new 依赖的东西，控制权在开发者手里，耦合度高。
IOC：把对象的创建和管理权交给 Spring 容器，由容器统一创建和管理 Bean。控制权从开发者"反转"给了容器，所以叫控制反转。

**2. 什么是 DI（依赖注入）？**

DI 是 IOC 的**具体实现方式**。容器在创建对象时，自动把该对象依赖的其他对象注入进来，不用手动获取。

三种注入方式：
- 构造器注入（**Spring 官方推荐必填依赖**，保证不可变、不为 null、方便测试）
- Setter 注入（推荐可选依赖）
- 字段注入（`@Autowired` 直接注字段上，**不推荐**，但很多老项目在用）

**3. 为什么需要 IOC？**

- **解耦**：依赖接口不依赖实现，换实现类很方便
- **方便测试**：单元测试可以用 Mock 对象替换
- **可复用**：同一个 Bean 多处复用
- **统一管理**：容器管理生命周期

> 记这个问答题的技巧：IOC 是"思想/目标"，DI 是"实现手段"，然后说好处。别把两者混成一个东西。

### 易错点 3：三层架构的职责

经典三层架构，填空题考过：

- **Controller / 表现层**：接收 HTTP 请求，返回响应
- **Service / 业务层**：处理业务逻辑、事务
- **Mapper / DAO / 持久层**：访问数据库

分层的核心目的是**关注点分离**，每层只干自己的事，降低耦合。

---

## 四、SQL 与 MyBatis：写错就是生产事故

### 易错点 1：SQL 语句的书写顺序

口诀：**S-F-W-G-H-O-L**

```
select → from → where → group by → having → order by → limit
```

注意：where 是在 group by **之前**过滤行，having 是在 group by **之后**过滤分组。顺序不能乱。

### 易错点 2：SQL 分类

| 分类 | 全称 | 关键字 |
|------|------|--------|
| DDL | Data Definition Language（数据定义语言）| create / drop / alter |
| DML | Data Manipulation Language（数据操作语言）| insert / update / delete |
| DQL | Data Query Language（数据查询语言）| select |
| DCL | Data Control Language（数据控制语言）| grant / revoke |

`CREATE TABLE` 属于 **DDL**，不是 DML。这个经常有人搞混——以为 create 是"操作数据"，其实它操作的是**表结构**。

### 易错点 3：update / delete 必须带 where

这是填空题的"送命题"。**不带 where 的 update/delete 会全表操作**，在生产环境就是事故。

> 真实工作中的经验：执行 update/delete 前，先用 select 查一下 where 条件对不对，确认了再动手。

### 易错点 4：下划线转驼峰配置

MyBatis 配置中，开启下划线转驼峰的配置项是：

```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
```

数据库用 `create_time`（下划线），Java 用 `createTime`（驼峰），开了这个配置自动映射。**这行配置几乎每个项目都会加，但很多人说不出它的确切名字。**

### 易错点 5：`<where>` 标签的细节（第三周重点）

第三周单选第 6 题考了 `<where>` 标签，错误率极高。关键知识点：

- ✅ `<where>` 会自动去掉 SQL **开头多余的 and/or**
- ✅ 如果所有 `<if>` 都不成立，`<where>` **不会生成 where 关键字**
- ✅ 每个 `<if>` 内部**仍然要写 and 前缀**——`<where>` 就是用来处理第一个条件前多余的 and 的
- ❌ `<where>` 不会深入每个 `<if>` 内部处理，只处理最外层第一个条件前的 and

> 一句话：**每个 if 都写 and，where 会帮你收拾第一个的烂摊子。** 别反过来以为有了 where 就不用写 and 了。

### 易错点 6：多参数用 @Param

Mapper 接口方法有**多个参数**时，每个参数前要加 `@Param` 注解，XML 里才能用 `#{xxx}` 按名字引用。

```java
List<Course> list(@Param("keyword") String keyword, 
                  @Param("status") Integer status);
```

为什么要加？因为 Java 编译后参数名会变成 arg0、arg1，MyBatis 认不出来。`@Param` 给它们起名字。

### 易错点 7：`<association>` vs `<collection>`

MyBatis 多表联查的 resultMap 中：

- **`<association>`**：映射**一对一**关系（比如课程关联一个教练）
- **`<collection>`**：映射**一对多**关系（比如一个部门有多个员工）

别用混了。

### 易错点 8：PageHelper 只对第一条查询生效

PageHelper 的 `startPage()` **只对紧跟在它后面的第一条 select 语句生效**，不是所有后续查询都分页。中间不能插别的 select，否则分页就跑到那条上去了。

所以标准写法是：**Service 层里，PageHelper.startPage() 之后立刻紧跟你要分页的查询。**

---

## 五、Spring AOP：第三周最难的一块

AOP（面向切面编程）是第三周的重头戏，也是坑最多的地方。

### 易错点 1：五个核心术语

这是问答题，五个术语必须都能说清楚：

| 术语 | 含义 |
|------|------|
| **切面（Aspect）** | 横切关注点的模块化，比如"日志切面""事务切面" |
| **切点（Pointcut）** | 匹配哪些方法的规则（表达式），决定"切哪里" |
| **通知（Advice）** | 切面要执行的具体逻辑，决定"什么时候切、干什么" |
| **连接点（JoinPoint）** | 程序执行过程中的点（通常是方法调用），是"可以被切的地方" |
| **织入（Weaving）** | 把切面逻辑应用到目标对象上的过程，生成代理对象 |

### 易错点 2：五种通知注解

| 注解 | 执行时机 |
|------|---------|
| `@Before` | 方法执行之前 |
| `@After` | 方法执行之后（无论正常返回还是抛异常） |
| `@AfterReturning` | 方法正常返回之后 |
| `@AfterThrowing` | 方法抛出异常之后 |
| `@Around` | **环绕通知**，方法执行前后都可以加逻辑，还能控制是否调用目标方法 |

其中 `@Around` 最强大，也最常用——统计耗时、事务管理这些都用它。

### 易错点 3：JDK 动态代理 vs CGLIB 动态代理

这是问答题高频题，三个核心区别：

| 对比点 | JDK 动态代理 | CGLIB 动态代理 |
|--------|-------------|---------------|
| 代理方式 | 基于**接口**，生成实现接口的代理类 | 基于**继承**，生成目标类的子类 |
| 适用条件 | 目标类必须实现接口 | 目标类不能是 final 的，方法不能是 final/static 的 |
| 性能 | 创建代理快，调用稍慢 | 创建代理慢（字节码操作），调用更快（继承调用） |

**Spring 默认用哪种？**
- 如果目标类实现了接口，默认用 **JDK 动态代理**
- 如果目标类没有接口，自动用 **CGLIB**

**什么情况强制用 CGLIB？**
- 设置 `spring.aop.proxy-target-class=true`（或 `@EnableAspectJAutoProxy(proxyTargetClass = true)`）
- Spring Boot 2.0+ 默认已经是 CGLIB 了（`spring.aop.proxy-target-class` 默认 true）

### 易错点 4：自调用导致 AOP 失效（必考点）

这是最经典的坑：**同一个类里，方法 A 调用方法 B（`this.methodB()`），方法 B 上的 AOP 不生效。**

原因：Spring AOP 是通过**代理对象**实现的。外部调用走的是代理对象（代理对象在前后加逻辑），但 `this.xxx()` 走的是原始对象本身，跳过了代理，所以 AOP 失效。

解决办法（了解即可）：
1. 注入自己（`@Autowired` 注入自身，用注入的对象调用）
2. 用 `AopContext.currentProxy()` 获取代理对象
3. 把方法拆到另一个 Bean 里（最推荐，符合单一职责）

### 易错点 5：切点表达式的写法

常见写法：

```text
execution(* com.example.controller..*.*(..))
```

解释一下每个部分：
- 第一个 `*`：返回值任意
- `com.example.controller..`：controller 包及其子包
- 第二个 `*`：类名任意
- 第三个 `*`：方法名任意
- `(..)`：参数任意

另外还有注解式切点：
- `@annotation(com.example.annotation.OperationLog)`：匹配标了某个注解的方法

多种表达式可以用 `&&` 组合。

### 易错点 6：操作日志为什么异步落库

AOP 做操作日志时，通常用 `@Async` 异步写库，而不是同步写。最关键的原因是：

> **异步落库不会阻塞业务方法的返回，提升接口响应速度。**

日志不是核心业务路径，没必要让用户等着日志写完再拿到结果。另外，异步还能避免日志操作和业务在同一个事务里——业务回滚了，操作日志不该跟着消失。

---

## 六、全局异常处理：Spring Boot 的标配

### 为什么要有统一异常处理？

- 前后端联调友好：不管出什么错，都返回统一格式的 `Result`
- 不用每个 Controller 都 try-catch，代码干净
- 可以做异常日志记录、报警
- 不同异常返回不同的状态码和提示信息

### 核心注解

- `@RestControllerAdvice`：标记全局异常处理类（返回 JSON）
- `@ExceptionHandler(Exception.class)`：标记处理某种异常的方法

### 常见异常分类（至少 4 类）

| 异常类型 | 典型场景 | 状态码/处理 |
|---------|---------|------------|
| `MethodArgumentNotValidException` | 参数校验失败（@Valid） | 400 / 返回字段错误信息 |
| `NoHandlerFoundException` | 请求路径不存在 | 404 |
| `HttpRequestMethodNotSupportedException` | 请求方法不对（GET 用成 POST） | 405 |
| `ArithmeticException` / `NullPointerException` | 业务代码 bug | 500 / "系统内部错误" |
| 自定义业务异常（如 `BusinessException`） | 业务逻辑不满足 | 自定义 code + 提示信息 |
| `Exception`（兜底） | 所有未捕获的异常 | 500 / 记录日志，返回友好提示 |

> 注意：Bean Validation 校验 `@RequestBody` 时用 `@Valid`，校验 Query 参数时用 `@Validated` 加在 Controller 类上。

---

## 七、Git：命令别记混了

### 易错点 1：fetch vs pull vs merge

- `git fetch`：**只拉取**远程最新代码到本地，**不合并**
- `git pull`：拉取远程并**自动合并**到当前分支（= fetch + merge）
- `git merge`：合并指定分支到当前分支

一句话区分：**fetch 是"看看有什么更新"，pull 是"拿过来并合并"。**

### 易错点 2：git reset 的三种模式

| 模式 | 回退 commit | 工作区修改 | 暂存区 |
|------|------------|-----------|--------|
| `--soft` | ✅ 回退 | ✅ 保留 | ✅ 保留 |
| `--mixed`（默认）| ✅ 回退 | ✅ 保留 | ❌ 清空 |
| `--hard` | ✅ 回退 | ❌ 丢弃 | ❌ 清空 |

> **`--hard` 是危险操作**，会彻底丢掉工作区所有未提交的修改。用之前一定要确认。

记忆技巧：soft 最"软"（保留最多），hard 最"硬"（全删干净），mixed 在中间。

---

## 八、编程题模板：两周各一道

### 第二周：Spring Boot + MyBatis 标准 CRUD

这是最基础的编程题，必须能手写出来。以 Dept 部门表为例：

**1. 实体类（Lombok @Data）**

```java
@Data
public class Dept {
    private Integer id;
    private String name;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
}
```

**2. Mapper 接口（注解方式）**

```java
@Mapper
public interface DeptMapper {

    @Select("select * from dept order by update_time desc")
    List<Dept> list();

    @Select("select * from dept where id = #{id}")
    Dept getById(Integer id);

    @Insert("insert into dept(name, create_time, update_time) values(#{name}, now(), now())")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    void insert(Dept dept);

    @Update("update dept set name = #{name}, update_time = now() where id = #{id}")
    void update(Dept dept);

    @Delete("delete from dept where id = #{id}")
    void deleteById(Integer id);
}
```

**3. Service 层（构造器注入）**

```java
@Service
@RequiredArgsConstructor
public class DeptServiceImpl implements DeptService {
    private final DeptMapper deptMapper;
    // 方法直接调用 mapper
}
```

**4. Controller（RESTful 风格）**

```java
@RestController
@RequestMapping("/depts")
@RequiredArgsConstructor
public class DeptController {
    private final DeptService deptService;

    @GetMapping          public List<Dept> list() { return deptService.list(); }
    @GetMapping("/{id}") public Dept get(@PathVariable Integer id) { return deptService.getById(id); }
    @PostMapping         public void save(@RequestBody Dept dept) { deptService.save(dept); }
    @PutMapping          public void update(@RequestBody Dept dept) { deptService.update(dept); }
    @DeleteMapping("/{id}") public void delete(@PathVariable Integer id) { deptService.deleteById(id); }
}
```

> 记住几个关键注解：`@Mapper`、`@Service`、`@RestController`、`@RequestMapping`、`@RequiredArgsConstructor`（Lombok 构造器注入）、`@Options`（主键回填）。

### 第三周：AOP 方法耗时统计切面

第三周的编程题，写一个统计 Service 层方法耗时的切面：

```java
package com.example.aspect;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
@Slf4j
public class MethodCostAspect {

    // 拦截 Service 层所有方法（包含子包）
    @Around("execution(* com.example..service..*(..))")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        // 1. 记录开始时间
        long start = System.currentTimeMillis();
        try {
            // 2. 调用目标方法（必须返回结果）
            return pjp.proceed();
        } finally {
            // 3. 计算耗时（用 finally 保证即使抛异常也输出日志）
            long cost = System.currentTimeMillis() - start;
            // 4. 获取类名和方法名
            String className = pjp.getTarget().getClass().getSimpleName();
            String methodName = pjp.getSignature().getName();
            log.info("业务方法 {}.{} 耗时 {}ms", className, methodName, cost);
        }
    }
}
```

这道题的几个关键细节：
- 用 `@Around` 环绕通知（前后都要记时间）
- 必须调用 `pjp.proceed()` 执行目标方法，并且把返回值 return 出去
- 用 `finally` 保证即使抛异常也输出耗时日志
- 异常继续向上抛（不吞异常）——因为 `proceed()` 抛出的异常我们没 catch，直接往上走了
- `@Aspect` + `@Component` 两个注解都不能少

---

## 九、两周下来最大的收获

回顾这两周的考试，有几个感受特别深：

**1. "知道"和"能说清楚"是两回事。** 比如 IOC 和 DI，好像都听过，但真要按"是什么 → 关系是什么 → 为什么需要"的逻辑讲清楚，没练过还真不行。问答题的失分基本都在这。

**2. 细节决定对错。** `<where>` 标签里每个 if 要不要写 and？PageHelper 对几条查询生效？AOP 自调用为什么失效？这些都是"平时写代码可能能跑对，但考试一问就犹豫"的细节。

**3. 编程题要练手速，更要练规范。** CRUD 看起来简单，但注解有没有写全？主键回填有没有加？构造器注入还是字段注入？RESTful 风格对不对？这些都是评分点。

---

## 接下来的计划

- 把这篇里的易错点做成速查表，考前翻一遍
- AOP 动态代理和自调用问题，要写代码亲自验证一下
- MyBatis 多表联查的 resultMap，手动写几个例子练熟
- 下一周应该会进入更深的内容，提前预习

> 考试不是目的，查漏补缺才是。分数低不丢人，丢人的是错了的题下次还错。
