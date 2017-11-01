---
layout: post
title: Spring Factories
categories: spring
---

# Spring.factories中的那些key

* org.springframework.boot.env.PropertySourceLoader  
  用来加载一个PropertySource，实现类有： 
    * PropertiesPropertySourceLoader 加载一个properties文件或者xml properties文件
    * YamlPropertySourceLoader 加载一个ymal 或 yml 文件

  在spring-boot中， ConfigFileApplicationListener 通过工具类 PropertySourcesLoader 来加载 application 配置文件

  在工具类PropertySourcesLoader 中， 通过SpringFactoriesLoader.loadFactories(PropertySourceLoader.class, classLoader)方法来创建接口PropertySourceLoader的实例

* org.springframework.boot.SpringApplicationRunListener  
  监听SpringApplication.run方法， 触发以下事件： 
    * started
    * environmentPrepared
    * contextPrepared
    * contextLoaded
    * finished

  通过SpringFactoriesLoader来创建接口SpringApplicationRunListener的实例， 并保存在SpringApplicationRunListeners中

* org.springframework.context.ApplicationContextInitializer  
  在spring的 refreshed方法之前，初始化 ConfigurableApplicationContext， 特别是用在web应用程序中，需要通过代码来做一些application context的初始化工作的情况下。 例如， 根据context的环境来注册一些property source或者设置profile

  在spring-boot中， SpringApplication.initialize方法将会通过SpringFactoriesLoader来加载并创建ApplicationContextInitializer的实例， 然后在SpringApplication.prepareContext方法中来调用这些initializer

* org.springframework.context.ApplicationListener  
  Application event listener的接口，基于观察者模式。

  在spring-boot中， SpringApplication.initialize方法将会通过SpringFactoriesLoader来加载并创建ApplicationListener的实例

* org.springframework.boot.env.EnvironmentPostProcessor  
  用于自定义application的environment， 接口的实现必须要注册到META-INF/spring.factories中

  该接口的常用实现类：
  * ConfigFileApplicationListener： 读取application配置文件
  * SpringApplicationJsonEnvironmentPostProcessor： 实现通过spring.application.json参数来添加 property source

  可实现该接口来加载外部的配置信息， 例如， 从数据库或者config server加载配置信息

* org.springframework.boot.diagnostics.FailureAnalyzer  
  分析错误，提供诊断信息给用户

* org.springframework.boot.diagnostics.FailureAnalysisReporter  
  报告错误信息

* org.springframework.boot.autoconfigure.EnableAutoConfiguration  
  为某个java config开启auto-configuration功能

  EnableAutoConfigurationImportSelector.selectImports方法将会读取META-INF/spring.factories，取出所有的auto-configuration class

* org.springframework.boot.autoconfigure.template.TemplateAvailabilityProvider  
  判断某个模板引擎是否可用

* org.springframework.boot.actuate.autoconfigure.ManagementContextConfiguration  
  这是一个注解，是一个特殊的@Configuration，用于management context

* org.springframework.cloud.bootstrap.BootstrapConfiguration  
  一个标识注解，用于创建bootstrap application context

<!--description-->

# spring.factories 详解：

## spring-boot  

* PropertySource Loaders  
  org.springframework.boot.env.PropertySourceLoader=\  
  org.springframework.boot.env.PropertiesPropertySourceLoader,\  
  org.springframework.boot.env.YamlPropertySourceLoader  
  spring-boot默认配置了2种PropertySourceLoader，用来加载properties文件或者yaml文件  

* Run Listeners  
  org.springframework.boot.SpringApplicationRunListener=\  
  org.springframework.boot.context.event.EventPublishingRunListener  
  EventPublishingRunListener被创建时，同时创建了一个SimpleApplicationEventMulticaster， 并为它添加ApplicationListener。当SpringApplication.run方法中各种事件触发时，为所有监听的ApplicationListener来广播  

* Application Context Initializers  
  org.springframework.context.ApplicationContextInitializer=\  
  org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\  
  注册一个BeanFactoryPostProcessor，用来为错误的配置打印warnings，这里的错误是指： @ComponentScan的包包含了 org 或者 org.springframework。

  org.springframework.boot.context.ContextIdApplicationContextInitializer,\
  设置Spring ApplicationContext ID

  org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
  委托给其他的initializers。  可通过context.initializer.classes参数指定

  org.springframework.boot.context.web.ServerPortInfoApplicationContextInitializer
  监听EmbeddedServletContainerInitializedEvent事件，设置server.ports属性

* Application Listeners  
  org.springframework.context.ApplicationListener=\
  org.springframework.boot.ClearCachesApplicationListener,\
  监听ContextRefreshedEvent事件， 清除ClassLoader缓存

  org.springframework.boot.builder.ParentContextCloserApplicationListener,\
  监听ParentContextAvailableEvent事件， 如果parent application context关闭，则关闭当前application context

  org.springframework.boot.context.FileEncodingApplicationListener,\
  如果设置了spring.mandatory_file_encoding参数， 将会跟file.encoding去比较，如果不相等，则抛出异常，终止application启动

  org.springframework.boot.context.config.AnsiOutputApplicationListener,\
  根据spring.output.ansi.enabled参数， 配置是否启用AnsiOutput

  org.springframework.boot.context.config.ConfigFileApplicationListener,\
  设置active profile，并根据profile来加载application配置
  默认将被加载的配置是 application.properties 和 application.yml， 默认加载路径是： 
  * classpath:
  * file:./
  * classpath:config/
  * file:./config/:   
  
  可通过spring.config.name参数指定配置文件的名称， 通过spring.config.location参数指定加载路径

  org.springframework.boot.context.config.DelegatingApplicationListener,\
  委托给其他ApplicationListener， 可通过context.listener.classes参数指定

  org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener,\
  替换liquibase的ServiceLocator

  org.springframework.boot.logging.ClasspathLoggingApplicationListener,\
  监听ApplicationEnvironmentPreparedEvent和ApplicationFailedEvent， 日志记录当前线程上下文ClassLoader的类路径， DEBUG level.

  org.springframework.boot.logging.LoggingApplicationListener
  配置LoggingSystem， 可以通过logging.config参数来配置logging config路径

* Environment Post Processors
  org.springframework.boot.env.EnvironmentPostProcessor=\
  org.springframework.boot.cloud.CloudFoundryVcapEnvironmentPostProcessor,\
  如果是在Cloud Foundry平台， 加载Cloud Foundry相关参数

  org.springframework.boot.env.SpringApplicationJsonEnvironmentPostProcessor
  通过spring.application.json或者SPRING_APPLICATION_JSON参数来添加 property source

* Failure Analyzers
  org.springframework.boot.diagnostics.FailureAnalyzer=\
  org.springframework.boot.diagnostics.analyzer.BeanCurrentlyInCreationFailureAnalyzer,\
  分析BeanCurrentlyInCreationException错误

  org.springframework.boot.diagnostics.analyzer.BeanNotOfRequiredTypeFailureAnalyzer,\
  分析BeanNotOfRequiredTypeException错误

  org.springframework.boot.diagnostics.analyzer.BindFailureAnalyzer,\
  分析BindException错误

  org.springframework.boot.diagnostics.analyzer.NoUniqueBeanDefinitionFailureAnalyzer,\
  分析NoUniqueBeanDefinitionException错误

  org.springframework.boot.diagnostics.analyzer.PortInUseFailureAnalyzer,\
  分析PortInUseException错误

  org.springframework.boot.diagnostics.analyzer.ValidationExceptionFailureAnalyzer
  分析ValidationException错误

* FailureAnalysisReporters  
  org.springframework.boot.diagnostics.FailureAnalysisReporter=\
  org.springframework.boot.diagnostics.LoggingFailureAnalysisReporter
  日志记录错误信息

## spring-boot-autoconfigure

* Initializers  
  org.springframework.context.ApplicationContextInitializer=\
  org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
  在ConfigurationClassPostProcessor和 Spring Boot之间创建一个共享的 CachingMetadataReaderFactory。
  CachingMetadataReaderFactory是接口MetadataReaderFactory的一个基于缓存的实现， 它为每一个Spring resource缓存一个MetadataReader

  org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer
  日志记录ConditionEvaluationReport

* Application Listeners  
  org.springframework.context.ApplicationListener=\
  org.springframework.boot.autoconfigure.BackgroundPreinitializer
  监听ApplicationEnvironmentPreparedEvent事件， 后台线程触发early initialization

  5种early initialization: 
  * MessageConverterInitializer
  * MBeanFactoryInitializer
  * ValidationInitializer
  * JacksonInitializer
  * ConversionServiceInitializer

* Auto Configure  
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
  org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
  org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
  org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration,\
  org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration,\
  org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
  org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
  org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
  org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
  org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
  org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
  org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
  org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
  org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
  org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
  org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
  org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
  org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
  org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
  org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
  org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
  org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
  org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
  org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
  org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
  org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
  org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
  org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
  org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
  org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
  org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
  org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
  org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
  org.springframework.boot.autoconfigure.jms.hornetq.HornetQAutoConfiguration,\
  org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
  org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
  org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
  org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
  org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
  org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
  org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
  org.springframework.boot.autoconfigure.mobile.DeviceResolverAutoConfiguration,\
  org.springframework.boot.autoconfigure.mobile.DeviceDelegatingViewResolverAutoConfiguration,\
  org.springframework.boot.autoconfigure.mobile.SitePreferenceAutoConfiguration,\
  org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
  org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
  org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
  org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
  org.springframework.boot.autoconfigure.reactor.ReactorAutoConfiguration,\
  org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration,\
  org.springframework.boot.autoconfigure.security.SecurityFilterAutoConfiguration,\
  org.springframework.boot.autoconfigure.security.FallbackWebSecurityAutoConfiguration,\
  org.springframework.boot.autoconfigure.security.oauth2.OAuth2AutoConfiguration,\
  org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
  org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
  org.springframework.boot.autoconfigure.social.SocialWebAutoConfiguration,\
  org.springframework.boot.autoconfigure.social.FacebookAutoConfiguration,\
  org.springframework.boot.autoconfigure.social.LinkedInAutoConfiguration,\
  org.springframework.boot.autoconfigure.social.TwitterAutoConfiguration,\
  org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
  org.springframework.boot.autoconfigure.velocity.VelocityAutoConfiguration,\
  org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
  org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
  org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
  org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration,\
  org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration,\
  org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration,\
  org.springframework.boot.autoconfigure.web.HttpEncodingAutoConfiguration,\
  org.springframework.boot.autoconfigure.web.HttpMessageConvertersAutoConfiguration,\
  org.springframework.boot.autoconfigure.web.MultipartAutoConfiguration,\
  org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration,\
  org.springframework.boot.autoconfigure.web.WebClientAutoConfiguration,\
  org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration,\
  org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration,\
  org.springframework.boot.autoconfigure.websocket.WebSocketMessagingAutoConfiguration,\
  org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration

* Failure analyzers  
  org.springframework.boot.diagnostics.FailureAnalyzer=\
  org.springframework.boot.autoconfigure.jdbc.DataSourceBeanCreationFailureAnalyzer
  分析DataSourceBeanCreationException错误

* Template availability providers  
  org.springframework.boot.autoconfigure.template.TemplateAvailabilityProvider=\
  org.springframework.boot.autoconfigure.freemarker.FreeMarkerTemplateAvailabilityProvider,\
  org.springframework.boot.autoconfigure.mustache.MustacheTemplateAvailabilityProvider,\
  org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAvailabilityProvider,\
  org.springframework.boot.autoconfigure.thymeleaf.ThymeleafTemplateAvailabilityProvider,\
  org.springframework.boot.autoconfigure.velocity.VelocityTemplateAvailabilityProvider,\
  org.springframework.boot.autoconfigure.web.JspTemplateAvailabilityProvider


## spring-boot-actuator

* Auto Configure  
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.boot.actuate.autoconfigure.AuditAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.CacheStatisticsAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.CrshAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.EndpointAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.EndpointMBeanExportAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.EndpointWebMvcAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.HealthIndicatorAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.InfoContributorAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.JolokiaAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.ManagementServerPropertiesAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.ManagementWebSecurityAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.MetricFilterAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.MetricRepositoryAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.MetricsDropwizardAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.MetricsChannelAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.MetricExportAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.PublicMetricsAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.TraceRepositoryAutoConfiguration,\
  org.springframework.boot.actuate.autoconfigure.TraceWebFilterAutoConfiguration

* Management context configure  
  org.springframework.boot.actuate.autoconfigure.ManagementContextConfiguration=\
  org.springframework.boot.actuate.autoconfigure.EndpointWebMvcManagementContextConfiguration,\
  org.springframework.boot.actuate.autoconfigure.EndpointWebMvcHypermediaManagementContextConfiguration
  通过spring mvc暴露endpoint

  ManagementContextConfigurationsImportSelector负责从META-INF/spring.factories读取相关配置

## spring-cloud-commons

* AutoConfiguration  
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.cloud.client.CommonsClientAutoConfiguration,\
  org.springframework.cloud.client.discovery.noop.NoopDiscoveryClientAutoConfiguration,\
  org.springframework.cloud.client.hypermedia.CloudHypermediaAutoConfiguration,\
  org.springframework.cloud.client.loadbalancer.AsyncLoadBalancerAutoConfiguration,\
  org.springframework.cloud.client.loadbalancer.LoadBalancerAutoConfiguration,\
  org.springframework.cloud.client.serviceregistry.ServiceRegistryAutoConfiguration,\
  org.springframework.cloud.commons.util.UtilAutoConfiguration,\
  org.springframework.cloud.client.discovery.simple.SimpleDiscoveryClientAutoConfiguration

* Environment Post Processors  
  org.springframework.boot.env.EnvironmentPostProcessor=\
  org.springframework.cloud.client.HostInfoEnvironmentPostProcessor
  添加hostname和IP到property source

## spring-cloud-context 

* AutoConfiguration  
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration,\
  org.springframework.cloud.autoconfigure.RefreshAutoConfiguration,\
  org.springframework.cloud.autoconfigure.RefreshEndpointAutoConfiguration,\
  org.springframework.cloud.autoconfigure.LifecycleMvcEndpointAutoConfiguration

* Application Listeners  
  org.springframework.context.ApplicationListener=\
  org.springframework.cloud.bootstrap.BootstrapApplicationListener,\
  创建一个新的SpringApplication，在META-INF/spring.factories中加载BootstrapConfiguration配置来设置source，用bootstrap.properties来代替application.properties来加载配置

  org.springframework.cloud.bootstrap.LoggingSystemShutdownListener,\
  org.springframework.cloud.context.restart.RestartListener

* Bootstrap components
  org.springframework.cloud.bootstrap.BootstrapConfiguration=\
  org.springframework.cloud.bootstrap.config.PropertySourceBootstrapConfiguration,\
  通过PropertySourceLocator来加载property source

  org.springframework.cloud.bootstrap.encrypt.EncryptionBootstrapConfiguration,\
  org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration,\
  org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration

## spring-cloud-zookeeper-core

* Auto Configuration
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.cloud.zookeeper.ZookeeperAutoConfiguration
  配置zookeeper客户端 CuratorFramework

## spring-cloud-zookeeper-config

* Auto Configuration  
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.cloud.zookeeper.config.ZookeeperConfigAutoConfiguration
  配置zookeeper config watcher (ConfigWatcher)

* Bootstrap Configuration  
  org.springframework.cloud.bootstrap.BootstrapConfiguration=\
  org.springframework.cloud.zookeeper.config.ZookeeperConfigBootstrapConfiguration
  配置ZookeeperPropertySourceLocator，加载zookeeper数据到property source
  PropertySourceBootstrapConfiguration将会调用ZookeeperPropertySourceLocator， 将配置信息写入environment


