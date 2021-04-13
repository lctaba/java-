### spring-boot自动装配，用户自定义配置的流程

- 使用@EnableAutoConfiguration注解

- @EnableAutoConfiguration中有两个注解：
  - @AutoConfigurationPackage：用于扫描导入用户定义的配置类
  - @Import(AutoConfigurationImportSelector.class)：用于导入springboot程序默认配置类
- 利用@Import(AutoConfigurationImportSelector.class)导入springboot默认配置类时，程序会到spring-boot-autoconfigure-2.3.4.RELEASE.jar包中获取META-INF中的配置信息，该配置信息包含了自动配置的127个配置类的类名。
- 通过类名向容器中导入这些默认配置类。
- 这些默认配置类利用@Conditonal注解进行条件装配。
- 满足条件装配的条件之后，程序会先判断用户是否手动向容器中注册了该配置类：
  - 如果用户手动注册，则使用用户的。
  - 如果未注册，则使用默认的。
- 配置类的信息可以在properties文件中修改。通过@ConfigurationProperties(prefix = "mycar")注解获取配置文件中具有该前缀的配置信息，修改对应的配置信息，实现对配置属性的修改。

