## 以下内容摘自[Cyc2018大佬](http://www.cyc2018.xyz/)的笔记，对原文做了略微修改和添加了少量内容

## 一、数据类型

### 基本类型

- byte/8
- char/16
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/~

boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将

boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 支持 boolean 数组，但是是通过读写 byte

数组来实现的。

### 包装类型

#### 自动拆装箱

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。













 





```
Integer x = 2; // 装箱
int y = x; // 拆箱
```





#### 缓存池

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123) 每次都会新建一个对象；
- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。













 





```
        Integer x = new Integer(123);
        Integer y = new Integer(123);
        System.out.println(x == y); // false
        Integer z = Integer.valueOf(123); // false
        System.out.println(y == z);
        Integer k = Integer.valueOf(123);
        System.out.println(z == k); // true
```





valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。













 





```
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```





在 Java 8 中，Integer 缓存池的大小默认为 -128~127。













 





```
    static final int low = -128;
    static final int high;
    static final Integer cache[];
    static {
    // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
        // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
        // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;
        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);
        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }
```





编译器会在自动装箱过程调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象。













 





```
Integer m = 123;
Integer n = 123;
System.out.println(m == n); // true
```





#### 基本类型对应的缓冲池如下：

- boolean values true and false
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F

在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以直接使用缓冲池中的对象。

在 jdk 1.8 所有的数值类缓冲池中，Integer 的缓冲池 IntegerCache 很特殊，这个缓冲池的下界是 - 128，上界默认

是 127，但是这个上界是可调的，在启动 jvm 的时候，通过 -XX:AutoBoxCacheMax=\<size\> 来指定这个缓冲池的大

小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始

化的时候就会读取该系统属性来决定上界。

## 二、String

### 概览 String 被声明为 final，因此它不可被继承。

### 存储

在 Java 8 中，String 内部使用 char 数组存储数据。













 





```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```





在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 coder 来标识使用了哪种编码。













 





```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;
    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```





value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

### 不可变的好处

#### 1. 可以缓存 hash 值

因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

#### 2. String Pool 的需要

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

​                                ![img](https://github.com/luoChunhui-1024/JavaInterview/blob/master/docs/static/img/java/string.png)

#### 3. 安全性

String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

#### 4. 线程安全

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

### String, StringBuffer and StringBuilder

#### 1. 可变性

- String 不可变
- StringBuffer 和 StringBuilder 可变

#### 2. 线程安全

- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步

### String Pool

字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程中将字符串添加到 String Pool 中。

当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同字符串，而 s3 和 s4 是通过 s1.intern() 方法取得一个字符串引用。intern() 首先把 s1 引用的字符串放到 String Pool 中，然后返回这个字符串引用。因此 s3 和 s4 引用的是同一个字符串, s5 和 s3引用的也是同一个字符串













 





```
        String s1 = new String("aaa");
        String s2 = new String("aaa");
        System.out.println(s1 == s2); // false
        String s3 = s1.intern();
        String s4 = s2.intern();
        String s5 = s1.intern();
        System.out.println(s3 == s4); // true
        System.out.println(s3 == s5); // true
```





如果是采用 "bbb" 这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。













 





```
String s5 = "bbb";
String s6 = "bbb";
System.out.println(s5 == s6); // true
```





在 Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7，String Pool 被移到堆中。这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。

### new String("abc")

- 使用这种方式一共会创建两个字符串对象（前提是 String Pool 中还没有 "abc" 字符串对象）。
- "abc" 属于字符串字面量，因此编译时期会在 String Pool 中创建一个字符串对象，指向这个 "abc" 字符串字面量；

而使用 new 的方式会在堆中创建一个字符串对象。

创建一个测试类，其 main 方法中使用这种方式来创建字符串对象。













 





```
public class NewStringTest {
    public static void main(String[] args) {
        String s = new String("abc");
    }
}
```





使用 javap -verbose 进行反编译，得到以下内容：













 





```
// ...
Constant pool:
// ...
    #2 = Class #18 // java/lang/String
    #3 = String #19 // abc
// ...
    #18 = Utf8 java/lang/String
    #19 = Utf8 abc
// ...
    public static void main(java.lang.String[]);
        descriptor: ([Ljava/lang/String;)V
        flags: ACC_PUBLIC, ACC_STATIC
        Code:
            stack=3, locals=2, args_size=1
             0: new #2 // class java/lang/String
            3: dup
            4: ldc #3 // String abc
            6: invokespecial #4 // Method java/lang/String."<init>":
            (Ljava/lang/String;)V
            9: astore_1
// ...
```





在 Constant Pool 中，#19 存储这字符串字面量 "abc"，#3 是 String Pool 的字符串对象，它指向 #19 这个字符串字面量。在 main 方法中，0: 行使用 new #2 在堆中创建一个字符串对象，并且使用 ldc #3 将 String Pool 中的字符串对象作为 String 构造函数的参数。

以下是 String 构造函数的源码，可以看到，在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 数组内容，而是都会指向同一个 value 数组。













 





```
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```





虽然共用一个 value 数组 和一个hash值，但是orginal 和 新构建的数组并不是同一个对象，只是有部分成员函数有交叉而已













 





```
        String s6 = new String("bbb");
        String s7 = new String(s6);
        System.out.println(s6 == s7);   // false
```





## 三、运算

### 参数传递

Java 的参数是以值传递的形式传入方法中，而不是引用传递。（拷贝一份地址值，也就是拷贝一份引用而不是同一个引用）

以下代码中 Dog dog 的 dog 是一个指针，存储的是对象的地址。在将一个参数传入一个方法时，本质上是将**对象的****地址以值的方式传递到形参中**。因此在方法中使指针引用其它对象，那么这两个指针此时指向的是完全不同的对象，在一方改变其所指向对象的内容时对另一方没有影响。













 





```
    public class Dog {
        String name;
        Dog(String name) {
            this.name = name;
        }
        String getName() {
            return this.name;
        }
        void setName(String name) {
            this.name = name;
        }
        String getObjectAddress() {
            return super.toString();
        }
    }
    public class PassByValueExample {
        public static void main(String[] args) {
            Dog dog = new Dog("A");
            System.out.println(dog.getObjectAddress()); // Dog@4554617c
            func(dog);
            System.out.println(dog.getObjectAddress()); // Dog@4554617c
            System.out.println(dog.getName()); // A
        }
        private static void func(Dog dog) {
            System.out.println(dog.getObjectAddress()); // Dog@4554617c
            dog = new Dog("B");
            System.out.println(dog.getObjectAddress()); // Dog@74a14482
            System.out.println(dog.getName()); // B
        }
    }
```





如果在方法中改变对象的字段值会改变原对象该字段值，因为改变的是同一个地址指向的内容。













 





```
    class PassByValueExample {
        public static void main(String[] args) {
            Dog dog = new Dog("A");
            func(dog);
            System.out.println(dog.getName()); // B
        }
        private static void func(Dog dog) {
            dog.setName("B");
        }
    }
```





### float 与 double

Java 不能隐式执行向下转型，因为这会使得精度降低。

1.1 字面量属于 double 类型，不能直接将 1.1 直接赋值给 float 变量，因为这是向下转型。













 





```
// float f = 1.1;
```





1.1f 字面量才是 float 类型。













 





```
float f = 1.1f;
```





### 隐式类型转换

因为字面量 1 是 int 类型，它比 short 类型精度要高，因此不能隐式地将 int 类型下转型为 short 类型。













 





```
short s1 = 1;
// s1 = s1 + 1;     // 等式右边的 s1 会自动隐式转换成int类型
```





但是使用 += 或者 ++ 运算符可以执行隐式类型转换。













 





```
s1 += 1;
// s1++;
```





上面的语句相当于将 s1 + 1 的计算结果进行了向下转型：













 





```
s1 = (short) (s1 + 1);
```





### switch

从 Java 7 开始，可以在 switch 条件判断语句中使用 String 对象。













 





```
        String s = "a";
        switch (s) {
            case "a":
                System.out.println("aaa");
                break;
            case "b":
                System.out.println("bbb");
                break;
        }
```





switch 不支持 long，是因为 switch 的设计初衷是对那些只有少数的几个值进行等值判断，如果值过于复杂，那么还是用 if 比较合适。













 





```
long x = 111;
switch (x) { // Incompatible types. Found: 'long', required: 'char, byte, short, int,Character, Byte, Short, Integer, String, or an enum'
    case 111:
        System.out.println(111);
        break;
    case 222:
        System.out.println(222);
        break;
}
```





## 四、继承

### 访问权限

Java 中有三个访问权限修饰符：private、protected 以及 public，如果不加访问修饰符，表示包级可见。

可以对类或类中的成员（字段以及方法）加上访问修饰符。

- 类可见表示其它类可以用这个类创建实例对象。
- 成员可见表示其它类可以用这个类的实例对象访问到该成员；

protected 用于修饰成员，表示在继承体系中成员对于子类可见，但是这个访问修饰符对于类没有意义。

设计良好的模块会隐藏所有的实现细节，把它的 API 与它的实现清晰地隔离开来。模块之间只通过它们的 API 进行通信，一个模块不需要知道其他模块的内部工作情况，这个概念被称为信息隐藏或封装。因此访问权限应当尽可能地使每个类或者成员不被外界访问。

如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别。这是为了确保可以使用父类实例的地方都可以使用子类实例（**多态性**），也就是确保满足里氏替换原则。

字段决不能是公有的，因为这么做的话就失去了对这个字段修改行为的控制，客户端可以对其随意修改。例如下面的例子中，AccessExample 拥有 id 公有字段，如果在某个时刻，我们想要使用 int 存储 id 字段，那么就需要修改所有

的客户端代码。













 





```
public class AccessExample {
    public String id;
}
```





可以使用公有的 getter 和 setter 方法来替换使用字段公有，这样的话就可以控制对字段的修改行为。













 





```
public class AccessExample {
    private int id;
    public String getId() {
        return id + "";
    }
    public void setId(String id) {
        this.id = Integer.valueOf(id);
    }
}
```





但是也有例外，如果是包级私有的类或者私有的嵌套类，那么直接暴露成员不会有特别大的影响。













 





```
public class AccessWithInnerClassExample {
    private class InnerClass {
        int x;
    }
    private InnerClass innerClass;
    public AccessWithInnerClassExample() {
        innerClass = new InnerClass();
    }
    public int getValue() {
        return innerClass.x; // 直接访问
     }
}
```





### 抽象类与接口

#### 1. 抽象类

抽象类和抽象方法都使用 abstract 关键字进行声明。如果一个类中包含抽象方法，那么这个类必须声明为抽象类。抽象类和普通类最大的区别是，抽象类不能被实例化，需要继承抽象类才能实例化其子类。（**包含抽象方法的类的一定是抽象类，抽象类不一定包含抽象方法**）













 





```
public abstract class AbstractClassExample {
    protected int x;
    private int y;
    public abstract void func1();
    public void func2() {
        System.out.println("func2");
    }
}
```

















 





```
public class AbstractExtendClassExample extends AbstractClassExample {
    @Override
    public void func1() {
        System.out.println("func1");
    }
}
```

















 





```
// AbstractClassExample ac1 = new AbstractClassExample(); // 'AbstractClassExample' is
abstract; cannot be instantiated
AbstractClassExample ac2 = new AbstractExtendClassExample();
ac2.func1();
```





#### 2. 接口

接口是抽象类的延伸，在 Java 8 之前，它可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现。

从 Java 8 开始，接口也可以拥有默认的方法实现，**这是因为不支持默认方法的接口的维护成本太高了**。在 Java 8 之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类，如果有默认的方法，需要该方法的子类可以重写该方法，不需要的方法的子类则不重写即可，所有已经实现了该接口的类也不需要改动）。

接口的成员（字段 + 方法）默认都是 public 的，并且不允许定义为 private 或者 protected。

接口的字段默认都是 static 和 final 的。













 





```
public interface InterfaceExample {
    void func1();
    default void func2(){
        System.out.println("func2");
     }
    int x = 123;
    // int y; // Variable 'y' might not have been initialized
    public int z = 0; // Modifier 'public' is redundant for interface fields
    // private int k = 0; // Modifier 'private' not allowed here
    // protected int l = 0; // Modifier 'protected' not allowed here
    // private void fun3(); // Modifier 'private' not allowed here
}
```

















 





```
public class InterfaceImplementExample implements InterfaceExample {
    @Override
    public void func1() {
        System.out.println("func1");
    }
}
```

















 





```
// InterfaceExample ie1 = new InterfaceExample(); // 'InterfaceExample' is abstract; cannot
be instantiated
InterfaceExample ie2 = new InterfaceImplementExample();
ie2.func1();
System.out.println(InterfaceExample.x);
```





#### 3. 比较

- 从设计层面上看，抽象类提供了一种 IS-A 关系(父子关系），那么就必须满足里式替换原则，即子类对象必须能够替换掉所有父类对象，子类会继承父类所有的属性和方法。而接口更像是一种 LIKE-A 关系（师徒关系），它只是提供一种方法实现契约，需要实现什么样的功能就实现什么样的接口。
- 从使用上来看，一个类可以实现多个接口，但是不能继承多个抽象类。
- 接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制。
- 接口的成员只能是 public 的，而抽象类的成员可以有多种访问权限。
- 在 jdk8 以前，接口中的方法默认全是抽象方法，不能有方法体，而 抽象类中可以有不声明为抽象方法的方法，这些方法可以有方法体。但是jdk 8以后，接口也支持有方法体的默认方法了。
- 接口的体量相对都比较小，而类的体量相对就比较大，体量大的话会增大维护成本和降低使用效率

#### 4. 使用选择

##### 使用接口的场景：

- 需要让不相关的类都实现一个方法，增加某个功能时。例如不相关的类都可以实现 Compareable 接口中的 compareTo() 方法；
- 需要使用多重继承。

##### 使用抽象类的场景：

- 需要在几个相关的类中共享代码。
- 需要能控制继承来的成员的访问权限，而不是都为 public。
- 需要继承非静态和非常量字段。

在很多情况下，接口优先于抽象类。因为接口没有抽象类严格的类层次结构要求，可以灵活地为一个类添加行为。并且从 Java 8 开始，接口也可以有默认的方法实现，使得修改接口的成本也变的很低。

### super

- 访问父类的构造函数：可以使用 super() 函数访问父类的构造函数，从而委托父类完成一些初始化的工作。
- 访问父类的成员：如果子类重写了父类的某个方法，可以通过使用 super 关键字来引用父类的方法实现。













 





```
public class SuperExample {
    protected int x;
    protected int y;
    public SuperExample(int x, int y) {
        this.x = x;
        this.y = y;
    }
    public void func() {
        System.out.println("SuperExample.func()");
    }
}
```

















 





```
public class SuperExtendExample extends SuperExample {
private int z;
    public SuperExtendExample(int x, int y, int z) {
        super(x, y);
        this.z = z;
    }
    @Override
    public void func() {
        super.func();
        System.out.println("SuperExtendExample.func()");
    }
}
```

















 





```
SuperExample e = new SuperExtendExample(1, 2, 3);
e.func();
```

















 





```
SuperExample.func()
SuperExtendExample.func()
```





### 重写与重载

#### 1. 重写（Override）

存在于继承体系中，指子类实现了一个与父类在方法声明上完全相同的一个方法。

为了满足里式替换原则，重写有以下三个限制：

- 子类方法的访问权限必须大于等于父类方法；
- 子类方法的返回类型必须是父类方法返回类型或为其子类型。
- 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型。

使用 @Override 注解，可以让编译器帮忙检查是否满足上面的三个限制条件。

下面的示例中，SubClass 为 SuperClass 的子类，SubClass 重写了 SuperClass 的 func() 方法。其中：

- 子类方法访问权限为 public，大于父类的 protected。
- 子类的返回类型为 ArrayList，是父类返回类型 List 的子类。
- 子类抛出的异常类型为 Exception，是父类抛出异常 Throwable 的子类。
- 子类重写方法使用 @Override 注解，从而让编译器自动检查是否满足限制条件













 





```
class SuperClass {
    protected List<Integer> func() throws Throwable {
        return new ArrayList<>();
    }
}
class SubClass extends SuperClass {
    @Override
    public ArrayList<Integer> func() throws Exception {
        return new ArrayList<>();
    }
}
```





在调用一个方法时，先从本类中查找看是否有对应的方法，如果没有查找到再到父类中查看，看是否有继承来的方法。否则就要对参数进行转型，转成父类之后看是否有对应的方法。总的来说，方法调用的优先级为：

- this.func(this)
- super.func(this)
- this.func(super)
- super.func(super)













 





```
/*
A
|
B
|
C
|
D
*/
class A {
    public void show(A obj) {
        System.out.println("A.show(A)");
    }
    public void show(C obj) {
        System.out.println("A.show(C)");
    }
}
class B extends A {
    @Override
    public void show(A obj) {
        System.out.println("B.show(A)");
    }
}
class C extends B {
}
class D extends C {
}
```

















 





```
public static void main(String[] args) {
    A a = new A();
    B b = new B();
    C c = new C();
    D d = new D();
    // 在 A 中存在 show(A obj)，直接调用
    a.show(a); // A.show(A)
    // 在 A 中不存在 show(B obj)，将 B 转型成其父类 A
    a.show(b); // A.show(A)
    // 在 B 中存在从 A 继承来的 show(C obj)，直接调用
    b.show(c); // A.show(C)
    // 在 B 中不存在 show(D obj)，但是存在从 A 继承来的 show(C obj)，将 D 转型成其父类 C
    b.show(d); // A.show(C)
    // 引用的还是 B 对象，所以 ba 和 b 的调用结果一样
    A ba = new B();
    ba.show(c); // A.show(C)
    ba.show(d); // A.show(C)
}
```





#### 2. 重载（Overload）

存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是参数类型、个数、顺序至少有一个不同。应该注意的是，返回值不同，其它都相同不算是重载。

## 五、Object 通用方法

### 概览













 





```
public native int hashCode()
public boolean equals(Object obj)
protected native Object clone() throws CloneNotSupportedException
public String toString()
public final native Class<?> getClass()
protected void finalize() throws Throwable {}
public final native void notify()
public final native void notifyAll()
public final native void wait(long timeout) throws InterruptedException
public final void wait(long timeout, int nanos) throws InterruptedException
public final void wait() throws InterruptedException
```





### equals()

#### 1. 等价关系

##### Ⅰ 自反性













 





```
x.equals(x); // true
```





##### Ⅱ 对称性













 





```
x.equals(y) == y.equals(x); // true
```





##### Ⅲ 传递性













 





```
if (x.equals(y) && y.equals(z))
    x.equals(z); // true;
```





##### Ⅳ 一致性

多次调用 equals() 方法结果不变













 





```
x.equals(y) == x.equals(y); // true
```





##### Ⅴ 与 null 的比较

对任何不是 null 的对象 x 调用 x.equals(null) 结果都为 false













 





```
x.equals(null); // false;
```





#### 2. 等价与相等

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
- 对于引用类型，== 判断两个变量是否引用同一个对象，而 equals() 判断引用的对象是否等价













 





```
Integer x = new Integer(1);
Integer y = new Integer(1);
System.out.println(x.equals(y)); // true
System.out.println(x == y); // false
```





#### 3. 代码实现

实现步骤

- 检查是否为同一个对象的引用，如果是直接返回 true；
- 检查是否是同一个类型，如果不是，直接返回 false；
- 将 Object 对象进行转型；
- 判断每个关键域是否相等













 





```
public class EqualExample {
    private int x;
    private int y;
    private int z;
    public EqualExample(int x, int y, int z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        EqualExample that = (EqualExample) o;
        if (x != that.x) return false;
        if (y != that.y) return false;
        return z == that.z;
    }
}
```





### hashCode()

hashCode() 返回散列值，而 equals() 是用来判断两个对象是否等价。等价的两个对象散列值一定相同，但是散列值

相同的两个对象不一定等价。

在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象散列值也相等。

下面的代码中，新建了两个等价的对象，并将它们添加到 HashSet 中。我们希望将这两个对象当成一样的，只在集

合中添加一个对象，但是因为 EqualExample 没有实现 hashCode() 方法，因此这两个对象的散列值是不同的，最终

导致集合添加了两个等价的对象













 





```
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size()); // 2
```





理想的散列函数应当具有均匀性，即不相等的对象应当均匀分布到所有可能的散列值上。这就要求了散列函数要把所有域的值都考虑进来。可以将每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。R 一般取 31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位。一个数与 31 相乘可以转换成移位和减法： 31*x == (x<<5)-x ，编译器会自动进行这个优化。













 





```
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + x;
    result = 31 * result + y;
    result = 31 * result + z;
    return result;
}
```





### toString()

默认返回 ToStringExample@4554617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。













 





```
public class ToStringExample {
    private int number;
    public ToStringExample(int number) {
        this.number = number;
    }

    public static void main(String[] args) {
        ToStringExample example = new ToStringExample(123);
        System.out.println(example.toString());
        System.out.println(example.hashCode());
    }
}
```

















 





```
cn.ncu.ToStringExample@1540e19d
```





### clone()（浅拷贝，拷贝前后指向的是同一个对象）

#### 1. cloneable

##### protected

clone() 是 Object 的 protected 方法，它不是 public，一个类不显式去重写 clone()，其它类就不能直接去调用该类实例的 clone() 方法。













 





```
public class CloneExample {
    private int a;
    private int b;
}
```

















 





```
CloneExample e1 = new CloneExample();
// CloneExample e2 = e1.clone(); // 'clone()' has protected access in 'java.lang.Object'
```





##### cloneable

重写 clone() 得到以下实现：













 





```
public class CloneExample {
    private int a;
    private int b;
    @Override
    public CloneExample clone() throws CloneNotSupportedException {
        return (CloneExample)super.clone();
    }
}
```

















 





```
CloneExample e1 = new CloneExample();
try {
    CloneExample e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
```

















 





```
java.lang.CloneNotSupportedException: CloneExample
```





以上抛出了 CloneNotSupportedException，这是因为 CloneExample 没有实现 Cloneable 接口。

应该注意的是，clone() 方法并不是 Cloneable 接口的方法，而是 Object 的一个 protected 方法。Cloneable 接口只是规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException。













 





```
public class CloneExample implements Cloneable {
    private int a;
    private int b;
    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```





#### 2. 浅拷贝

拷贝对象和原始对象的引用类型引用同一个对象。













 





```
public class ShallowCloneExample implements Cloneable {
    private int[] arr;
    public ShallowCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }
    public void set(int index, int value) {
        arr[index] = value;
    }
    public int get(int index) {
        return arr[index];
    }
    @Override
    protected ShallowCloneExample clone() throws CloneNotSupportedException {
        return (ShallowCloneExample) super.clone();
    }
}
```

















 





```
 public static void main(String[] args) {
     ShallowCloneExample e1 = new ShallowCloneExample();
     ShallowCloneExample e2 = null;
     try {
         e2 = e1.clone();
     } catch (CloneNotSupportedException e) {
         e.printStackTrace();
     }
     e1.set(2, 222);
     System.out.println(e2.get(2)); // 222
 }
```





#### 3. 深拷贝

拷贝对象和原始对象的引用类型引用不同对象。













 





```
public class DeepCloneExample implements Cloneable {
    private int[] arr;
    public DeepCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }
    public void set(int index, int value) {
        arr[index] = value;
     }
    public int get(int index) {
        return arr[index];
    }
    @Override
    protected DeepCloneExample clone() throws CloneNotSupportedException {
        DeepCloneExample result = (DeepCloneExample) super.clone();
        result.arr = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            result.arr[i] = arr[i];
        }
        return result;
    }
}
```

















 





```
DeepCloneExample e1 = new DeepCloneExample();
DeepCloneExample e2 = null;
try {
    e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```





#### 4. clone() 的替代方案(拷贝构造函数或者拷贝工厂)

使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，

最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。













 





```
public class CloneConstructorExample {
private int[] arr;
public CloneConstructorExample() {
    arr = new int[10];
    for (int i = 0; i < arr.length; i++) {
        arr[i] = i;
    }
    }
    public CloneConstructorExample(CloneConstructorExample original) {
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = original.arr[i];
        }
    }
    public void set(int index, int value) {
        arr[index] = value;
     }
    public int get(int index) {
        return arr[index];
    }
}
```

















 





```
public static void main(String[] args) {
    CloneConstructorExample e1 = new CloneConstructorExample();
    CloneConstructorExample e2 = new CloneConstructorExample(e1);
    e1.set(2, 222);
    System.out.println(e2.get(2)); // 2
}
```





## 六、关键字

### final

#### 1. 数据

声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。













 





```
final int x = 1;
// x = 2; // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```





#### 2. 方法

声明方法不能被子类重写。

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

#### 3. 类

声明类不允许被继承。

### static

#### 1. 静态变量

- 静态变量：又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它。静态变量在内存中只存在一份。
- 实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死。













 





```
public class A {
    private int x; // 实例变量
    private static int y; // 静态变量
    public static void main(String[] args) {
        // int x = A.x; // Non-static field 'x' cannot be referenced from a static context
        A a = new A();
        int x = a.x;
        int y = A.y;
    }
}
```





#### 2. 静态方法

静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。













 





```
public abstract class A {
    public static void func1(){
    }
    // public abstract static void func2(); // Illegal combination of modifiers: 'abstract' and 'static'
}
```





只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字。













 





```
public class A {
    private static int x;
    private int y;
    public static void func1(){
        int a = x;
        // int b = y; // Non-static field 'y' cannot be referenced from a static context
        // int b = this.y; // 'A.this' cannot be referenced from a static context
    }
}
```





#### 3. 静态语句块

静态语句块在类初始化时运行一次。













 





```
public class A {
    static {
    System.out.println("123");
    }
    public static void main(String[] args) {
        A a1 = new A();
        A a2 = new A();
    }
}
```

















 





```
123
```





#### 4. 静态内部类

非静态内部类依赖于外部类的实例，而静态内部类不需要。













 





```
public class OuterClass {
    class InnerClass {
    }
    static class StaticInnerClass {
    }
    public static void main(String[] args) {
        // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be
        referenced from a static context
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        StaticInnerClass staticInnerClass = new StaticInnerClass();
    }
}
```





静态内部类不能访问外部类的非静态的变量和方法

#### 5. 静态导包

在使用静态变量和方法时不用再指明 ClassName，从而简化代码，但可读性大大降低。













 





```
import static com.xxx.ClassName.*
```





#### 6. 初始化顺序

静态变量和静态语句块优先于实例变量和普通语句块，普通语句块优先于构造方法，静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序













 





```
public static String staticField = "静态变量";
```

















 





```
static {
    System.out.println("静态语句块");
}
```

















 





```
public String field = "实例变量";
```

















 





```
{
    System.out.println("普通语句块");
}
```





最后才是构造函数的初始化。













 





```
public InitialOrderTest() {
    System.out.println("构造函数");
}
```





存在继承的情况下，初始化顺序为：

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）

## 七、反射

#### 概述

​    JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；利用class 字节码对象，可以使得对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

#### 获取 Class 字节码对象的方法

1. Object类的getClass()方法,判断两个对象是否是同一个字节码文件

2. 静态属性class,锁对象

3. Class类中静态方法forName(),读取配置文件， 类名必须是完整类名



#### 方法的使用：

Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：

- Field ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
- Method ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
- Constructor ：可以用 Constructor 创建新的对象。

通过调用相应的get()方法，我们可以获取到相应的field，Method 和 Constructor ，如果是这些字段或方法不是公有的，那么还需要用setAccessible() 来修改访问权限，最后用某个实例对象作为参数即可调用我们想要执行的方法或修改某个字段。

#### 反射机制优缺点

优点：可扩展性高，在运行期类型的判断，动态加载类，提高代码灵活度。

缺点：

1. 因为反射涉及了动态类型的解析，所以 JVM 无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。

2. 安全问题，让我们可以动态操作改变类的属性同时也增加了类的安全隐患。比如利用反射给类型为 Integer 的List 添加Sting 元素 

#### 使用场景：

1. 动态代理
2. 我们在使用JDBC连接数据库时使用Class.forName()通过反射加载数据库的驱动程序；
3. Spring框架也用到很多反射机制，最经典的就是xml的配置模式。Spring 通过 XML 配置模式装载 Bean 的过程：1) 将程序内所有 XML 或 Properties 配置文件加载入内存中; 2)Java类里面解析xml或properties里面的内容，得到对应实体类的字节码字符串以及相关的属性信息; 3)使用反射机制，根据这个字符串获得某个类的Class实例; 4)动态配置实例的属性。

### 静态编译和动态编译

- **静态编译：**在编译时确定类型，绑定对象
- **动态编译：**运行时确定类型，绑定对象

#### 使用举例

##### Class.forName()读取配置文件举例

根据配置文件的 string 获取对应类的class字节码对象，利用反射创建这个class 对象的实例，将这个实例，作为参数传递到 Juicer 的 run()方法中。













 





```
public class Demo1_Reflect {
    public static void main(String[] args) throws IOException, ClassNotFoundException, IllegalAccessException, InstantiationException {
        BufferedReader br = new BufferedReader(new FileReader("config.properties"));

        // 根据配置文件的 string 获取对应类的class对象
        Class<?> clazz = Class.forName(br.readLine());
        // 利用反射创建这个class 对象的实例
        Fruit f = (Fruit) clazz.newInstance();
        // 作为参数传递到 Juicer 的 run()方法中
        Juicer juicer = new Juicer();
        juicer.run(f);
    }

}

interface Fruit{
    public void squeeze();      // 根据不同的水果榨不同的汁
}

class Apple implements Fruit{
    @Override
    public void squeeze() {
        System.out.println("榨出一杯苹果汁");
    }
}

class Orange implements Fruit{
    @Override
    public void squeeze() {
        System.out.println("榨出一杯橙汁");
    }
}

class Juicer{
    public void run(Fruit f){
        f.squeeze();
    }
}
```





##### 通过反射获取带参构造方法并使用

如果有参构造方法是公有的，直接使用 getConstructor(Class\<?\>... parameterTypes) 即可获取构造器；如果构造方法不是 public的，需要使用 getDeclaredConstructor()方法强制获取构造器，并且需要设置访问属性，消除私有权限













 





```
        Class clazz = Class.forName("cn.ncu.reflect.Person");
        // 直接使用newInstance() 方法创建的是无参构造实例
        Person p = (Person) clazz.newInstance();
        System.out.println(p);  // Person{name='null', age=0}

        // 如果有参构造方法是公有的，直接使用 getConstructor(Class<?>... parameterTypes) 即可获取构造器
        Constructor c = clazz.getConstructor(String.class, int.class);
        Person p2 = (Person)c.newInstance("张三", 23);
        System.out.println(p2); // Person{name='张三', age=23}

       /* // 如果构造方法不是 public的，并且需要设置访问属性，消除私有权限
        Constructor c = clazz.getDeclaredConstructor(String.class, int.class);
        c.setAccessible(true);
        Person p2 = (Person)c.newInstance("张三", 23);
        System.out.println(p2); // Person{name='张三', age=23}*/
```





##### 通过反射获取成员变量并使用













 





```
        Field f = clazz.getField("name");
        f.set(p, "王五");
        System.out.println(p);  // Person{name='王五', age=0}
```





```
如果name 属性时非public 的，需要强行获取 field 并修改访问权限为public
```













 





```
        Field f = clazz.getDeclaredField("name");
        f.setAccessible(true);
        f.set(p, "王五");
        System.out.println(p);  // Person{name='王五', age=0}
```





##### 通过反射获取方法并使用

如果该方法的访问权限是 public , 那么直接调用getMethod();方法然后调用 invoke()方法执行即可。













 





```
        Method m = clazz.getMethod("eat", int.class);
        m.invoke(p, 10);  // 张三eat 10 apples
```





否则需要调用 getDeclaredMethod() 强行获取方法并修改访问权限

## 八、异常

Throwable 可以用来表示任何可以作为异常抛出的类，分为两种： Error 和 Exception。其中 Error 用来表示 JVM无法处理的错误，Exception 分为两种：

- 受检异常 ：需要用 try...catch... 语句捕获并进行处理，并且可以从异常中恢复；
- 非受检异常 ：是程序运行时错误，例如除 0 会引发 Arithmetic Exception，此时程序崩溃并且无法恢复

Error我觉得是代码逻辑错误或者是外部资源不足，Exception一般是代码的非逻辑错误无，比如语法错误，或者是使用错误，比如除0异常或者下标越界异常



​                ![img](https://github.com/luoChunhui-1024/JavaInterview/blob/master/docs/static/img/java/exception.png)

![ddd](https://dss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=4207578672,1058536368&fm=15&gp=0.jpg)

## 九、泛型

（后期有一篇文章专门讲泛型）













 





```
public class Box<T> {
    // T stands for "Type"
    private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```





### 1.  把泛型定义在类上，定义格式为：

​      public class 类名\泛型类型1,…\>

​    静态方法必须定义自己的泛型

### 2. 把泛型定义在方法上， 定义格式    

​      public \<泛型类型\> 返回类型 方法名(泛型类型 变量名)

### 3. 泛型定义在接口上，定义格式为：

public interface 接口名\<泛型类型\>

### 4. 泛型高级之通配符* A:泛型通配符\<?\>

​     \* 任意类型，如果没有明确，那么就是Object以及任意的Java类了
​            ArrayList\<?\> list = new ArrayList\<String\>();        //当右边的泛型不确定时，左边的泛型可以用通配符？表示
\* B:? extends E
​     \* 向下限定，E及其子类
​        \* boolean addAll(Collection<? extends E> c) 
\* C:? super E
​     \* 向上限定，E及其父类

#### 通过反射越过泛型检查（泛型擦除）

Java 中的泛型是伪泛型，为什么说它是伪泛型呢？因为它采用的是类型擦除的方式来实现的。在生成的Java字节码中是不包含泛型中的类型信息的。Java 中的泛型仅用于编译器做类型检查,就是保证我们代码写的不会出现类型错误。 List\<String\>,List\<Integer\>编译之后可以理解成统统变成List\<Object\>了，而之所以在使用内部元素的时候察觉不到泛型被擦除了，觉得这个ArrayList 正是按我们设定的泛型来使用元素，是因为编译的时候凡是使用到了泛型的地方，编译器自动加上了类型强转，我们也察觉不到泛型被擦除了。又因为加上了强转，所以擦除掉泛型也不会出问题。

证明类型擦除的方式有三种，



一种是定义两个数组列表，一个是参数为 String 的 ArrayList, l另一个是参数为 Integer 的ArrayList , 对这两个对象分别调用getClass()方法然后做等等于的运算，会发现结果是返回 true













 





```
        ArrayList<String> arrayList1=new ArrayList<String>();
        arrayList1.add("abc");
        ArrayList<Integer> arrayList2=new ArrayList<Integer>();
        arrayList2.add(123);
        System.out.println(arrayList1.getClass()==arrayList2.getClass());
```





第二种是用反射获取一个 泛型类型为 Integer 的 ArrayList 的 add 方法，给这add 方法传参 Object.class， 然后调用 invoke() 存入一个字符串，会发现存入成功，这也说明泛型在运行时不存在的。

第三种是通过反编译代码来证明

ArrayList\<Integer\>的一个对象，在这个集合中添加一个字符串数据













 





```
        ArrayList<Integer> list = new ArrayList<>();
        list.add(111);
        list.add(22);

        // list.add("你好");
        Class clazz = Class.forName("java.util.ArrayList");
        Method m = clazz.getMethod("add", Object.class);
        m.invoke(list, "你好");
        System.out.println(list);   // [111, 22, 你好]
```





##### 参考：[10 道 Java 泛型面试题](https://cloud.tencent.com/developer/article/1033693)

## 十、注解

Java 注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。

在web框架中可以用注解来代替配置文件，可能更方便。

## 十一、特性

Java 各版本的新特性

## Java 与 C++ 的区别

- Java 是纯粹的面向对象语言，所有的对象都继承自 java.lang.Object，C++ 为了兼容 C 即支持面向对象也支持面向过程。
- Java 通过虚拟机从而实现跨平台特性，但是 C++ 依赖于特定的平台。
- Java 的 goto 是保留字，但是不可用，C++ 可以使用 goto。
- Java 支持自动垃圾回收，而 C++ 需要手动回收。
- Java 没有指针，它的引用可以理解为安全指针，而 C++ 具有和 C 一样的指针。
- Java 不支持多重继承，只能通过实现多个接口来达到相同目的，而 C++ 支持多重继承。
- Java 不支持操作符重载，虽然可以对两个 String 对象执行加法运算，但是这是语言内置支持的操作，不属于操作符重载，而 C++ 可以。
- Java 不支持条件编译，C++ 通过 #ifdef #ifndef 等预处理命令从而实现条件编译。