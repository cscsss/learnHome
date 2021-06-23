# java实现注解的底层原理和概念
-  java注解是JDK1.5引入的一种注释机制，java语言的类、方法、变量、参数和包都可以被注解标注。和Javadoc不同，java注解可以通过反射获取标注内容
-  在编译器生成.class文件时，注解可以被嵌入字节码中，而jvm也可以保留注解的内容，在运行时获取注解标注的内容信息
-  java提供的注解可以分成两类：

<br/>**作用在代码上的功能注解（部分）**:

注解名称 | 功能描述
--- | ---
@Override | 检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误
@Deprecated | 标记过时方法。如果使用该方法，会报编译警告
@SuppressWarnings | 指示编译器去忽略注释解中声明的警告
@FunctionalInterface | java8支持，标识一个匿名函数或函数式接口

<br/>**让给程序员开发自定义注解的元注解（和关键字@interface配合使用的注解）**

元注解名称 | 功能描述
--- | ---
@Retention | 标识这个注释解怎么保存，是只在代码中，还是编入类文件中，或者是在运行时可以通过反射访问
@Documented | 标识这些注解是否包含在用户文档中
@Target | 标识这个注解的作用范围
@Inherited | 标识注解可被继承类获取
@Repeatable | 标识某注解可以在同一个声明上使用多次

-	Annotation是所有注解类的共同接口，不用显示实现。注解类使用@interface定义（代表它实现Annotation接口），搭配元注解使用，如下
```java
package java.lang.annotation;
public interface Annotation {
    boolean equals(Object obj);
    int hashCode();
    String toString();
    // 返回定义的注解类型，你在代码声明的@XXX,相当于该类型的一实例
    Class<? extends Annotation> annotationType();
}
-----自定义示例-----
@Retention( value = RetentionPolicy.RUNTIME)
@Target(value = ElementType.TYPE)
public @interface ATest {
	String hello() default  "siting";
}
```
ATest的字节码文件，编译器让自定义注解实现了Annotation接口
```java
public abstract @interface com/ATest implements java/lang/annotation/Annotation {
  // compiled from: ATest.java
  @Ljava/lang/annotation/Retention;(value=Ljava/lang/annotation/RetentionPolicy;.RUNTIME)
  @Ljava/lang/annotation/Target;(value={Ljava/lang/annotation/ElementType;.TYPE})
  // access flags 0x401
  public abstract hello()Ljava/lang/String;
    default="siting"
}
```
-	自定义注解类型时，一般需要用@Retention指定注解保留范围RetentionPolicy，@Target指定使用范围ElementType。RetentionPolicy保留范围只能指定一个，ElementType使用范围可以指定多个
-	注解信息怎么和代码关联在一起，java所有事物都是类，注解也不例外，加入代码`System.setProperty("sum.misc.ProxyGenerator.saveGeneratedFiles","true");` 可生成注解相应的代理类
![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6882addffc7d42048cfc84508a3bdea9~tplv-k3u1fbpfcp-zoom-1.image)
在代码里定义的注解，会被jvm利用反射技术生成一个代理类，然后和被注释的代码（类，方法，属性等）关联起来

# 五种元注解详解
-  @Retention：指定注解信息保留阶段，有如下三种枚举选择。只能选其一
```java
public enum RetentionPolicy {
    /** 注解将被编译器丢弃，生成的class不包含注解信息 */
    SOURCE,
    /** 注解在class文件中可用，但会被JVM丢弃;当注解未定义Retention值时，默认值是CLASS */
    CLASS,
    /** 注解信息在运行期(JVM)保留（.class也有），可以通过反射机制读取注解的信息,
      * 操作方法看AnnotatedElement(所有被注释类的父类) */
    RUNTIME
}
```
-  @Documented：作用是告诉JavaDoc工具，当前注解本身也要显示在Java Doc中(不常用)
-  @Target：指定注解作用范围，可指定多个
```java
public enum ElementType {
    /** 适用范围：类、接口、注解类型，枚举类型enum */
    TYPE,
    /** 作用于类属性 (includes enum constants) */
    FIELD,
    /** 作用于方法 */
    METHOD,
    /** 作用于参数声明 */
    PARAMETER,
    /** 作用于构造函数声明 */
    CONSTRUCTOR,
    /** 作用于局部变量声明 */
    LOCAL_VARIABLE,
    /** 作用于注解声明 */
    ANNOTATION_TYPE,
    /** 作用于包声明 */
    PACKAGE,
    /** 作用于类型参数（泛型参数）声明 */
    TYPE_PARAMETER,
    /** 作用于使用类型的任意语句(不包括class) */
    TYPE_USE
}
```
TYPE_PARAMETER的用法示例
```java
class D<@PTest T> { } // 注解@PTest作用于泛型T
```
TYPE_USE的用法示例
```java
//用于父类或者接口 
class Test implements @Parent TestP {} 

//用于构造函数
new @Test String("/usr/data")

//用于强制转换和instanceof检查,注意这些注解中用于外部工具
//它们不会对类型转换或者instanceof的检查行为带来任何影响
String path=(@Test String)input;
if(input instanceof @Test String) //注解不会影响

//用于指定异常
public Person read() throws @Test IOException.

//用于通配符绑定
List<@Test ? extends Data>
List<? extends @Test Data>

@Test String.class //非法，不能标注class
```
-  @Inherited：表示**当前注解会被注解类的子类继承**。即在子类Class<T\>通过getAnnotations()可获取父类被@Inherited修饰的注解。而注解本身是不支持继承
```java
@Inherited
@Retention( value = RetentionPolicy.RUNTIME)
@Target(value = ElementType.TYPE)
public @interface ATest {  }
----被ATest注解的父类PTest----
@ATest
public class PTest{ }

---Main是PTest的子类----
public class Main extends PTest {
    public static void main(String[] args){
        Annotation an = Main.class.getAnnotations()[0];
  		//Main可以拿到父类的注解ATest，因为ATest被元注解@Inherited修饰
        System.out.println(an);
    }
}  
---result--
@com.ATest()  
```
-	@Repeatable：JDK1.8新加入的，表明自定义的注解可以在同一个位置重复使用。在没有该注解前，是无法在同一个类型上使用相同的注解多次
```java
  //Java8前无法重复使用注解
  @FilterPath("/test/v2")
  @FilterPath("/test/v1")
  public class Test {}
```
  
# 使用动态代理机制处理注解
-	反射机制获取注解信息
```java
--- 作用于注解的注解----
@Inherited
@Retention( value = RetentionPolicy.RUNTIME)
@Target(value = {ElementType.ANNOTATION_TYPE})
public @interface AnnotationTest {
    String value() default "AnnotationTest";
}
------父类-------
public class PTest {}
------被注解修饰的package-info.java------
//package-info.java
@AnTest("com-package-info")
package com;
-------------
@AnnotationTest("AnnotationTest")
@Inherited
@Retention( value = RetentionPolicy.RUNTIME)
@Target(value = {ElementType.TYPE_USE,ElementType.PACKAGE,ElementType.FIELD,
        ElementType.TYPE_PARAMETER,ElementType.CONSTRUCTOR,ElementType.LOCAL_VARIABLE})
public @interface AnTest  {
    String value() default  "siting";
}
```
运行示例
```java
//注解类
@AnTest("mainClass")
//注解泛型参数                                       //注解继承父类
public class Main<@AnTest("parameter") T > extends @AnTest("parent") PTest {
    @AnTest("constructor")  //注解构造函数
    Main(){ }
    //注解字段域
    private @AnTest("name") String name;
    //注解泛型字段域
    private @AnTest("value") T value;
    //注解通配符
    private @AnTest("list")List<@AnTest("generic") ?>list;
    //注解方法
    @AnTest("method")                       //注解方法参数
    public String hello(@AnTest("methodParameter") String name)
            throws @AnTest("Exception") Exception { // 注解抛出异常
        //注解局部变量,现在运行时暂时无法获取（忽略）
        @AnTest("result") String result;
        result = "siting";
        System.out.println(name);
        return  result;
    }

    public static void main(String[] args) throws Exception {

        Main<String>  main = new Main<> ();
        Class<Main<Object>> clazz = (Class<Main<Object>>) main.getClass();
        //class的注解
        Annotation[] annotations = clazz.getAnnotations();
        AnTest testTmp = (AnTest) annotations[0];
        System.out.println("修饰Main.class注解value: "+testTmp.value());
        //构造器的注解
        Constructor<Main<Object>> constructor = (Constructor<Main<Object>>) clazz.getDeclaredConstructors()[0];
        testTmp = constructor.getAnnotation(AnTest.class);
        System.out.println("修饰构造器的注解value: "+testTmp.value());
        //继承父类的注解
        AnnotatedType annotatedType = clazz.getAnnotatedSuperclass();
        testTmp = annotatedType.getAnnotation(AnTest.class);
        System.out.println("修饰继承父类的注解value: "+testTmp.value());
        //注解的注解
        AnnotationTest annotationTest = testTmp.annotationType().getAnnotation(AnnotationTest.class);
        System.out.println("修饰注解的注解AnnotationTest-value: "+annotationTest.value());
        //泛型参数 T 的注解
        TypeVariable<Class<Main<Object>>> variable = clazz.getTypeParameters()[0];
        testTmp = variable.getAnnotation(AnTest.class);
        System.out.println("修饰泛型参数T注解value: "+testTmp.value());
        //普通字段域 的注解
        Field[] fields = clazz.getDeclaredFields();
        Field nameField = fields[0];
        testTmp = nameField.getAnnotation(AnTest.class);
        System.out.println("修饰普通字段域name注解value: "+testTmp.value());
        //泛型字段域 的注解
        Field valueField = fields[1];
        testTmp = valueField.getAnnotation(AnTest.class);
        System.out.println("修饰泛型字段T注解value: "+testTmp.value());
        //通配符字段域 的注解
        Field listField = fields[2];
        AnnotatedParameterizedType annotatedPType = (AnnotatedParameterizedType)listField.getAnnotatedType();
        testTmp = annotatedPType.getAnnotation(AnTest.class);
        System.out.println("修饰泛型注解value: "+testTmp.value());
        //通配符注解 的注解
        AnnotatedType[] annotatedTypes = annotatedPType.getAnnotatedActualTypeArguments();
        AnnotatedWildcardType annotatedWildcardType = (AnnotatedWildcardType) annotatedTypes[0];
        testTmp = annotatedWildcardType.getAnnotation(AnTest.class);
        System.out.println("修饰通配符注解value: "+testTmp.value());
        //方法的注解
        Method method = clazz.getDeclaredMethod("hello", String.class);
        annotatedType = method.getAnnotatedReturnType();
        testTmp = annotatedType.getAnnotation(AnTest.class);
        System.out.println("修饰方法的注解value: "+testTmp.value());
        //异常的注解
        annotatedTypes =  method.getAnnotatedExceptionTypes();
        testTmp = annotatedTypes[0].getAnnotation(AnTest.class);
        System.out.println("修饰方法抛出错误的注解value: "+testTmp.value());
        //方法参数的注解
        annotatedTypes = method.getAnnotatedParameterTypes();
        testTmp = annotatedTypes[0].getAnnotation(AnTest.class);
        System.out.println("修饰方法参数注解value: "+testTmp.value());
        //包的注解
        Package p = Package.getPackage("com");
        testTmp = p.getAnnotation(AnTest.class);
        System.out.println("修饰package注解value: "+testTmp.value());
        main.hello("hello");
    }
}
```
结果
```java
修饰Main.class注解value: mainClass
修饰构造器的注解value: constructor
修饰继承父类的注解value: parent
修饰注解的注解AnnotationTest-value: AnnotationTest
修饰泛型参数T注解value: parameter
修饰普通字段域name注解value: name
修饰泛型字段T注解value: value
修饰泛型注解value: list
修饰通配符注解value: generic
修饰方法的注解value: method
修饰方法抛出错误的注解value: Exception
修饰方法参数注解value: methodParameter
修饰package注解value: com-package-info
hello
```
# spring.AOP和注解机制
spring.AOP相当于**动态代理和注解机制**在spring框架的结合实现

- 前要知识：面向切面编程(AOP)和动态代理
	- C是面向过程编程的，java则是面向对象编程，C++则是两者兼备，它们都是一种规范和思想。面向切面编程也一样，可以简单理解为：**切面**编程专注的是局部代码，主要为某些点植入增强代码
	- 考虑要局部加入增强代码，使用动态代理则是最好的实现。在被代理方法调用的前后，可以加入需要的增强功能；因此spring的切面编程是基于动态代理的
- 切面的概念
	
   概念 |   描述
--- | ----
   通知（Advice） | 需要切入的增强代码逻辑被称为通知
   切点（Pointcut） | 定义增强代码在何处执行
   切面（Aspect） | 定义：切面是通知和切点的集合
   连接点（JoinPoint） | 在切点基础上，指定增强代码在切点执行的时机(在切点前，切点后，抛出异常后等)
   目标（target）| 被增强目标类

-	spring.aop提供的切面注解

切面编程相关注解 | 功能描述
--- | ---
@Aspect | 作用于类，声明当前方法类是增强代码的切面类
@Pointcut |	作用于方法，指定需要被拦截的其他方法。当前方法则作为拦截集合名使用

-	spring的通知注解其实是**通知+指定连接点**组成，分五种（Before、After、After-returning、After-throwing、Around）

spring通知（Advice）注解 | 功能描述
--- | ---
@After | 增强代码在@Pointcut指定的方法之后执行
@Before | 增强代码在@Pointcut指定的方法之前执行
@AfterReturning | 增强代码在@Pointcut指定的方法 return返回之后执行
@Around | 增强代码可以在被拦截方法前后执行
@AfterThrowing | 增强代码在@Pointcut指定的方法抛出异常之后执行

-	在spring切面基础上，开发具有增强功能的自定义注解 **(对注解进行切面)**
```java
新建spring-web + aop 项目；新建如下class
------ 目标Controller ------
@RestController
public class TestController {
    @STAnnotation
    @RequestMapping("/hello")
    public String hello(){  return "hello@csc";  }
}
------ Controller注解 -------
@Retention( value = RetentionPolicy.RUNTIME)
@Target(value = ElementType.METHOD)
public @interface STAnnotation {
    String value() default "注解hello!";
}
------ Controller切面 ------
@Aspect
@Component
public class ControllerAspect {
    //切点:注解指定关联 (对注解进行切面)
    @Pointcut("@annotation(STAnnotation)")
    public void controllerX(){}
    //切点:路径指定关联
    @Pointcut("execution(public * com.example.demo.TestController.*(..))")
    public void controllerY(){}
    //在controllerY()切点执行之前的连接点加入通知
    @Before("controllerY()")
    public void yBefore(JoinPoint joinPoint) throws Throwable {
    	//可以加入增强代码
        MethodSignature methodS = (MethodSignature)joinPoint.getSignature();
        Method method = methodS.getMethod();
        if (method.isAnnotationPresent(STAnnotation.class)) {
            STAnnotation annotation = method.getAnnotation(STAnnotation.class);
            System.out.println(annotation.value());
        }
        System.out.println("controllerY");
    }
    //controllerX()切点执行之后的连接点加入通知
    @After("controllerX()")
    public void xBefore(JoinPoint joinPoint) throws Throwable {
    	//可以加入增强代码
        System.out.println("controllerX");
    }
}
```
**启动项目；执行`curl  http://127.0.0.1:8080/hello`**,控制台输出如下
![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5020c36ee193492b85902c0dc035a168~tplv-k3u1fbpfcp-zoom-1.image)
# （题外）@FunctionalInterface原理介绍
-	Lambda 表达式的结构：（...args）-> { ... code } 
	-	lambda在python,C++都对应的定义，java也有，lambda一般由入参，处理过程组成。如果处理代码只有一行，中括号{} 可以省略。其实就是简化的函数。在java里，lambda用函数式接口实现
-	@FunctionalInterface作用于接口，接口可以接受lambda表达式作为右值，此类接口又叫函数式接口，其规定修饰的接口只能有一个抽象的方法（不包扣静态方法和默认、私有方法）。attention:不加@FunctionalInterface修饰，只定义一个抽象方法的接口默认也是函数式接口
```java
@FunctionalInterface
public interface Func {  void hello(String name); }
---------------------
public static void main(String[] args) {
        Func func = (name) -> System.out.println(name);
        func.hello("siting");
    }
```
查看对应的Main.class字节码文件 `javap.exe -p -v -c Main.class`
```java
Constant pool:
   #1 = Methodref          #8.#28         // java/lang/Object."<init>":()V
   //常量值中前面的#0表示引导方法取BootstrapMethods属性表的第0项(字节码在最下面)
   #2 = InvokeDynamic      #0:#33         // #0:hello:()Lcom/Func;
   #3 = String             #34            // siting
   #4 = InterfaceMethodref #35.#36        // com/Func.hello:(Ljava/lang/String;)V
   #5 = Fieldref           #37.#38        // java/lang/System.out:Ljava/io/PrintStream;
   #6 = Methodref          #39.#40        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #7 = Class              #41            // com/Main
 .... // main执行字节码
 public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
      	 // 动态获得一个CallSite对象，该对象是一个内部类，实现了Func接口
         0: invokedynamic #2,  0              // InvokeDynamic #0:hello:()Lcom/Func;
         5: astore_1
         6: aload_1
         7: ldc           #3                  // String siting
         // 调用CallSite对象的hello方法
         9: invokeinterface #4,  2            // InterfaceMethod com/Func.hello:(Ljava/lang/String;)V
        14: return
.... //lambda表达式 会编译出私有静态类        
private static void lambda$main$0(java.lang.String);
    descriptor: (Ljava/lang/String;)V
    flags: ACC_PRIVATE, ACC_STATIC, ACC_SYNTHETIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: aload_0
         4: invokevirtual #6                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         7: return
.... //lambda表达式 会编译出一个对应的内部类         
SourceFile: "Main.java"
InnerClasses:
     public static final #59= #58 of #62; //Lookup=class java/lang/invoke/MethodHandles$Lookup of class java/lang/invoke/MethodHandles
BootstrapMethods:
  0: #30 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lan
g/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #31 (Ljava/lang/String;)V
      //调用Main方法里的lambda$main$0静态方法（真正执行lambda的逻辑的方法）
      #32 invokestatic com/Main.lambda$main$0:(Ljava/lang/String;)V
      #31 (Ljava/lang/String;)V
```
从上面的字节码可看出，1：lambda表达式会被编译成一个私有静态方法和一个内部类；2：内部类实现了函数式接口，而实现方法会调用一个Main.class里一静态方法 3：静态方法lambda$main$0里是我们自己写的代码逻辑。运行参数加上`-Djdk.internal.lambda.dumpProxyClasses`可以查看lambda对应内部类的具体信息
![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6eaac73ce6844dba82e37277300d0a33~tplv-k3u1fbpfcp-zoom-1.image)
-	常用函数式接口

接口 | 描述
--- | ---
Predicate |判断：传入一个参数，返回一个bool结果， 方法为boolean test(T t)
Consumer|消费：传入一个参数，无返回值， 方法为void accept(T t)
Function|转化处理：传入一个参数，返回一个结果，方法为R apply(T t)
Supplier|生产：无参数传入，返回一个结果，方法为T get()
BiFunction |转化处理：传入两个个参数，返回一个结果，方法R apply(T t, U u)
BinaryOperator |二元操作符， 传入的两个参数的类型和返回类型相同， 继承 BiFunction

---
欢迎指正文中错误
---
**关注公众号，一起交流，微信搜一搜: 潜行前行**
---
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE5LzE3MzY3Yjk0ZGEwZTlmZDI?x-oss-process=image/format,png)
# 参考文章
-	[Annotation详解](https://blog.csdn.net/u013703461/article/details/66259013)  
-	[Java注解（Annotation）原理详解](https://blog.csdn.net/lylwo317/article/details/52163304)
-	[Java Lambda表达式 实现原理分析](https://blog.csdn.net/jiankunking/article/details/79825928)
