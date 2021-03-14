## Spring Boot 自动装配

### @Autowired

从源码中，我们可以看到以下几点：

1. 该注解可以用在 **构造函数、方法、方法内的参数、类属性和注解上**；
2. 该注解在程序运行时有效；
3. 有一个属性 required，默认true，表示该装配是否是必须的，若是，则找不到对象的类时抛出异常；



一般用法：

1. 先添加接口，声明接口方法；
2. 添加接口实现，并使用spring三层结构的注解添加到bean中；
3. 使用时，使用注解进行自动装配；

```java
@Autowired
private ProductInterface productServer;
```



ProductInterface 接口有两个实现类，FoodProductImpl 和 FrultProductImpl

```java
@Autowired
@Qualifer("FoodProductImpl")
private ProductInterface productServer;
```



其他自动装配：

**@Resource** 是 J2EE 的注解，JSR250中的规范，其作用和 **@Autowired** 类似，匹配不到则报错

**@Inject** 注解也可以实现自动装配的功能，它是 JSR330 中的规范



总结：

　　自动装配还是利用了SpringFactoriesLoader来加载META-INF/spring.factoires文件里所有配置的EnableAutoConfgruation，它会经过exclude和filter等操作，最终确定要装配的类

　　处理@Configuration的核心还是ConfigurationClassPostProcessor，这个类实现了BeanFactoryPostProcessor, 因此当AbstractApplicationContext执行refresh方法里的invokeBeanFactoryPostProcessors(beanFactory)方法时会执行自动装配





## SpringBean的生命周期管理？

普通JAVA对象和spring 管理的bean实例化的过程是有区别的。



在普通JAVA环境下创建对象的步骤主要为：

1. java源码被编译为class文件
2. 等到类要被初始化时，如：new、反射等
3. class被虚拟机通过类加载器加载到 JVM
4. 初始化对象供我们使用



### Spring Bean的生命周期过程？

而spring所管理的bean不同的是，除了class对象外还会使用BeanDefinition的实例来描述对象的信息。

比如说我们可以在Spring所管理的Bean做一些描述，比如@Scope、@Lazy、@DependsOn等等

流程：

Spring在启动的时候会去扫描在XML、JAVAconfig、注解中需要被Spring管理的Bean信息，随后会将这些信息封装成BeanDefinition，最后把他们放到一个BeanDefinitionMap中，这个Map的key是BeanName，value是BeanDefinition对象，到这里就是吧定力的元数据加载起来，单真正的对象还没实例化

接着会遍历这个BeanDefinitionMap，执行BeanFactoryPostProcessor这个Bean工厂的后置处理逻辑

比如我们我们平时定义的占位符，就是通过BeanFactoryPostProcessor的子类PropertyPlaceholderConfigurer进行注入的

当然这里我们可以使用我们自定义的BeanFactoryPostProcessor来对我们定义好的Bean元数据进行获取和修改。很少用

后置处理器执行完了就到了实例化对象了

在Spring里是通过反射来实现实例化对象的，一般会反射选择合适的构造器来实例化对象，但是这里实例化只是创建对象，对象具体的属性还是没有实例化的，比如我的对象是 用户service，那它依赖的 SendService对象这时候还是null的

所以下一步就是注入对象的属性，再往下就是初始化工作了

首先会判断该Bean是否实现了Aware相关的接口，如果存在就会填充相关资源

Aware的相关接口处理完就会到BeanPostProcessor后置处理器，BeanPostProcessor后置处理器有两个方法，before 、after，这个BeanPostProcessor后置处理器是AOP实现的关键，关键子类是AnnottationAwareAspectJAutoProxyCreator

此时会执行BeanPostProcessor相关子类的before方法，

执行完后，接着会执行init相关方法，比如	@PostConstruct、实现InitializingBean接口、定义的init-method方法

去他们官网查看了他们的执行顺序分别是：@PostConstruct、实现了InitializingBean接口以及init-method方法

等到init方法执行完之后就会执行BeanPostProcessor的after方法，此时基本流程走完，我们就可以获取到对象去使用了

销毁就去看有没有配置相关destroy方法，执行就完事了





1. Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化
2. Bean实例化后对将Bean的引入和值注入到Bean的属性中
3. 如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法
4. 如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
5. 如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来。
6. 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。
7. 如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
8. 如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。
9. 此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
10. 如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。





### Spring 怎么解决循环依赖的？

例：A=B.a B=A.b

##### 过程：

上面说的spring 是先实例化对象再注入对象的属性，所以循环依赖是说A实例化完毕后会去注入其对象的属性，此时发现A依赖B，此时B还没被创建，就会去实例化B对象，实例化B对象后发现B依赖A，此时A已经被创建了，所以B对象最终能完成创建。

B对象返回到A的属性注入的方法上，完成A的创建。

##### 原理：

原理就是用到了三级缓存。三级缓存其实就是三个Map

* singletonObjects（一级，日常获取Bean的地方）

* earlySingletonObjects（二级，以实例化对象，还没进行属性注入，由三级缓存放进来）

* singletonFactories（三级，Value是一个对象工厂）

回到上面讲述的过程中，A对象实例化后，会吧A放入三级缓存中，key是BeanName val是ObjecFactory，等A属性注入时发现依赖B，又去实例化B，B注入时会去从三级缓存里拿出A（ObjecFactory），从中得到对应的Bean（就是对象A）此时把三级缓存干掉，放到二级缓存当中去，此时key时BeanName value是Bean A（这里A还未完成属性注入操作）等完全初始化后，就会把二级缓存remove掉，塞到一级缓存中。我们自己去getBean的时候，拿到的是一级缓存

为什么是三级缓存？

我们的对象是单例的，有可能A对象依赖的B对象是有AOP操作的，假如没有三级缓存，只有二级缓存，那如果有AOP的情况下，岂不是在存入二级缓存级之前都需要先去做AOP代理，这不合适，这里主要考虑的是代理的情况，从代理里拿到代理对象。而二级缓存的必要时为了性能，从三级缓存的工厂里创建出对象，再扔到二级缓存（这样就不用每次都到三级缓存的工厂里去拿）。









<img src="assets/640" alt="图片" style="zoom:50%;" />



![图片](assets/640-20210314214114239)

![图片](assets/640-20210314214115054)



