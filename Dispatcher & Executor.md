#  Dispatcher & Executor

​	分发器和执行器采用：

## 1.Dispatcher

	### AbstractDispatcher

​	实现了ApplicationEventPublisherAware（需实现ApplicationEventPublisher方法，由ApplicationContext调用并实现）

```java
@Autowired
private RabbitHelper rabbitHelper;
@Autowired
private RedisTemplate redisTemplate;
@Autowired
@Qualifier("mvcConversionService")
private ConversionService conversionService;
@Autowired
ScheduledThreadPoolExecutor executor;	//定时线程池
private ApplicationEventPublisher applicationEventPublisher;	//应用事件发布者
private final MessageProperties defaultMessageProperties = new MessageProperties();	//消息参数
private final String exchangeName;	//通道名
private final String route;			//路由名
private final String redisPrifix;
private final Integer currentMax;
private final Integer messageTtlInSeconds;
private final String keyPattern;
private final AbstractDispatcher.LoadFunction<DATA, KEY> loadFunction;
private final Function<DATA, KEY> keyGetter;
private final Class<KEY> keyClass;
private final Method keyMethod;
private final String messageType;	//消息类型
```

dispatcher分为主动调用和定时调用

主动调用为：`offer()`和`publish()`

定时调用为 `dispatch()`

当调用offer()时，会调用dispatchSingle方法，通过rabbit发送

```java
public void offer(DATA data) {
    this.offerInternal(data);
}

  private void dispatchSingle(DATA data) {
        try {
            //生成分发key值
            String dispatchKey = (String)this.conversionService.convert(this.keyGetter.apply(data), String.class);
            dispatchKey = this.redisPrifix + dispatchKey;
            //放入redis,并设置过期时间
            this.redisTemplate.opsForValue().set(dispatchKey, data, (long)(this.messageTtlInSeconds + RandomUtils.nextInt(60)), TimeUnit.SECONDS);
            //通过rabbit发送消息到队列
            this.rabbitHelper.send(this.exchangeName, this.route, data, this.messageType);
        } catch (Exception var3) {
            this.logger.error("发送消息发生未知异常", var3);
        }

    }
```

~~~  java
//publish 方法，会通过事件的方式处理请求，TransactionEvent是匿名内部类 继承了ApplicationEvent事件，
public void publish(DATA data) {
        this.applicationEventPublisher.publishEvent(new AbstractDispatcher.TransactionEvent(data, this));
    }
~~~

​	**ApplicationContext在运行期会自动检测到所有实现了ApplicationListener的bean对象，并将其作为事件接收对象。当ApplicationContext的publishEvent方法被触发时，每个实现了ApplicationListener接口的bean都会收到ApplicationEvent对象，每个ApplicationListener可根据事件类型只接收处理自己感兴趣的事件**

`dispatch()`	

current() 从redis中获取存放的id 集合

执行loadFunction函数，也就项目实现的`toDispatch`方法，获取数据库中相应的data数据

~~~JAVA
protected void dispatch() {
        List<KEY> current = this.current();	//从redis获取id集合
        if (this.currentMax <= 0 || current.size() < this.currentMax) {
            List<DATA> todo = this.load(current);	//获取data数据
            if (Objects.nonNull(todo) && todo.size() > 0) {
                todo.forEach(this::dispatchSingle);
            }

        }
    }
   
~~~

## 2.Executor

执行由dispatcher分发来的数据

```java
@Autowired
private RedisTemplate redisTemplate;
@Autowired
@Qualifier("mvcConversionService")
private ConversionService conversionService;
@Autowired
PlatformTransactionManager transactionManager;		//事务管理器
@Autowired
ListenerRegister listenerRegister;		//注册监听器
private final Queue targetQueue;		
private final String dispatchPrefix;
private final String lockPrefix;
private final Integer lockSeconds;
```



第一步，首先做初始化操作init(),将targetQueue和messageHandler方法传入，如果有消息过来，则执行messageHandler方法。

```java
@PostConstruct
public void init() {    this.listenerRegister.addListener(this.targetQueue,this::messageHandler);//autostart
}
```

`checkStatus()`,`checkExpire()`,`UpdateTargetStatus()`,`handler()`这几个方法是可以重写的， **handler()**方法是执行executor必须要重写的。



## 3.ListenerRegister

注册监听器

addListener 将队列和method添加到SimpleMessageListenerContainer中

```java
public <X> void addListener(Queue targetQueue, Consumer<X> method, boolean autoStart) {
    SimpleHandler<X> handler = new SimpleHandler<>(method, this.rabbitHelper);
    SimpleMessageListenerContainer container = this.create(handler, this.simpleHandlerMethod, targetQueue);
    this.listeners.put(targetQueue.getName(), container);
    if (autoStart) {
        this.start(container);
    }
}
```

SimpleMessageListenerContainer：最简单的消息监听器容器，只能处理固定数量的JMS会话，且不支持事务。





![1539596745193](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1539596745193.png)