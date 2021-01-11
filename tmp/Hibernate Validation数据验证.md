 

# Validation数据验证

## 1.引入Validation依赖

### 1.第一种方式：

> 对于SpringBoot项目可以通过starter启动器引入Javax.validation

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
    <version>2.3.3.RELEASE</version>
</dependency>
```

### 2.第二种方式：

> 对于普通项目可以通过jar包引入Javax.validation

```xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
```

**说明：**

​	1.通过上述引入的Validation验证框架提供的验证注解均在javax.validation.constraints包下；

## 2.Validation提供的验证注解

### 1.关于空值验证相关:

|   注解    |               作用目标               |                    功能                     |                备注                |
| :-------: | :----------------------------------: | :-----------------------------------------: | :--------------------------------: |
|   @Null   |                Object                |            验证Object是否是null             |       Object为null即返回true       |
| @NotNull  |                Object                |           验证Object是否不是null            |      Object不是null即返回true      |
| @NotEmpty | Charsequence、Collection、Map、Array |     验证目标是否不为null，并且长度大于0     |    Charsequence至少包含一个字符    |
| @NotBlank |             Charsequence             | 验证目标是否不为null，并且trim之后长度大于0 | Charsequence至少包含一个非控制字符 |

### 2.关于范围验证相关：

|                注解                |                         作用目标                         |                             功能                             | 备注 |
| :--------------------------------: | :------------------------------------------------------: | :----------------------------------------------------------: | :--: |
|            @Max(value)             | BigDecimal、BigInteger，byte、short、int、long以及包装类 |                       数值不能大于该值                       |      |
|            @Min(value)             | BigDecimal、BigInteger，byte、short、int、long以及包装类 |                       数值不能小于该值                       |      |
|         @DecimalMax(value)         | BigDecimal、BigInteger，byte、short、int、long以及包装类 |                       数值不能大于该值                       |      |
|         @DecimalMin(value)         | BigDecimal、BigInteger，byte、short、int、long以及包装类 |                       数值不能小于该值                       |      |
| @Length(max=maxValue,min=minValue) |                          String                          |                      String长度不能小于                      |      |
| @Range(max=maxValue,min=minValue)  |                       数值和String                       | 数值的大小或者String的长度不能大于maxValue也不能小于minValue |      |
| @Size(max=maxValue, min=minValue)  |           Charsequence、Collection、Map、Array           |       序列的元素个数不能少于minValue，不能大于maxValue       |      |

### 3.关于正负数验证

|      注解       |                           作用目标                           |       功能        |        备注         |
| :-------------: | :----------------------------------------------------------: | :---------------: | :-----------------: |
|    @Positive    | BigDecimal、BigInteger，byte、short、int、long、float、double以及包装类 |  数值必须为正数   |    不能为0或负数    |
| @PositiveOrZero | BigDecimal、BigInteger，byte、short、int、long、float、double以及包装类 | 数值必须为正数或0 | 可以为0，不能为负数 |
|    @Negative    | BigDecimal、BigInteger，byte、short、int、long、float、double以及包装类 |  数值必须为负数   |   不能为0或者正数   |
| @NegativeOrZero | BigDecimal、BigInteger，byte、short、int、long、float、double以及包装类 | 数值必须为负数或0 | 可以为0，不能为正数 |

### 4.关于日期时间验证

|      注解      |                           作用目标                           |                   功能                   | 备注 |
| :------------: | :----------------------------------------------------------: | :--------------------------------------: | :--: |
|    @Future     | 日期时间相关类型(LocalDateTime、LocalDate、LocalTime、Date、Calendar、Instant) |    参数代表的日期时间必须晚于当前时间    |      |
| @FuturePresent | 日期时间相关类型(LocalDateTime、LocalDate、LocalTime、Date、Calendar、Instant) | 参数代表的日期时间必须晚于或等于当前时间 |      |
|     @Past      | 日期时间相关类型(LocalDateTime、LocalDate、LocalTime、Date、Calendar、Instant) |    参数代表的日期时间必须早于当前时间    |      |
|  @PastPresent  | 日期时间相关类型(LocalDateTime、LocalDate、LocalTime、Date、Calendar、Instant) | 参数代表的日期时间必须早于或等于当前时间 |      |

### 5.其他类型验证

|  注解  | 作用目标 |    功能     | 备注 |
| :----: | :------: | :---------: | :--: |
|  @URL  |  String  | URL地址验证 |      |
| @Email |  String  |  Email验证  |      |

## 3.验证方式

### 1.获取Validator

```java
// 第一种方式
ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();
Validator validator = validatorFactory.getValidator();

//第二种方式
ValidatorFactory validatorFactory = Validation.byDefaultProvider()
    .configure()
    .buildValidatorFactory();
Validator validator = validatorFactory.getValidator();

//第三中方式
ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
    .configure()
    .failFast(true)
    .buildValidatorFactory();
Validator validator = validatorFactory.getValidator();
```

说明：

​	1.Validator默认会将验证目标的所有状态，并将不符合要求的错误信息一起返回；但是可以通过第三中方式配置为一旦出错立即返回；

### 2.使用Validator进行Bean验证

#### 1.验证方法：

`Validator#validate()`                   验证所有bean的所有约束

`	Validator#validateProperty()`   验证单个属性

`	Validator#validateValue()`         检查给定类的单个属性是否可以成功验证

#### 2.使用示例：

```java
//创建对象 
User user = new User(null, "", "!@#$", null, 11);

// 验证所有bean的所有约束
Set<ConstraintViolation<User>> constraintViolations = validator.validate(user);
// 验证单个属性
Set<ConstraintViolation<User>> constraintViolations2 = validator.validateProperty(user, "name");
// 检查给定类的单个属性是否可以成功验证
Set<ConstraintViolation<User>> constraintViolations3 = validator.validateValue(User.class, "password", "sa!");

//获取验证出错信息
constraintViolations.forEach(constraintViolation -> System.out.println(constraintViolation.getMessage()));
constraintViolations2.forEach(constraintViolation -> System.out.println(constraintViolation.getMessage()));
constraintViolations3.forEach(constraintViolation -> System.out.println(constraintViolation.getMessage()));
```

### 3.使用Validator进行方法参数和返回值验证

#### 1.获取Validator

进行方法参数和返回值的验证需要使用ExecutableValidator类型的Validator;获取方式如下：

```java
ExecutableValidator executableValidator = validator.forExecutables();
```

#### 2.验证方法：

`executableValidator#validateParameters()`                   验证方法的参数

`	executableValidator#validateReturnValue()`   			  验证方法的返回值

#### 3.使用示例：

```java
//获取验证方法参数和返回值的Validator
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
executableValidator = factory.getValidator().forExecutables();

```

### 4.使用Validator进行构造函数参数和返回值验证

#### 1.获取Validator

进行构造函数参数和返回值的验证需要使用ExecutableValidator类型的Validator;获取方式如下：

```java
ExecutableValidator executableValidator = validator.forExecutables();
```

#### 2.验证方法：

executableValidator#validateConstructorParameters()`                   验证方法的参数

executableValidator#validateConstructorReturnValue()`   			  验证方法的返回值

#### 3.使用示例：

```java
//获取验证方法参数和返回值的Validator
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
executableValidator = factory.getValidator().forExecutables();

```

### 5.在web应用中验证参数：

## 4.分组验证

## 5.嵌套验证

