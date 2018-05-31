#作者： 汪冲

#本项目为很牛x很高大上的springcloud项目，前方高能，请非战斗人员离场		> .<  	开个玩笑

#项目起源：
说实话，项目起源很简单，老项目的业务拆分，代码优化，业务代码的解耦

#这个项目应该学到什么

##1.业务熟悉
没有业务，一切都是纸上谈兵，之前也研究过springboot和springcloud，并没有研究出什么结果出来，但是有了业务，项目要在规定时间内上线，还是要抓紧熟悉业务，学习相关知识的（O。O不然怎么办啊，项目要上线，不会也要强行上啊，尼玛坑爹啊），熟悉业务才能更好的拆分项目。

##2.项目协作
springcloud的角色较多，比如注册中心，配置中心，消费者，生产者，负载均衡，熔断器，路由，网关，队列等等（本项目没有用到，只用到部分），一个人肯定是干不完的，主要是代码和搭环境要双线进行，需要一个团队协作，比如本项目中，我就主要负责搭环境（简单来说就是踩坑，填坑），我队友迁移代码。

##3.技术
前面说了半天的屁话，终于到技术层面了，这次拆分项目，主要还是学习技术（学不到真的可惜了，微服务，分布式，肯定是未来的主流，国外已经是主流，别跟我说dubbo，dubbo基本上只有springcloud的一个服务发现模块，其他功能都要自己实现）

###spring相关基础

了解spring基础才能更好的理解，效率的拆分项目成springcloud

spring ioc：英文全称是Inversion of Control，控制反转，是一种思想转变，网上帖子说的很清楚了，可以去百度，这里我拿个例子描述我的理解，传统的开发，我们想做某个操作，需要先拿到对象，再用对象去	 进行这个操作（毕竟java是面向对象编程），而ioc，就是由spring管理的ioc容器在初始化的时候就实例化好了对象，要做某个操作的时候，直接给你这个对象进行操作就行了。

spring di：英文全称是Dependency Injection，依赖注入，上面提到spring初始化的时候实例化了对象放入了容器之中，di就是把对象注入到你需要用到的地方使用就行了。PS：
>>>> spring整个对象必须是一条链，也就是说，从头到位都是spring管理，如果是不受spring管理的对象，需要用拿到容器，通过容器获取，static修饰的方法，属性不受管理。。

spring aop：英文全称是Aspect Oriented Programming，面向切面编程，即对于面向对象的补充和完善，利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重 用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。权限认证、日志、事物都可以用到此技术，具体实现为动态代理。事物的手动回滚和自动回滚都需要会用。

spring 的基础注解，这里就不一一列举了。

###mybatis

mybatis 的数据库配置，此处请注意数据库参数，连接数等等
示例：jdbc:mysql://172.18.11.127:3306/trymysql?&useUnicode=true&characterEncoding=utf-8
mybatis动态sql，这个很重要，自行学习吧，批处理操作划重点。

###redis

redis连接，pool的管理（单例，表示内存为唯一一块，本次拆分，redis全部交给了spring容器管理，默认为单例，别再纠结于一个static了 O.O ），批处理（redis的批处理跟单个处理的速度一样，甚至更快，代码应该尽量考虑批处理，多次连接做同一个或者类似操作，还不如批量查出来集中处理），流的关闭等问题。

原谅我们的项目就这么点东西。>_< \\\

###分布式

springcloud是一个分布式服务框架，分布式解决的是并发可用，扩容，解耦等问题。

分布式集群的好处：

1. 系统之间的耦合度大大降低，可以独立开发、独立部署、独立测试，系统与系统之间的边界非常明确，排错也变得相当容易，开发效率大大提升。

2. 系统之间的耦合度降低，从而系统更易于扩展。我们可以针对性地扩展某些服务。假如某个设备业务量比较多，增加这个业务所在节点的机器就可以了，而对于后台管理系统、数据分析系统而言，节点数量维持原有水平即可。

3. 服务的复用性更高。比如，当我们将用户系统作为单独的服务后，公司所有的产品都可以使用该系统作为用户系统，无需重复开发。

 	。。。此次省略十万个好处。
 	
###springcloud 配置

####注册中心
跟dubbo一样，肯定需要的三个角色，注册中心，消费者，生产者。

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>
引入此jar包，启动类上加@EnableEurekaServer注解即可（比dubbo不知道简单多少倍）
配置文件：（我这里配置文件是bootstrap而不是application，不是我画蛇添足，而是因为bootstrap加载优于application，为什么需要这个，留个悬念）
>>>
server:
  port: 8761    #表示设置该服务注册中心的端口
eureka: 
  environment: dev    #表示当前环境，dev为开发环境，slave为仿真，master为线上
  instance: 
    hostname: ${hostname:localhost}     #表示设置该服务注册中心的hostname
    prefer-ip-address: true
  client: 
    register-with-eureka: false    #由于我们目前创建的应用是一个服务注册中心，而不是普通的应用，默认情况下，这个应用会向注册中心（也是它自己）注册它自己，设置为false表示禁止这种默认行为
    fetch-registry: false    #表示不去检索其他的服务，因为服务注册中心本身的职责就是维护服务实例，它也不需要去检索其他服务
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/  #注册中心地址
  server: 
    enable-self-preservation: true  #表示开启自我保护模式

所有配置我都写了注释了，自行理解吧。

####生产者

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>
除了写业务代码需要的包之外，还需要这个jar包，然后在启动类上加@EnableEurekaClient注解即可，然后你会机智的发现，配置文件呢，bootstrap.yml里面的内容跟服务器无关，他的配置在哪，数据库在哪？
这就是dubbo和springcoloud 的不同之处了，前面我说过了，dubbo三种角色，注册中心，消费者，生产者，而springcloud远远不止，这里我用到了一种角色，分布式配置中心，springcloud config。

####消费者
web项目即为消费者，对外开放的web接口服务

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-feign</artifactId>
		</dependency>
		<dependency>
			<groupId>io.github.openfeign.form</groupId>
			<artifactId>feign-form</artifactId>
			<version>2.1.0</version>
		</dependency>
		<dependency>
			<groupId>io.github.openfeign.form</groupId>
			<artifactId>feign-form-spring</artifactId>
			<version>2.1.0</version>
		</dependency>
在启动类上加上@EnableDiscoveryClient（标记为普通消费者）@EnableFeignClients（标记为自带Feign服务的消费者）
用法：在消费者的service接口上加注解@FeignClient(value = "service-gt")，value为服务名，即可实现rpc调用服务


####配置中心

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
除了注册中心，所有节点引入此jar包，表示支持springcloud config，配置中心项目的启动类上加上@EnableConfigServer（标记为配置中心）@EnableEurekaClient（注册给注册中心提供分布式支持）
配置：
>>>	
spring.application.name=config-server
	#配置中心端口
server.port=8888
	# 配置git仓库地址
spring.cloud.config.server.git.uri=http://git.muheda.com/muheda/service-framework-config.git
	# 配置仓库路径
spring.cloud.config.server.git.searchPaths=config-dev
	# 配置仓库的分支
spring.cloud.config.label=develop
	# 访问git仓库的用户名密码
spring.cloud.config.server.git.username= xxx
spring.cloud.config.server.git.password= xxxx
	#注册中心地址
eureka.client.serviceUrl.defaultZone=http://127.0.0.1:8761/eureka/
	#关闭安全访问限制
management.security.enabled = false

注释都写全了，自行理解。
配置文件中的spring.cloud.config.server.git.searchPaths表示在git中的文件夹名称，也就是所有的配置都是读自这个文件夹中的配置文件，{serverName}-{label}.yml 为各个服务的配置，修改配置需要提交到git上才能生效（读取的是配置中心配置文件中的git的地址里面的searchPaths路径下的配置文件，不提交代码鬼晓得你改了什么）

#####配置热加载

这里附送一个配置中心配置热加载（本项目我只是写了一个能够热加载的demo，并没有用到重构中）

	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>
增加此jar包，在需要热加载的节点配置中加入management.security.enabled=false关闭安全验证（都说了是个demo，开启安全验证在刷新配置之前还要去认证权限，太麻烦了，干脆关闭了，偷懒中），在需要热加载某些配置的类上增加注解@Configuration 和 @RefreshScope，在热加载的配置所用到的方法上加上注解 @RefreshScope（在web项目的com.muheda.web.controller.grading.BehaviorViewController里面，自己看吧）。这样就行了吗？你会发现，你怎么改变配置，还是原来的配置，因为他的加载是被动加载，也就是需要我们主动推给他，post调用http://localhost:8765/refresh 刷新配置即可。

####熔断器
熔断器是dubbo所没有的一种熔断机制，具体是干什么的，自行百度～。～
熔断器处于生产者和消费者之间，用于熔断处理

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix</artifactId>
		</dependency>
引入此jar包，启动类加注解@EnableHystrix，配置文件中加入feign.hystrix.enabled = true 
用法： 
在消费者的service接口上的@FeignClient(value = "service-gt")注解中加入fallback = xxxx.class属性，xxxx.class为此接口的实现类（实现类中即可做熔断处理）。

###踩坑
不用多说，日常踩坑系列之-----分布式坑

####分布式session
提到分布式，不得不说的肯定是分布式session啊。此次的分布式session的实现比较简单，用spring session实现，即序列化session到redis，rpc取session即去session中取

		<dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session-data-redis</artifactId>
		</dependency>
在项目启动类上加注解@EnableRedisHttpSession，在配置中心的配置文件中加入spring.session.store-type = redis 表示session用redis传递，然后配置redis地址即可spring.redis.host,spring.redis.password,spring.redis.port。原本我以为这么弄就完事了。其实不存在的，怎么可能这么简单。还需要额外的java config的支持，作为session的中转站。
		
	@Configuration
	public class FeignSessionConfig {
		@Bean
		public RequestInterceptor requestInterceptor() {
			return new RequestInterceptor() {
				@Override
				public void apply(RequestTemplate requestTemplate) {
					String sessionId = RequestContextHolder.currentRequestAttributes().getSessionId();
					if (!StringUtil.isEmpty(sessionId)) {
						requestTemplate.header("Cookie", "SESSION=" + sessionId);
					}
				}
			};
		}
	}
加入此java config才能让session在所有springcloud节点中共享。

####	页面
本来打算用新的thymeleaf模版，后来被支付宝恶心到了，支付宝的属性传递和页面都是非标准的html，thymeleaf太严格，后摒弃（妈的，这里坑了我好几天），后来换成freemarker，谢天谢地，终于完美解决（除了html需要变成ftl文件之外）。

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-freemarker</artifactId>
		</dependency>
引入jar包即可
		
####数据源
这里指的是多个数据源的配置问题，三个以上，正常状态下的读写分离的2个数据源，这里不考虑，三个及三个以上的数据源。我这里用的是java config的方式实现的。

	@Configuration
	@MapperScan(basePackages = "com.muheda.api.dao.mysql.mapper.reader", sqlSessionTemplateRef  = "readerSqlSessionTemplate")
	public class ReaderDataSource{
		@Bean(name = "slaveDataSource")
    		@ConfigurationProperties(prefix = "spring.datasource.reader")
    		public DataSource readerDataSource() {
        		return DataSourceBuilder.create().build();
    		}
	    Bean(name = "readerSqlSessionFactory")
	    public SqlSessionFactory readerSqlSessionFactory(@Qualifier("slaveDataSource") DataSource dataSource) throws Exception {
	        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
	        bean.setDataSource(dataSource);
	        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/reader/*.xml"));
	        return bean.getObject();
	    }
	    @Bean(name = "readerTransactionManager")
	    public DataSourceTransactionManager readerTransactionManager(@Qualifier("slaveDataSource") DataSource dataSource) {
	        return new DataSourceTransactionManager(dataSource);
	    }
	    @Bean(name = "readerSqlSessionTemplate")
	    public SqlSessionTemplate readerSqlSessionTemplate(@Qualifier("readerSqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
	        return new SqlSessionTemplate(sqlSessionFactory);
	    }
	}
你以为坑就这么结束了吗。NO！@Bean(name = "slaveDataSource") 这个位置的slaveDataSource ，之前我按照常规思想，配置的readDataSource和writeDataSource，然而，他内部默认读写分离自己装配的bean就是这2个名字。没错，这里再用这个名字就是重复注入。（坑爹啊，我tm名字太标准也是一种错吗，不过仔细想想，我的命名跟开发者雷同，说明还是有点东西的）。

####文件上传
这个就不用多说了，直接上代码吧。

	@RequestMapping(value = "/api/uploadexcel/toFile", method = RequestMethod.POST, produces = {
			MediaType.APPLICATION_JSON_UTF8_VALUE }, consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
	public ResultData uploadSimData(@RequestPart(value = "excelFile")MultipartFile excelFile) {
		return memberForJryjService.uploadSimData(excelFile);
	}
反正就是各种配置各种加属性就完事了

####熔断器
这个东西应该是我在springcloud中遇到的最大的坑了，本来初衷是好的，想在服务宕机的时候，也有一个熔断器让服务熔断，不会直接把错误暴露出来，然而。。。。
这边有几个坑
1.他对于页面的影响。加了熔断器之后，spring session 的共享会报错了，需要加入特殊的熔断策略hystrix.command.default.execution.isolation.strategly = SEMAPHORE
2.加了熔断器之后，接口反应时间过长会自动进入熔断，但是这个默认时间太短了，会影响到接口调用，加入hystrix.command.default.execution.isolation.thrad.timeoutInMilliseconds = 75000这个配置即可
3.熔断器对并发请求是有拦截的，默认是10个并发就开始进入熔断器，这在压力环境下根本太低了，需要增加并发请求数，线上的这个并发请求数数根据压测的cpu状况决定，加入配置hystrix.command.default.execution.isolation.semaphor.maxConcurrentRequests=60 当前把请求数加到60.（这里基本上算上本项目最大的坑了，加了熔断器，默认约束了请求并发，真tm坑）

####rpc请求
在消费者调用生产者服务的时候，经常出现调用超时，因为，ribbon限制了调用时间，默认100ms，也就是说服务需要100ms内调用完毕，尼玛这里坑爹啊，每个服务都能这么块才是见鬼了，加入配置
ribbon.ReadTimeout = 70000和ribbon.ConnectTimeout = 70000,这里的时间根据项目服务中最慢的一个决定，本项目最慢的一个是64秒。。。

####mysql超时不重连问题
mysql默认8小时连接不干活就自动断开连接，断开连接之后不会通知这个连接，所以下次连接操作数据库的时候会报错，这里在数据库配置后面加2个属性就行了autoReconnect=true（支持重连）和failOverReadOnly=false（重连后支持读写）

#总结
实际上本项目很关键的分布式锁,分布式事物没有涉及到，分布式锁准备用redis实现，分布式事物准备用rocketMq实现，后续补上。
