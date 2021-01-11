# spring-Aop

## 1.切面编程:

​		在业务代码中，除过业务的核心逻辑以外，通常还会有一些非业务代码，比如日志记录、事务控制、权限校验等；这些非核心业务逻辑如果直接写在业务逻辑中，会和核心业务代码耦合在一起，不方便对核心业务逻辑进行梳理，也不方便对这些非核心业务逻辑进行复用！那么可以在编码时将这些非核心业务逻辑独立出来，运行或者编译时再以某种方式将这些非核心业务代码`填充至`合适的地方即可；

​		那么，将非核心业务逻辑在编码时独立出来，在编译或者运行时通过某种机制将代码填充至需要执行的地方，可以清晰的组织代码结构，复用非核心业务代码；这种组织方式称之为切面编程，即AOP；

## 2.切面编程中的基本概念:

​		AOP不仅仅只是把非核心业务逻辑和核心的业务逻辑剥离，最终还是需要以某种机制将非核心业务逻辑和业务逻辑像连接 '铆钉' 连接在一起，才能在核心业务代码中执行必要的非核心代码！

​		在spring中，表示连接核心业务代码和非核心业务代码的'铆钉'称之为**切点**和**连接点**；切点表示对外暴露的非核心业务代码，连接点表示需要连接的核心业务代码，切点和连接点互相结合表示需要在哪个核心业务逻辑中执行哪个非核心业务逻辑！在spring中定义切点和连接点的方式如下:

```java
/*
 * @Pointcut表示一个切点，切点连接着由通知定义的非核心业务代码；
 * 其中的execution(...)内部是一个定义方法的表达式，表示一个连接点，连接某一个或者某一类方法；
 * 而pointcuta方法是一个切点的载体！pointcuta称为该切点的名称！
 */
@Pointcut(execution(* com.sslike.TestAop.t(..)))
public void pointcuta(){}
```

​	其中，切点连接的非核心业务代码称之为**通知**，表示具体要执行的非核心业务逻辑，通知和具体的连接点相连；其定义方式如下:

```java
/*
 * @Before表示通知方式，即何时执行该非核心业务代码；
 * doSomething方法表示具体的非核心业务代码；通知和切点通过切点的名称相连！
 */
@Before("pointcuta")
public void doSomething(){
		//非核心业务逻辑
}
```

​	切点所在的类称之为**切面**，表示承载切点的平台；		

​	AOP的整体工作原理如下：

![aop1](aop1.png)

## 3.注解方式定义切面、切点、连接点、通知

### 1.定义切面：

​		切面是承载非核心业务代码的平台，体现在代码上是一个类；需要由@Aspect修饰

```java
@Aspect
public class MyAspect{
}
```

### 2.定义切点和连接点:

​		spring中必须以空方法作为切点载体，并在切点中定义连接点表示当前切点需要和哪个连接点相连！

#### 1.定义切点:

```java
@Aspect
public class MyAspect{
  /*
   * 1.切点必须以一个空方法作为载体，方法的名称为切点的名称；
   * 2.切点必须由@Pointcut修饰；
   * 3.定义切点时必须定义当前切点要连接的连接点；
   */
  @Pointcut("....")
  public void pointcuta(){}
}
```

#### 2.定义连接点

```java
@Aspect
public class MyAspect{
  /*
   * 1.连接点必须和切点一起定义；
   * 2.连接点时一个表达式，其中execution表示执行方法时触发通知，执行非核心业务逻辑；
   * 3.连接点表达式最终必须定位到一个或者一类方法，该表达式第一位表示返回值，如果任意使用*表示；第二位表示通过全路径名称定义的一个方法，如果某个部分任意可以使用*代替，()表示方法的参数类型，如果任意则使用..代替；
   */
  @Pointcut("execution(* com.sslike.packagename.classname.funname(..))")
  public void pointcuta(){}
}
```

### 3.定义通知:

​	通知类型有5中，通知必须指定切点，表示为切点和连接点连接的方法指定待执行的非核心业务逻辑！

```java
@Aspect
public class MyAspect{
  @Pointcut("execution(* com.sslike.packagename.classname.funname(..))")
  public void pointcuta(){}
  
  //表示执行方法之前触发
  @Before("pointcuta()")
  public void doa(){
    //do something...
  }
  
  //表示执行方法之后触发
  @After("pointcuta()")
  public void dob(){
    //do something...
  }
  
  //表示在方法完成且正常返回之后触发
  @AftertReturning("pointcuta()")
  public void doc(){
    //do something...
  }
  
  //表示发生异常之后触发
  @AfterThrowing("pointcuta()")
  public void dod(){
    //do something...
  }
  
  //表示在方法执行之前和之后分别执行
  @Around("pointcuta()")
  public void doe(ProceedingJoinPoint point){
    try{
      //方法执行前执行某些操作
      point.proceed();
      //方法执行之后执行某些操作
    }
  }
}
```

## 4.配置方式定义切面、切点、连接点、通知:

​		除过使用注解的方式定义aop配置信息以外，还可以早spring-config.xml文件中通过xml的方式定义！

```xml
<!--aop:config定义aop配置信息 -->
<aop:config>
  <!--aop:aspect定义aop切面 -->
   <aop:aspect id="t" ref="test" order="1">
     <!--aop:pointcut定义aop切点，并通过expression属性定义待连接的连接点 -->
     <aop:pointcut id="testAop" expression="execution(* packagename.classname.funname(..))" />
     <!--aop:before定义aop通知，并通过pointcut-ref属性与切点相关联 -->
     <aop:before method="printTime" pointcut-ref="testAop" />
   </aop:aspect>
</aop:config>
```



 