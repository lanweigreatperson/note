Spring Boot提供了两个接口`ApplicationRunner `和`CommandLineRunner`用于`SpringApplication`启动后运行一些特定的代码， 只需要实现他们的`run`方法就可以了。详细看下面的贴出的源码，注意参数。如果存在多个实现了接口的bean，并且要求有顺序的话，需要实现`org.springframework.core.Ordered `接口或者使用`org.springframework.core.annotation.Order `注解。

`ApplicationRunner `源码：

~~~java
/**
*@FunctionalInterface是jdk8后引入的，网上有很多解释。lambda表达式
*
*@since 1.3.0 spring boot1.3后支持
*/
@FunctionalInterface
public interface ApplicationRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming application arguments
	 * @throws Exception on error
	 */
	void run(ApplicationArguments args) throws Exception;

}
~~~

`CommandLineRunner`源码：

~~~java
@FunctionalInterface
public interface CommandLineRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming main method arguments
	 * @throws Exception on error
	 */
	void run(String... args) throws Exception;

}
~~~

以下是源码分析：`SpringApplication#run`启动spring应用核心方法，spring这些源码方法名字起的相当牛逼。看名字就只其意。通过下面这段源码我们将会看到实现接口后的bean，是在`ApplicationStartedEvent`之后`ApplicationReadyEvent`事件之前执行

~~~java
public class SpringApplication {
    ...
	public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(
					SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass)
						.logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
            //这里调用Runner，就是实现了ApplicationRunner和CommandLineRunner接口的代码逻辑。在容器refresh完成后调用。由此也将会看到他是处于事件ApplicationStartedEvent之后ApplicationReadyEvent事件之前
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
    
    private void callRunners(ApplicationContext context, ApplicationArguments args) {
		List<Object> runners = new ArrayList<>();
        //从容器中获取实现ApplicationRunner接口的beans
		runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
        //从容器中获取实现CommandLineRunner接口的beans
		runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
        //对bean进行排序
		AnnotationAwareOrderComparator.sort(runners);
        //循环调用
		for (Object runner : new LinkedHashSet<>(runners)) {
			if (runner instanceof ApplicationRunner) {
				callRunner((ApplicationRunner) runner, args);
			}
			if (runner instanceof CommandLineRunner) {
				callRunner((CommandLineRunner) runner, args);
			}
		}
	}
    
    private void callRunner(ApplicationRunner runner, ApplicationArguments args) {
		try {
			(runner).run(args);
		}
		catch (Exception ex) {
			throw new IllegalStateException("Failed to execute ApplicationRunner", ex);
		}
	}

	private void callRunner(CommandLineRunner runner, ApplicationArguments args) {
		try {
			(runner).run(args.getSourceArgs());
		}
		catch (Exception ex) {
			throw new IllegalStateException("Failed to execute CommandLineRunner", ex);
		}
	}
    ......
}

~~~

