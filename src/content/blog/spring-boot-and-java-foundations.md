---
title: "开始学 Spring Boot：从一次登录提交，重新补 Java 的地基"
description: "结合三个本地练习，记录 Spring Boot 路由、表单提交、JSON 请求的第一轮实践，以及同步补强 Java 基础的计划。"
pubDate: 2026-07-28
tags:
  - Spring Boot
  - Java
  - 学习记录
  - 后端开发
---

最近开始系统接触 Spring Boot。它给人的第一印象很直接：新建项目、写一个 Controller、启动应用，再在浏览器里访问一个路径，页面很快就能跑起来。

但把几个小练习写下来后，我反而更清楚地意识到：Spring Boot 不是绕过 Java 基础的捷径。注解能帮我减少配置，Lombok 能减少样板代码，可对象、字符串比较、方法参数、异常和集合这些东西，最终都还是要靠 Java 本身来理解。

所以这段时间的学习节奏是两条线并行：一边用 Spring Boot 建立对后端请求链路的直觉，一边把 Java 进阶中掌握不牢的基础慢慢补回来。

> 不急着追求做出多大的项目，先把“一次请求是怎样进来、被处理、再返回页面或数据”的过程走明白。

## 从页面路由开始：Controller 和模板并不神秘

最简单的练习是 `submit-simple`：访问 `/login`，Controller 返回 `login`；访问 `/page`，Controller 返回 `page`。对应的 HTML 模板放在 `resources/templates` 下。

```java
@GetMapping("/login")
public String goLogin() {
    return "login";
}

@GetMapping("/page")
public String goPage() {
    return "page";
}
```

这段代码不复杂，却让我第一次把几个概念连在了一起：浏览器发出 URL 请求，Spring 根据 `@GetMapping` 找到方法，方法返回模板名，随后由模板引擎定位并渲染页面。

以前看见“路由”时，我容易只记住“写一个注解”。真正自己跑起来后，才发现更重要的是路径、HTTP 方法、方法返回值和模板位置之间的对应关系。任何一环对不上，页面就不会按照预期出现。

## 从一次登录提交，理解 GET、POST 与参数绑定

在 `submit` 练习中，我又往前走了一步：`GET /login` 负责显示登录页，表单提交时通过 `POST /login` 将用户名和密码交给后端；验证成功后返回模拟数据页，失败则重定向回登录页并带上错误标记。

```java
@PostMapping("/login")
public String login(@RequestParam String username,
                    @RequestParam String password) {
    if ("admin".equals(username) && "123456".equals(password)) {
        return "mock-data";
    }
    return "redirect:/login?error=1";
}
```

这里有两个很具体的收获。

第一，`@RequestParam` 不是“魔法接收参数”，而是把表单中同名的字段绑定到 Java 方法参数上。前端 `input` 的 `name`、请求方法和 Controller 中的参数名，必须对得上。

第二，代码中使用了 `"admin".equals(username)`，而不是 `username.equals("admin")`。两种写法在正常输入时结果相同，但前者即使 `username` 是 `null` 也不会抛出空指针异常。这是一个很小的细节，却提醒我：Java 的基础并不只在考试题里，写每一段业务判断时都在起作用。

当然，目前的账号密码写死在代码里，只用于学习流程。真实项目还需要密码哈希、身份认证、权限控制、数据库持久化和更完整的异常处理；这些不能因为“页面已经跳转成功”就被忽略。

## 从表单到 JSON：前后端开始传递对象

`yoga2026` 这个练习让我接触到了另一种更接近接口开发的请求方式。前端使用 `fetch` 发送 JSON：

```js
fetch('http://localhost:8080/getInfo', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ username: 'admin', password: '123456' })
});
```

后端则使用 `@RequestBody` 接收 JSON，并把它映射为 `LoginModel` 对象；再通过 `@ResponseBody` 将对象作为 JSON 响应返回：

```java
@PostMapping(value = "/getInfo", produces = "application/json")
@ResponseBody
public LoginModel getInfo(@RequestBody LoginModel user) {
    user.setUsername("aaa");
    user.setPassword("bbb");
    return user;
}
```

这一步让我对“对象”有了更具体的感受。`LoginModel` 不是抽象概念，而是前后端约定的数据形状：字段名、类型和序列化结果都需要保持一致。Lombok 的 `@Data` 确实省去了 getter、setter 等重复代码，但也提醒我不能只知道“加一个注解就能用”，还要知道对象里究竟有哪些属性、方法是怎样生成和调用的。

## Spring Boot 在推进，Java 基础也不能欠账

回头看这些练习，我觉得当前最需要补的不是更多框架名词，而是能支撑后续开发的 Java 基础。我的复习会先聚焦在下面几项：

1. **对象与引用**：弄清楚对象在方法之间如何传递，修改对象字段会带来什么影响。
2. **字符串、空值与条件判断**：继续把 `equals`、`null`、分支逻辑这些高频细节写扎实。
3. **集合与泛型**：后面无论返回列表、处理查询结果还是做数据转换，都离不开 `List`、`Map` 和类型约束。
4. **异常与调试**：不只看报错文本，还要顺着请求路径、参数和调用栈定位问题。
5. **线程与 JVM 基础**：这部分暂时不急着一次学完，但会在前面几项稳定后继续补齐，不再只停留在名词记忆上。

我不想把“Java 进阶基础薄弱”当成一个模糊的压力，而是把它拆成能跟着项目验证的小问题。每多写一个接口、多走通一次请求，就回头确认其中用了哪些 Java 知识；哪些还说不清，就记录下来再专门复习。

## 下一步：让链路再完整一点

接下来，我准备在现有练习上继续往前推进：先完善表单校验和错误提示，再尝试把 Controller 中的处理逻辑逐步拆到更清晰的结构中；同时复习集合、泛型和异常处理，让“能跑”慢慢变成“知道为什么这样写”。

这段学习也再次验证了一句我想提醒自己的话：光看到问题还不够，必须想办法落实。文档可以看，教程可以跟，但一定要自己动手敲、边敲边记，最后再用运行结果检查理解是否真的落地。

> Keep learning, keep testing, keep building.
