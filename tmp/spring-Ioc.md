# spring-控制反转(IOC)

## 1.何为控制反转？

​	控制反转，需要拆开来看，即什么是控制，又控制了什么，反转是什么，如何反转的；

​	所谓**控制**是指：***控制依赖对象的创建过程***；

​	 通常情况下，在当前类中依赖另一个类的实例对象时，需要在当前类中通过依赖对象的构造方法来实例化依赖对象，这个创建依赖对象的过程是由当前类来自行控制的，比如说：当前类是直接使用new关键字通过构造方法创建实例对象还是通过反射的方式创建？依赖对象的依赖是由构造方法初始化还是调用`set`方法初始化？这些问题都需要当前使用依赖对象的类来控制；

​	所谓**反转**是指：**将依赖对象的创建过程交给外部容器来控制，使用依赖对象的类只需要通过外部容器即可获得依赖对象，不再由当类自行控制依赖对象的创建过程**；这个机制也称之为**控制反转**！

## 2.**控制反转的优势：**

​	**避免使用依赖对象的类和依赖对象直接耦合，增加代码的可维护性；**

​	通过控制反转的方式，将依赖对象的创建过程交给外部容器管理，使用依赖对象的类不再和依赖对象的创建过程直接耦合；使用时只需要从外部容器获取即可，简化了对依赖对象的使用过程；而依赖对象的创建过程交给外部容器管理之后，全局只有一处创建依赖对象的地方，如果依赖对象的创建过程有所修改，那么也只需要修改外部容器创建依赖对象的过程即可，无需对使用依赖对象的类大面积修改；这就是**控制反转**的优点！

## 3.控制反转和依赖注入：

​	**控制反转和依赖注入是站在不同角度描述了同一个事情，控制反转是站在使用依赖对象的类的角度，而依赖注入则是站在容器的角度，并共同描述了类获取依赖对象的方式；**

​	**在spring中，既可以通过getBean这种方式从容器中获取一个依赖对象，也可以通过注入的方式给类中注入一个依赖对象；**

​	依赖注入是指：一个类的依赖对象不应该由该类直接通过构造方法创建，而应该由容器向该类中自动注入该类依赖的实例对象；

​	控制反转是指：将依赖对象的创建过程交给外部容器，使用依赖对象的类无需自己控制依赖对象的创建过程，只需从外部容器获取即可；

​	由此可见，控制反转和依赖注入解决的都是同一个问题，即：如何控制依赖对象的创建过程！因此控制反转和依赖注入实际上描述的是同一个事情，只是角度不同而已：**控制反转是从使用依赖对象的类的角度描述，而自动注入是从容器的角度描述；**

## 4.Spring Framework中的控制反转

### 4.1Spring Framework中的Ioc依赖的包：

​		Spring Framework中实现了控制反转需要的外部容器以及自动注入机制，其中**org.springframework.beans**和**org.springframework.context**包是使用`Spring Framework`控制反转的基础，在使用Spring Framework的Ioc时必须先导入上述两个包；

​		其中**org.springframework.beans**提供了对bean的创建、管理等功能；**org.springframework.context**提供了上下文管理功能，存储了spring启动之后的运行环境等信息，通过该包可以获取到容器；

### 4.2Spring中的bean：

​		在`Spring Framework`中，把通过容器管理的实例对象称之为`bean`，没有通过容器管理的实例对象称之为普通的对象；

​		将实例对象交给容器管理时需要配置以下几项信息:

​		1.bean的名称

​		2.bean的创建方式；

​		3.bean的依赖；

​		4.bean的属性；	

​		告知容器上述bean信息的过程也称为配置容器元数据的过程

### 4.3通过xml方式配置Ioc元数据:

#### 1.指定创建方式:

```xml
<!-- 通过构造方法创建 -->
<bean id="bean_name" name="BeanName" class="com.sslike.ioc.Bean"></bean>
<!-- 通过工厂实例方法创建 -->
<bean id="bean_factory" name="BeanFactory" class="com.sslike.ioc.BeanFactory"></bean>
<bean id="bean_name" name="BeanName" factory-bean="bean_factory" class="com.sslike.ioc.Bean" factory-method="getBean"></bean>
<!-- 通过工厂静态方法创建 -->
<bean id="bean_factory" name="BeanFactory" class="com.sslike.ioc.BeanFactory" factory-method="getBean"></bean>
```

#### 2.注入bean的依赖:

##### 1.通过构造方法注入依赖:

```xml
<!-- 通过construct-arg注入基本类型数据 -->
<bean id="bean_name" name="BeanName" class="com.sslike.ioc.Bean">
<construct-arg name="名称定为" index="位置定位" type="类型定位" value="具体值"></construct-arg>
</bean>
<!-- 通过construct-arg注入bean -->
<bean id="bean_name" name="BeanName" class="com.sslike.ioc.Bean">
<construct-arg name="名称定为" index="位置定位" type="类型定位" ref="其他bean"></construct-arg>
</bean>
<!-- 通过c注入基本类型数据 -->
<bean id="bean_name" name="BeanName" class="com.sslike.ioc.Bean" c:0="通过位置定位设置具体值" c:bean2="通过名称定位设置具体值">
</bean>
<!-- 通过c注入bean -->
<bean id="bean_name" name="BeanName" class="com.sslike.ioc.Bean" c:0_ref="通过位置定位设置bean" c:bean2_ref="通过名称定位设置bean">
</bean>
```

##### 2.通过set方法注入依赖:

```xml
<!-- 通过property注入基本类型数据 -->

<!-- 通过property注入bean -->

<!-- 通过property注入list -->

<!-- 通过property注入set -->

<!-- 通过property注入map -->

<!-- 通过property注入null -->

<!-- 通过p注入基本类型数据 -->

<!-- 通过p注入bean -->
```



#### 3.设置bean的属性:

##### 1.设置bean的唯一标识:

```xml
<!-- 通过id指定 -->
<bean id="testBean" class="package.TestBean" scope="singleton"></bean>

<!-- 通过name指定 -->
<bean name="testBean" class="package.TestBean" scope="singleton"></bean>

<!-- 通过alias指定别名 -->
<alias name="testBean" alias="tBean"></alias>

<!-- id、name、alias同时使用 -->
<bean id="testBean" name="testBeanName" class="package.TestBean" scope="singleton"></bean>
<alias name="testBeanName" alias="tBean"></alias>
```

##### 2.设置bean的作用域:

​	常见的作用域类型：

​		singleton：单利作用域：在容器初始化时创建，单利模式，无论获取多少次仅创建一次该实例对象；默认作用域

​		prototype：原型作用域：获取bean时创建，每获取一次就会创建一次；

​		request：请求作用域：接收到新的请求时创建，每接收一个新的请求就创建一次，请求结束即销毁；

​		session：会话作用域：创建新的会话时创建，每创建一个新的会话就会创建一次，会话结束即销毁；

​		application：应用作用域：限定其作用域为ServletContext的生命周期，应用启动时创建，每个引用内部只会创建一个bean；

​	作用域配置方式：

```xml
<!-- 设置单例作用域:以单利模式创建bean,每次从容器中获取的都是同一个bean,默认作用域 -->
<bean id="testBean" class="package.TestBean" scope="singleton"></bean>
<!-- 设置原型作用域:以原型多例模式创建bean,每次从容器中获取bean时都会重新创建bean -->
<bean id="testBean" class="package.TestBean" scope="prototype"></bean>

<!-- 单例原型共同使用:原型嵌套原型：符合bean生命周期 -->
<bean id="testBean" class="package.TestBean" scope="prototype">
	<bean id="testBean2" class="package.TestBean" scope="prototype"></bean>
</bean>

<!-- 单例原型共同使用:原型嵌套单例：符合bean生命周期 -->
<bean id="testBean" class="package.TestBean" scope="prototype">
	<bean id="testBean2" class="package.TestBean" scope="singleton"></bean>
</bean>

<!-- 单例原型共同使用:单例嵌套单例：符合bean生命周期 -->
<bean id="testBean" class="package.TestBean" scope="singleton">
	<bean id="testBean2" class="package.TestBean" scope="singleton"></bean>
</bean>

<!-- 单例原型共同使用:单例嵌套原型：原型bean成单例模式，不符合生命周期 -->
<bean id="testBean" class="package.TestBean" scope="singleton">
	<bean id="testBean2" class="package.TestBean" scope="prototype"></bean>
</bean>

```

​	单例原型共同使用:通过lookup-method来协同生命周期

```java
public abstract class TestBean{
  private TestBean2 testBean2;
  
  public TestBean2 setTestBean2(){
    //改写TestBean2的参数注入方式，TestBean2每次都通过getBean2获取,避免退化成单例
    this.testbean2 = getbean2();
  }
 
  public abstract TestBean2 getBean2();
}
```

```xml
<bean id="testBean" class="package.TestBean" scope="singleton">
  <!--不再使用property来注入TestBean2,改用lookup-method来注入-->
  <lookup-method name="getBean2" bean="testBean2"></lookup-method>
</bean>
```

​		自定义作用域

```java
import org.springframework.beans.factory.config.Scope;
public class MyScope implements Scope{
  //实现Scope中的get和remove方法，在从容器中获取自定义作用域的bean时，会调用get方法获得Bean实例对象
}
```

```xml
<!-- 将自定义作用域设置为bean -->
<bean id="selfScope" class="package.MyScope"></bean>
  
<!-- 将自定义作用域注册到spring -->
<bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
	<property name="scopes">
  	<map>
    	<entry key="myScope" value-ref="selfScope"></entry>
    </map>
  </property>
</bean>

<!-- 使用自定义作用域 -->
<bean id="testBean" class="package.TestBean" scope="myScope"></bean>
```

​		线程作用域

```xml
<!-- 将线程作用域设置为bean -->
<bean id="threaScope" class="org.springframework.context.support.SimpleThreadScope"></bean>
  
<!-- 将自定义作用域注册到spring -->
<bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
	<property name="scopes">
  	<map>
    	<entry key="thScope" value-ref="threadScope"></entry>
    </map>
  </property>
</bean>

<!-- 使用自定义作用域 -->
<bean id="testBean" class="package.TestBean" scope="thScope"></bean>
```



##### 3.设置bean的懒加载:

```xml
<!-- 通过lazy-init设置懒加载,仅对单利有效 -->
<bean id="testBean" class="package.TestBean" lazy-init="true"></bean>

<!-- 通过default-lazy-init设置默认懒加载,仅对单利有效,默认所有单利作用域的bean都会懒加载 -->
<beans default-lazy-init="true"></beans>
```

##### 4.设置bean的初始化和销毁逻辑:

​		通过继承InitializingBean和DisposableBean 自定义初始化和销毁逻辑:

```java
public class TestBean implements InitializingBean,DisposableBean{
  @Override
  public void afterPrpertiesSet() throws Exception{
    //初始化逻辑
  }
  
  @Override
  public void destory() throws Exceptions{
    //销毁逻辑
  }
}

```

​		通过init-method和destory-method自定义初始化和销毁逻辑:

```java
public class TestBean{
  public void onInit(){
    //初始化逻辑
  }
  
  public void onDestory(){
    //销毁逻辑
  }
}
```

```xml
<!-- 通过init-method和destory-method自定义初始化和销毁逻辑 -->
<bean id="testBean" class="package.TestBean" init-method="onInit" destory-method="onDestory"></bean>
```

​	通过default-init-method和default-destory-method自定义初始化和销毁逻辑:

```xml
<!-- 通过default-init-method和default-destory-method自定义初始化和销毁逻辑 -->
<beans default-init-method="onInit" default-destory-method="onDestory"></beans>
```



##### 5.设置bean的继承结构:

```xml
<!-- bean之间具有真实继承结构，父bean真实存在，父bean也需要被实例化-->
<bean id="parentBean" class="package.ParentClass">
	<property name="p1" ref="b1"></property>
  <property name="p2" value="b2"></property>
</bean>
<bean id="childrenBean1" class="package.ChildrenClass1"  parent="parentBean">
  <!--childrenBean1自动继承父bean的p1和p2属性-->
	<property name="p3" ref="b3"></property>
  <property name="p4" value="b4"></property>
</bean>
<bean id="childrenBean2" class="package.ChildrenClass2"  parent="parentBean">
	<!--childrenBean2自动继承父bean的p1和p2属性-->
	<property name="p5" ref="b5"></property>
  <property name="p6" value="b6"></property>  
</bean>
  
<!-- bean之间具有真实继承结构，父bean真实存在，但父bean无需被容器实例化，仅起复用作用-->
<bean id="parentBean" class="package.ParentClass" abstract="true">
	<property name="p1" ref="b1"></property>
  <property name="p2" value="b2"></property>
</bean>
<bean id="childrenBean1" class="package.ChildrenClass1"  parent="parentBean">
  <!--childrenBean1自动继承父bean的p1和p2属性-->
	<property name="p3" ref="b3"></property>
  <property name="p4" value="b4"></property>
</bean>
<bean id="childrenBean2" class="package.ChildrenClass2"  parent="parentBean">
	<!--childrenBean2自动继承父bean的p1和p2属性-->
	<property name="p5" ref="b5"></property>
  <property name="p6" value="b6"></property>  
</bean>

<!-- bean之间不具有真实继承结构，父bean并不真实存在，仅为复用bean的属性-->
<bean id="parentBean" class="package.ParentClass" abstract="true">
	<property name="p1" ref="b1"></property>
  <property name="p2" value="b2"></property>
</bean>
<bean id="childrenBean1" class="package.ChildrenClass1"  parent="parentBean">
  <!--childrenBean1自动继承父bean的p1和p2属性-->
	<property name="p3" ref="b3"></property>
  <property name="p4" value="b4"></property>
</bean>
<bean id="childrenBean2" class="package.ChildrenClass2"  parent="parentBean">
	<!--childrenBean2自动继承父bean的p1和p2属性-->
	<property name="p5" ref="b5"></property>
  <property name="p6" value="b6"></property>  
</bean>
 
```



### 4.3通过注解方式配置Ioc元数据:

#### 1.通过configure代替xml配置文件:

```java
@Configuration
public class Configure{
}
```

#### 2.指定获取bean的方式:

```java
/* 通过在Configure中定义获取bean的方法创建bean*/
@Configuration
public class Configure{
  /* 通过@Bean注解修饰的方法被认为是一个bean的创建方法，
   * 方法创建的bean的名称默认为方法名称
   * 返回的bean类型由返回值决定
   */
  @Bean
  public TestBean testbean(){
    return new TestBean();
  }
}

/* 自动扫描指定包路径下的类为bean*/
@Configuration
//通过添加ComponentScan可以将指定包路径下被@Component/@Services/@Controller/@Repository修饰的类自动扫描为bean；
@ComponentScan("packagepath")
public class Configure{
}
```

#### 3.注入bean的依赖:

##### 1.通过构造方法注入

```java
@Component
public class Bean1{
  private Bean2 bean2;
 	/* 在构造方法中声明需要注入的bean即可
 	 * 此种方式仅能够注入bean，不能注入基本类、集合等其他类型数据；
 	 */
  public Bean1(Bean2 bean2){
    this.bean2 = bean2;
  }
}
```

##### 2.通过set方法注入

```java
@Component
public class Bean1{
  private Bean2 bean2;
  /*
   * 通过Autowired注解修饰set方法来注入bean
   */
  @Autowired
  public setBean2(Bean2 bean2){
    this.bean2 = bean2;
  }
}
```

##### 3.通过属性注入

```java
@Component
public class Bean1{
  /*
   * 通过Autowired注解修饰类型为bean的属性即可注入bean
   */
  @Autowired
  private Bean2 bean2;
  
}
```

##### 4.注入集合数据:

​	自动从容器中查找bean注入List属性:(Set集合类注入方式和List相同)

```java
/*通过set方法注入List*/
@Component
public class Bean1{
  private List<String> list;
  /*
   * 通过Autowired注解set方法注入list数据
   */
  @Autowired
  public List<String> setList(List<String> list){
    this.list = list;
  }
}

//自动注入类型与List泛型相同的bean作为bean的元素；
@Configuration
public class Configure{
  @Bean
  public String str1(){
    return "str1";
  }
  
  @Bean
  public String str2(){
    return "str2";
  }
}

//list数据:["str1","str2"]
```

​		指定注入List属性的bean(Set集合类注入方式和List相同)：

```java
//指定需要为List注入的bean；
@Configuration
public class Configure{
  @Bean
  public ArrayList<String> strlist(){
    return "str1";
  }
}

/*通过set方法注入List*/
@Component
public class Bean1{
  private List<String> list;
  /*
   * 通过Autowired注解set方法注入list数据
   * 通过Qualifier注解指定要从容器中注入的bean
   */
  @Autowired
  @Qualifier("strlist")
  public List<String> setList(List<String> list){
    this.list = list
  }
}
//list数据:["str1"]
```

​	自动注入map数据

```java
//指定需要为List注入的bean；
@Configuration
public class Configure{
  @Bean
  public Integer intnum1(){
    return 11;
  }
  
  @Bean
  public Integer intnum2(){
    return 22;
  }
}

/*通过set方法注入List*/
@Component
public class Bean1{
  private Map<String,Integer> map;
  /*
   * 通过Autowired注解set方法注入list数据
   * 通过Qualifier注解指定要从容器中注入的bean
   */
  @Autowired
  public Map<String,Integer> setMap(Map<String,Integer> map){
    this.map = map
  }
}

//map数据:{"intnum1":11,"intnum2":12}
```

​	手动指定要为map注入的bean

```java
//指定需要为List注入的bean；
@Configuration
public class Configure{
  @Bean
  public HashMap<String,Integer> map1(){
    HashMap<String,Integer> hmap = new HashMap<String,Integer>;
    hmap.put("map1",11);
    return hmap;
  }
  
  @Bean
  public Integer intnum2(){
    return 22;
  }
}

/*通过set方法注入List*/
@Component
public class Bean1{
  private Map<String,Integer> map;
  /*
   * 通过Autowired注解set方法注入list数据
   * 通过Qualifier注解指定要从容器中注入的bean
   */
  @Autowired
  @Qualifier("map1")
  public Map<String,Integer> setMap(Map<String,Integer> map){
    this.map = map
  }
}

//map数据:{"map1":11}
```

##### 5.注入ApplicationContext

​	其他如BeanFactory等也都可以直接注入到bean中

```java
@Component
public class Bean1{
  private ApplicationContext context;
  /*
   * 通过Autowired注解set方法注入list数据
   * 通过Qualifier注解指定要从容器中注入的bean
   */
  @Autowired
  public ApplicationContext setContext(ApplicationContext context){
    this.context = context;
  }
}
```

#### 4.设置bean的属性：

##### 1.设置bean的名称:

```java
/*通过@Bean声明bean的名称*/
@Configuration
public class Configure{
  /*
   * 默认bean的名称为bean1，即方法的名称
   * 也可通过@Bean指定bean的名称
   */
  @Bean("b1")
  public Bean1 bean1(){
    return new Bean1();
  }
}

/*通过@Component/@Services/@Controller/@Repository声明bean的名称*/
@Component("b1")
public class Bean1{}
```

##### 2.设置作用域:

```java
/*通过@Scope声明bean的作用域*/
@Configuration
@Scope("singleton")
public class Configure{}

@Configuration
public class Configure{
  /*
   * 默认bean的名称为bean1，即方法的名称
   * 也可通过@Bean指定bean的名称
   */
  @Bean("b1")
  @Scope("prototype")
  public Bean1 bean1(){
    return new Bean1();
  }
}

//自定义作用域
@Configuration
public class Configure{
  /*
   * 默认bean的名称为bean1，即方法的名称
   * 也可通过@Bean指定bean的名称
   */
  @Bean("b1")
  //使用自定义作用域
  @Scope("myscope")
  public Bean1 bean1(){
    return new Bean1();
  }
  
  @Bean
  public MyScope myscope(){
    return new MyScope();
  }
  
  @Bean
  public CustomScopeConfigurer cusscope(){
    CustomScopeConfigurer cusscope = new CustomScopeConfigurer();
    cusscope.add("myscope",myscope());
    return cusscope;
  }
}

//结局单利嵌套原型导致生命周期不协同的问题:lookup-method
public abstract class Bean1(){
  private Bean2 bean2;
  
  @Lookup
  public abstract Bean2 getbean2();
  
  public Bean2 getBean2(){
    return getbean2();
  }
  
}
```

##### 3.设置懒加载:

```java
/* 通过在类上添加@Lazy*/
@Component("b1")
@Lazy
public class Bean1{}

/* 通过在Configuration中添加@Lazy注解*/
@Configuration
@Lazy
public class Configure{
  /*
   * 默认bean的名称为bean1，即方法的名称
   * 也可通过@Bean指定bean的名称
   */
  @Bean("b1")
  public Bean1 bean1(){
    return new Bean1();
  }
}
        
/* 通过在Bean创建方法中添加@Lazy注解*/
@Configuration
public class Configure{
  /*
   * 默认bean的名称为bean1，即方法的名称
   * 也可通过@Bean指定bean的名称
   */
  @Bean("b1")
  @Lazy
  public Bean1 bean1(){
    return new Bean1();
  }
}
```

##### 4.设置初始化和销毁逻辑:

```java
//继承InitializingBean,DisposableBean实现
public class TestBean implements InitializingBean,DisposableBean{
  @Override
  public void afterPrpertiesSet() throws Exception{
    //初始化逻辑
  }
  
  @Override
  public void destory() throws Exceptions{
    //销毁逻辑
  }
}

//通过@PostConstruct和@PreDestory实现
public class TestBean{
  @PostConstruct
  public void onInit(){
    //初始化逻辑
  }
  
  @PreDestory
  public void onDestory(){
    //销毁逻辑
  }
}

//通过@Bean注解的initMethod和destoryMethod指定初始化方法和销毁方法实现
@Configuration
public class Configure{
  @Bean("b1",initMethod="onInit",destoryMethod="onDestory")
  public Bean1 bean1(){
    return new Bean1();
  }
}

```

