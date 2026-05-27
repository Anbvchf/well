# IDEA + Tomcat 运行 Java Web 项目的过程，以及与 Spring Boot 的对比

这份文档说明的是：一个传统 Java Web 项目放在 IDEA 中，为什么需要配置某个 Tomcat，IDEA 启动时具体发生了什么，以及它和现在常见的 Spring Boot 启动方式有什么相同和不同。

当前这个目录本身就是一套 Tomcat 运行环境：

```text
C:\Users\Administrator\Desktop\task\tomcat\apache-tomcat-7.0.59_mp_uat
```

它不是普通的项目源码目录，而是一个 Web 容器目录。里面的 `bin`、`conf`、`lib`、`webapps` 等目录共同组成了 Tomcat 的运行环境。

---

## 1. 你的理解是否正确

你的理解整体是对的，可以整理成下面这条主线：

```text
IDEA 中的 Java Web 项目
        |
        v
IDEA 编译项目，并生成可部署的 Web Artifact
        |
        v
IDEA 启动你配置好的那个 Tomcat
        |
        v
Tomcat 读取自己的配置，比如 conf/server.xml、conf/context.xml
        |
        v
Tomcat 加载自己的公共 jar，比如 lib 目录下的 jar
        |
        v
Tomcat 部署 IDEA 编译出来的 war 或 war exploded
        |
        v
Tomcat 读取项目自己的 Web 配置，比如 WEB-INF/web.xml、Servlet 注解、Spring MVC 配置
        |
        v
浏览器访问 URL，Tomcat 根据映射关系把请求交给对应的 Java 代码处理
```

不过这里有一个细节需要更准确一点：

IDEA 本地运行 Tomcat 时，不一定每次都真的生成一个 `.war` 压缩包。更常见的是使用 `war exploded`。

`war exploded` 可以理解成“解压后的 war 目录”，结构类似这样：

```text
xxx_war_exploded/
  index.jsp
  static/
  WEB-INF/
    web.xml
    classes/
    lib/
```

它和真正的 `.war` 文件本质上是同一种 Web 应用，只是一个是压缩包，一个是展开后的目录。IDEA 本地开发时喜欢用 `war exploded`，因为改代码、改 JSP、改静态资源后更新更方便。

---

## 2. 为什么 IDEA 里要配置 Tomcat

普通 Java 项目可以直接运行 `main()` 方法：

```text
IDEA -> java.exe -> 调用 main()
```

但是传统 Java Web 项目通常没有一个直接给你运行的业务 `main()`。它的入口不是你自己写的某个主函数，而是由 Web 容器负责启动。

也就是说，传统 Java Web 项目的运行方式是：

```text
IDEA -> 启动 Tomcat -> Tomcat 加载你的 Web 项目 -> Tomcat 接收 HTTP 请求 -> Tomcat 调用你的代码
```

Tomcat 在这里负责这些事情：

- 开启 HTTP 端口，接收浏览器或前端发来的请求
- 管理 Servlet、Filter、Listener 的生命周期
- 加载 Web 项目的 class 和 jar
- 解析 `web.xml` 或注解
- 管理 JSP 编译
- 管理 Session
- 提供 JNDI、数据源等容器能力
- 把 URL 请求转发给对应的 Servlet 或 Spring MVC Controller

所以 IDEA 里面配置 Tomcat，本质上是告诉 IDEA：

```text
这个项目不是直接 java main 启动，
请用这套 Tomcat 环境来运行它。
```

---

## 3. IDEA 中配置 Tomcat 后，启动时发生了什么

假设你在 IDEA 里配置了这个 Tomcat：

```text
C:\Users\Administrator\Desktop\task\tomcat\apache-tomcat-7.0.59_mp_uat
```

然后在 `Run/Debug Configurations` 里创建了一个 `Tomcat Server -> Local` 的运行配置，并在 `Deployment` 里添加了当前项目的 `war` 或 `war exploded`。

点击 Run 之后，大致会发生下面这些步骤。

---

## 4. 第一步：IDEA 编译你的项目

IDEA 会先编译你的 Java 代码。

如果是 Maven 项目，可能会按照 `pom.xml` 的配置处理依赖、资源文件和编译输出。

编译后的内容通常包括：

```text
target/classes/
target/xxx.war
```

或者 IDEA 自己的输出目录：

```text
out/artifacts/xxx_war_exploded/
```

这些内容会形成一个 Web 应用结构。

典型结构如下：

```text
xxx_war_exploded/
  index.jsp
  static/
  WEB-INF/
    web.xml
    classes/
      com/example/...
    lib/
      项目依赖的 jar
```

其中：

- `WEB-INF/classes` 放你项目编译后的 `.class` 文件
- `WEB-INF/lib` 放你项目自己的依赖 jar
- `WEB-INF/web.xml` 是传统 Web 项目的核心配置文件之一
- JSP、HTML、CSS、JS 等资源会放在 Web 根目录或静态资源目录下

---

## 5. 第二步：IDEA 启动指定的 Tomcat

IDEA 不会随便找一个 Tomcat。它会启动你在 IDEA 中配置的那个 Tomcat。

也就是说，如果某个项目必须配这个目录：

```text
C:\Users\Administrator\Desktop\task\tomcat\apache-tomcat-7.0.59_mp_uat
```

那通常说明这个项目运行时依赖这套 Tomcat 里的配置或公共 jar。

Tomcat 的启动入口通常在：

```text
bin/catalina.bat
bin/startup.bat
```

IDEA 启动 Tomcat 时，本质上也是调用 Tomcat 的启动逻辑，只是 IDEA 会帮你拼好 JVM 参数、classpath、部署路径、调试参数等。

Tomcat 启动时会用到两个很重要的概念：

```text
CATALINA_HOME
CATALINA_BASE
```

简单理解：

- `CATALINA_HOME`：Tomcat 程序本体在哪里
- `CATALINA_BASE`：当前这个 Tomcat 实例的配置、日志、部署目录在哪里

很多时候本地开发里这两个路径是同一个目录。

---

## 6. 第三步：Tomcat 读取自己的配置

Tomcat 启动后，会先读取自己的配置文件。

主要包括：

```text
conf/server.xml
conf/context.xml
conf/web.xml
conf/logging.properties
```

### 6.1 server.xml

`conf/server.xml` 是 Tomcat 的服务器级配置。

当前这个 Tomcat 的 HTTP 端口配置在这里：

```xml
<Connector port="8081" protocol="HTTP/1.1" ... />
```

这表示 Tomcat 启动后会监听：

```text
http://localhost:8081
```

`server.xml` 里常见配置包括：

- Tomcat 关闭端口
- HTTP 端口
- HTTPS 端口
- AJP 端口
- Host 配置
- appBase，也就是 Web 应用默认部署目录
- 是否自动部署 webapps 目录下的应用

例如：

```xml
<Host name="localhost" appBase="webapps"
      unpackWARs="true" autoDeploy="true">
</Host>
```

意思是：

- 主机名是 `localhost`
- 默认部署目录是 `webapps`
- 如果放进去的是 `.war`，可以自动解压
- 可以自动部署

### 6.2 context.xml

`conf/context.xml` 是 Tomcat 的上下文默认配置。

当前这个 Tomcat 的 `context.xml` 里配置了 JNDI 数据源，例如：

```xml
<Resource name="jdbc/chic"
          type="javax.sql.DataSource"
          auth="Container"
          driverClassName="com.mysql.cj.jdbc.Driver"
          ... />
```

这类配置的意思是：

Tomcat 启动时帮你创建一个数据库连接池，并把它注册到 JNDI 环境里。项目代码或 Spring 配置可以通过名字去找它，例如：

```text
java:comp/env/jdbc/chic
```

所以如果项目里配置的是从 JNDI 取数据源，而你换了一个没有这些 `Resource` 配置的 Tomcat，就可能启动失败。

常见错误包括：

```text
javax.naming.NameNotFoundException
Cannot create JDBC driver
No suitable driver
ClassNotFoundException: com.mysql.cj.jdbc.Driver
```

### 6.3 conf/web.xml

`conf/web.xml` 是 Tomcat 的全局 Web 默认配置。

它会给所有部署在这个 Tomcat 上的 Web 应用提供一些默认行为，比如：

- 默认 Servlet
- JSP Servlet
- MIME 类型
- 欢迎页规则

项目自己的 `WEB-INF/web.xml` 优先描述项目自己的行为。

---

## 7. 第四步：Tomcat 加载 lib 里的公共 jar

当前 Tomcat 的公共 jar 在：

```text
lib/
```

这个目录里的 jar 属于 Tomcat 容器级 classpath。

也就是说，这里的 jar 对部署到 Tomcat 上的 Web 应用通常是可见的。

当前这个 Tomcat 的 `lib` 里能看到一些项目运行相关的 jar，例如：

```text
mysql-connector-java-8.0.11.jar
commons-dbcp.jar
commons-pool-1.5.4.jar
itext-4.2.1.jar
iTextAsian.jar
jasperreports-htmlcomponent-5.0.1.jar
AsianFonts-1.0.jar
```

这说明这个 Tomcat 不只是一个纯净 Tomcat，它还承担了一部分项目运行环境的职责。

例如：

- MySQL 驱动放在 Tomcat `lib`，JNDI 数据源才能创建 MySQL 连接
- DBCP 相关 jar 用于数据库连接池
- iText、JasperReports、字体 jar 可能用于报表或 PDF 导出

这也是为什么这个项目可能必须使用“对应的 Tomcat”。

因为项目运行时不只依赖自己的代码，还依赖容器里提前放好的东西。

---

## 8. 第五步：Tomcat 部署 IDEA 生成的 Web 应用

IDEA 编译完成后，会把你的 Web 应用部署到 Tomcat。

部署方式常见有两种：

### 8.1 war 部署

真正生成一个 `.war` 文件，然后交给 Tomcat。

例如：

```text
my-project.war
```

Tomcat 可以把它解压成目录后运行。

### 8.2 war exploded 部署

IDEA 本地开发中更常见。

它不是一个压缩包，而是一个已经展开好的目录：

```text
my-project_war_exploded/
```

优点是：

- 启动更方便
- 改 JSP 或静态资源后更容易热更新
- IDEA 可以更快地同步编译结果

所以你可以这样理解：

```text
生产环境常见：打成 war 包，放到 Tomcat
本地 IDEA 常见：生成 war exploded，直接让 Tomcat 加载这个展开目录
```

但两者的 Web 应用结构和运行原理基本一致。

---

## 9. 第六步：Tomcat 读取项目自己的配置

Tomcat 部署你的项目后，会开始读取项目自己的 Web 配置。

传统 Java Web 项目里最重要的是：

```text
WEB-INF/web.xml
```

这里面通常会配置：

- Servlet
- Filter
- Listener
- Spring 的 ContextLoaderListener
- Spring MVC 的 DispatcherServlet
- URL 映射
- 欢迎页
- 错误页

例如传统 Servlet 项目可能有：

```xml
<servlet>
    <servlet-name>HelloServlet</servlet-name>
    <servlet-class>com.example.HelloServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>HelloServlet</servlet-name>
    <url-pattern>/hello</url-pattern>
</servlet-mapping>
```

这表示：

```text
访问 /hello
    |
    v
Tomcat 调用 com.example.HelloServlet
```

如果是 Spring MVC 项目，`web.xml` 里通常会配置一个核心 Servlet：

```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

这表示：

```text
大部分请求先交给 DispatcherServlet
```

然后 Spring MVC 再根据 Controller 上的注解决定具体调用哪个方法：

```java
@Controller
public class UserController {

    @RequestMapping("/user/list")
    public String list() {
        return "user/list";
    }
}
```

请求过程就是：

```text
浏览器访问 /user/list
        |
        v
Tomcat 收到请求
        |
        v
Tomcat 根据 web.xml 发现这个请求应该交给 DispatcherServlet
        |
        v
DispatcherServlet 进入 Spring MVC
        |
        v
Spring MVC 找到 @RequestMapping("/user/list")
        |
        v
调用 UserController.list()
```

所以你说的“接口对应着什么”，可以更准确地分成两层：

第一层是 Tomcat 层：

```text
URL -> Servlet
```

第二层是 Spring MVC 层：

```text
URL -> DispatcherServlet -> Controller 方法
```

如果项目不是 Spring MVC，而是原生 Servlet，那么就是：

```text
URL -> 某个 Servlet
```

如果项目用了 Spring MVC，那么通常是：

```text
URL -> DispatcherServlet -> Controller
```

---

## 10. 第七步：浏览器访问项目

当前这个 Tomcat 的 HTTP 端口是 `8081`。

所以启动后，访问地址一般是：

```text
http://localhost:8081/项目路径
```

这个“项目路径”也叫 `context path`。

如果 IDEA 里 Deployment 配的是：

```text
/myplat
```

那么访问地址类似：

```text
http://localhost:8081/myplat
```

如果配置的是：

```text
/
```

那么访问地址就是：

```text
http://localhost:8081/
```

---

## 11. 传统 Tomcat Web 项目的完整运行链路

可以把整个过程串起来看：

```text
1. 你点击 IDEA 的 Run

2. IDEA 编译 Java 源码

3. IDEA 生成 war 或 war exploded

4. IDEA 启动你配置的 Tomcat

5. Tomcat 读取 conf/server.xml
   确定端口、Host、部署目录等

6. Tomcat 读取 conf/context.xml
   创建 JNDI 数据源等容器资源

7. Tomcat 加载 lib 下的公共 jar
   比如 MySQL 驱动、连接池、报表 jar 等

8. Tomcat 部署 IDEA 生成的 Web 应用

9. Tomcat 加载项目的 WEB-INF/classes 和 WEB-INF/lib

10. Tomcat 读取项目的 WEB-INF/web.xml、Servlet 注解、Spring 配置

11. Tomcat 创建 Servlet、Filter、Listener

12. 如果是 Spring MVC，Tomcat 创建 DispatcherServlet，Spring 容器开始加载 Controller、Service、Dao 等 Bean

13. 浏览器访问 http://localhost:8081/xxx

14. Tomcat 接收请求，根据 URL 映射找到 Servlet

15. 如果请求进入 DispatcherServlet，则由 Spring MVC 找到对应 Controller 方法

16. Controller 调用 Service、Dao、数据库等

17. 返回 JSP、JSON、HTML 或其他响应

18. Tomcat 把响应返回给浏览器
```

---

## 12. 为什么某个项目要对应某个 Tomcat

理论上，一个标准的 Java Web 项目可以部署到任意兼容版本的 Tomcat 上。

但实际项目里经常不是这么理想。

某个项目可能依赖特定 Tomcat 的这些内容：

- 特定的 Tomcat 版本，比如 Tomcat 7
- 特定端口，比如当前的 `8081`
- `conf/context.xml` 里的 JNDI 数据源
- `conf/server.xml` 里的 Host、Connector、编码配置
- `lib` 里的数据库驱动
- `lib` 里的报表、PDF、字体、工具 jar
- 特定 JVM 参数
- 特定日志配置
- 特定文件路径

因此，这个项目在 IDEA 里要配置“对应的 Tomcat”。

换一个干净 Tomcat 后，可能缺少：

```text
JNDI 数据源
数据库驱动
公共 jar
端口配置
编码配置
项目需要的容器参数
```

于是项目就可能启动失败，或者启动成功但访问时报错。

---

## 13. Spring Boot 的启动方式

现在很多 Java 项目会使用 Spring Boot。

Spring Boot 的典型启动方式和传统 Tomcat Web 项目不一样。

传统 Java Web 项目通常是：

```text
外部 Tomcat 启动
        |
        v
Tomcat 加载你的 war
```

Spring Boot 常见方式是：

```text
你的 Spring Boot 应用启动
        |
        v
应用内部启动嵌入式 Tomcat
```

Spring Boot 项目通常会有一个启动类：

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

这时候启动入口又回到了普通 Java 的 `main()`。

也就是说：

```text
IDEA -> java.exe -> main() -> SpringApplication.run() -> 启动 Spring 容器 -> 启动内嵌 Tomcat
```

Spring Boot 默认会把 Tomcat 作为项目依赖带进来。

例如 Maven 里常见：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

这个 starter 默认包含：

```text
Spring MVC
Jackson
Validation
嵌入式 Tomcat
```

所以 Spring Boot 项目一般不需要你在 IDEA 里手动配置外部 Tomcat。

你直接运行启动类即可。

---

## 14. Spring Boot 启动时发生了什么

Spring Boot 的典型启动链路是：

```text
1. 你点击 IDEA 中启动类的 Run

2. IDEA 编译项目

3. IDEA 执行 main() 方法

4. main() 调用 SpringApplication.run()

5. Spring Boot 创建 Spring ApplicationContext

6. Spring Boot 读取 application.properties 或 application.yml

7. Spring Boot 根据自动配置机制创建各种 Bean

8. 如果引入了 spring-boot-starter-web，Spring Boot 创建并启动嵌入式 Tomcat

9. Spring Boot 注册 DispatcherServlet

10. Spring MVC 扫描 Controller

11. 嵌入式 Tomcat 监听端口，比如 8080

12. 浏览器访问 http://localhost:8080/xxx

13. 请求进入嵌入式 Tomcat

14. 请求进入 DispatcherServlet

15. Spring MVC 找到 Controller 方法

16. 返回 JSON、HTML 或其他响应
```

Spring Boot 里很多传统 Tomcat 的配置，会被放到：

```text
application.properties
application.yml
```

例如端口：

```properties
server.port=8081
```

项目路径：

```properties
server.servlet.context-path=/myplat
```

数据库：

```properties
spring.datasource.url=jdbc:mysql://...
spring.datasource.username=...
spring.datasource.password=...
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

也就是说，Spring Boot 把很多原来放在外部 Tomcat 里的东西，转移到了项目自己的配置文件里。

---

## 15. 传统 Tomcat 项目和 Spring Boot 项目的相同点

两者底层有很多东西是一样的。

### 15.1 都可以使用 Tomcat

传统项目使用的是外部 Tomcat：

```text
先有 Tomcat，再部署项目
```

Spring Boot 默认使用嵌入式 Tomcat：

```text
先启动项目，项目内部启动 Tomcat
```

但本质上都可以是 Tomcat 在接收 HTTP 请求。

### 15.2 都可以使用 Spring MVC

传统项目可以通过 `web.xml` 配置 `DispatcherServlet`。

Spring Boot 会自动注册 `DispatcherServlet`。

但请求进入 Spring MVC 后，Controller 的处理方式很像：

```java
@RestController
public class UserController {

    @GetMapping("/user/list")
    public List<User> list() {
        return userService.list();
    }
}
```

### 15.3 都是 URL 映射到 Java 方法

不管是哪种方式，最终目标都是：

```text
HTTP 请求 -> URL 匹配 -> Java 代码处理 -> 返回响应
```

### 15.4 都需要依赖管理

传统项目可能把一部分 jar 放在：

```text
Tomcat/lib
```

也可能放在：

```text
WEB-INF/lib
```

Spring Boot 通常通过 Maven 或 Gradle 管理依赖，并打进最终 jar 或 war。

### 15.5 都需要配置端口、数据库、日志等

只是配置位置不同。

传统项目常见位置：

```text
Tomcat/conf/server.xml
Tomcat/conf/context.xml
项目/WEB-INF/web.xml
Spring XML 配置
```

Spring Boot 常见位置：

```text
application.properties
application.yml
Java Config
注解
```

---

## 16. 传统 Tomcat 项目和 Spring Boot 项目的不同点

下面是重点对比。

| 对比项 | 传统 Java Web + 外部 Tomcat | Spring Boot |
| --- | --- | --- |
| 启动入口 | Tomcat 的启动脚本 | Java `main()` 方法 |
| IDEA 运行方式 | 配置 Tomcat Server | 直接运行启动类 |
| 部署产物 | 通常是 war 或 war exploded | 通常是可执行 jar，也可以是 war |
| Tomcat 位置 | 项目外部单独安装 | 默认嵌入在项目依赖里 |
| 端口配置 | `Tomcat/conf/server.xml` | `application.properties` 或 `application.yml` |
| 数据源配置 | 可能在 `Tomcat/conf/context.xml` 里配置 JNDI | 通常在 `application.yml` 中配置 `spring.datasource` |
| Servlet 配置 | 常见于 `WEB-INF/web.xml` | 多数自动配置，也可以 Java Config |
| DispatcherServlet | 通常手动在 `web.xml` 中配置 | Spring Boot 自动注册 |
| 依赖位置 | `WEB-INF/lib` 加 `Tomcat/lib` | Maven/Gradle 统一管理 |
| 环境耦合 | 容易依赖某台 Tomcat 的配置 | 配置更多随项目走 |
| 部署方式 | 把 war 放进 Tomcat | `java -jar xxx.jar` |
| 运维方式 | 先维护 Tomcat，再部署多个 war | 每个服务自己带运行环境 |
| 适合场景 | 老项目、传统单体、多应用共用 Tomcat | 微服务、新项目、独立部署 |

---

## 17. 两种方式的核心差异

最核心的差异可以用一句话概括：

传统 Tomcat 项目是：

```text
容器启动项目
```

Spring Boot 项目是：

```text
项目启动容器
```

传统方式：

```text
Tomcat 是主角
项目是被部署进去的应用
```

Spring Boot 方式：

```text
Spring Boot 应用是主角
Tomcat 是应用内部启动的一个组件
```

---

## 18. 传统项目的请求流转

传统外部 Tomcat 项目：

```text
浏览器
  |
  v
外部 Tomcat 的 Connector，例如 8081
  |
  v
Tomcat Engine / Host / Context
  |
  v
项目的 Filter 链
  |
  v
Servlet
  |
  v
如果是 Spring MVC，则进入 DispatcherServlet
  |
  v
Controller
  |
  v
Service
  |
  v
Dao / Mapper
  |
  v
数据库
```

---

## 19. Spring Boot 项目的请求流转

Spring Boot 内嵌 Tomcat 项目：

```text
浏览器
  |
  v
Spring Boot 内嵌 Tomcat，例如 8080 或 8081
  |
  v
Filter 链
  |
  v
DispatcherServlet
  |
  v
Controller
  |
  v
Service
  |
  v
Dao / Mapper
  |
  v
数据库
```

可以看到，请求进入 Spring MVC 之后，两者非常像。

不同主要在启动、部署、配置管理这些地方。

---

## 20. 一个形象的对比

传统外部 Tomcat：

```text
Tomcat 像一个酒店。
你的 Web 项目像住进去的客人。
酒店先开门，然后客人入住。
```

Spring Boot：

```text
你的应用自己带了一套小型酒店。
应用启动的时候，酒店也一起开门。
```

从工程角度说就是：

```text
传统方式：外部容器 + 应用
Spring Boot：应用 + 内嵌容器
```

---

## 21. Spring Boot 也能打成 war 放到外部 Tomcat 吗

可以。

Spring Boot 并不是只能打 jar。

它也可以打成 war，然后部署到外部 Tomcat。

这种方式一般需要：

1. 修改打包方式为 `war`
2. 排除或 provided 内嵌 Tomcat
3. 启动类继承 `SpringBootServletInitializer`

示例：

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Application.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

这种模式下，它又接近传统项目：

```text
外部 Tomcat -> 加载 Spring Boot war
```

不过现在更常见的 Spring Boot 部署方式还是：

```text
java -jar xxx.jar
```

---

## 22. 回到你当前这个 Tomcat

当前这个 Tomcat 有几个明显特点：

1. HTTP 端口是 `8081`
2. AJP 端口配置成了 `8010`
3. `conf/context.xml` 中配置了 JNDI 数据源
4. `lib` 中放了 MySQL 驱动、连接池、报表、字体等 jar
5. 它应该是为某个具体项目或某个环境准备过的 Tomcat

所以这个项目在 IDEA 里要选择这个 Tomcat，是合理的。

更准确地说：

```text
不是 Java 项目天然必须绑定这个 Tomcat，
而是这个项目运行时依赖这个 Tomcat 里的配置和公共依赖。
```

如果要把它迁移成更现代的 Spring Boot 方式，通常需要把这些外部 Tomcat 配置逐步迁移进项目：

- 把端口迁移到 `server.port`
- 把 context path 迁移到 `server.servlet.context-path`
- 把 JNDI 数据源改成 Spring Boot 的 `spring.datasource`
- 把 Tomcat/lib 里的业务 jar 改成 Maven/Gradle 依赖
- 把 `web.xml` 中的 Servlet、Filter、Listener 改成注解或 Java Config
- 把 Spring XML 配置逐步改成 Java Config 或 Boot 自动配置

---

## 23. 最后总结

你的理解可以总结为：

```text
IDEA 负责：
  编译项目
  生成 war 或 war exploded
  启动指定 Tomcat
  把项目部署到 Tomcat

Tomcat 负责：
  读取自己的 server.xml、context.xml
  加载 Tomcat/lib 公共 jar
  创建端口、数据源、容器环境
  加载并运行你的 Web 应用
  根据 URL 映射调用 Servlet 或 Spring MVC

项目负责：
  提供 WEB-INF/web.xml、Controller、Service、Dao、JSP 等业务代码和配置
```

传统外部 Tomcat 的核心是：

```text
Tomcat 启动项目
```

Spring Boot 的核心是：

```text
项目启动 Tomcat
```

这就是两种模式最重要的区别。

---

## 24. 补充问题与回答：Spring Boot 内嵌 Tomcat 到底怎么运行

这一节专门回答读完前面内容之后容易产生的几个问题。

---

### 问题 1：Spring Boot 是内置了 Tomcat 对吧？

回答：通常是的。

如果 Spring Boot 项目引入了这个依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

那么默认情况下，它会引入内嵌 Tomcat。

也就是说，Spring Boot 项目不需要你单独下载一个 Tomcat，然后在 IDEA 里配置 `Tomcat Server -> Local`。

传统外部 Tomcat 是：

```text
先安装 Tomcat
再把项目 war 部署进去
```

Spring Boot 默认是：

```text
项目自己带着 Tomcat 相关 jar
启动项目的时候，顺便把内嵌 Tomcat 启动起来
```

不过要注意一点：

Spring Boot 默认用 Tomcat，但不是只能用 Tomcat。

它也可以换成 Jetty 或 Undertow。只是大多数 Spring Boot Web 项目默认就是内嵌 Tomcat。

---

### 问题 2：启动 Spring Boot 项目的时候，需要把代码放进内嵌 Tomcat 中吗？

回答：不需要手动把代码放进内嵌 Tomcat。

这是 Spring Boot 和传统外部 Tomcat 最大的区别之一。

传统外部 Tomcat 的方式是：

```text
Tomcat 是一个已经存在的外部程序
你的项目要被打成 war 或 war exploded
然后部署到 Tomcat 里面
```

Spring Boot 的方式是：

```text
你的项目本身就是一个 Java 程序
Tomcat 是这个 Java 程序依赖的一部分
启动你的 main 方法时，Tomcat 跟着一起启动
```

Spring Boot 项目一般有这样的启动类：

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

你运行的是这个 `main()` 方法。

启动过程可以理解成：

```text
IDEA 点击 Run
    |
    v
java.exe 启动当前 Spring Boot 应用
    |
    v
执行 main()
    |
    v
SpringApplication.run()
    |
    v
创建 Spring 容器
    |
    v
创建并启动内嵌 Tomcat
    |
    v
把 DispatcherServlet、Filter、Listener 等注册到内嵌 Tomcat
    |
    v
Tomcat 开始监听端口
```

所以 Spring Boot 不会像传统 Tomcat 那样，把你的项目复制到：

```text
Tomcat/webapps/
```

也不会要求你在本机单独准备一个：

```text
apache-tomcat-xxx/
```

Spring Boot 项目打包后，通常是一个可执行 jar：

```text
my-project.jar
```

这个 jar 里面包含：

```text
你的业务代码
你的配置文件
你的依赖 jar
Spring Boot 启动相关代码
内嵌 Tomcat 相关 jar
```

然后通过下面命令就能启动：

```bash
java -jar my-project.jar
```

---

### 问题 3：那 Spring Boot 的代码是不是不在 Tomcat 中运行？

回答：这句话要分两层理解。

如果你说的是：

```text
代码是不是不放在外部 Tomcat 安装目录里面？
```

那答案是：对。

Spring Boot 的代码不需要放到外部 Tomcat 的 `webapps` 目录中。

但是如果你说的是：

```text
请求是不是完全绕过 Tomcat，直接进我的 Controller？
```

那答案是：不是。

Spring Boot Web 项目默认仍然是通过内嵌 Tomcat 接收 HTTP 请求的。

更准确的说法是：

```text
你的业务代码和内嵌 Tomcat 运行在同一个 Java 进程、同一个 JVM 里面。
Tomcat 不再是外部单独安装的服务器，而是你的 Spring Boot 程序内部的一个组件。
```

可以这样理解：

```text
外部 Tomcat 模式：

Tomcat 进程
  |
  |-- 加载你的 war
  |-- 调用你的代码
```

```text
Spring Boot 模式：

Spring Boot 进程
  |
  |-- 启动 Spring
  |-- 启动内嵌 Tomcat
  |-- Tomcat 接收请求
  |-- Tomcat 调用 Spring MVC
  |-- Spring MVC 调用你的 Controller
```

所以不是“代码完全不在 Tomcat 中运行”，而是：

```text
代码不部署到外部 Tomcat 目录里；
但是 HTTP 请求仍然先由内嵌 Tomcat 接收，再由 Tomcat 调用 Spring MVC 和你的业务代码。
```

---

### 问题 4：Spring Boot 是中心，还是 Tomcat 是中心？

回答：在 Spring Boot 模式下，Spring Boot 应用是中心。

传统外部 Tomcat 模式是：

```text
Tomcat 是中心
项目是被部署进去的应用
```

Spring Boot 模式是：

```text
Spring Boot 应用是中心
Tomcat 是应用内部启动的 Web 服务器组件
```

也就是前面总结的：

```text
传统外部 Tomcat：Tomcat 启动项目
Spring Boot：项目启动 Tomcat
```

但是从处理 HTTP 请求的角度看，Tomcat 仍然很重要。

因为浏览器访问项目时，最先接住请求的还是内嵌 Tomcat。

---

### 问题 5：Spring Boot 的端口是配置在 Tomcat 里吗？

回答：本质上端口最终还是配置到内嵌 Tomcat 上，但配置位置不再是外部 Tomcat 的 `server.xml`。

传统外部 Tomcat 的端口通常写在：

```text
Tomcat/conf/server.xml
```

例如：

```xml
<Connector port="8081" protocol="HTTP/1.1" />
```

Spring Boot 的端口通常写在项目自己的配置文件里：

```text
application.properties
application.yml
```

例如 `application.properties`：

```properties
server.port=8081
```

或者 `application.yml`：

```yaml
server:
  port: 8081
```

Spring Boot 启动时会读取这个配置，然后把端口设置到内嵌 Tomcat 的 Connector 上。

所以可以这样理解：

```text
传统 Tomcat：
  端口写在外部 Tomcat 的 server.xml

Spring Boot：
  端口写在项目的 application.yml / application.properties
  Spring Boot 启动时把这个端口应用到内嵌 Tomcat
```

如果不配置，Spring Boot 默认端口通常是：

```text
8080
```

如果同一台电脑上已经有一个程序占用了 `8080`，再启动另一个也使用 `8080` 的 Spring Boot 项目，就会报端口占用错误。

---

### 问题 6：请求打到端口后，是不是先经过 Tomcat，再经过过滤器，然后进入 Spring Boot 程序？

回答：整体理解是对的，但可以说得更精确一点。

请求流转大概是：

```text
浏览器 / 前端 / Postman
    |
    v
访问 http://localhost:8081/user/list
    |
    v
内嵌 Tomcat 的 Connector 接收到 HTTP 请求
    |
    v
Tomcat 把请求封装成 HttpServletRequest / HttpServletResponse
    |
    v
进入 Filter 过滤器链
    |
    v
进入 DispatcherServlet
    |
    v
Spring MVC 根据 URL 找到 Controller 方法
    |
    v
Controller 调用 Service、Mapper、数据库等
    |
    v
返回结果
```

更完整一点：

```text
请求进入：

客户端
  -> Tomcat Connector
  -> Tomcat Servlet 容器
  -> Filter 链
  -> DispatcherServlet
  -> Spring MVC HandlerMapping
  -> Controller
  -> Service
  -> Dao / Mapper
  -> 数据库
```

返回出去：

```text
数据库
  -> Dao / Mapper
  -> Service
  -> Controller
  -> DispatcherServlet
  -> Filter 链返回阶段
  -> Tomcat
  -> 客户端
```

所以你说的这句话是成立的：

```text
所有请求先打到 Tomcat 监听的端口，
Tomcat 接住请求，
然后经过过滤器，
再进入 Spring Boot 里的 Spring MVC 程序。
```

只不过需要补一句：

这里的 Tomcat 是内嵌在 Spring Boot 应用里的 Tomcat，不是外面单独安装的那个 Tomcat。

---

### 问题 7：过滤器 Filter 是干什么的？

回答：Filter 是 Servlet 规范里的组件，作用是在请求真正进入 Servlet 或 Controller 之前，以及响应真正返回客户端之前，做一些统一处理。

Filter 像一个“关卡”。

请求进来时，它可以先检查、修改、记录、拦截。

响应出去时，它也可以再处理一次。

Filter 的典型作用包括：

1. 登录校验

   判断用户有没有登录。如果没登录，直接拦截，不让请求继续进 Controller。

2. 权限校验

   判断当前用户有没有权限访问这个接口。

3. 编码处理

   比如统一设置请求和响应的字符编码，避免中文乱码。

4. 跨域处理

   处理 CORS，让前端项目可以访问后端接口。

5. 日志记录

   记录请求 URL、请求方法、耗时、IP、用户信息等。

6. 安全处理

   比如防止 XSS、处理 Token、校验签名等。

7. 链路追踪

   给每个请求生成 traceId，方便排查日志。

8. 统一包装请求或响应

   比如读取请求体、包装响应内容、做压缩或特殊格式处理。

Filter 的核心方法可以简单理解成：

```java
public void doFilter(ServletRequest request,
                     ServletResponse response,
                     FilterChain chain) {

    // 1. 请求进入 Controller 之前做点事情

    chain.doFilter(request, response);

    // 2. Controller 处理完，响应返回客户端之前再做点事情
}
```

关键是这行：

```java
chain.doFilter(request, response);
```

它的意思是：

```text
放行，让请求继续往后走。
```

如果 Filter 不调用 `chain.doFilter()`，请求就不会继续进入后面的 Filter、Servlet 或 Controller。

例如登录拦截可以这样理解：

```text
请求进来
  |
  v
登录 Filter 检查用户是否登录
  |
  |-- 没登录：直接返回 401 或跳转登录页
  |
  |-- 已登录：chain.doFilter() 放行
                      |
                      v
                  继续进入 Controller
```

---

### 问题 8：Filter 和 Spring MVC 的 Interceptor 是一回事吗？

回答：不是一回事，但作用有点像。

Filter 是 Servlet 规范里的东西，位置更靠外。

Interceptor 是 Spring MVC 里的东西，位置更靠里。

大概顺序是：

```text
Tomcat
  |
  v
Filter
  |
  v
DispatcherServlet
  |
  v
Spring MVC Interceptor
  |
  v
Controller
```

也就是说：

```text
Filter 在进入 Spring MVC 之前执行。
Interceptor 在进入 Spring MVC 之后、调用 Controller 之前执行。
```

常见理解：

```text
Filter 更偏底层、容器级、通用 Web 请求处理。
Interceptor 更偏 Spring MVC 内部、业务接口拦截。
```

Spring Security 很多核心逻辑就是基于 Filter 链完成的。

---

### 问题 9：返回结果也是从 Tomcat 发出去的吗？

回答：是的。

请求是 Tomcat 接进来的，响应最终也是 Tomcat 发出去的。

过程大概是：

```text
Controller 返回数据
    |
    v
Spring MVC 处理返回值
    |
    v
如果是 @ResponseBody 或 @RestController，就转换成 JSON
    |
    v
写入 HttpServletResponse
    |
    v
经过 Filter 返回阶段
    |
    v
Tomcat 把 HTTP 响应写回客户端连接
    |
    v
浏览器 / 前端 / Postman 收到响应
```

所以可以理解成：

```text
Tomcat 负责网络层的接收和返回；
Spring Boot / Spring MVC 负责业务分发和业务处理；
Controller / Service / Dao 负责具体业务逻辑。
```

---

### 问题 10：一台电脑可以启动多个 Spring Boot 项目吗？

回答：可以。

前提是：

1. 机器性能足够
2. 每个项目使用不同的端口
3. 数据库、Redis、文件路径、日志路径等外部资源不要互相冲突

例如同一台电脑上可以这样启动：

```text
项目 A：server.port=8080
项目 B：server.port=8081
项目 C：server.port=8082
```

访问时分别是：

```text
http://localhost:8080
http://localhost:8081
http://localhost:8082
```

每个 Spring Boot 项目通常都是一个独立的 Java 进程：

```text
java.exe 进程 A -> Spring Boot 项目 A -> 内嵌 Tomcat A -> 监听 8080
java.exe 进程 B -> Spring Boot 项目 B -> 内嵌 Tomcat B -> 监听 8081
java.exe 进程 C -> Spring Boot 项目 C -> 内嵌 Tomcat C -> 监听 8082
```

只要端口不冲突，就可以同时运行。

如果两个项目都配置：

```properties
server.port=8080
```

那么第二个启动时就会失败，因为同一个 IP 的同一个端口同一时间只能被一个进程监听。

常见错误类似：

```text
Port 8080 was already in use
Address already in use
```

---

### 问题 11：多个 Spring Boot 项目为什么只要端口不一样就能同时跑？

回答：因为操作系统会根据端口把请求分发给不同进程。

例如：

```text
http://localhost:8080/order/list
```

操作系统会把这个请求交给监听 `8080` 的那个 Java 进程。

```text
http://localhost:8081/user/list
```

操作系统会把这个请求交给监听 `8081` 的那个 Java 进程。

它们之间是独立的。

可以理解成：

```text
同一台电脑
  |
  |-- 8080 -> 项目 A
  |-- 8081 -> 项目 B
  |-- 8082 -> 项目 C
```

所以本地开发时，经常会同时启动多个服务：

```text
用户服务：8081
订单服务：8082
支付服务：8083
网关服务：8080
```

前端或网关通过不同端口访问不同服务。

---

### 问题 12：Spring Boot 和外部 Tomcat 在请求处理上的最终对比

传统外部 Tomcat：

```text
浏览器
  |
  v
外部 Tomcat 监听端口
  |
  v
外部 Tomcat 加载部署进去的 war
  |
  v
Filter
  |
  v
DispatcherServlet
  |
  v
Controller
  |
  v
返回响应给外部 Tomcat
  |
  v
外部 Tomcat 返回给浏览器
```

Spring Boot 内嵌 Tomcat：

```text
浏览器
  |
  v
Spring Boot 进程里的内嵌 Tomcat 监听端口
  |
  v
Filter
  |
  v
DispatcherServlet
  |
  v
Controller
  |
  v
返回响应给内嵌 Tomcat
  |
  v
内嵌 Tomcat 返回给浏览器
```

两者进入 `DispatcherServlet` 之后非常像。

核心区别仍然是：

```text
外部 Tomcat：
  先有 Tomcat，再部署项目。

Spring Boot：
  先启动项目，项目内部启动 Tomcat。
```

---

### 问题 13：一句话总结你的理解

可以这样总结：

```text
Spring Boot Web 项目默认内置 Tomcat。
启动 Spring Boot 时，不需要把代码放到外部 Tomcat 里。
Spring Boot 应用启动后，会在同一个 JVM 里启动内嵌 Tomcat。
请求先到内嵌 Tomcat 监听的端口，再经过 Filter 链，再进入 DispatcherServlet 和 Controller。
响应处理完成后，也由 Tomcat 写回给客户端。
一台电脑可以同时启动多个 Spring Boot 项目，只要端口和其他资源不冲突即可。
```
