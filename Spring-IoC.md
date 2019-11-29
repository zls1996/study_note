## Spring IoC核心实现

### IoC容器概述

​	“依赖反转”：**依赖对象的获得被反转**。如果合作对象的引用或者依赖关系的管理由具体对象来完成，会导致代码的高度耦合和可测试性的降低，这对复杂的面向对象系统的设计是非常不利的。在面向对象系统中，对象封装了数据和对数据的处理，对象的依赖关系常常体现在对数据和方法的依赖上。这些依赖关系可以通过把对象的依赖注入交给框架或者IoC容器来完成，这种从具体对象手中交出控制的做法是非常有价值的，它可以在解耦代码的同时提高代码的可测试性。

## IoC容器设计与实现：BeanFactory和ApplicationContext

​	在Spring IoC容器设计中，我们可以看到两个主要的容器系列，一个是实现BeanFactory接口的简单容器实例，这系列容器只实现了容器最基本的功能；另一个是ApplicationContext应用上下文，它作为容器的高级形态存在。应用上下文在简单容器的基础上，增加了许多面向框架的特性，同时对应用环境做了许多适配。有了这两种基本容器系列，基本可以满足用户对IoC容器使用的大部分需求了。

​	在Spring提供的基本IoC容器的接口定义和实现的基础之上，Spring通过定义BeanDefinition来管理给予Spring应用的各种对象以及它们之间的相互依赖关系。**BeanDefinition抽象了我们对于Bean的定义，是让容器起作用的主要数据类型**。

​	同时，在使用IoC容器时，了解BeanFactory和ApplicationContext之间的区别对于我们理解和使用IoC容器也是比较重要的。弄清楚这两种容器之间的区别和联系，意味着我们具备了解辨别容器系列中不同容器产品的能力。还有一个好处就是，如果需要定制特定功能的容器实现，也能比较方便的在容器系列中找到一款恰当的产品作为参考，不需要重新设计。

#### Spring IoC容器的设计

​	首先我们可以看一下BeanFactory和ApplicationContext的架构图。	![1570766158366](C:\Users\leeson\AppData\Roaming\Typora\typora-user-images\1570766158366.png)



![1570774529008](C:\Users\leeson\AppData\Roaming\Typora\typora-user-images\1570774529008.png)



- BeanFactory接口定义了基本的IoC容器规范。在BeanFactory接口定义中，包括了getBean()这样的IoC容器的基本方法。HierarchicalBeanFactory接口在继承了BeanFactory的基本接口之后，增加了getParentBeanFactory()的接口功能。
- 第二条设计主线是：以ApplicationContext应用上下文为核心的接口设计，这里主要涉及的主要接口设计有，从BeanFactory到ListableBeanFactory，再到ApplicationContext,再到我们常用的WebApplicationContext或者ConfigurableApplicationContext的实现。在这个接口体系中，ListableBeanFactory和HierarchicalBeanFactory两个接口，连接BeanFactory接口定义和ApplicationContext应用上下文的接口定义。在ListableBeanFactory接口中，细化了许多BeanFactory功能，比如定义了getBeanDefinitionNames()接口方法；对于HierarchicalBeanFactory接口，我们已经提到过；对于ApplicationContext接口，它通过继承MessageSource、ResourceLoader、ApplicationEventPublisher接口，在BeanFactory简单IoC容器的基础上添加了许多对高级容器的支持。





##### BeanFactory的应用场景

​	BeanFactory提供的是最基本的IoC容器的功能，关于这些功能定义，我们可以在接口BeanFactory代码中看到。

```java

public interface BeanFactory {

	// FactoryBean的name前缀
	String FACTORY_BEAN_PREFIX = "&";

	Object getBean(String name) throws BeansException;
    
	<T> T getBean(String name, Class<T> requiredType) throws BeansException;
    
	Object getBean(String name, Object... args) throws BeansException;

	<T> T getBean(Class<T> requiredType) throws BeansException;

	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	boolean containsBean(String name);

	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
    
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
    
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
	
    // 查询指定名字的Bean的所有别名
	String[] getAliases(String name);

}

```

​	BeanFactory接口定义了IoC容器最基本的格式，并且提供了IoC容器所应该遵守的最基本的服务契约，同时，这也是我们使用IoC容器所应该遵守的最底层和最基本的编程规范，这些接口定义勾画出了IoC的基本轮廓。很显然，在Spring的代码实现中，BeanFactory只是一个接口类，并没有给出容器的具体实现。

​	用户在使用容器时，可以使用转移符"&"来得到FactoryBean本身，用来区分通过容器来获取FactoryBean产生的对象和获取FactoryBean本身。举例来说，如果myJndiObject是一个FactoryBean，那么使用&myJndiObject所得到的是FactoryBean，而不是myJndiObject这个FactoryBean产生出来的对象。

##### BeanFactory容器的设计原理

​	BeanFactory接口提供了使用IoC容器的规范。在这个基础上，Spring还提供了符合这个IoC容器接口的一系列容器的实现供开发人员使用。我们以XmlBeanFactory的实现为例，首先来看XmlBeanFactory的类继承关系。

![1570775873511](C:\Users\leeson\AppData\Roaming\Typora\typora-user-images\1570775873511.png)

​	可以看到，作为一个简单的容器系列最底层实现的XmlBeanFactory，与我们在Spring应用中用到的那些上下文相比，有一个非常显著的特点：它只提供最基本的IoC容器的功能。理解这一点有助于我们理解ApplicationContext与基本的BeanFactory之间的区别和联系。我们可以认为直接的BeanFactory实现是IoC容器的基本形式，而各种ApplicationContext的实现是IoC容器的高级表现形式。

​	XmlBeanFactory继承自DefaultListableBeanFactory这个类，后者非常重要，使我们经常要用到的一个IoC容器的实现，比如在设计应用上下文ApplicationContext时就会用到它。我们会看到这个DefaultListableBeanFactory实际上包含了基本IoC容器所具有的重要功能，也是在很多地方都会用到的容器系列中的一个基本产品。

​	在Spring中，实际上是把DefaultListableBeanFactory作为一个默认的功能完整的IoC容器来使用的。XmlBeanFactory在继承了DefaultListableBeanFactory容器功能的同时，增加了新的功能，这些功能很容易从XmlBeanFactory名字上猜到。它是一个XML相关的BeanFactory，也就是它是一个可以读取XML文件方式定义的BeanDefinition的IoC容器。

```java
public class XmlBeanFactory extends DefaultListableBeanFactory {

	private final XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);

	public XmlBeanFactory(Resource resource) throws BeansException {
		this(resource, null);
	}
    
	public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
		super(parentBeanFactory);
		this.reader.loadBeanDefinitions(resource);
	}

}
```

​	这些实现XML读取的功能是怎样实现的呢？对这些XML文件定义信息的处理并不是由XmlBeanFactory直接完成的。在XmlBeanFactory中，初始化了一个XmlBeanDefinitionReader对象，有了这个Reader对象，那些以XML方式定义的BeanDefinition就有了处理的地方。我们可以看到，对这些XML形式的信息的处理实际上是由这些XmlBeanDefinitionReader来完成的。

​	构造XmlBeanFactory这个IoC容器时，需要指定BeanDefinition的信息来源，而这个信息来源需要封装成Spring中的Resource类来给出。Resource是Spring用来封装I/O操作的类。比如，我们的BeanDefinition的信息是以XML文件形式存在的，那么可以使用像如下代码：

```java
Resource resource = new ClassPathResource("beans.xml");
```

​	这样的具体的ClassPathResource来构造需要的Resource，然后将Resource作为构造参数传递给XmlBeanFactory构造参数。这样，IoC容器就可以了方便的定位到需要的BeanDefinition信息来对Bean完成容器的初始化和依赖注入过程。

​	XmlBeanFactory的功能是建立在DefaultListableBeanFactory这个基本容器的基础上的，并在这个基本容器的基础上实现了其它诸如XML读取的附加功能。对于这些功能的实现原理，看一看XmlBeanFactory构造方法中需要得到Resource对象。对XmlBeanDefinitionReader对象的初始化，以及使用这个对象来完成对
loadBeanDefintion的调用，就是这个调用启动从Resource中载入BeanDefinitions的过程，loadBeanDefinitions同时也是IoC容器初始化的重要组成部分。

​	在上面的代码中，我们可以看到XmlBeanFactory使用了DefaultListableBeanFactory。从中我们可以看到IoC容器使用的一些基本过程，尽管我们在应用中使用IoC容器很少会使用这样原始的方式，但是了解一下这个基本过程，很清楚揭示了在IoC容器使用的一些基本过程，对我们了解IoC容器的工作原理是非常有帮助的。因为这个编程式使用容器的过程，很清楚地揭示了在IoC容器实现中的那些关键的类(比如Resource、DefaultListableBeanFactory和BeanDefinitionReader)之间的相互关系，例如它们是如何把IoC容器的功能解耦的，又是如何结合在一起为IoC容器服务的，等等。在以下代码中，我们可以看到编程式使用IoC容器的过程。

```java
Resource resource = new ClassPathResource("beans.xml");
BeanFactory parentBeanFactory = new DefaultListableBeanFactory();
BeanDefinitionReader reader = new XmlBeanDefinitionReader(parentBeanFactory);
reader.loadBeanDefinitions(resource);
```

​	这样，我们就可以通过factory对象来使用DefaultListableBeanFactory这个IoC容器了。在使用IoC容器时，需要如下几个步骤：

1. 创建IoC配置文件的抽象资源，这个抽象资源包含了BeanDefinition的定义信息
2. 创建一个BeanFactory，这个使用了DefualtListableBeanFactory
3. 创建一个载入BeanDefinition的读取器，这里使用XmlBeanDefinitionReader来载入XML文件形式的BeanDefinition,通过一个回调配置给BeanFactory。
4. 从定义好的资源位置读取配置信息，具体的解析过程由XmlBeanDefinitionReader来完成。完成整个载入和注册Bean定义之后，需要的IoC容器就建立起来了。这个时候就可以直接使用IoC容器了。

##### ApplicationContext的使用场景

​	在Spring中，系统已经为用户提供了许多定义好的容器实现，而不需要开发人员事必躬亲。相比那些简单拓展BeanFactory的基本IoC容器，开发人员常用的ApplicationContext除了能提供前面介绍的容器的基本功能之外，还为用户提供了以下的附加服务，可以让用户更加方便使用。所以说，ApplicationContext是一个高级形态的IoC容器。ApplicationContext在BeanFactory的基础上添加了如下附加功能：

1. 支持不同的信息源。我们可以看到ApplicationContext拓展了MessageResource接口，这些信息源的拓展功能可以支持国际化的实现，为开发多语言版本的应用提供服务。
2. **访问资源**。这一特性体现在对ResourceLoader和Resource的支持上，这样我们**可以从不同的地方得到Bean定义资源**。这种抽象使得用户程序可以灵活地定义Bean定义信息，尤其是从不同的I/O途径得到Bean定义信息。这在接口关系上看不出来，不过一般来说，具体ApplicationContext都是继承了DefaultResourceLoader的子类。因为DefaultResourceLoader是AbstractApplicationContext的父类。
3. **支持应用事件**。**ApplicationContext继承了接口ApplicationEventPublisher，从而在上下文中引入了事件机制，这些事件和Bean的声明周期的结合为Bean的管理提供了便利**。

##### ApplicationContext容器的设计原理

​	在ApplicationContext中，我们以常用的FileSystemXmlApplicationContext的实现为例来说明ApplicationContext容器的设计原理。

![1570783744289](C:\Users\leeson\AppData\Roaming\Typora\typora-user-images\1570783744289.png)

​	在FileSystemXmlApplicationContext的设计中，我们看到ApplicatoinContext应用上下文的主要功能已经在FileSystemXmlApplicationContext的基类AbstractXmlApplicationContext中实现了，在FileSystemXmlApplicationContext中，作为一个具体的应用上下文，只需要实现和它自身设计相关的两个功能。

```java

public class FileSystemXmlApplicationContext extends AbstractXmlApplicationContext {

	public FileSystemXmlApplicationContext() {
	}

	public FileSystemXmlApplicationContext(ApplicationContext parent) {
		super(parent);
	}

	public FileSystemXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[] {configLocation}, true, null);
	}

	public FileSystemXmlApplicationContext(String... configLocations) throws BeansException {
		this(configLocations, true, null);
	}

	public FileSystemXmlApplicationContext(String[] configLocations, ApplicationContext parent) throws BeansException {
		this(configLocations, true, parent);
	}
    
	public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh) throws BeansException {
		this(configLocations, refresh, null);
	}
	
	public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}

	@Override
	protected Resource getResourceByPath(String path) {
		if (path != null && path.startsWith("/")) {
			path = path.substring(1);
		}
		return new FileSystemResource(path);
	}

}

```

​	一个功能是，如果应用直接使用FileSystemXmlApplicationContext，对于实例化这个应用上下文的支持，同时启动IoC容器的**refresh()**过程。这在FileSystemApplicationContext的代码中可以看到。refresh函数在AbstractApplicationContext中实现。

​	这里的refresh会牵涉IoC容器启动的一系列复杂操作，同时，对于不同的容器实现，这些操作都是类似的，因此在基类AbstractApplicationContext就将它们封装好。所以，我们在FileSystemXmlApplicationContext的设计中看到的只是一个简单的调用。关于这个refresh在IoC的具体实现，会在下面重点分析。

​	另一个功能是与FileSystemXmlApplicationContext设计具体相关的功能，这部分会怎样从文件系统中加载XML的Bean定义资源有关。通过这个过程，可以为在文件系统中读取XML形式存在的BeanDefinition做准备，因为不同的应用上下文实现对应着不同的读取BeanDefinition的方式，在FileSystemXmlApplicationContext中的实现方法为getResourceByPath()方法。通过调用这个方法，可以得到FileSystemResource的资源定位。



### IoC容器的初始化过程

​	IoC容器的初始化是由前面介绍的refresh()方法来启动的，这个方法标志着IoC容器的正式启动。具体来说，这个**启动包括BeanDefinition的Resource定位、载入和注册三个基本过程**。如果我们了解如何编程式使用IoC容器，就可以清楚地看到Resource定位和载入过程以及接口的调用。

​	Spring把Resource定位、载入和注册的三个过程分开了，并使用不同的模块来完成。如使用相应的ResourceLoader、BeanDefinitionReader模块等，通过这样的设计方式，可以让用户更加灵活地对这三个过程进行裁剪或拓展，定义出最适合自己的IoC容器初始化过程。

##### Resource定位

​	Resource定位指的是BeanDefinition的资源定位，它由ResourceLoader通过统一的Resource接口来完成，这个Resource对各种形式的BeanDefinition的使用都提供了统一的接口，对于这些BeanDefinition的存在形式，我们都不陌生。比如，在文件系统中的Bean定义信息可以使用FileSystemResource来进行抽象；在类路径中的Bean定义信息可以使用前面提到的ClassPathResource等使用。这个定位过程类似于容器寻找数据的过程。

​	通过编程的方式使用DefaultListableBeanFactory时，首先定义一个Resource去定位容器使用的BeanDefinition。这时使用的是ClassPathResource,这意味着Spring会在类路径中去寻找以文件形式存在的BeanDefinition信息。

```java
Resource resource = new ClassPathResource("beans.xml");
```

​	这里定义的Resource并不能由DefaultListableBeanFactory直接使用，Spring通过BeanDefinitionReader来对这些信息进行处理。在这里，我们也可以看到使用ApplicationContext相对于直接使用DefaultListableBeanFactory的好处。因为在ApplicationContext中，Spring已经为我们提供了一系列加载不同Resource的读取器的实现，而DefaultListableBeanFactory只是一个纯粹的IoC容器，需要为它配置特定的读取器次才能完成这些功能。当然，有利就有弊，使用DefaultListableBeanFactory这种更底层的容器，能提高定制IoC容器的灵活性。

​	回到我们常用的ApplicationContext上来，例如FileSystemXmlApplicationContext、ClassPathXmlApplicationContext以及XmlWebApplicationContext等。简单地从这些类的名字上分析，可以清楚地看到它们可以提供哪些不同的Resource读入功能，比如FileSystemXmlApplicationContext可以从文件系统载入Resource，ClassPathXmlApplicationContext可以从classpath中载入Resource，XMLWebApplicationContext可以在web容器中载入Resource等。

​	我们以FileSystemXmlApplicationContext为例，分析这个ApplicationContext是如何实现Resource定位的。

![1571130491807](C:\Users\leeson\AppData\Roaming\Typora\typora-user-images\1571130491807.png)

​	我们可以从FileSystemXmlApplicationContext的继承关系看到，FileSystemXmlApplicationContext继承了AbstractApplicationContext，因此具备了从ResourceLoader读入以Resource定义的BeanDefinition能力，AbstractApplicationContext的父类是DefaultResourceLoader。我们来看看FileSystemXmlApplicationContext的具体实现。

```java

public class FileSystemXmlApplicationContext extends AbstractXmlApplicationContext {

	
	public FileSystemXmlApplicationContext() {
	}

	public FileSystemXmlApplicationContext(ApplicationContext parent) {
		super(parent);
	}
	// 这里的构造函数的configLocation包含的是BeanDefinition的文件路径
	public FileSystemXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[] {configLocation}, true, null);
	}
    // 允许包含多个beanDefinition路径
	public FileSystemXmlApplicationContext(String... configLocations) throws BeansException {
		this(configLocations, true, null);
	}
    // 还允许指定父ApplicationContext(双亲)
	public FileSystemXmlApplicationContext(String[] configLocations, ApplicationContext parent) throws BeansException {
		this(configLocations, true, parent);
	}

	public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh) throws BeansException {
		this(configLocations, refresh, null);
	}
	
    // 在对象的初始化过程中，调用refresh函数载入BeanDefinition，这里的refresh()函数就是	ApplicationContext的启动过程
	public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}
	
    // 这是应用于文件系统中Resource的实现，通过构造一个FileSystemResource来得到一个在文件系统中定位的
   	// BeanDefinition。这里的getResourceByPath采用了模板模式，具体的定位实现是由于各个子类来实现的。
	@Override
	protected Resource getResourceByPath(String path) {
		if (path != null && path.startsWith("/")) {
			path = path.substring(1);
		}
		return new FileSystemResource(path);
	}

}

```

​	我们可以看到，关于IoC容器的载入功能，在这里并没有被涉及到，因为它继承了AbstractXmlApplicationContext，关于IoC容器的过程，都是在refresh()方法中实现的，这里的refresh非常重要，使我们分析容器初始化过程的一个重要入口，这里的refresh调用时在FileSystemXmlBeanFactory的构造函数中启动的。



##### BeanDefinition的载入和解析

​	在完成了对代表BeanDefinition的Resource定位分析之后，我们需要来了解整个BeanDefinition信息的载入过程。这个载入过程，就是把定义的BeanDefinition在IoC容器中转换成一个Spring内部的数据结构(BeanDefinition)的过程。

​	我们再从FileSystemApplicationContext入手，看看IoC容器是怎样完成BeanDefinition载入的。

```java
public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}
```

refresh()方法在AbstractApplicationContext中实现。AbstractApplicationContext详细地描述了整个ApplicationContext的启动过程，比如BeanFactory的更新，MessageSource以及PostProcessor的注册等等。AbstractApplicationContext更像是对ApplicationContext进行初始化模板或执行提纲，这个执行过程为Bean的生命周期管理提供了条件。

我们来直接看AbstractApplicationContext的refresh执行流程。

```java
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// 在子类中启动refreshBeanFactory
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// 设置BeanFactory的后置处理
				postProcessBeanFactory(beanFactory);

				// 调用BeanFactory的后处理器，这些后处理器是在
                // Bean定义中向容器注册的
				invokeBeanFactoryPostProcessors(beanFactory);

				// 注册Bean的后置处理器，在Bean创建过程中调用
				registerBeanPostProcessors(beanFactory);

				// 对上下文消息进行初始化
				initMessageSource();

				// 初始化上下文中的事件机制
				initApplicationEventMulticaster();

				// 初始化其他的特殊bean
				onRefresh();

				// 检查监听Bean并且将这些Bean向容器注册
				registerListeners();

				// 实例化所有单例
				finishBeanFactoryInitialization(beanFactory);

				// 发布容器事件，结束refresh过程
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
```

#### ApplicationContext和Bean的初始化和销毁

​	对于BeanFactory和ApplicationContext，容器自身也有一个初始化和销毁关闭的过程。对于ApplicationContext启动的过程是在AbstractApplicationContext中实现的。在使用应用上下文时需要做一些准备工作，这些准备工作在prepareBeanFactory()方法中实现。在这个方法中，为容器配置了ClassLoader、PropertyEditor以及BeanPostProcessor等，从而为容器的启动做好了必要的准备工作。

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// Tell the internal bean factory to use the context's class loader etc.
		beanFactory.setBeanClassLoader(getClassLoader());
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

		// Configure the bean factory with context callbacks.
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// Register early post-processor for detecting inner beans as ApplicationListeners.
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

		// Detect a LoadTimeWeaver and prepare for weaving, if found.
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// Register default environment beans.
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
	}
```

​	同样，在容器关闭时，也需要完成一系列的工作，这些工作在doClose()方法中完成。在这个方法中，先发出容器关闭的信号，然后将Bean逐个关闭，最后关闭容器本身。

```java
protected void doClose() {
		if (this.active.get() && this.closed.compareAndSet(false, true)) {
			if (logger.isInfoEnabled()) {
				logger.info("Closing " + this);
			}

			LiveBeansView.unregisterApplicationContext(this);

			try {
				// Publish shutdown event.
				publishEvent(new ContextClosedEvent(this));
			}
			catch (Throwable ex) {
				logger.warn("Exception thrown from ApplicationListener handling ContextClosedEvent", ex);
			}

			// Stop all Lifecycle beans, to avoid delays during individual destruction.
			if (this.lifecycleProcessor != null) {
				try {
					this.lifecycleProcessor.onClose();
				}
				catch (Throwable ex) {
					logger.warn("Exception thrown from LifecycleProcessor on context close", ex);
				}
			}

			// Destroy all cached singletons in the context's BeanFactory.
			destroyBeans();

			// Close the state of this context itself.
			closeBeanFactory();

			// Let subclasses do some final clean-up if they wish...
			onClose();

			this.active.set(false);
		}
	}
```

​	容器的实现是通过IoC管理bean的生命周期来实现的。Spring IoC在对Bean的生命周期进行管理时提供了Bean生命周期各个时间点的回调。

​	Bean的生命周期为：

- Bean实例的创建
- 为Bean实例设置属性
- 调用Bean的初始化方法
- 应用可以通过IoC容器使用Bean
- 当容器关闭时，调用Bean的销毁方法



​	Bean的初始化方法在initializeBean方法中实现：

​	// TODO





在调用Bean的初始化方法之前，会调用一系列的aware接口实现，把相关的BeanName，BeanClassLoader，以及BeanFactory注入到Bean中去。接着看对invokeInitMethods的调用，这时会看到启动afterPropertySet的过程，当然，需要Bean实现InitializingBean接口，这里同样是对Bean的回调。

// TODO





​	最后，还会看到判断Bean是否配置有initMethod,如果有，那么通过invokeCustomInitMethod方法来直接调用，最终完成Bean的初始化。

​	



与Bean初始化类似，当容器关闭时，可以看到Bean销毁方法的调用。Bean销毁过程

























