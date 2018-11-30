spring的事件机制是基于观察者设计模式的，`ApplicationListener#onApplicationEvent(Event)`方法，用于对事件的处理 。在容器初始化的时候执行注册到容器中的Listener。逆向来查看执行过程

~~~java
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
	/**
	 * Handle an application event.
	 * @param event the event to respond to
	 */
	void onApplicationEvent(E event);
}
~~~

`SimpleApplicationEventMulticaster#multicastEvent`执行调用所有实现了`ApplicationListener`接口的Bean.

~~~java
/*
*广播所有注册监听的事件
*设置个Excutor可以异步执行
*/
public class SimpleApplicationEventMulticaster extends AbstractApplicationEventMulticaster {
    private Executor taskExecutor;//可以异步执行listener
    ......
    @Override
	@SuppressWarnings({ "unchecked", "rawtypes" })
	public void multicastEvent(final ApplicationEvent event) {
        //遍历执行listener，getApplicationListeners调用AbstractApplicationEventMulticaster父类方法
		for (final ApplicationListener listener : getApplicationListeners(event)) {
			Executor executor = getTaskExecutor();
			if (executor != null) {
				executor.execute(new Runnable() {
					@Override
					public void run() {
						listener.onApplicationEvent(event);
					}
				});
			}
			else {
				listener.onApplicationEvent(event);
			}
		}
	}
}
~~~

父类`AbstractApplicationEventMulticaster#getApplicationListeners()`部分源码：

~~~java
/*
*抽象AbstractApplicationEventMulticaster 事件广播者
*/
public abstract class AbstractApplicationEventMulticaster
		implements ApplicationEventMulticaster, BeanClassLoaderAware, BeanFactoryAware {
	.....
    //实现此BeanFactoryAware接口，初始化时，将会将beanFactory注入到该实例中。便于后续获取bean
   	private BeanFactory beanFactory;
    @Override
	public void setBeanFactory(BeanFactory beanFactory) {
		this.beanFactory = beanFactory;
		if (this.beanClassLoader == null && beanFactory instanceof ConfigurableBeanFactory) 		{
			this.beanClassLoader = ((ConfigurableBeanFactory) beanFactory).getBeanClassLoader();
		}
	}
    ......
    /*
    *获取容器监听
    */
	protected Collection<ApplicationListener<?>> getApplicationListeners(ApplicationEvent event) {
		Class<? extends ApplicationEvent> eventType = event.getClass();
		Object source = event.getSource();	
		Class<?> sourceType = (source != null ? source.getClass() : null);
		ListenerCacheKey cacheKey = new ListenerCacheKey(eventType, sourceType);
		ListenerRetriever retriever = this.retrieverCache.get(cacheKey);
		if (retriever != null) {
			return retriever.getApplicationListeners();
		}
		else {
			retriever = new ListenerRetriever(true);
			LinkedList<ApplicationListener<?>> allListeners = new LinkedList<ApplicationListener<?>>();
			Set<ApplicationListener<?>> listeners;
			Set<String> listenerBeans;
			synchronized (this.defaultRetriever) {
				listeners = new LinkedHashSet<ApplicationListener<?>>(this.defaultRetriever.applicationListeners);
				listenerBeans = new LinkedHashSet<String>(this.defaultRetriever.applicationListenerBeans);
			}
			for (ApplicationListener<?> listener : listeners) {
				if (supportsEvent(listener, eventType, sourceType)) {
					retriever.applicationListeners.add(listener);
					allListeners.add(listener);
				}
			}
			if (!listenerBeans.isEmpty()) {
				BeanFactory beanFactory = getBeanFactory();
				for (String listenerBeanName : listenerBeans) {
					try {
						Class<?> listenerType = beanFactory.getType(listenerBeanName);
						if (listenerType == null || supportsEvent(listenerType, event)) {
							ApplicationListener<?> listener =
									beanFactory.getBean(listenerBeanName, ApplicationListener.class);
							if (!allListeners.contains(listener) && supportsEvent(listener, eventType, sourceType)) {
								retriever.applicationListenerBeans.add(listenerBeanName);
								allListeners.add(listener);
							}
						}
					}
					catch (NoSuchBeanDefinitionException ex) {
						// Singleton listener instance (without backing bean definition) disappeared -
						// probably in the middle of the destruction phase
					}
				}
			}
			OrderComparator.sort(allListeners);
			if (this.beanClassLoader == null ||
					(ClassUtils.isCacheSafe(eventType, this.beanClassLoader) &&
							(sourceType == null || ClassUtils.isCacheSafe(sourceType, this.beanClassLoader)))) {
				this.retrieverCache.put(cacheKey, retriever);
			}
			return allListeners;
		}
	}
....
}
~~~

`AbstractApplicationContext#publishEvent()`调用发布事件源码

~~~java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
implements ConfigurableApplicationContext, DisposableBean{
	......
	public static final String APPLICATION_EVENT_MULTICASTER_BEAN_NAME = "applicationEventMulticaster";
	......
	/**
	 * 初始化事件广播器，如果未配置，默认使用SimpleApplicationEventMulticaster
	 *这个何时执行？稍后将会贴出代码
	 */
	protected void initApplicationEventMulticaster() {
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
			this.applicationEventMulticaster =
					beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
			if (logger.isDebugEnabled()) {
				logger.debug("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
			}
		}
		else {
			this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
			beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
			if (logger.isDebugEnabled()) {
				logger.debug("Unable to locate ApplicationEventMulticaster with name '" +
						APPLICATION_EVENT_MULTICASTER_BEAN_NAME +
						"': using default [" + this.applicationEventMulticaster + "]");
			}
		}
	}
	......
	//获取容器事件广播器
	ApplicationEventMulticaster getApplicationEventMulticaster() throws IllegalStateException 		{
		if (this.applicationEventMulticaster == null) {
			throw new IllegalStateException("ApplicationEventMulticaster not initialized - " + "call 'refresh' before multicasting events via the context: " + this);
		}
		return this.applicationEventMulticaster;
	}
	......
	@Override
	public void publishEvent(ApplicationEvent event) {
		Assert.notNull(event, "Event must not be null");
		if (logger.isTraceEnabled()) {
			logger.trace("Publishing event in " + getDisplayName() + ": " + event);
		}
		//获取到事件广播器，发布事件
		getApplicationEventMulticaster().multicastEvent(event);
		//如果存在父容器，父容器也将发布事件	
		if (this.parent != null) {
			this.parent.publishEvent(event);
		}
	}
	......
}
~~~

`AbstractApplicationContext#finishRefresh()`源码

~~~java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
implements ConfigurableApplicationContext, DisposableBean{
	......
	protected void finishRefresh() {
		// Initialize lifecycle processor for this context.
		initLifecycleProcessor();

		// Propagate refresh to lifecycle processor first.
		getLifecycleProcessor().onRefresh();

		// Publish the final event. 发布事件
		publishEvent(new ContextRefreshedEvent(this));

		// Participate in LiveBeansView MBean, if active.
		LiveBeansView.registerApplicationContext(this);
	}
	......
}
~~~

`AbstractApplicationContext#refresh()`源码

~~~java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
implements ConfigurableApplicationContext, DisposableBean{
	.....
	@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.初始化事件广播器
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.完成refresh
				finishRefresh();
			}

			catch (BeansException ex) {
				logger.warn("Exception encountered during context initialization - cancelling refresh attempt", ex);

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}
		}
	}
	.....
}

~~~

spring容器启动将会执行到`AbstractApplicationContext#refresh()`。由此可以看到spring事件的流程。

下面写了一个demo看看，如何实现事件的异步。

~~~java
/**
 * @Auther: lanwei
 * @Date: 2018/11/1 23:38
 * @Description:
 */
public class ApplicationListenerAsynTest {
    public static void main(String[] args) {
        final AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.addApplicationListener(new ApplicationListener<ApplicationEvent>() {
            @Override
            public void onApplicationEvent(ApplicationEvent event) {
                println(event.getSource().toString());
            }
        });
        context.register(MyConfiguration.class);
        context.refresh();
        context.close();
    }

    @Configuration
    public static class MyConfiguration {

        @Bean
        public SimpleApplicationEventMulticaster applicationEventMulticaster() {
            SimpleApplicationEventMulticaster multicaster = new SimpleApplicationEventMulticaster();
            multicaster.setTaskExecutor(new ThreadPoolExecutor(5, 10, 60, TimeUnit.SECONDS, new ArrayBlockingQueue<>(10)));
            return multicaster;
        }
    }

    private static void println(String message) {
        System.out.printf("[线程 : %s] %s\n",
                Thread.currentThread().getName(), // 当前线程名称
                message);
    }
}
~~~

