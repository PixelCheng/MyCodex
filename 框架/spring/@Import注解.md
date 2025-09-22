`@Import` 是 **Spring Framework** 中的一个**核心注解**，主要用于**将额外的配置类（或组件）导入到当前的配置类中**。它的核心目的是实现**模块化配置**和**代码组织**，让你能够将配置分散在多个类中，然后在需要时组合起来。

**主要作用和场景：**

1. **导入其他 `@Configuration` 类：**

    - 这是最常见的用法。如果你的 Spring 配置分散在多个用 `@Configuration` 注解的类中，你可以使用 `@Import` 在一个主配置类中引入它们。

    - 示例：

      ```java
      @Configuration
      public class DatabaseConfig {
          @Bean
          public DataSource dataSource() {
              // 配置数据源
              return ...;
          }
      }
      
      @Configuration
      public class ServiceConfig {
          @Bean
          public MyService myService() {
              return new MyServiceImpl();
          }
      }
      
      // 主配置类，导入其他配置
      @Configuration
      @Import({DatabaseConfig.class, ServiceConfig.class}) // 导入 DatabaseConfig 和 ServiceConfig
      public class AppConfig {
          // 这里可以定义自己的 Bean 或依赖其他导入的 Bean
      }
      ```

    - 当 Spring 容器启动并处理 `AppConfig` 时，它会自动处理 `DatabaseConfig` 和 `ServiceConfig` 中定义的 Bean，就像这些 Bean 是直接在 `AppConfig` 中定义的一样。

2. **导入 `@Component` 类（普通组件）：**

    - 虽然 `@ComponentScan` 通常是扫描组件的首选方式，但 `@Import` 也可以用来显式地引入单个或多个用 `@Component`（或其派生注解如 `@Service`, `@Repository`, `@Controller`）标记的类。

    - 示例：

      ```java
      @Service
      public class SpecialService {
          // ...
      }
      
      @Configuration
      @Import(SpecialService.class) // 显式导入 SpecialService 组件
      public class AppConfig {
          // ...
      }
      ```

3. **导入 `ImportSelector` 实现：**

    - 这是更高级的用法，用于**动态地、有条件地**选择要导入哪些配置。`ImportSelector` 是一个接口，你需要实现它的 `selectImports` 方法，该方法返回一个包含要导入的配置类全限定名的字符串数组。

    - **场景：** 根据环境变量、系统属性、条件注解等决定加载哪些配置。Spring Boot 的自动配置机制大量使用了 `ImportSelector`。

    - 示例 (简化)：

      ```java
      public class MyImportSelector implements ImportSelector {
          @Override
          public String[] selectImports(AnnotationMetadata importingClassMetadata) {
              // 根据条件判断，返回需要导入的配置类名
              if (someCondition) {
                  return new String[]{"com.example.ConfigA"};
              } else {
                  return new String[]{"com.example.ConfigB"};
              }
          }
      }
      
      @Configuration
      @Import(MyImportSelector.class) // 导入选择器
      public class AppConfig {
          // ...
      }
      ```

4. **导入 `ImportBeanDefinitionRegistrar` 实现：**

    - 这是最灵活的用法，允许你**以编程方式直接向 Spring 容器注册 Bean 定义**。`ImportBeanDefinitionRegistrar` 也是一个接口，你需要实现它的 `registerBeanDefinitions` 方法。

    - **场景：** 需要非常精细地控制 Bean 的注册过程，例如基于注解动态生成代理、注册大量相似 Bean 等。很多框架（如 MyBatis-Spring）利用它来集成。

    - 示例 (简化)：

      ```java
      public class MyBeanRegistrar implements ImportBeanDefinitionRegistrar {
          @Override
          public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
              // 使用 registry 对象手动注册 BeanDefinition
              RootBeanDefinition beanDef = new RootBeanDefinition(MySpecialBean.class);
              registry.registerBeanDefinition("mySpecialBean", beanDef);
          }
      }
      
      @Configuration
      @Import(MyBeanRegistrar.class) // 导入注册器
      public class AppConfig {
          // ...
      }
      ```

**`@Import` 与 `@ComponentScan` 的区别：**

- **`@ComponentScan`:** 用于**扫描**指定包（及其子包）下的组件（`@Component`, `@Service`, `@Repository`, `@Controller`）和配置类（`@Configuration`），并自动将它们注册为 Spring Bean。它是一种**基于包路径的批量发现机制**。
- **`@Import`:** 用于**显式地、精确地**引入**特定的**配置类、组件类或特殊的导入器（`ImportSelector`/`ImportBeanDefinitionRegistrar`）。它是一种**显式声明依赖配置的方式**，不涉及包扫描。

**总结：**

`@Import` 注解是 Spring 配置模块化的关键工具。它允许你：

1. **组合配置：** 将多个配置类合并到一个入口点 (`@Import({ConfigA.class, ConfigB.class})`)。
2. **显式引入特定组件：** 直接引入单个非配置类组件 (`@Import(SpecialComponent.class)`)。
3. **实现动态配置：** 通过 `ImportSelector` 根据条件选择加载哪些配置。
4. **实现高级注册：** 通过 `ImportBeanDefinitionRegistrar` 以编程方式精细控制 Bean 的注册。