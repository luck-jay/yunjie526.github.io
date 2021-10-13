## 一、Spring boot的使用

#### 1. 创建maven工程

#### 2. 引入依赖

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

#### 3. 创建主程序

```java
/**
 * 主程序类固定写法
 * 这是一个Springboot应用
 */
@SpringBootApplication
public class test {
    public static void main(String[] args) {
        SpringApplication.run(test.class, args);
    }
}
```

#### 4. 编写业务

```java
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello01(){
        return " Hello Spring Boot";
    }
}
```

#### 5. 配置文件

application.properties

```yaml
server.port=8080
```

#### 6. 简化部署

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

把项目打成jar包形式，直接在目标服务器运行。

## 二、了解自动配置原理

### 1. Spring boot的特点

#### 1.1依赖管理

- 父项目依赖管理

  ``` xml
      <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>2.3.4.RELEASE</version>
      </parent>
  ```

  > 它的父项目是

  ```xml
    <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>2.3.4.RELEASE</version>
    </parent>
  ```

  > 几乎声明了所有开发中常见的依赖版本号，版本自动仲裁机制。

- 开发导入starter场景启动器

  > 1、见到很多spring-boot-starter-*：*就代表某种场景
  > 2、只要引入starter这个场景的所有常规需要的依赖都会自动引入
  >
  > 支持的场景：https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using

  

  

- 无需关注版本号，自动版本仲裁

- 可自定义版本号

  > 1、查看spring-boot-dependencies里面规定的当前依赖的版本用的key
  > 2、在当前项目里面重写配置

  ```xml
      <properties>
          <mysql.version>5.1.43</mysql.version>
      </properties>
  ```


#### 1.2 自动配置

- 自动配置tomcat

  - 引入tomcat依赖

    ``` xml
          <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <version>2.3.4.RELEASE</version>
          </dependency>
    ```

  - 配置tomcat

- 自动配好SpringMVC
- 自动配好web常见功能，如：字符编码问题
- 默认的包结构
  - 主程序下的所有包及其下面的所有子包里面的组件都会默认扫描出来
  - 想要改变扫描路径`@SpringBootApplication（scanBasePackages="包名"）`或者`@ComponentScan`指定扫描路径
- 各种配置拥有默认值
- 按需加载所有配置项
- ……

### 2. 容器功能

#### 2.1 组件添加

1. `@Configuration`使用

   ```java
   ##########################@Configuration使用事例###########################################################
   /**
    * 1、配置类里面使用@Bean标注在方法上给容注册组件，默认是单实例的
    * 2、配置类本身也是组件
    * 3、proxyBeanMethods 是不是代理bean方法
    *      full(proxyBeanMethods = true)[保证每个@Bean标注在方法调用多少次都是单实例的]
    *      lite(proxyBeanMethods = false)[每个@Bean方法调用多少次返回的组件都是新创建的]
    *      组件依赖
    *      配置组件有依赖设置成true
    *      组件之间没有依赖设置成false
    * */
   @Configuration(proxyBeanMethods = true) // 告诉Spring boot这是一个配置类 == 配置文件
   public class Myconfig {
       /**
        * 外部无论对配置类中的组件注册方法调用多少次，获取的都是之前注册到容器中的单实例对象
        * @return
        */
       @Bean// 给容器中添加组件。以方法名作为组件的id，返回类型就是组件类型。方法返回的值就是组件在容器中的实例
       public User user01(){
           return new User("zhangsan", 18);
       }
   }
   ##########################@Configuration测试代码###########################################################
   /**
    * 主程序类
    * 这是一个Springboot应用
    */
   @SpringBootApplication
   public class MainApplication {
       public static void main(String[] args) {
           // 返回ICO容器
           ConfigurableApplicationContext run = SpringApplication.run(MainApplication.class, args);
           // 查看容器中的组件
           String[] names = run.getBeanDefinitionNames();
           for (String name : names) {
               System.out.println(name);
           }
           // 从容器中获取组件
           User bean = run.getBean("user01", User.class);
           System.out.println(bean.toString());
           //@Configuration(proxyBeanMethods = true)代理对象调用方法。Springboot总会检查这个组件是否在容器中
           // 保持组件的但实例
           Myconfig myconfig = run.getBean(Myconfig.class);
           User user01 = myconfig.user01();
           User user02 = myconfig.user01();
       }
   }
   ```

### 三、常见注解

1. @PathVariable   路径变量
2. @RequestHeader  获取请求头
3. @RequestParam   获取请求参数
4. @CookieValue      获取cookie值
5. @RequestBody   获取请求体[Post]
6. @RequestAttribute  获取request域属性
7. MatrixVariable  矩阵变量
