# 1 JAVA的Type类型体系
-  先了解下java的Type类型体系(类的类=>类型)，Type是所有类型（原生类型-Class、参数化类型-Parameterizedtype、数组类型-GenericArrayType、类型变量-TypeVariable、基本类型-Class）的共同接口;前两篇反射和注解讲到的Class<T\>就是Type的一实现类
![](https://img-blog.csdnimg.cn/img_convert/d6f000b133974d55a0a9dae24d981525.png)
-  Type下面又有四个子接口类ParameterizedType、TypeVariable、GenericArrayType、WildcardType
    -  **List<E\>表示泛型，E是TypeVariable类型，List<String\>则是ParameterizedType(参数化类型)**，List<String\>里的String称为实际参数类型
    -  具体化泛型中的类型时，可以使用 ? extends 或 ? super来表示继承关系；如`List<? extends Data>`，而里面的 ? 称为通配符类型WildcardType
    -  GenericArrayType 表示一种元素类型是ParameterizedType（参数化类型）或者TypeVariable（类型变量）的数组类型，如T[] 或者 List<E\>[]
-  注解是JDK1.5才出现了的，为了表示被注解的类型的，加入AnnotatedElement类型，字面意思就是被注解的元素。JDK1.8又有了AnnotatedType将Type和被注解元素的概念关联起来。
![](https://img-blog.csdnimg.cn/img_convert/349c5ed6f53a23a31bea0f3ed0e4d953.png)
-  AnnotatedType也有四个子接口，和Type的四个子接口一一对应，如：ParameterizedType类型被注解则被编译器解析成AnnotatedParameterizedType: `@AnTest("list")List<String>list`

# 2 泛型的概念
-  Java 泛型（generics）是JDK1.5中引入的一个新特性，其本质是参数化类型，解决不确定具体对象类型的问题;其所操作的数据类型被指定为一个参数（type parameter）这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法
> 泛型: 把类型明确的工作推迟到创建对象或调用方法的时候才去明确的特殊的类型

# 3 泛型类和泛型方法的示例
-  泛型类的定义
```java
public class MainTest<T> {
    private  T param;
}
public static void main(String[] args){
        MainTest<String> data = new MainTest<String>(){};
        ParameterizedType genType1 = (ParameterizedType)data.getClass().getGenericSuperclass();
  }
```
-  泛型方法的定义
```java
public class MainTest{
    public static void main(String[] args){
        printData("siting");
    }
    static  <T> T printData(T t){
        System.out.println(t);
        return t;
    }
}
```
- 接口和抽象类都可以使用泛型
# 4 类型擦除
-	创建泛型的实例时，jvm是会把具体类型擦除的；编译生成的字节码中不包含泛型中的类型参数，即ArrayList<String\>和ArrayList<Integer\>都擦除成了ArrayList，也就是被擦除成"原生类型"，这就是泛型擦除
```java
public class MainTest {
    public static void main(String[] args){
        List<String> strArr  = new ArrayList<>();
        List<Integer> intArr  = new ArrayList<>();
        Type strClazz = strArr.getClass();
        Type intClazz = intArr.getClass();
    }
}
```
![](https://img-blog.csdnimg.cn/img_convert/48db2091b33a5d09b3d80af561c3b671.png)
-	查看编译后的字节码文件是如何表示的: idea菜单 -> view -> show ByteCode
```java
public class MainTest<T> {
    T param;
    public static void main(String[] args){
        MainTest<String> test = new MainTest<>();
        test.setParam("siting");
    }
    public T getParam() {  return param;   }
    public void setParam(T param) {  this.param = param;  }
}
```
```java
public class com/MainTest {
  ...省略
  public static main([Ljava/lang/String;)V
   L0
    LINENUMBER 7 L0
    NEW com/MainTest
    DUP
    INVOKESPECIAL com/MainTest.<init> ()V
    ASTORE 1
   L1
    LINENUMBER 8 L1
    ALOAD 1
    LDC "siting"     // 调用类型擦除后的setParam(Object)
    INVOKEVIRTUAL com/MainTest.setParam (Ljava/lang/Object;)V
   L2
   ...省略//getParam 的返回值是Object
  public getParam()Ljava/lang/Object;
   L0
    LINENUMBER 10 L0
    ALOAD 0
    GETFIELD com/MainTest.param : Ljava/lang/Object;
    ARETURN
   ...省略//setParam 的入参是Object
  public setParam(Ljava/lang/Object;)V
   L0
    LINENUMBER 11 L0
    ALOAD 0
    ALOAD 1
    PUTFIELD com/MainTest.param : Ljava/lang/Object;
    RETURN
   ...
}
```
-	可以看出T(String)都被转换为Object类型，最初的初始化的String不见了
# 5 泛型的继承
-	子类可以指定父类的泛型参数，可以是已知类（Integer、String等），也可以用子类自己的泛型参数指定
-	泛型被继承时，且指定父类泛型参数，则额外生成的ParameterizedType类型作为子类的父类；如果没有指定父类泛型参数，则直接继承原生类型
```java
public class MainTest<T> {
    T param;
    static public class SubTest1 extends MainTest<String>{}
    static public class SubTest2<R> extends MainTest<R>{}
    //SubTest3继承的时原生类型
    static public class SubTest3 extends MainTest{}
}
```

![](https://img-blog.csdnimg.cn/img_convert/b911244755cc4ea9aa45d1d3eca21a1d.png)
# 6 泛型变量TypeVariable
-	（先临时定义一个名称，Test<E\>里的E为泛型参数）；泛型变量TypeVariable：泛型的泛型参数就是TypeVariable；当父类使用子类的泛型参数指定自身的泛型参数时；或者泛型属性定义在泛型类A<T\>中，并使用泛型类A<T\>的泛型参数T时，其泛型参数都会被编译器定为泛型变量TypeVariable，而不是被擦除
```java
public class MainTest<T> {
    List<T> param;
    public static void main(String[] args) throws Exception{
        Class clazz =  MainTest.class;
        TypeVariable[] typeVariable = clazz.getTypeParameters();
        // 1
        Field field = clazz.getDeclaredField("param");
        ParameterizedType arrayType = (ParameterizedType)field.getGenericType();
        // interface List<E> 的泛型类型E被T,具体化，因此其被识别为 TypeVariable
        TypeVariable variable1 = (TypeVariable)arrayType.getActualTypeArguments()[0];
        // 2
        ParameterizedType type = (ParameterizedType)SubTest.class.getGenericSuperclass();
        TypeVariable variable2 = (TypeVariable)type.getActualTypeArguments()[0];
    }
    static class SubTest<R> extends MainTest<R>{}
}
```
![](https://img-blog.csdnimg.cn/img_convert/d7255078df0218cb3849cec1fd31f564.png)
# 7 参数化类型ParameterizedType
```java
public interface ParameterizedType extends Type {
    //获取实际参数，List<String>里的String; 如果是List<T>则是TypeVariable类型
    Type[] getActualTypeArguments(); 
    // 获取原始类型List<String> -> List<E>
    Type getRawType();  
    Type getOwnerType();
}
```
-  需要注意的点，我们不能直接获取指定具体参数的泛型的类型，如`Class clazz = List<String>.class`编译时不通过的；还有就是直接通过泛型类new创建的对象，其Class并非ParameterizedType类型，而是泛型本身的class，示例如下
```java
public class MainTest<T> {
    public static void main(String[] args){
        MainTest<String> str = new MainTest<String>();
        Class variable = str.getClass();
        Type genType1 = variable.getGenericSuperclass();
    }
}
```
![](https://img-blog.csdnimg.cn/img_convert/7c67cc357f2615649a7a6016b3737069.png)
- 被具体参数化的泛型才能被编译器识别为ParameterizedType类型，有三种方式获取ParameterizedType类型
```java
// 1 子类继承泛型时，指定具体参数（可以是String等已知类型,也可以是子类的泛型参数）
// 2 获取在类内部定义的泛型属性，需指定具体泛型参数
// 3 局部代码，可以通过泛型的匿名内部子类(需指定具体泛型参数)获取ParameterizedType类型
public class MainTest<T> {
    List<T> list;
    public static void main(String[] args) throws NoSuchFieldException {
        SubTest<String> str = new SubTest<>();
        // 方式一
        Class variable = str.getClass();
        // 父类是(521)ParameterizedType类型
        ParameterizedType genType = (ParameterizedType)variable.getGenericSuperclass();
        // (521)ParameterizedType类型的原生类型是（479）class com.MainTest
        Type clazz = genType.getRawType();
        //MainTest.class 的原生类型是 （479）class com.MainTest
        Class rawClazz = MainTest.class;

        //方式二，泛型属性
        Field field = rawClazz.getDeclaredField("list");
        //属性list 类型是(546)ParameterizedType类型List<T>
        ParameterizedType fieldType = (ParameterizedType)field.getGenericType();

        // 方式三
        MainTest<String> sub3 = new MainTest<String>(){};
        // clazz3是匿名子类
        Class clazz3 =  sub3.getClass();
        //父类是(555)ParameterizedType类型
        ParameterizedType genType3 = (ParameterizedType) clazz3.getGenericSuperclass();
        // (555)ParameterizedType类型的原生类型是（479）class com.MainTest
        Type type3 = genType3.getRawType();
    }
    public static class SubTest<R> extends MainTest<R>{ }
}
```
![](https://img-blog.csdnimg.cn/img_convert/7a2294c8128c33048fc12128b10b1d1f.png)

# 8 通配符(WildcardType)
## 无边界通配符：无界通配符 ? 可以适配任何引用类型:
-	当方法参数需要传入一个泛型时，而且无法确定其类型时。直接使用无具体泛型变量的泛型，容易造成安全隐患；若在方法代码里进行类型转换，极容易出现ClassCastException错误
-	那泛型变量用Object代替不就行了？**但是泛型类+具体参数转变的ParameterizedType(参数化类型)是不存在继承关系；即Object是String的父类，但是List<Object\>和List<String\>的类型是不同的两个ParameterizedType，不存在继承关系**。于是有了类型通配符 ？
```java
public static void print(List list){} 
----->>>
public static void print(List<?> list){} 
```
![](https://img-blog.csdnimg.cn/img_convert/51d9e699b55585f85f55644b108045f8.png)
-	无界通配符可以匹配任意类型；但是在使用？时，不能给泛型类的变量设置值，因为我们不知道具体类型是什么；如果强行设置新值，后面的读容易出现ClassCastException错误。因此编译器限制了**通配符 ？**的泛型只能读不能写
## 上界限定通配符 < ? extends E>
-	想接收一个List集合，它只能操作数字类型的元素【Float、Integer、Double、Byte等数字类型都行】，怎么做？可以使用`List<? extends Number的子类>`，表明List里的元素都是Number的子类
```java
    public static void print(List<? extends Number> list) {
        Number n = new Double("1.0");
        list.add(n);
        Number tmp = list.get(0);
    }
```
![](https://img-blog.csdnimg.cn/img_convert/7cee62b6b003cc81e43bc287b4cf9d8b.png)
-	图片里可以看出，存在上界通配符，因为具体类型不确定，也是只能读不能写的
## 下界限定通配符 < ? super E>
```
class Parent{ }
class Child extends Parent{ }
public class MainTest<T> {
    T param;
    public static void main(String[] args){
        MainTest<? super Child> parent_m = new MainTest<>();
        parent_m.setParam(new Child());
        Object parent = parent_m.getParam();
    }
    public T getParam() {  return param;  }
    public void setParam(T param) {  this.param = param; }
}
```
![](https://img-blog.csdnimg.cn/img_convert/67f96c8e57cdf30ddb7013fbe5a468ca.png)
-	如果定义了通配符是谁的父类，则是下界限定通配符；此类通配符可读可写，转成任意父类都不会出现ClassCastException错误。
-	个人猜想：难道是因为**通配符**和**上界限定通配符**的泛型 向下转型容易出现ClassCastException错误，而**下界限定通配符**向上转型不会出现ClassCastException错误，因此java规范限制前者编译出错，而后面编译通过？
# 9 泛型数组(GenericArrayType)
```
public interface GenericArrayType extends Type {
    //获得这个数组元素类型，即获得：A<T>（A<T>[]）或  T（T[]）
    Type getGenericComponentType();
}
```
-	GenericArrayType，泛型数组，描述的是ParameterizedType类型以及TypeVariable类型数组，即形如：Test<T\>[][]、T[]等，是GenericArrayType的子接口
```java
public class MainTest<T> {
    T[] param;
    public static void main(String[] args) throws Exception{
        Class clazz =  MainTest.class;
        Field field = clazz.getDeclaredField("param");
        GenericArrayType arrayType = (GenericArrayType)field.getGenericType();
        TypeVariable variable = (TypeVariable) arrayType.getGenericComponentType();
    }
}
```
![](https://img-blog.csdnimg.cn/img_convert/3277f7af8834f4cbf69f078fbb001e8b.png)

---
欢迎指正文中错误
---
**关注公众号，一起交流，微信搜一搜: 潜行前行**
---
- [泛型就这么简单](https://segmentfault.com/a/1190000014120746)
- [Java程序员必备基础：泛型解析](https://juejin.im/post/6844904050673057799)
- [Java 获取泛型的类型实例详解](jb51.net/article/114805.htm)
