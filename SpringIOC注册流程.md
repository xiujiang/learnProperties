~~~
Spring 是如何通过XML方式启动项目的：
	new XmlBeanFactory(new ClassPathResource("****.xml"))
	1.Spring 的配置文件通过ClassPathResource读取封装，将xml转换为Resource资源
	2.XmlBeanFactory接收到Resource后，调用XmlBeanDefinitionReader.loadBeanDefinitions(Resource)		//加载BeanDefinition
	3.首先对Resource使用EncodedResource封装
	4.调用doLoadBeanDefinitions(InputSource,Resource)
	5.获取Xml文件的验证模式(DTD,XSD)
		  (DTD 获取方式，就是看有没有DOCTYPE，如果有就是DTD，没有就是XSD)
	6.加载XML文件，得到对应的Document
		6.1 EntityResolver 提供一个如何寻找DTD声明的方法
	7.根据返回的Document注册Bean信息
		7.1 调用registerBeanDefinitions(Document doc,Resource resource)
			7.1.1 使用DefaultBeanDefinitionDocumentReader 实例化BeanDefinitionDocumentReader
			7.1.2 设置环境变量
			7.1.3 获取加载Bean的个数
			7.1.4 调用 document.registerBeanDefinitions(doc,createReaderContext(resource))
				 7.1.4.1 得到 Element root 对象， 调用 doRegisterBeanDefinitions(root)
					7.1.4.1.1 处理profile属性
					7.1.4.1.2 在解析前后都有留给子类实现的方法，中间调用 parseBeanDefinitions(root,this.delegate)
						7.1.4.1.2.1 对默认命名空间的处理 调用parseDefaultElement(ele,delegate),默认标签(beans,bean,import,alias)
							7.1.4.1.2.1.1 bean 标签的解析
								7.1.4.1.2.1.1.1 delegate.parseBeanDefinitionElement获取Holder
										7.1.4.1.2.1.1.1.1 解析id,name，生成beanName和alias数组
										7.1.4.1.2.1.1.1.2 调用parseBeanDefinitionElement统一封装到GenericBeanDefinition类型的实例中
											7.1.4.1.2.1.1.1.2.1 解析:class,parent
											7.1.4.1.2.1.1.1.2.2 创建属性承载的GenericBeanDefinition
											7.1.4.1.2.1.1.1.2.3 调用parseBeanDefinitionAttributes解析element
												7.1.4.1.2.1.1.1.2.3.1 解析scope,singleton,abstract,lazy-init,autowire,dependcy-check,depends-on,autowire-candidate,primary,init-method,destory-method,factory-method,factory-bean
											7.1.4.1.2.1.1.1.2.4 解析子元素meta
											7.1.4.1.2.1.1.1.2.5 解析lookup-method
											7.1.4.1.2.1.1.1.2.6 解析子元素replaced-method
											7.1.4.1.2.1.1.1.2.7 解析子元素Constructor-Arg,遍历所有Node 节点，调用parseConstructorArgElement
												7.1.4.1.2.1.1.1.2.7.1 提取index,type,name
												7.1.4.1.2.1.1.1.2.7.2 有index 则直接封装在ConstructorArgumentValues.ValueHolder，并添加至BeanDefinition的constructorArgumentValues的indexArgumentValue属性中	
												7.1.4.1.2.1.1.1.2.7.3 没有index,解析子元素，使用ConstructorArgumentValues.ValueHolder封装解析的元素，然后封装type,index,name,并保存在BeanDefinition的constructorArgusmentValues的genericArgusmentValues
											7.1.4.1.2.1.1.1.2.8 解析子元素qualifier	
										7.1.4.1.2.1.1.1.3 检测到bean没有beanName，使用默认规则生成beanName
										7.1.4.1.2.1.1.1.4 将获取到的信息封装到BeanDefinitionHolder实例中返回。
								7.1.4.1.2.1.1.2 bdHolder不为空，若有自定义属性，则调用decorateBeanDefinitionIfRequired解析
									7.1.4.1.2.1.1.2.1 对节点调用decorateIfRequired
										7.1.4.1.2.1.1.2.1.1 获取自定义标签命名空间
										7.1.4.1.2.1.1.2.1.2 找到自定义标签所属的NamespaceHandler并进行进一步解析 
								7.1.4.1.2.1.1.3 解析结束，注册BeanDefinition， BeanDefinitionReaderUtils.registerBeanDefinition()
									7.1.4.1.2.1.1.3.1 使用BeanName注册BeanDefinition
										7.1.4.1.2.1.1.3.1.1 校验AbstractBeanDefinition属性中的MethodOverrides 是否与工厂方法并存或者methodOverides对应的方法不存在
										7.1.4.1.2.1.1.3.1.2 beanDefinitionMap尝试取beanName对应的BeanDefinition 如果存在，则判断是否可以覆盖
										7.1.4.1.2.1.1.3.1.3 如果不存在，加锁BeanDefinitionMap,put进去，添加BeanName到BeanDefinitionNames，如果manualSingletonNames有BeanName，就移除updatedSingletons的BeanName
										7.1.4.1.2.1.1.3.1.4 重置所有BeanName对应的缓存
									7.1.4.1.2.1.1.3.2 注册所有别名aliases
										7.1.4.1.2.1.1.3.2.1 校验别名，是否与BeanName相同，是否
										7.1.4.1.2.1.1.3.2.2 alias 循环校验
										7.1.4.1.2.1.1.3.2.3 注册alias
								7.1.4.1.2.1.1.4 通知监视器，加载完成。
							7.1.4.1.2.1.2 alias 标签的解析
							7.1.4.1.2.1.3 import 标签的解析
								7.1.4.1.2.1.3.1 获取resource 属性
								7.1.4.1.2.1.3.2 解析路径中的系统属性："${user.dir}"
								7.1.4.1.2.1.3.3 路径为绝对路径的话，递归调用bean的解析过程
								7.1.4.1.2.1.3.4 路径为相对路径的话，计算成绝对路径解析
								7.1.4.1.2.1.3.5 通知监听器，解析完成
							7.1.4.1.2.1.4 beans 标签的解析
								7.1.4.1.2.1.4.1 递归调用beans的解析过程
						7.1.4.1.2.2 对其他空间的处理 delegate.parseCustomElement(ele) 自定义标签
							7.1.4.1.2.2.1 获取命名空间名称
							7.1.4.1.2.2.2 调用this.readerContext.getNameSpaceHandlerResolver 根据命名空间名称找到相应的Handler解析器
								7.1.4.1.2.2.2.1 获取所有已经配置的handler映射 getHandlerMappings()
									7.1.4.1.2.2.2.1.1 如果没有缓存，则读取Spring.handlers配置文件，并缓存到map中
								7.1.4.1.2.2.2.2 如果解析过，直接返回NameSpaceHandler
								7.1.4.1.2.2.2.3 没有做过解析，拿到的是类路径，反射获取到NamespaceHandler，调用init方法进行自定义BeanDefinitionParser的注册，记录到缓存
							7.1.4.1.2.2.3 调用自定义的解析器parse方法
								7.1.4.1.2.2.3.1 获取元素名称
								7.1.4.1.2.2.3.2 在Parsers中找到元素对应的parser并返回
							
			7.1.5 记录加载BeanDefinition的个数
			
			
getBean() 通过getBean来加载bean
	1.doGetBean
		1.1 转换name,获取beanName name可能是beanName，也可能是alias,也有可能是FactoryBean
		1.2 尝试从缓存中获取Object getSingleton()
			1.2.1 检查换成是否有实例 singletonFactories
			1.2.2 如果是空，锁定全局变量singletonObjects，加载实例
				1.2.2.1 判断earlySingletonObjects，当前bean是否正在加载
				1.2.2.2 未加载，判断singletonFactories,如果不为空，调用预先设定的getObject方法，记录到缓存earlySingletonObjects
			1.2.3 如果不为空，返回
		1.3 存在，调用getObjectForBeanInstance 返回bean实例，如果是FactoryBean 则解析并返回bean实例
			1.3.1 如果指定的name 是工厂相关(&为前缀) 且beanInstance又不是FactoryBean的类型则验证失败
			1.3.2 正常bean 返回
			1.3.3 FactoryBean 首先尝试缓存中加载
			1.3.4 加载不到，之前的GernericBeanDefinition转换为RootBeanDefinition，如果指定BeanName是子Bean则同时会合并父类相关属性
			1.3.5 getObjectFromFactoryBean-doGetObjectFromFactoryBean
				1.3.5.1 调用getBean方法
				1.3.5.2 调用postProcessObjectFromFactoryBean 后处理器（Spring获取bean的规则中：尽可能保证所有Bean初始化后都调用注册的后置处理器处理）
		1.4 不存在
			1.4.1 检查prototype是否有循环依赖，只有单例才会处理循环依赖
			1.4.2 在已经加载bean中找不到，则父类中尝试加载bean
			1.4.3 不是类型检查，则记录bean
			1.4.4 将之前的GernericBeanDefinition转换为RootBeanDefinition，如果指定BeanName是子Bean则同时会合并父类相关属性
			1.4.5 如果有dependsOn ，存在依赖，则递归实例化调用
				1.4.5.1 getBean
				1.4.5.2 缓存依赖调用
			1.4.6 单例模式
				1.4.6.1 获取单例的实例 getSingleton(){createBean()}
					1.4.6.1.1 锁singletonObject，如果不为空直接返回
					1.4.6.1.2 检查循环依赖
					1.4.6.1.3 加载单例前记录加载状态
					1.4.6.1.4 调用ObjectFactory的getObject实例化Bean 
					1.4.6.1.5 加载单例后的处理调用
					1.4.6.1.6 记录结果到缓存并删除singletonFactories,earlySingletonObjects
					1.4.6.1.7 返回处理结果
				1.4.6.2 获取createBean()
					1.4.6.2.1 解析BeanClass,根据设置的Class属性或者ClassName来解析Class
					1.4.6.2.2 对override属性进行标记和验证 prepareMethodOverrides() 
						1.4.6.2.2.1 如果对应类中方法个数为1 那么标记MethodOverride暂未被覆盖，避免参数类型检查的开销
					1.4.6.2.3 对BeanDefinition做前置处理，给BeanPostProcessors一个机会返回代理来替代真正的实例 resolveBeforeInstantion(beanName,mdb)
						1.4.6.2.3.1 bean = applyBeanPostProcessorsBeforeInstantiation()
							1.4.6.2.3.1.1 这个方法调用后，beanDefinition已经被修改，bean可能已经被代理（AOP），调用InstantiationAwareBeanPostProcessor 的后处理器进行PostProcessBeforeInstantiation
						1.4.6.2.3.2 如果bean不为空的话，调用 applyBeanPostProcessorsAfterInitialization ，AOP的功能就是这里判断的,bean创建规则，会尽可能调用后置处理器
							1.4.6.2.3.2.1 调用BeanPostProcessor的postProcessAfterInitialization
					1.4.6.2.4 doCreateBean() 常规创建Bean
						1.4.6.2.4.1 如果是单例，则先清除缓存
						1.4.6.2.4.2 createBeanInstance 实例化bean，将beanDefinition转换为BeanWrapper 转换方式（工厂、构造函数、默认构造函数）
							1.4.6.2.4.2.1 解析class
							1.4.6.2.4.2.2 如果工厂方法不为空，则使用工厂方法创建 instantiateUsingFactoryMethod()
							1.4.6.2.4.2.3 根据resolvedConstructorOrFactoryMethod 判断是否需要查找使用哪个构造函数 
							1.4.6.2.4.2.4 如果解析过，并且可以注入，则使用 autowireConstructor 带参数的实例化
								1.4.6.2.4.2.4.1 构造函数参数的确定
									1.4.6.2.4.2.4.1.1 根据explicitArgs 判断是否有参数，如果有，则直接确定参数 explicitArgs 是getBean（）用户自定义的
									1.4.6.2.4.2.4.1.2 缓存中获取参数
									1.4.6.2.4.2.4.1.3 配置文件获取
								1.4.6.2.4.2.4.2 构造函数的确定 先根据参数个数匹配，对method根据public降序，非public方法构造参数数量降序，然后参数名称的获取：注解，工具类ParameternameDiscoverer
								1.4.6.2.4.2.4.3 根据确定的构造函数转换对应的参数类型
								1.4.6.2.4.2.4.4 构造函数不确定性的验证 例：不同构造函数参数为父子关系
								1.4.6.2.4.2.4.5 根据实例化策略以及得到的构造函数及构造参数实例化Bean
									1.4.6.2.4.2.4.5.1 SimpleInstanceStrategy 实例化策略，如果有需要动态改变的方法（用户是否使用replace,lookup属性），则调用cglib动态代理，如果不需要，则直接反射生成instance
							1.4.6.2.4.2.5 否则使用instantiateBean 默认构造函数解析
								
						1.4.6.2.4.3 MergedBeanDefinitionPostProcessor 合并后的处理 Autowired就是通过此方法实现诸如类型的预解析
						1.4.6.2.4.4 循环依赖处理 addSingletonFactory 添加ObjectFactory到SingletonFactory中
							1.4.6.2.4.1 AOP的操作就是在这里执行的
						1.4.6.2.4.5 属性填充 将所有属性填充到bean实例中
							1.4.6.2.4.5.1 如果有属性可以填充，先给InstantiationAwareBeanPostProcessors 最后一次机会改变bean,可控制是否进行属性填充
							1.4.6.2.4.5.2 如果是根据名称自动注入，调用autowireByName(),存入PropertyValues
								1.4.6.2.4.5.2.1 寻找需要依赖注入的属性
								1.4.6.2.4.5.2.2 循环所有属性
									1.4.6.2.4.5.2.2.1 递归调用getBean()
									1.4.6.2.4.5.2.2.2 注册依赖 registerDependentBean()
							1.4.6.2.4.5.3 如果是根据类型自动注入，调用autowireByType(),存入PropertyValues
								1.4.6.2.4.5.3.1 获取类型解析器
								1.4.6.2.4.5.3.2 获取需要依赖注入的属性
								1.4.6.2.4.5.3.3 循环所有属性
									1.4.6.2.4.5.3.3.1 探测指定属性的set方法
									1.4.6.2.4.5.3.3.2 解析指定beanName属性所匹配的值，并把解析到的属性名称存储到autowiredBeanNames里面，当属性存在多个封装bean时，会找到所有匹配的将其注入 resolveDependency
									1.4.6.2.4.5.3.3.3 如果匹配参数不为空，添加到参数列表中
									1.4.6.2.4.5.3.3.4 对匹配参数做循环，注册registerDependentBean()
							1.4.6.2.4.5.4 如果后置处理器已经初始化，对所有需要依赖检查的属性进行处理
							1.4.6.2.4.5.5 如果需要依赖检查，依赖检查，对应depends-on属性，3.0已经弃用该属性
							1.4.6.2.4.5.6 将属性应用到bean中， applyPropertyValues()
								1.4.6.2.4.5.6.1 如果有设置了spel表达式，就是在这里通过AbstractBeanFactory的evaluateBeanDefinitionString进行表达式的解析
						1.4.6.2.4.6 如果有init-method ，则进行初始化 initializeBean()
							1.4.6.2.4.6.1 激活Aware方法invokeAwareMethods (如果是BeanNameAware,BeanClassLoaderAware,BeanFactoryAware,则注入beanName,,ClassLoader,beanFactory)(实现Arare的接口，在bean初始化后，会注入一些特定实例)
							1.4.6.2.4.6.2 处理器BeanPostProcessor的应用，在实例化init方法前调用postProcessBeforeInitialization
							1.4.6.2.4.6.3 invokeInitMethods() 激活自定义init方法
								1.4.6.2.4.6.3.1 如果自定义方法实现了InitializingBean接口，就会有afterPropertiesSet方法，则先执行这个方法
								1.4.6.2.4.6.3.2 调用自定义初始化方法
							1.4.6.2.4.6.4 实例化后调用 postProcessAfterInitialization
						1.4.6.2.4.7 循环依赖检查
						1.4.6.2.4.8 注册DisposableBean 如果配置了destory-method 这里需要注册以便销毁的时候调用
							1.4.6.2.4.8.1 除了配置destory-method，还有注册后处理器DestructionAwareBeanPostProcessor统一处理销毁方法
						1.4.6.2.4.9 返回bean
					1.4.6.2.5 返回bean
				1.4.6.2 getObjectForBeanInstance 如果是BeanFactory的，则解析，返回bean
					
			1.4.7 prototype 模式
				1.4.7.1 beforePrototypeCreation(beanName)
				1.4.7.2 createBean()
				1.4.7.3 afterPrototypeCreation(beanName)
				1.4.7.4 getObjectForBeanInstance()
			1.4.8 其他模式
				1.4.8.1 根据scopeName获取Scope
				1.4.8.2 根据beanName和ObjectFactory在Scope中获取Instance
					1.4.8.2.1 ObjectFactory 如同protoptye一样 beforePrototypeCreation,createBean,afterPrototypeCreation
				1.4.8.3 getObjectForBeanInstance()
		1.5 检查需要的类型是否与bean的类型相同
			1.3.1 不同的话，调用convertIfNecessary() 转换类型
	

ClassPathXmlApplicationContext
	1.setConfigLocations 设置配置路径
		1.1 根据传入的多个路径，循环解析路径 resolvePath()
	2.refresh()
		2.1 prepareRefresh() 准备刷新的上下文环境，对系统属性或者环境变量准备及验证
			2.1.1 initPropertySources() 留给子类覆盖，系统未做处理，
			2.1.2 验证需要的属性文件是否已经都在环境中，也由用户自定义处理
		2.2 obtainFreshBeanFactory() 初始化BeanFactory 并进行Xml读取
			2.2.1 初始化 BeanFactory，并进行XML文件读取，并将得到的BeanFact鱼肉记录在当前实体的属性中 refreshBeanFactory()
				2.2.1.1 如果当前Beanfactory已经存在，就先销毁Bean，关闭beanFactory
				2.2.1.2 创建DefaultListableBeanFactory  createBeanFactory()
				2.2.1.3 为序列化指定id,如果需要的话，让这个BeanFactory从id反序列化到BeanFactory对象
				2.2.1.4 定制beanFactory，设置相关属性，包括是否允许覆盖同名的对象以及循环依赖和设置@Autowired @Qualifier注解解析器  customizeBeanFactory()
					2.2.1.4.1 允许覆盖和允许依赖如果不为空，则设置 
					2.2.1.4.2 设置AutowireCandidateResolver ,会注入QualifierAnnotationAutowireCandidateResolver的解析器 2.0没有该方法
				2.2.1.5 初始化DocumentReader，也就是加载BeanDefinition 并进行Xml 文件读取及解析
					2.2.1.5.1 初始化XmlBeanDefinitionReader 
					2.2.1.5.2 设置环境变量 
					2.2.1.5.3 读取xml文件，跟之前BeanFactory的读取方式一样
				2.2.1.6 使用全局变量记录BeanFactory类实例
			2.2.2 返回当前实体的beanFactory属性 到这一步，已经完成了beanFactory的解析，下面也就是ApplicationContext的高级功能
		2.3 prepareBeanFactory() 对BeanFactory进行功能填充 @Qualifier @Autowired
			2.3.1 设置ClassLoader
			2.3.2 设置spel解析器
			2.3.3 添加bean属性设置管理的工具 当某些类型在xml中无法被解析为正确格式，可以设置自定义属性编辑器来解决，如：类型为Date，但xml为String类型
			2.3.4 添加ApplicationContextAwareProcessor 
				2.3.4.1 在bean init的时候会调用BeanPostProcessor的 postProcessBeforeInitialization 和after的方法，这里就是到时候会调用的方法 内部会接着调用invokeAwareInterfaces 方法
			2.3.5 设置忽略自动装配的接口
				2.3.5.1 在注册processor 后，在invokeAwareInterfaces调用Aware类已经不是不同bean,就需要在这里忽略它们
			2.3.6 设置自动装配的特殊规则 当注册了依赖解析后，例如当注册了对BeanFactory.class的解析依赖后，当bean的属性注入的时候，一旦检测到属性为BeanFactory类型便会将Beanfactory实例注入进去
			2.3.7 增加对AspectJ的支持
			2.3.8 将相关环境变量及属性注册以单例模式注册
			
		2.4 postProcessBeanFactory() 子类覆盖方法做额外处理
			2.4.1 激活注册的BeanfactoryPostProcessor 
		2.5 invokeBeanFactoryPostProcessors() 激活各种BeanFactory处理器
			2.5.1 对BeanDefinitionRegister类型做处理
				2.5.1.1 对于硬编码注册的后处理器的处理
			2.5.2 对配置中读取的BeanFactoryPostProcessor的处理
		2.6 registerBeanPostProcessors() 注册拦截Bean创建的Bean处理器，只是注册，真正的调用在getBean
			2.6.1 注册所有实现PriorityOrdered的BeanPostProcessor
			2.6.2 注册所有实现Ordered的BeanPostProcessor
			2.6.3 注册所有无序的BeanPostProcessor
			2.6.4 注册所有MergedBeanDefinitionPostProcessor类型的BeanPostProcessor，并非重复注册
			2.6.5 添加ApplicationListener探测器
		2.7 initMessageSource() 为上下文初始化Message源，即不同语言的消息体，国际化处理
			2.7.1 主要为提取配置中定义的messageSource，并将其记录在Spring的容器中，也就是AbstractApplicationContext，定义资源文件必须为messageSource,否则抛异常
			2.7.2 如果已经配置了messageSource,那么将messageSource提取并记录在this.messageSource中
			2.7.3 如果没有定义配置文件，那么使用临时的DelegatingMessageSource作为调用getMessage方法的返回。
		2.8 initApplicationEventMulticaster() 初始化应用消息广播器，并放入“applicationEventMulticaster”bean中
			2.8.1 如果自定义了事件广播器，则使用用户自定义的
			2.8.2 如果没有自定义，则使用默认的ApplicationEventMulticaster
				2.8.2.1 SimpleApplicationEventMulticaster 当产生事件后，会默认使用multicastEvent进行广播，遍历所有监听器，使用监听器中的onApplicationEvent来进行监听器的处理。每个监听器都能获得event，由监听器决定是否处理
		2.9 onRefresh()	留给子类来初始化其他的Bean
		2.10 registerListeners() 在所有注册的bean中查找Listener bean 注册到消息广播器中
		2.11 finishBeanFactoryInitialization() 初始化剩下的单实例（非惰性的） 包括ConversionService设置，配置冻结以及非延迟加载的bean的初始化工作
			2.11.1 如果设置了conversionService，则设置ConversionService
			2.11.2 如果没有内置的valueResolver 则注册
			2.11.3 冻结所有Bean定义，说明注册的bean定义将不能被修改或进行其他处理
			2.11.4 初始化非延迟加载 调用getBean加载
		2.12 finishRefresh() 完成刷新过程，通知生命周期处理器 lifecycleProcessor 刷新过程，同时发出ContextRefreshEvent通知别人
			2.12.1 初始化initLifecycleProcessor 
			2.12.2 onRefresh 启动所有实现Lifecycle接口的bean
			2.12.3 发出ContextRefreshedEvent事件，以保证对应的监听器可以做进一步逻辑处理
		2.13 在过程中失败后
			2.13.1 destroyBeans() 	销毁已经创建的bean
			2.13.2 cancelRefresh()	取消刷新
	
	
	
	
	
	
明白：
	1.BeanFactoryPostProcessor
		BeanFactoryPostProcessor跟BeanPostProcessor类似，可以对Bean的定义(配置元数据)进行处理。也就是说，IOC容器允许BeanFactoryPostProcessor在容器实际实例化任何其他bean之前读取配置元数据
	并有可能修改它，还可以通过设置order属性控制BeanFactoryPostProcessor的执行顺序。
		如果你想改变Bean实例，那么最好用BeanPostProcessor,BeanFactoryPostProcessor作用域范围是容器级的，他只是和你使用的容器有关，如果在容器中定义后，只是对当前容器bean进行后置处理，不会对其他容器中bean有影响
		典型应用：PropertyPlaceholderConfigurer
	2.BeanFactory 和 FactoryBean
	

~~~

