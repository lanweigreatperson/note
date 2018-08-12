`SpringApplication`试图为您创建正确的`ApplicationContext`类型。`WebApplicationType`主要是用于确定Springboot为我们启动什么样的容器的，具体看`WebApplicationType`枚举类型。确定`WebApplicationType`的办法相当简单.源码如下：

~~~java
public class SpringApplication {
	......
    //构造SpringApplication
	@SuppressWarnings({ "unchecked", "rawtypes" })
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
        //deduceWebApplicationType方法是推出WebApplicationType
		this.webApplicationType = deduceWebApplicationType();
		setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
	}
    //查看逻辑就清楚了
    private WebApplicationType deduceWebApplicationType() {
		if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
				&& !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)
				&& !ClassUtils.isPresent(JERSEY_WEB_ENVIRONMENT_CLASS, null)) {
			return WebApplicationType.REACTIVE;
		}
		for (String className : WEB_ENVIRONMENT_CLASSES) {
			if (!ClassUtils.isPresent(className, null)) {
				return WebApplicationType.NONE;
			}
		}
		return WebApplicationType.SERVLET;
	}
	......

}
~~~

当然在实例化SpringApplication后还可以调用`SpringApplication#setWebApplicationType`或者`SpringApplication#setApplicationContextClass`来更改`WebApplicationType`环境，下面是`setWebApplicationType`和`setApplicationContextClass`源码

~~~java
public void setWebApplicationType(WebApplicationType webApplicationType) {
	Assert.notNull(webApplicationType, "WebApplicationType must not be null");
	this.webApplicationType = webApplicationType;
}

public void setApplicationContextClass(
			Class<? extends ConfigurableApplicationContext> applicationContextClass) {
	this.applicationContextClass = applicationContextClass;
	if (!isWebApplicationContext(applicationContextClass)) {
		this.webApplicationType = WebApplicationType.NONE;
	}
}
~~~



`WebApplicationType`类型。Reactive Web和Servlet Web区别可以在网上搜。

~~~java

public enum WebApplicationType {
	NONE,
	SERVLET,
	REACTIVE
}
~~~

