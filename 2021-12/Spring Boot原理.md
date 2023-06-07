## SpringBoot自动装配的原理
SpringBoot自动装配主要依赖的是`@EnableAutoConfiguration`注解，在`@SpringBootApplication`注解中贴了该注解，该注解可以让SpringBoot在
启动时，从classpath中扫所有jar包的META-INF/spring.factories文件，获取到EnableAutoConfiguration.class映射的所有类名列表，Spring会在所有非配置类Bean初始化前
根据对应AutoConfiguration类上的条件，如果满足的话就加载类并进行初始化，放到Spring容器中

## @SpringBootApplication注解

包含注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
```


`@SpringBootConfiguration` 和 `@Configuration` 作用类似

比较关键的注解为 `@EnableAutoConfiguration` 

### @EnableAutoConfiguration
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	/**
	 * Environment property that can be used to override when auto-configuration is
	 * enabled.
	 */
	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	/**
	 * Exclude specific auto-configuration classes such that they will never be applied.
	 * @return the classes to exclude
	 */
	Class<?>[] exclude() default {};

	/**
	 * Exclude specific auto-configuration class names such that they will never be
	 * applied.
	 * @return the class names to exclude
	 * @since 1.3.0
	 */
	String[] excludeName() default {};

}
```

其中通过 `@Import` 导入 `AutoConfigurationImportSelector`

`AutoConfigurationImportSelector` 实现了 `DeferredImportSelector` 的 `selectImports`方法

`selectImports` 最终会通过 SpringFactoriesLoader.loadFactoryNames 加载classpath下META-INF/spring.factories文件, 返回 `EnableAutoConfiguration.class` 映射的所有class



### ConfigurationClassPostProcessor
`ConfigurationClassPostProcessor` 实现了 `BeanFactoryPostProcessor` (和BeanPostProcessor接口类似，但是会在创建所有bean之前创建), 如文档所说

> ApplicationContext在其 bean 定义中自动检测BeanFactoryPostProcessor bean，并在创建任何其他 bean 之前应用它们

`ConfigurationClassPostProcessor` 调用了 `ConfigurationClassParser` 的 parse 方法, 最终调用 `doProcessConfigurationClass` 会解析 `Component`, `PropertySources`, `ComponentScan`,
`Import` 和 `ImportResource`等注解


一篇很不错的文章: [https://blog.csdn.net/qq_34436819/article/details/100944204](https://blog.csdn.net/qq_34436819/article/details/100944204)

![ConfigurationClassPostProcessor加载流程](imgs/ConfigurationClassPostProcessor.png)

在处理Import注解时，会根据Import的Class类型来分别处理`ImportSelector`、`ImportBeanDefinitionRegistrar`和其他类型的class


Spring Boot的 `AutoConfigurationImportSelector`就是在这里加载的

