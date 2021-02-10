# Spring Boot  @Enable*注解源码解析及自定义@Enable*


&emsp;&emsp;Spring Boot 一个重要的特点就是自动配置，约定大于配置，几乎所有组件使用其本身约定好的默认配置就可以使用，大大减轻配置的麻烦。其实现自动配置一个方式就是使用@Enable*注解，见其名知其意也，即“使什么可用或开启什么的支持”。

### Spring Boot 常用@Enable*

首先来简单介绍一下Spring Boot 常用的@Enable*注解及其作用吧。

1. `@EnableAutoConfiguration` 开启自动扫描装配Bean，组合成@SpringBootApplication注解之一 <br/><br/>
2. `@EnableScheduling` 开启计划任务的支持<br/><br/>
3. `@EnableTransactionManagement` 开启注解式事务的支持。<br/><br/>
4. `@EnableCaching` 开启注解式的缓存支持。<br/><br/>
5. `@EnableAspectJAutoProxy` 开启对AspectJ自动代理的支持。<br/><br/>
6. `@EnableEurekaServer` 开启Euraka Service 的支持，开启spring cloud的服务注册与发现<br/><br/>
7. `@EnableDiscoveryClient` 开启服务提供者或消费者，客户端的支持，用来注册服务或连接到如Eureka之类的注册中心<br/><br/>
8. `@EnableFeignClients` 开启[Feign](https://segmentfault.com/a/1190000011675354)功能<br/><br/>




还有一些不常用的比如：<br/>

0. `@EnableAsync` 开启异步方法的支持<br/><br/>
10. `@EnableWebMvc` 开启Web MVC的配置支持。<br/><br/>
11. `@EnableConfigurationProperties`  开启对@ConfigurationProperties注解配置Bean的支持。<br/><br/>
12. `@EnableJpaRepositories` 开启对Spring Data JPA Repository的支持。<br/><br/>



参考：http://tangxiaolin.com/learn/show?id=402881d2648c88cc01648c89d8730001


### @Enable*的源码解析
#### 查看它们的源码
@EnableAutoConfiguration 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190214154013463.PNG)

@EnableCaching 开启注解式的缓存支持。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190214154113111.PNG)

@EnableDiscoveryClient（@EnableEurekaServer 也是使用了这个组合注解） 开启服务提供者或消费者，客户端的支持，用来注册服务或连接到如Eureka之类的注册中心

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190214154152136.PNG)

@EnableAspectJAutoProxy 开启对AspectJ自动代理的支持。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190214154124740.PNG)

@EnableFeignClients 开启Feign功能

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190214154202827.PNG)

@EnableScheduling（这个比较**特殊**，为自己直接新建相关类，不继承***Selector和***Registrar） 开启计划任务的支持

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190214154042196.PNG)

#### 源码规律及解析
可以发现它们都使用了@Import注解（其中@Target:注解的作用目标，@Retention：注解的保留位置，@Inherited：说明子类可以继承父类中的该注解，@Document：说明该注解将被包含在javadoc中）

该元注解是被用来整合所有在@Configuration注解中定义的bean配置，即相当于我们将多个XML配置文件导入到单个文件的情形。

而它们所引入的配置类，主要分为***Selector和***Registrar，其分别实现了`ImportSelector`和`ImportBeanDefinitionRegistrar`接口，

![ImportSelector接口源码](https://img-blog.csdnimg.cn/2019021416233925.PNG)

![ImportBeanDefinitionRegistrar接口源码](https://img-blog.csdnimg.cn/20190214162414379.PNG)

两个的大概意思都是说，会根据AnnotationMetadata元数据注册bean类，即返回的Bean 会自动的被注入，被Spring所管理。

既然他们功能都相同，都是用来返回类，为什么 Spring 有这**两种不同的接口类**的呢？

其实刚开始的时候我也以为它们功能应该都是一样的，后面我在组内分享的时候，我的导师就问了我这个问题，然后当时我没有留意这个点所以答不出来😂。后面回去细看了一下和搜索了相关资料，发现它们的功能有些细微差别。首先我们从上面截图可以清楚地看到`ImportBeanDefinitionRegistrar`接口类中 `registerBeanDefinitions`方法多了一个参数 `BeanDefinitionRegistry`（点击这个参数进入看这个参数的Javadoc，可以知道，它是用于保存bean定义的注册表的接口），所以如果是实现了这个接口类的首先可以应用比较复杂的注册类的判断条件，例如：可以判断之前的类是否有注册到 Spring 中了。另外就是实现了这个接口类能修改或新增 Spring 类定义`BeanDefinition`的一些属性（查看其中一个实现了这个接口例子如：`AspectJAutoProxyRegistrar`，追查 `BeanDefinitionRegistry`参数可以查看到）。



#### 具体实现例子@EnableDiscoveryClient
可以看一下具体的一个例子在@EnableDiscoveryClient引入了EnableDiscoveryClientImportSelector，通过查看其继承实现图

![EnableDiscoveryClientImportSelector的继承实现图](https://img-blog.csdnimg.cn/20190214163950265.PNG)

可以看到其最终实现了ImportSelector接口，查看其具体实现源码

![EnableDiscoveryClientImportSelector的具体实现](https://img-blog.csdnimg.cn/20190214164232153.PNG)

知道其先得到父类注册的bean类，然后如果在查看AnnotationMetadata中是否存在autoRegister，是否需要注册该类，如果存在，则继续将`org.springframework.cloud.client.serviceregistry.AutoServiceRegistrationConfiguration`该类返回，加入到spring容器中。

####  源码小结
通过查看@Enable*源码，我们可以清楚知道其实现自动配置的方式的底层就是通过@Import注解引入相关配置类，然后再在配置类将所需的bean注册到spring容器中和实现组件相关处理逻辑去。

### 自定义@Enable*注解（EnableSelfBean）
&emsp;&emsp;在这里我们利用@Import和ImportSelector动手自定义一个自己的EnableSelfBean。该Enable注解可以将某些包下的所有类自动注册到spring容器中，对于一些实体类的项目很多的情况下，可以考虑一下通过这种方式将某包下所有类自动加入到spring容器，不再需要每个类再加上@Component等注解。

1. 先创建一个spring boot项目。
2. 创建包entity，并新建类Role，将其放入到entity包中。
```java
/**
 * 测试自己的自动注解的实体类
 * @author zhangcanlong
 * @since 2019/2/14 10:41
 **/
public class Role {
    public String test(){
     return "hello";
    }
}
```
3. 创建自定义配置类SelfEnableAutoConfig并实现ImportSelector接口。其中使用到ClassUtils类是用来获取自己某个包下的所有类的名称的。
```java
/**
 * 自己的定义的自动注解配置类
 * @author zhangcanlong
 * @since 2019/2/14 10:45
 **/
public class SelfEnableAutoConfig implements ImportSelector {
    Logger logger = LoggerFactory.getLogger(SelfEnableAutoConfig.class);
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        //获取EnableEcho注解的所有属性的value
        Map<String, Object> attributes = annotationMetadata.getAnnotationAttributes(EnableSelfBean.class.getName());
        if(attributes==null){return new String[0]; }
        //获取package属性的value
        String[] packages = (String[]) attributes.get("packages");
        if(packages==null || packages.length<=0 || StringUtils.isEmpty(packages[0])){
            return new String[0];
        }
        logger.info("加载该包所有类到spring容器中的包名为："+ Arrays.toString(packages));
        Set<String> classNames = new HashSet<>();

        for(String packageName:packages){
            classNames.addAll(ClassUtils.getClassName(packageName,true));
        }
        //将类打印到日志中
        for(String className:classNames){
            logger.info(className+"加载到spring容器中");
        }
        String[] returnClassNames = new String[classNames.size()];
        returnClassNames= classNames.toArray(returnClassNames);
        return  returnClassNames;
    }
}
```
ClassUtil类

```java
/**
 * 获取所有包下的类名的工具类。参考：https://my.oschina.net/cnlw/blog/299265
 * @author zhangcanlong
 * @since 2019/2/14
 **/
@Component
public class ClassUtils {
	private static final String FILE_STR= "file";
	private static final String JAR_STR = "jar";

	/**
	 * 获取某包下所有类
	 * @param packageName 包名
	 * @param isRecursion 是否遍历子包
	 * @return 类的完整名称
	 */
	public static Set<String> getClassName(String packageName, boolean isRecursion) {
		Set<String> classNames = null;
		ClassLoader loader = Thread.currentThread().getContextClassLoader();
		String packagePath = packageName.replace(".", "/");
		URL url = loader.getResource(packagePath);
		if (url != null) {
			String protocol = url.getProtocol();
			if (FILE_STR.equals(protocol)) {
				classNames = getClassNameFromDir(url.getPath(), packageName, isRecursion);
			} else if (JAR_STR.equals(protocol)) {
				JarFile jarFile = null;
				try{
	                jarFile = ((JarURLConnection) url.openConnection()).getJarFile();
				} catch(Exception e){
					e.printStackTrace();
				}
				if(jarFile != null){
					getClassNameFromJar(jarFile.entries(), packageName, isRecursion);
				}
			}
		} else {
			/*从所有的jar包中查找包名*/
			classNames = getClassNameFromJars(((URLClassLoader)loader).getURLs(), packageName, isRecursion);
		}
		return classNames;
	}

	/**
	 * 从项目文件获取某包下所有类
	 * @param filePath 文件路径
	 * @param isRecursion 是否遍历子包
	 * @return 类的完整名称
	 */
	private static Set<String> getClassNameFromDir(String filePath, String packageName, boolean isRecursion) {
		Set<String> className = new HashSet<>();
		File file = new File(filePath);
		File[] files = file.listFiles();
		if(files==null){return className;}
		for (File childFile : files) {
			if (childFile.isDirectory()) {
				if (isRecursion) {
					className.addAll(getClassNameFromDir(childFile.getPath(), packageName+"."+childFile.getName(), isRecursion));
				}
			} else {
				String fileName = childFile.getName();
				if (fileName.endsWith(".class") && !fileName.contains("$")) {
					className.add(packageName+ "." + fileName.replace(".class", ""));
				}
			}
		}
		return className;
	}

	private static Set<String> getClassNameFromJar(Enumeration<JarEntry> jarEntries, String packageName, boolean isRecursion){
		Set<String> classNames = new HashSet<>();

		while (jarEntries.hasMoreElements()) {
			JarEntry jarEntry = jarEntries.nextElement();
			if(!jarEntry.isDirectory()){
				String entryName = jarEntry.getName().replace("/", ".");
				if (entryName.endsWith(".class") && !entryName.contains("$") && entryName.startsWith(packageName)) {
					entryName = entryName.replace(".class", "");
					if(isRecursion){
						classNames.add(entryName);
					} else if(!entryName.replace(packageName+".", "").contains(".")){
						classNames.add(entryName);
					}
				}
			}
		}
		return classNames;
	}

	/**
	 * 从所有jar中搜索该包，并获取该包下所有类
	 * @param urls URL集合
	 * @param packageName 包路径
	 * @param isRecursion 是否遍历子包
	 * @return 类的完整名称
	 */
	private static Set<String> getClassNameFromJars(URL[] urls, String packageName, boolean isRecursion) {
		Set<String> classNames = new HashSet<>();
		for (URL url : urls) {
			String classPath = url.getPath();
			//不必搜索classes文件夹
			if (classPath.endsWith("classes/")) {
				continue;
			}
			JarFile jarFile = null;
			try {
				jarFile = new JarFile(classPath.substring(classPath.indexOf("/")));
			} catch (IOException e) {
				e.printStackTrace();
			}
			if (jarFile != null) {
				classNames.addAll(getClassNameFromJar(jarFile.entries(), packageName, isRecursion));
			}
		}
		return classNames;
	}
}
```


4. 创建自定义注解类EnableSelfBean
```java
/**
 * 自定义注解类，将某个包下的所有类自动加载到spring 容器中，不管有没有注解，并打印出
 * @author zhangcanlong
 * @since 2019/2/14 10:42
 **/
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(SelfEnableAutoConfig.class)
public @interface EnableSelfBean {
    //传入包名
    String[] packages() default "";
}
```

5. 创建启动类SpringBootEnableApplication
```java
@SpringBootApplication
@EnableSelfBean(packages = "com.kanlon.entity")
public class SpringBootEnableApplication {
    @Autowired
    Role abc;

    public static void main(String[] args) {
        ConfigurableApplicationContext context =  SpringApplication.run(SpringBootEnableApplication.class, args);
        //打印出所有spring中注册的bean
        String[] allBeans = context.getBeanDefinitionNames();
        for(String bean:allBeans){
            System.out.println(bean);
        }
        System.out.println("已注册Role："+context.getBean(Role.class));
        SpringBootEnableApplication application = context.getBean(SpringBootEnableApplication.class);
        System.out.println("Role的测试方法："+application.abc.test());
    }
}
```

启动类测试的一些感悟：重新复习了回了spring的一些基础东西，如：
1. @Autowired是默认通过by type（即类对象）得到注册的类，如果有多个实现才使用by name来确定。
2. 所有注册的类的信息存储在ApplicationContext中，可以通过ApplicationContext得到注册类，这个是很基础的，但是真的很久没看，没想到竟然又忘记了。
3. Spring boot中如果@ComponentScan没有，则默认是指扫描当前启动类所在的包里的对象。

自定义Enable注解源码地址：https://github.com/KANLON/practice/tree/master/spring-boot-enable



参考：
1. http://tangxiaolin.com/learn/show?id=402881d2648c88cc01648c89d8730001
2. SpringBoot @Enable* 注解 https://segmentfault.com/a/1190000015188776
3. 获取指定包名下的所有类的类名（全名) https://my.oschina.net/cnlw/blog/299265)
4. Spring-Boot之@Enable*注解的工作原理 https://www.jianshu.com/p/3da069bd865c
5. Spring源码解析------@Import注解解析与ImportSelector，ImportBeanDefinitionRegistrar以及DeferredImportSelector区别 https://www.xiaoquan.work/articles/2020/01/03/1578016154544.html
6. @import和@Bean的区别，以及ImportSelector和ImportBeanDefinitionRegistrar两个接口的简单实用 https://blog.csdn.net/qq_22701869/article/details/102561494

![CrudBoys](https://img-blog.csdnimg.cn/20201114143851114.png)
