---
title: "深入理解 Java 动态代理：JDK 与 CGLIB 的原理与实践"
description: "从静态代理的局限出发，拆解 JDK 动态代理和 CGLIB 动态代理的实现原理，用可运行的代码对比两者的差异，最后看 Spring AOP 如何在这两者之间做选择。"
pubDate: 2026-08-07
tags:
  - Java
  - 动态代理
  - Spring AOP
  - 设计模式
---

代理模式是面向对象编程中最常用的模式之一。它的核心思想很朴素：**不直接访问目标对象，而是通过一个"中间人"来间接访问**。这个中间人可以在调用前后做额外的事情——记日志、控权限、管事务、做缓存。

但在 Java 里，如果每个代理都手写，你会发现代码里充满了重复。动态代理就是用来解决这个问题的。

## 一、从静态代理说起：重复代码的困境

假设我们有一个用户服务接口和它的实现：

```java
public interface UserService {
    String getUser(Long id);
    void saveUser(String name);
}

public class UserServiceImpl implements UserService {
    @Override
    public String getUser(Long id) {
        System.out.println("查询用户: " + id);
        return "User-" + id;
    }

    @Override
    public void saveUser(String name) {
        System.out.println("保存用户: " + name);
    }
}
```

现在我们想在每次调用前后打印日志。用静态代理的写法：

```java
public class UserServiceProxy implements UserService {
    private final UserService target;

    public UserServiceProxy(UserService target) {
        this.target = target;
    }

    @Override
    public String getUser(Long id) {
        System.out.println("[日志] 方法 getUser 开始, 参数: " + id);
        String result = target.getUser(id);
        System.out.println("[日志] 方法 getUser 结束, 返回: " + result);
        return result;
    }

    @Override
    public void saveUser(String name) {
        System.out.println("[日志] 方法 saveUser 开始, 参数: " + name);
        target.saveUser(name);
        System.out.println("[日志] 方法 saveUser 结束");
    }
}
```

使用方式：

```java
UserService proxy = new UserServiceProxy(new UserServiceImpl());
proxy.getUser(1L);
proxy.saveUser("张三");
```

这段代码能用，但问题很明显：

1. **一个接口就要写一个代理类**。如果有 20 个 Service 接口，就要写 20 个代理类。
2. **每个方法都要手写横切逻辑**。日志、事务、权限……每加一个方法就要在代理类里同步加一份。
3. **接口变更时代理类也要跟着改**。接口加了一个方法，代理类不跟着加就编译不过。

这些问题的本质是：**横切逻辑是相同的，但被分散复制到了每个代理类中**。动态代理的思路就是——能不能在运行时自动生成这些代理类？

## 二、JDK 动态代理：基于接口的运行时生成

JDK 动态代理是 Java 标准库自带的方案，核心是 `java.lang.reflect.Proxy` 类和 `InvocationHandler` 接口。

### 2.1 核心机制

JDK 动态代理的原理可以用一句话概括：**在运行时动态生成一个实现了指定接口的代理类，把所有方法调用转发给 InvocationHandler 处理**。

关键角色：

- **Proxy**：`Proxy.newProxyInstance()` 在运行时生成代理类的字节码并加载到 JVM
- **InvocationHandler**：你实现的调用处理器，所有方法调用都会进入它的 `invoke()` 方法
- **目标对象**：被代理的真实对象，在 handler 中持有引用

### 2.2 代码实现

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class LoggingHandler implements InvocationHandler {
    private final Object target;

    public LoggingHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 调用前：记录日志
        System.out.println("[日志] " + method.getName() + " 开始, 参数: " +
                (args != null ? java.util.Arrays.toString(args) : "无"));

        // 调用目标方法
        Object result = method.invoke(target, args);

        // 调用后：记录返回值
        System.out.println("[日志] " + method.getName() + " 结束, 返回: " + result);
        return result;
    }

    // 通用工厂方法：给任意对象创建代理
    @SuppressWarnings("unchecked")
    public static <T> T create(Object target, Class<T> interfaceType) {
        return (T) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                new Class<?>[]{interfaceType},
                new LoggingHandler(target)
        );
    }
}
```

使用方式：

```java
UserService proxy = LoggingHandler.create(new UserServiceImpl(), UserService.class);
proxy.getUser(1L);
proxy.saveUser("张三");
```

输出：

```
[日志] getUser 开始, 参数: [1]
查询用户: 1
[日志] getUser 结束, 返回: User-1
[日志] saveUser 开始, 参数: [张三]
保存用户: 张三
[日志] saveUser 结束
```

和静态代理对比，最大的变化是：**一个 `LoggingHandler` 可以代理任意接口**。你不需要为 `OrderService`、`ProductService` 各写一个代理类了。

### 2.3 代理类到底长什么样

JDK 在运行时生成的代理类，本质上等价于这样的代码（简化版）：

```java
public final class $Proxy0 implements UserService {
    private InvocationHandler handler;

    @Override
    public String getUser(Long id) {
        // 把方法信息封装成 Method 对象，交给 handler
        Method m = UserService.class.getMethod("getUser", Long.class);
        return (String) handler.invoke(this, m, new Object[]{id});
    }

    @Override
    public void saveUser(String name) {
        Method m = UserService.class.getMethod("saveUser", String.class);
        handler.invoke(this, m, new Object[]{name});
    }
}
```

所有方法调用的最终归宿都是 `handler.invoke()`。你可以在 `invoke()` 里根据 `Method` 对象判断当前调用的方法，做不同的处理。

### 2.4 JDK 动态代理的限制

JDK 动态代理有一个硬性限制：**目标对象必须实现至少一个接口**。

```java
// 如果 UserServiceImpl 没有实现任何接口
public class UserServiceImpl {  // 没有实现接口
    public String getUser(Long id) { ... }
}

// 下面这行会抛异常：Proxy.newProxyInstance 要求传入接口
Proxy.newProxyInstance(...);  // 没有接口可传
```

这是因为 JDK 生成的代理类通过 `implements` 接口来保证类型兼容。如果目标类没有实现接口，JDK 动态代理就无能为力了。

这时就需要 CGLIB。

## 三、CGLIB 动态代理：基于继承的运行时生成

CGLIB（Code Generation Library）是一个第三方字节码生成库。和 JDK 动态代理不同，它不要求目标类实现接口——**它通过生成目标类的子类来实现代理**。

### 3.1 核心机制

CGLIB 的原理是：**在运行时动态生成目标类的子类，在子类中重写方法，插入横切逻辑**。

关键角色：

- **Enhancer**：CGLIB 的核心类，负责生成代理子类
- **MethodInterceptor**：方法拦截器，类似 JDK 的 InvocationHandler
- **目标类**：被代理的类，不需要实现任何接口

### 3.2 代码实现

先引入依赖（Maven）：

```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
```

实现拦截器：

```java
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

public class LoggingInterceptor implements MethodInterceptor {

    @Override
    public Object intercept(Object obj, Method method, Object[] args,
                            MethodProxy methodProxy) throws Throwable {
        // 调用前：记录日志
        System.out.println("[CGLIB日志] " + method.getName() + " 开始, 参数: " +
                (args != null ? java.util.Arrays.toString(args) : "无"));

        // 调用父类（目标类）的方法
        Object result = methodProxy.invokeSuper(obj, args);

        // 调用后：记录返回值
        System.out.println("[CGLIB日志] " + method.getName() + " 结束, 返回: " + result);
        return result;
    }
}
```

创建代理：

```java
import net.sf.cglib.proxy.Enhancer;

public class CglibDemo {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(UserServiceImpl.class);  // 目标类（不需要接口）
        enhancer.setCallback(new LoggingInterceptor());

        UserServiceImpl proxy = (UserServiceImpl) enhancer.create();
        proxy.getUser(1L);
        proxy.saveUser("张三");
    }
}
```

注意看：`enhancer.setSuperclass(UserServiceImpl.class)` 设置的是**类**而不是接口，生成的代理对象可以强转为 `UserServiceImpl` 类型，因为它就是 `UserServiceImpl` 的子类。

### 3.3 CGLIB 的限制

CGLIB 通过继承生成子类，所以有几个限制：

1. **不能代理 final 类**——final 类不能被继承
2. **不能代理 final 方法**——final 方法不能被重写，CGLIB 会跳过这些方法
3. **不能代理 private 方法**——子类无法访问父类的 private 方法

```java
public final class FinalService {  // final 类，CGLIB 无法代理
    public void doSomething() { ... }
}
```

### 3.4 MethodProxy 的作用

CGLIB 拦截器的参数比 JDK 多了一个 `MethodProxy`，这是 CGLIB 性能优于 JDK 的关键之一。

```java
// JDK 动态代理：通过反射调用
method.invoke(target, args);  // 反射调用，有性能开销

// CGLIB：通过 MethodProxy 调用
methodProxy.invokeSuper(obj, args);  // 生成了 FastClass 索引，避免反射
```

CGLIB 会为每个方法生成一个整数索引，调用时通过索引直接定位方法，不需要像 JDK 那样每次都走反射查找。这在频繁调用的场景下性能差距会体现出来。

## 四、JDK vs CGLIB：全面对比

| 维度 | JDK 动态代理 | CGLIB 动态代理 |
|------|-------------|----------------|
| 代理原理 | 实现接口 | 继承目标类生成子类 |
| 前提条件 | 目标类必须实现接口 | 目标类不能是 final |
| 依赖 | JDK 自带，无需额外依赖 | 需引入 cglib 依赖 |
| 方法调用 | 反射调用（Method.invoke） | FastClass 索引，避免反射 |
| 性能 | 创建快，调用稍慢 | 创建慢，调用更快 |
| final 限制 | 不受影响（final 方法在接口中不存在） | final 类/方法无法代理 |
| Spring 默认 | 有接口时使用 | 无接口时使用 |

### 4.1 代理对象的类型差异

```java
// JDK 代理：代理对象是 Proxy 的子类，实现目标接口
// proxy instanceof UserService → true
// proxy instanceof UserServiceImpl → false（代理类不继承 UserServiceImpl）

// CGLIB 代理：代理对象是目标类的子类
// proxy instanceof UserServiceImpl → true（代理类继承 UserServiceImpl）
```

这个差异在实际开发中很重要：如果你的代码强依赖具体类而不是接口，JDK 代理创建的对象可能无法强转为你期望的类型。

## 五、实战应用：Spring AOP 如何选择代理

Spring AOP 是动态代理最经典的应用场景。当你给一个方法加上 `@Transactional`、`@Async`、`@Cacheable` 注解时，Spring 在背后做的事情就是：**用动态代理包装这个 Bean，在方法调用前后插入额外逻辑**。

### 5.1 Spring 的选择策略

Spring 在 5.x 之后默认策略如下：

1. **目标类实现了接口** → 优先使用 JDK 动态代理
2. **目标类没有实现任何接口** → 使用 CGLIB 动态代理
3. **显式配置 `spring.aop.proxy-target-class=true`** → 强制使用 CGLIB

```java
// Spring Boot 2.x 开始默认开启 proxy-target-class=true
// 也就是说 Spring Boot 默认用 CGLIB，即使你有接口
// 这是为了避免前面提到的类型转换问题
```

### 5.2 @Transactional 的背后

当你写下面这段代码时：

```java
@Service
public class OrderService {
    @Transactional
    public void createOrder(Order order) {
        orderRepository.save(order);
        inventoryService.deduct(order.getProductId());
    }
}
```

Spring 实际创建的 Bean 是一个代理对象。调用 `createOrder` 时，执行流程是：

```
调用方 → 代理对象.createOrder(order)
            ↓
         开启事务（TransactionInterceptor）
            ↓
         目标对象.createOrder(order)
            ↓
         提交事务 / 回滚事务
```

这就是为什么 **同类内部方法调用时 `@Transactional` 会失效**——内部调用走的是 `this`，而不是代理对象：

```java
@Service
public class OrderService {
    @Transactional
    public void createOrder(Order order) { ... }

    public void batchCreate(List<Order> orders) {
        // 这里调用 this.createOrder()，不经过代理对象
        // @Transactional 不生效！
        for (Order order : orders) {
            this.createOrder(order);  // 直接调用，绕过了代理
        }
    }
}
```

理解了动态代理的原理，这类"坑"就不再神秘了。

## 六、动手验证：自己写一个简易 AOP

理解了原理之后，我们可以用 JDK 动态代理实现一个最简版的"注解驱动 AOP"：

```java
import java.lang.annotation.*;
import java.lang.reflect.*;

// 1. 定义注解
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface LogExecutionTime {
}

// 2. 定义 InvocationHandler
public class TimerHandler implements InvocationHandler {
    private final Object target;

    public TimerHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 检查方法上是否有 @LogExecutionTime 注解
        if (method.isAnnotationPresent(LogExecutionTime.class)) {
            long start = System.currentTimeMillis();
            Object result = method.invoke(target, args);
            long elapsed = System.currentTimeMillis() - start;
            System.out.println(method.getName() + " 耗时: " + elapsed + "ms");
            return result;
        }
        // 没有注解的方法直接放行
        return method.invoke(target, args);
    }
}

// 3. 使用
public class Demo {
    public static void main(String[] args) {
        UserService proxy = (UserService) Proxy.newProxyInstance(
                UserServiceImpl.class.getClassLoader(),
                UserServiceImpl.class.getInterfaces(),
                new TimerHandler(new UserServiceImpl())
        );
        proxy.getUser(1L);  // 自动计时
    }
}
```

这就是 Spring AOP 的雏形。Spring 的实现更复杂，但核心思想完全一致：**代理对象 + 方法拦截器 + 注解判断**。

## 七、总结

动态代理是 Java 框架设计的基石。很多看似"神奇"的特性——`@Transactional` 自动管事务、`@Async` 异步执行、`@Cacheable` 缓存、MyBatis 的 Mapper 接口——底层都是动态代理。

几个要点回顾：

1. **JDK 动态代理**基于接口，通过 `Proxy.newProxyInstance()` 生成代理类，方法调用通过反射转发给 `InvocationHandler`
2. **CGLIB 动态代理**基于继承，通过 `Enhancer` 生成子类，方法调用通过 `MethodProxy` 拦截
3. **Spring AOP** 根据目标类是否实现接口选择代理方式，Spring Boot 默认强制使用 CGLIB
4. **同类内部方法调用不走代理**，这是 `@Transactional` 等注解失效的常见原因

> 代理模式不是"魔法"，它只是在对象外面套了一层壳。理解了这层壳是怎么生成的、怎么转发调用的，框架的行为就有了可解释的依据。

> Keep learning, keep testing, keep building.
