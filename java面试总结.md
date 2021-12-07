# java面试总结

# 一. java基础

## 1. 类的创建

1. ### 创建类的方法

:one: 使用new关键字创建；

:two: 反射：Class的newInstance()，Constructor的newInstance(Xxx)

:three: 拷贝：使用Clone()

:four: 反序列化的方法；

2. 几种方式的创建效率如何？

通过new创建对象的效率比较高。通过反射时，先找查找类资源，使用类加载器创建，过程比较繁琐，所以效率较低

3. 构造方法new，序列化对象，反射，克隆分别创建一个对象的方法，，只有new和反射用到了构造方法.

## 2. 对象的深拷贝和浅拷贝

### 2.1 深拷贝和浅拷贝的区别

​	Java的拷贝技术在面试中的频率相比于序列化会少点，拷贝要实现的内容是将一个对象赋值一份出来。而对于一个基础类型，深拷贝和浅拷贝是一样的，都会复制一份新的，而对于一个对象类型，如果是浅拷贝的话，那其底层访问的是同一份对象，类似于对该对象起了个别名。而深拷贝的话才是我们真正说的复制，它会重新创建一个新的对象出来，和之前的对象的值一模一样，但是底层的地址是完全不同的。

### 2.2怎么实现一个java 的深拷贝：

​      在java中，一个最基本的父类Object中有clone方法，而这个方法是实现浅拷贝的，这是一个native方法。如果我们要想实现深拷贝的话，就需要实现Cloneable接口，并且实现clone方法，如果对象中包含其他对象，其他对象也要实现。下面看下通过clone来实现一个类的拷贝。

方式一：重写Object类的clone()方法

```java
/**
 * 用户
 */
public class User implements Cloneable {
 
    private String name;
    private Address address;
 
    // constructors, getters and setters
 
    @Override
    public User clone() throws CloneNotSupportedException {
        User user = (User) super.clone();
        user.setAddress(this.address.clone());
        return user;
    }
 
}

/**
 * 地址
 */
public class Address implements Cloneable {
 
    private String city;
    private String country;
 
    // constructors, getters and setters
 
    @Override
    public Address clone() throws CloneNotSupportedException {
        return (Address) super.clone();
    }
 
}

// 需要注意的是，super.clone()其实是浅拷贝，所以在重写User类的clone()方法时，address对象需要调用address.clone()重新赋值。
```

方法二：构造函数

```java
@Test
public void constructorCopy() {

  Address address = new Address( "杭州" , "中国");
  User user = new User("大山", address);

  // 调用构造函数时进行深拷贝
  User copyUser = new User(user.getName(), new Address(address.getCity(), address.getCountry()));

  // 修改源对象的值
  user.getAddress().setCity("深圳");

  // 检查两个对象的值不同
  assertNotSame(user.getAddress().getCity(), copyUser.getAddress().getCity());

}
```

方式三：第三方 JAR 包集成的序列化方式进行拷贝对象。例如Apache Commons Lang包下的clone方法(因为使用到了序列化，所以需要拷贝的类要实现Serializable接口)

```java
/**
 * 地址
 */
public class Address implements Serializable {
 
    private String city;
    private String country;
 
    // constructors, getters and setters
 
}

/**
 * 用户
 */
public class User implements Serializable {
 
    private String name;
    private Address address;
 
    // constructors, getters and setters
 
}

@Test
/**
使用Apache Commons Lang进行深拷贝
*/
public void serializableCopy() {
 
    Address address = new Address("杭州", "中国");
    User user = new User("大山", address);
 
    // 使用Apache Commons Lang序列化进行深拷贝
    User copyUser = (User) SerializationUtils.clone(user);
 
    // 修改源对象的值
    user.getAddress().setCity("深圳");
 
    // 检查两个对象的值不同
    assertNotSame(user.getAddress().getCity(), copyUser.getAddress().getCity());
 
}
```



![image-20210830144723608](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20210830144723608.png)


## 3. 序列化 

1. #### Java序列化的作用是什么？

​	我们都知道，java是以对象为基本核心理念的，在java中，万物皆为对象。在某些场景下，我们需要将对象传递给其他机器或者写到磁盘保存。这个时候就需要**将对象转换成二进制的方式**，它是一个对象转换为二进制流的过程，而序列化过程就是完成这项工作的，与之对应的是将二进制转换为对象的过程，我们称之为反序列化过程。

2. #### 序列化的实现步骤？

​	Java实现序列化的过程由以下几个流程：

1. 在类对象中实现Serializable接口

2. 在类中定义序列化id

3. #### 底层的原理是什么

​		java底层是通过java.io包中的ObjectOutputStream实现将对象转换为二进制流。ObjectOutputStream中的核心方法在于这个writeObject，其实现也是按深度，按照类型写入流中，数组和string，null的写入等等。而writeObject方法核心是writeObject0.与此相反的是反序列的方法，也就是ObjectInputStream.

java.io.serializable接口本身没有方法和字段，仅用于标识可序列化的语义。可序列化类的所有子类型本身都是可序列化的，否则会报 NotSerializableException

- **Java序列化算法**

1. **所有保存到磁盘的对象都有一个序列化编码号**
2. **当程序试图序列化一个对象时，会先检查此对象是否已经序列化过，只有此对象从未（在此虚拟机）被序列化过，才会将此对象序列化为字节序列输出。**
3. **如果此对象已经序列化过，则直接输出编号即可。**

4. #### Java 序列化类中有一个final的Long型id,这个序列化id的作用是什么？如果没有的话，是否可以序列化成功？

​		在序列化中，序列化id其实是一个版本管理的问题，其实在很多时候，我们都会使用到，因为一个对象在使用的过程中，难免出现修改字段，这时候需要将序列化id升级一个版本区分之前的版本。如果没有写序列化id能否序列化成功，当然是可以的，如果我们没有写序列化id 的时候，jvm会默认根据相应的规则生成一个，但这个时候可能会有一个问题是，但我们把jdk升级可能会导致序列化id和之前的不同，这就会导致代码出现无法序列化的问题。所以为了安全起见，最后在代码生成的时候加上一个序列化id。

5. #### 如果不想序列化某个字段，应该怎么办？

方法一：		

如果不想对某个字段进行序列化，可以使用transient。当然了static也不能被序列化，static是类的概念，而transient是临时的概念。

**使用transient修饰的属性，java序列化时，会忽略掉此字段，所以反序列化出的对象，被transient修饰的属性是默认值。对于引用类型，值是null；基本类型，值是0；boolean类型，值是false。**

方法二：	

使用transient虽然简单，但将此属性完全隔离在了序列化之外。java提供了**可选的自定义序列化。**可以进行控制序列化的方式，或者对序列化数据进行编码加密等。通过重写writeObject与readObject方法，可以自己选择哪些属性需要序列化， 哪些属性不需要。如果writeObject使用某种规则序列化，则相应的readObject需要相反的规则反序列化，以便能正确反序列化出对象。这里展示对名字进行**反转加密**。

```java
public class Person implements Serializable {
  private String name;
  private int age;
  //省略构造方法，get及set方法

    // 这里的方法必须为 private 和 void ，因为Java在调用ObjectOutPutStream类时检查其是否有私有的，无返回值的writeObject方法，如果有，其会委托该方法进行对象序列化。
  private void writeObject(ObjectOutputStream out) throws IOException {
    //将名字反转写入二进制流
    out.writeObject(new StringBuffer(this.name).reverse());
    out.writeInt(age);
  }

  private void readObject(ObjectInputStream ins) throws IOException,ClassNotFoundException{
    //将读出的字符串反转恢复回来
    this.name = ((StringBuffer)ins.readObject()).reverse().toString();
    this.age = ins.readInt();
  }
}
```
JDK源码ObjectStreamClass中的方法
```java
writeObjectMethod = getPrivateMethod(cl, "writeObject",
                                     new Class<?>[] { ObjectOutputStream.class },
                                     Void.TYPE);
readObjectMethod = getPrivateMethod(cl, "readObject",
                                    new Class<?>[] { ObjectInputStream.class },
                                    Void.TYPE);
readObjectNoDataMethod = getPrivateMethod(cl, "readObjectNoData", null, Void.TYPE);
hasWriteObjectData = (writeObjectMethod != null);
```



6. Externalizable接口


Externalizable继承了Serializable，该接口中定义了两个抽象方法：writeExternal()与readExternal()。当使用Externalizable接口来进行序列化与反序列化的时候需要开发人员重写writeExternal()与readExternal()方法。由于上面的代码中，并没有在这两个方法中定义序列化实现细节，所以输出的内容为空。还有一点值得注意：在使用Externalizable进行序列化的时候，**在读取对象时，会调用被序列化类的无参构造器去创建一个新的对象，然后再将被保存对象的字段的值分别填充到新对象中**。所以，**实现Externalizable接口的类必须要提供一个public的无参的构造器。**


6. #### 你了解的常见的序列化框架，其优缺点？

   |                        |                 **优点**                 |                           **缺点**                           |
   | ---------------------- | :--------------------------------------: | :----------------------------------------------------------: |
   | **Kryo**               |          速度快，序列化后体积小          |                       跨语言支持较复杂                       |
   | **Hessian**            |              默认支持跨语言              |                             较慢                             |
   | **Protostuff**         |           速度快，基于protobuf           |                          需静态编译                          |
   | **Protostuff-Runtime** | 无需静态编译，但序列化前需预先传入schema | 不支持无默认构造函数的类，反序列化时需用户自己初始化序列化后的对象，其只负责将该对象进行赋值 |
   | **Java**               |         使用方便，可序列化所有类         |                        速度慢，占空间                        |

## 4. 反射

![image-20210830152734616](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20210830152734616.png)

1. #### 介绍一下你了解的反射，反射的原理是什么？

反射主要是指程序可以访问、检测和修改它本身状态或行为的一种能力

Java反射：在Java运行时环境中，对于任意一个类，能否知道这个类有哪些属性和方法？对于任意一个对象，能否调用它的任意一个方法

Java反射机制主要提供了以下功能：

- 在运行时判断任意一个对象所属的类。
- 在运行时构造任意一个类的对象。
- 在运行时判断任意一个类所具有的成员变量和方法。
- 在运行时调用任意一个对象的方法。

Java Reflection API简介

在JDK中，主要由以下类来实现Java反射机制，这些类（除了第一个）都位于java.lang.reflect包中

- Class类：代表一个类，位于java.lang包下。
- Field类：代表类的成员变量（成员变量也称为类的属性）。
- Method类：代表类的方法。
- Constructor类：代表类的构造方法。
- Array类：提供了动态创建数组，以及访问数组的元素的静态方法。



2. #### 怎么实现一个反射？反射的几种调用方法？区别是什么？

1. 获取想要操作的类的Class 对象，他是反射的核心，通过Class 对象我们可以任意调用类的方
    法。
2. 调用Class 类中的方法，既就是反射的使用阶段。
3. 使用反射API 来操作这些信息。

3. #### 获取Class 对象的 3 种方法

1. Class.forName(“类的全路径”)

2. Object.class

3. Instance.getClass()

对于三者的区别是：

Class.forName会加载类，默认会对静态代码块初始化，当然也可以通过参数设置不初始化静态代码块。

Object.Class只返回类的信息，不会做任何初始化的操作。

Instance.getClass() 是返回对象对应的class对象。

可以构建一个含有静态代码块，动态代码块，和构造函数的进行试验。值得注意的是，无论是谁调用，类一旦被加载过，就会重缓存中获取，并不会每次都加载。如果我们需要将获取到的class进行实例的获取，那么会调用到另一个方法Class.newInstance。而newInstance会将类初始化完成，并调用构造函数初始化一个对象出来。这个在后续的jvm 面试题中会重点描述。

4. #### 创建对象的两种方法

Class 对象的 newInstance()
1. 使用Class 对象的newInstance()方法来创建该Class 对象对应类的实例，但是这种方法要求该Class 对象对应的**类有默认的空构造器**。

2. 调用 Constructor 对象的 newInstance()；

  先使用Class 对象获取指定的Constructor 对象，再调用Constructor 对象的newInstance()方法来创建 Class 对象对应类的实例,通过这种方法可以选定构造方法创建实例。

## 5. 字符串

主要涉及的知识点：equals和 == 的区别，jvm的内存模型，intern方法的使用，字符串String常量类。

#### String的基本特性

:one: String：字符串，使用一对 "" 引用起来表示，被声明为 final 的，即不可被继承。

:two: 实现了Serializable接口，表示支持序列化；

:three: 实现了Comparable接口，表示String是可以比较大小的；

:four: 在 jdk8 以前内部定义了 final char[] value 用于存储字符串数据， jdk9 时改为byte[] 数组

#### 补：jdk9的新特性

java9之前String底层数组的实现采用的是char数组，在这种方式下，每一个字符都将占用两个字节的空间。

而在java9之后，String底层采用byte数组和编码标识来识别，主要原因就是提高内存的占用率。

coder的用法：

当检测到变量按照latin1或ISO进行标识时，会为其分配一个字节大小的空间；
当检测到变量按照utf-16进行标识时，会为其分配而两个字节大小的空间。
同样地，与String相关的StringBuilder,StringBuffer底层都采用了byte[]来实现。

```java
// JDK9 之前
private final char value[];
// JDK9 之后
private final byte[] value;
private final byte coder;
```

#### 1. equals和 == 的区别

在java语言中，== 和equals **都是有判断两者相等**的意思，不过，使用场景上有些不同。其主要区别如下：

**== 是运算符，equals是Object的基本方法**，也就是说每个java类都有这个方法。在java语言中，**运算符是不允许重载的，而方法是可以重写的**，所以我们可以定义自己的equals的方法，这就意味着equals方法在每个类中有可能判断相等的方式不一样。

== 运算符判断相等的方法是：**如果是基本类型的话，则判断的是基本类型的值是否相等；如果是引用类型的话，则判断引用的地址是否相等**。而equals方法，我们知道如果不重写的话，那么其底层的实现其实就是 == （参考object的）。但是如果重写之后的话，我们就要根据类的重写去看下具体的实现了，在这里，我们看下Integer和String的判断思路。

```java
 //object 类 

public boolean equals(Object obj) { 

     return (this == obj); 

}
//string
public boolean equals(Object anObject) {
    	// 先判断对象的地址值是否相等
        if (this == anObject) {
            return true;
        }
    	// 依次对比每个字符
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
}

//Integer
public boolean equals(Object obj) {
    	//判断是否是整型
        if (obj instanceof Integer) {
            //使用==判断
            return value == ((Integer)obj).intValue();
        }
        return false;
}
```

#### 2. 字符串常量池的内存模型

因为在开发中会有大量的字符串被创建，但是永久代的回收效率很低，只有在Full GC时才被触发，所以在jdk7 的时候，将字符串常量池从永久代中转移到了堆空间内，可以保证及时回收，在jdk8 以后，将永久代改成了元空间，并且实现方式改为了在本地内存，但是字符串常量池依旧在堆空间中。

字符串常量池中不会存在相同的常量；

常量与常量拼接的结果在常量池，原理是编译器优化；只要其中有一个是变量，结果就在堆中，变量拼接的原理是StringBuilder。

```java
//字面量定义的方式，"abc"存储在字符串常量池中
String s1 = "abc";
// 都采用常量进行拼接时，jvm会对代码进行编译器优化，生成的字节码已经是“abc”
String s2 = "a" + "b" + "c"; 

System.out.println(s1 == s2);


System.out.println(s1);//
System.out.println(s2);//abc
```

#### 3. String.intern()方法

如果不是用双引号声明的对象，可以使用String提供的intern方法，会从字符串常量池中查询当前的字符串是否存在，若果存在就将当前的字符串常量池中查询当前的字符串是否存在，若不存在就将当前的字符串的放入常量池，并返回这个字符串的地址值；若存在，则直接返回字符串常量池中的地址，此方法可以保证字符串在内存中只有一份，可以节约内存。

![image-20210830164000123](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20210830164000123.png)

#### 有关String的代码练习

```java
// 例子1
	String s = new String("1");
    s.intern();//调用此方法之前，字符串常量池中已经存在了"1"
    String s2 = "1";
    System.out.println(s == s2);//jdk6：false   jdk7/8：false

// 例子1
String s = new String("a") + new String("b");
String s2 = s.intern();
System.out.println( s2 == "ab");	// JDK6,7,8 true
System.out.println( s == "ab");		// JDK6 false, JDK 7,8 true;
// 原因是，在jdk6中如果字符串常量池中没有，会首先复制一份对象放入池中(深拷贝)，然后返回池中对象的对象地址 所以 s2 和 s 是两个对象。
// 在jdk7以后，会把对象的引用地址复制一份(浅拷贝)，放入池中，并返回池中的引用地址，所以s2 和 s 是一个对象。

// 例子2
String x = "ab";
String s = new String("a") + new String("b");	// 堆中的一个 String对象
String s2 = s.intern();	//	此时因为常量池中已经有了ab，则不会放入。
System.out.println( s2 == x);	// JDK6,7,8 true
System.out.println( s == x);	// JDK6,7,8 false

// 例子3
String s1 = "a";
String s2 = "b";
String s3 = "ab";
String s4 = s1 + s2;
/**
如下的s1 + s2 的执行细节：
	① StringBuilder s = new StringBuilder();
	② s.append("a")
	③ s.append("b")
	④ s.toString()  --> 约等于 new String("ab")

补充：
1.在jdk5.0之后使用的是StringBuilder,在jdk5.0之前使用的是StringBuffer
2.字符串拼接操作不一定使用的是StringBuilder!
	如果拼接符号左右两边都是字符串常量或常量引用，则仍然使用编译期优化，即非StringBuilder的方式。
3.针对于final修饰类、方法、基本数据类型、引用数据类型的量的结构时，能使用上final的时候建议使用上。

*/


// 例子4 
/**
	String s = new String("ab")会创建几个对象?
	答：两个，一个是new关键字在堆中创建的对象，一个是常量池中的对象。
*/

// 例子5
/**
	String s = new String("a") + new String("b")会创建几个对象?
	答：六个，
	对象1: new StringBuilder()
	对象2: new String("a")
	对象3: "a" --> 字符串常量池
	对象4: new String("b")
	对象5: "b" --> 字符串常量池
	对象6: new String("ab")  底层是 StringBuilder 调用了toString方法，所以在常量池中并不存在字符串 "ab"
*/


// 例子6
	String s = new String("a") + new String("b");//new String("ab")
    //在上一行代码执行完以后，字符串常量池中并没有"ab"

    String s2 = s.intern();		//jdk6中：在串池中创建一个字符串"ab"
                                //jdk8中：串池中没有创建字符串"ab",而是创建一个引用，指向new String("ab")，将此引用返回

    System.out.println(s2 == "ab");//jdk6:true  jdk8:true
    System.out.println(s == "ab");//jdk6:false  jdk8:true

```



#### StringBuilder、StringBuffer和String的区别

这个是面试可能会出现的问题，虽然现在已经比较少使用StringBuilder或者StringBuffer了，但是我们还是应该要知道一下这两个类出现的作用是什么?

其实核心的问题在于String是不可变，那字符串的拼接就变得不是那么友好，String在做字符串拼接的时候，需要重新申请内存存入新的字符串。基于这样的一个因素，StringBuilder和StringBuffer出现了，其主要的作用是在字符串拼接的效率上。**一般来说字符串的拼接速度是StringBuilder>StringBuffer>String**。而StringBuilder和StingBuffer的区别在于StringBuilder是线程不安全的，StringBuffer是线程安全的。下面我们看下其append方法，底层都是基于父类实现的

String实现了三个接口：Serializable、Comparable<String>、CharSequence

StringBuilder只实现了两个接口 Serializable、CharSequence，相比之下String的实例可以通过compareTo方法进行比较，其他两个不可以。

**String为字符串常量，而StringBuilder和StringBuffer均为字符串变量，即String对象一旦创建之后该对象是不可更改的，但后两者的对象是变量，是可以更改的。以下面一段代码为例：**

```JAVA
public AbstractStringBuilder append(String str) {//都是基于父类实现的

    if (str == null)
      return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
  }

而StringBuilder和StringBuffer的区别在于：方法上是否加锁
 //StringBuilder
 public StringBuilder append(String str) {
    super.append(str);
    return this;
  }

//StringBuffer
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;

  }

/** 
通过StringBuilder的append()的方式添加字符串的效率要远高于使用String的字符串拼接方式！
StringBuilder的append()的方式：自始至终中只创建过一个StringBuilder的对象
		   使用String的字符串拼接方式：创建过多个StringBuilder和String的对象,占用了过多的内存
改进的空间：在实际开发中，如果基本确定要前前后后添加的字符串长度不高于某个限定值highLevel的情况下,建议使用构造器实例化：
  	  
*/
StringBuilder s = new StringBuilder(highLevel);//new char[highLevel]	

```



#### Java中hashCode 和equals方法的区别

​		我们知道hashCode 和equals方法都是父类Object中的基本方法。HashCode 方法定义的初衷是为了求解一个类的hashCode值，而equals方法的定义初衷是为了判断两个对象是否内容一致。一般情况下，我们会认为**equals相等的两个对象，其hashcode值也相等。而hashcode相等的两个对象，equals不一定相等**。在《effect java》中有一条重要的规则是：如果重写了equals方法，就一定要重写hashcode方法。规则很简单，原因在于如果我们只重写了其中的一个，可能会导致在某些java的基本类中判断两个对象是否相等的时候出错，尤其是在hashMap中存入的对象作为key的时候，该对象一定要重写两个方法。因为hashMap在判断key是否相同的时候，是先判断hashcode后再判断equals的。

## 6. java常见关键字

​		java面试中的一些基本概念和常用的关键字，涉及到重载和重写、覆盖；java的抽象类和接口；static、final、public等关键字。其中的接口抽象类的概念需要重点掌握，java 的权限关键字也是代码中可以去关注的点，可是由于我们平时开发的代码安全性没有那么高，所以一般就是public解决一切问题，但掌握了对我们本身的代码质量还是有帮助的。下面我就通过一些面试中常见的问题解答的方式对这些概念进行描述。

### 1. java的构造函数

​		构造函数是一类特殊的函数，它在创建一个对象的时候调用，一个对象的创建只会调用一次构造函数。任何一个类都会有对应的构造函数，如果我们不去写构造函数的时候会默认生成一个无参构造函数；一般在构造函数中初始化类的属性值和函数的方法，这样达到一旦对象构建出来之后，相应的属性值就已经初始化完成。

特点：

:one: **方法名和类名一样；不能定义返回类型；这个没有返回值和返回值为void可不一样**

:two: **每个类在没有自己显式声明构造器的时候，都会有一个默认的无参构造**

:three: **构造器可以重载，而且可以使用super()、this()相互调用**

:four: **每个构造器的默认第一行都是super()，但是一旦父类中没有无参构造，必须在子类的第一行显式的声明调用哪一个构造**



### 2. 继承

​		继承的关系时，描述的是父子类之间的关系，继承的实质是子类继承父类的方法和属性。在java中，使用关键字extends表示一个类继承另一个类。在继承的关系中，有一个叫做菱形继承的问题，这个在C++中是支持的，在java中，不支持所谓的菱形继承。所谓的菱形继承是指一个类C即继承了父类A，又继承父类B，这样就导致了这样的一个菱形关系网络。Java中支持单继承，多重继承，但是不支持多继承（菱形继承）。java中通过接口实现了多继承的需求。

1. 当一个类不想被其他类继承的时候，我们可以使用final来修饰这个类。

2. 构造函数是不能被继承的，如果要调用父类的构造函数的是，可以使用super关键字进行访问。

3. 在继承关系中，初始化子类的时候，一定是先调用其父类的构造函数，然后调用子类的构造函数；
4. 接口只能继承接口，但是可以多继承。类都是单继承，但是继承有传递性。



### 3. 重写和重载

重写（override）：在java中，子类继承了父类的属性和方法，但是当子类修改扩充了方法中的方法时，我们把它称作为重写。重写需要保证**返回值和形参都不变**；并且重写不能抛出新异常或者比父类更加宽泛的异常。重写的优势在于**子类可以修改父类的实现，来达到相同的方法不同的功能**。

重载（overload）：在同一个类中，由于**方法名相同，参数不同，返回值可以相同也可以不同的多个方法**，被称为重载。最常见的重载是构造函数重载。

下面我们看下重写和重载的区别：

| 区别点   | 重载     | 重写           |
| -------- | -------- | -------------- |
| 参数列表 | 必须修改 | 不能修改       |
| 返回类型 | 可以修改 | 不能修改       |
| 异常     | 可以修改 | 不能抛出更广泛 |
| 权限访问 | 可以修改 | 不能更严格限制 |

**注：**构造函数不能重写，因为构造函数不能被继承，就更不能说到重写，但是构造方法常常被重载，也就是我们常常使用的不同参数的构造函数。私有方法都不能被子类访问，所以私有方法也不能被重写。

### is-a & like-a & has-a

强调继承关系，is-a，如果A is-a B，那么B就是A的父类；

代表组合关系，like-a，接口，如果A like a B，那么B就是A的接口。 ；

强调从属关系，has-a，如果A has a B，那么B就是A的组成部分。

### 4. 修饰符

Java语言提供了很多修饰符，大概分为两类：

1. 访问权限修饰符
2. 非访问权限修饰符

#### 1. 访问权限修饰符

1. **public：**共有访问。对所有的类都可见。
2. **protected：**保护型访问。对同一个包可见，对不同的包的子类可见。
3. **default：**默认访问权限。只对同一个包可见，注意对不同的包的子类不可见。
4. **private：**私有访问。只对同一个类可见，其余都不见。

| 修饰符    | 当前类 | 同一个包内 | 子孙类（同包） | 子孙类（不同包） | 其他包 |
| --------- | ------ | ---------- | -------------- | ---------------- | ------ |
| Public    | Y      | Y          | Y              | Y                | Y      |
| Protected | Y      | Y          | Y              | Y/N              | N      |
| Default   | Y      | Y          | Y              | N                | N      |
| private   | Y      | N          | N              | N                | N      |

#### 2. 非访问权限修饰符

1. static 修饰符，用来创建类方法和类变量。
2. final 修饰符，用来修饰类、方法和变量，final 修饰的类不能够被继承，修饰的方法不能被继承类重新定义，修饰的变量为常量，是不可修改的。
3. abstract 修饰符，用来创建抽象类和抽象方法。
4. synchronized 用于多线程的同步。
5. volatile 修饰的成员变量在每次被线程访问时，都强制从共享内存中重新读取该成员变量的值。而且，当成员变量发生变化时，会强制线程将变化值回写到共享内存。这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。
6. transient：序列化的对象包含被 transient 修饰的实例变量时，java 虚拟机(JVM)跳过该特定的变量。

#### 3. 外部类修饰符

1. public（访问控制符），将一个类声明为公共类，它可以被任何对象访问，一个程序的主类必须是公共类。
2. default（访问控制符），类只对包内可见，包外不可见。
3. abstract（非访问控制符），将一个类声明为抽象类，抽象类不能用来实例化对象，声明抽象类的唯一目的是为了将来对该类进行扩充，抽象类可以包含抽象方法和非抽象方法。。
4. final（非访问控制符），将一个类生命为最终（即非继承类），表示它不能被其他类继承。

注：

1. protected 和 private 不能修饰外部类，是因为外部类放在包中，只有两种可能，包可见和包不可见。
2. final 和 abstract不能同时修饰外部类，因为该类要么能被继承要么不能被继承，二者只能选其一。
3. 不能用static修饰，因为类加载后才会加载静态成员变量。所以不能用static修饰类和接口，因为类还没加载，无法使用static关键字。

### 5. static关键字

**说说你了解到的static关键字的用法？为什么static方法中不能使用this关键字？静态内部类的作用是什么？**

​		Static是 java 的一个关键字，该关键字在java中被广泛使用，静态变量，静态方法是我们常用的方式，静态的关键字更多的时候，我会将它理解为从一个对象的概念提升到一个类的概念。比如一个成员变量升级为静态变量的时候，我们更多会认为这个变量是某个类的变量，不再是某个对象的。这样就导致jvm存入静态变量和普通变量的地方是有区别的。

**1.** **静态变量**：静态变量是将类的属性变量加上static关键字，加上静态变量的属性在类的加载过程中的第三阶段（准备）会被分配内存，这样静态变量就和类的方法一起被放在方法区，并且一个类只有一个地方存储静态变量，所以这就是我们说的类的静态变量是属于类的，而不属于某个对象。

**2.** **静态函数**： 由于函数本身是存在方法区中的，所以静态函数和非静态函数在存储上并没有太大的差异，但是静态函数和非静态函数的主要区别在于静态函数可以直接通过类进行调用，而非静态函数需要创建具体的实例，通过实例调用方法。

**3.** **静态代码块**：在java类中将一段代码不用任何函数，而是用static修饰，我们把这段代码称为静态代码块，一般静态代码块的作用是为静态变量赋初始值。

**4.** **内部静态类**：在java语言中，允许类种类，也就是通常我们说的内部类。一般内部类的作用是只有该类会调用，并且不想暴露出太多的信息给外部，并且这些信息是一个完整的个体，所以使用内部类来定义是比较合适的，jdK的源码中有很多内部类的定义。如果把这个内部类加上static修饰符的时候，那这个类就是一个静态内部类。静态内部类具有静态的一般特性，直接通过类名访问，外部可以不用创建对象就可以直接访问静态内部类。静态内部类和普通的内部类有较大的差别：

1） 静态内部类可以有静态方法和静态变量，非静态内部类不能有

2） 静态内部类不能访问外部类的非静态方法和非静态变量，而非静态内部类可以随意访问外部类的方法和变量，包含是私有的。

3） 创建静态内部类不需要创建外部类的对象，而使用非静态内部类的时候，一定是外部的对象进行调用的。

其实上面的差别，实质在于我们将内部类当做外部类的方法，静态内部类类似于静态方法，非静态内部类类似于非静态方法。

**5.** **静态导入**：这个是在引入包的时候，引入了静态方法或者静态变量，这个作用就是将函数中的静态放入包的引入中，以保证代码的简洁性。

```java
import static com.java.base.learn.reflect.StrTest.B;
public class StaticTest {
  //静态变量
  public static int a ;
 
  //静态代码块
  static{
    a = 1;
    System.out.println(B);
  }

  //静态方法
  public static void setA(int a0){
    a = a0;
  }

  //静态内部类
  public static class InnerClassTest{
    private static int c;
    private int d;
  }
}
```



### 8. switch语句

在Java7之前，switch只能支持 byte、short、char、int或者其对应的封装类以及Enum类型。

在Java7中，也支持了String类型 ：String byte short int char Enum 类型

### 9. final关键字

1、final关键字可以用于成员变量、本地变量、方法以及类。

2、 final成员变量必须在声明的时候初始化或者在构造器中初始化，否则就会报编译错误。

3、 你不能够对final变量再次赋值。

4、 本地变量必须在声明时赋值。

5、 在匿名类中所有变量都必须是final变量。

6、 final方法不能被重写。

7、 final类不能被继承。

8、 没有在声明时初始化final变量的称为空白final变量(blank final variable)，它们必须在构造器中初始化，或者调用this()初始化。不这么做的话，编译器会报错“final变量(变量名)需要进行初始化”。

### 10. super 和 this 关键字

super和this都只能位于构造器的第一行，而且不能同时使用，这是因为会造成初始化两次，this用于调用重载的构造器，super用于调用父类被子类重写的方法。

## 7. 异常

<img src="http://uploadfiles.nowcoder.com/images/20151113/140047_1447376765880_373DC390B08E99ABC340DB1F78F35FCB" alt="img" style="zoom:150%;" />

都是Throwable的子类：

1. Error（错误）:是程序无法处理的错误。这些错误表示故障发生于虚拟机内部错误以及资源耗尽的情形，一般不需要程序处理。

   例如：常见的 OotOfMemoryError 和 StackOverflowError

2. Exception（异常）:是程序本身可以处理的异常。
   1. 检查异常（**编译器要求必须处置的异常**）： 除了Error，RuntimeException及其子类以外，其他的Exception类及其子类都属于可查异常，一般是外部错误。这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。常见的例如：IOException

   2. 非检查异常(编译器不要求处置的异常)：包括运行时异常（RuntimeException与其子类）和错误（Error）。

      常见的：NullPointerException

##### Q1:final、finally、finalize的区别是什么？

这几个词只是长得有点相同，其实它们在java中没有任何关系。

final 是固定的意思，它的常见用法是用来修饰变量，类，方法，当我们修饰变量的时候final修饰的是常量，当修饰方法的时候，表示方法不能被覆盖，当修饰类的时候，表示类不能继承。当然还有一种是修饰方法内的变量，这种在lambda表达式的时候经常会被使用到，这个就是表示传过来的值是不允许改变的。

finally是最终的意思，这个是在java异常机制中，表示都要执行的意思，这个在上面的内容中已经提到了，这里不多说明。

finalize是java的一个方法，它不是关键字，它是jvm垃圾回收的时候使用。我们知道finalize是object的一个方法，任何一个对象都有该方法，jvm在垃圾回收的时候，回收线程会先判断对象是否不可达(GCRoots不能连接)，如果不可达就代表可以回收，这个时候判断该对象是否覆盖了finalize方法，如果没有覆盖，直接回收；如果有覆盖，就将该对象放置在一个F-Queue队列中，由优先级低的线程执行，如果执行完成之后，再次判断该对象是否不可达，如果不可达就直接回收，如果可达的话，就是我们常说的对象逃逸。这是finalize函数的设计初衷，一般我们在实际项目中很少会用到这个方法，毕竟垃圾回收我们希望虚拟机自己管理就好了。

## 8. 基本数据类型及其包装类

### 1.基本数据类型的介绍

整数类型 4 种

byte 占1个字节，8位

short 占2个字节，16位

int 占4个字节，32位

long 占8个字节，64位

浮点类型：2种

float 占4个字节，32位

double 占8个字节，64位

字符类型：

char 占2个字节，16位

从下往上转时，可以自动转换，但是从上往下转换时，则需要强制类型转换，否则会报编译时错误。

即 byte -> int  可以，但是int -> byte 需要强制类型转换。

| 默认值  | 存储需求（字节） | 取值范围 | 示例         |                    |
| ------- | ---------------- | -------- | ------------ | ------------------ |
| byte    | 0                | 1        | -2^7—2^7-1   | byte b=10;         |
| char    | ‘ \u0000′        | 2        | 0—2^16-1     | char c=’c’ ;       |
| short   | 0                | 2        | -2^15—2^15-1 | short s=10;        |
| int     | 0                | 4        | -2^31—2^31-1 | int i=10;          |
| long    | 0                | 8        | -2^63—2^63-1 | long o=10L;        |
| float   | 0.0f             | 4        | -2^31—2^31-1 | float f=10.0F      |
| double  | 0.0d             | 8        | -2^63—2^63-1 | double d=10.0;     |
| boolean | false            | 1        | true\false   | boolean flag=true; |

**当基本数据类型参加算术运算时：**

1. 所有的byte,short,char型的值将被提升为int型；
2. 如果有一个操作数是long型，计算结果是long型；
3. 如果有一个操作数是float型，计算结果是float型；
4. 如果有一个操作数是double型，计算结果是double型；
5. 被fianl修饰的变量不会自动改变类型，当2个final修饰相操作时，结果会根据左边变量的类型而转化。

```
/*
对于整数，默认为十进制的int类型，当想要long型时后缀要加L/l;
十六进制+前缀0X/0x;
八进制+前缀 0;
二进制+前缀 0b/0B;
对于浮点数，默认为 double 类型，当想要 float 类型时加后缀 F/f
*/
int i = 12;
long l1 = 12L;
int i = 010;	// 八进制，会转换成 十进制的8
int i = 0b1001;
int i = 0xCAFF;

```

例题：
```java
byte b1 = 1,b2 = 2, b3, b6, b8;
final byte b4 = 4,b5 = 6, b7;
b3 = (byte) (b1+b2);  /*语句1*/
b6 = b4 + b5;    	  /*语句2*/
b8 = (byte) (b1+b4);  /*语句3*/
b7 = (byte) (b2+b5);  /*final修饰，即只可赋值一次，便不可再改变*/
b3 += b1;		
// 这句话是可以正常编译的，语句中用的是a+=b的语句，此语句会将被赋值的变量自动强制转化为相对应的类型。

```

###  涉及包装类的 == 和 equals

:one:  基本数据类型只有 == 判断（判断值是否相等），没有equals方法

:two:  包装类的 == 方法会判断对象的地址值是否相等，而 equals方法经过重写判断的是值

```java
/*
注意两种赋值形式的区别:
方式一：由于直接赋值的话会进行自动的装箱。所以当值在[-128,127]中的时候，由于值缓存在IntegerCache中，那么当赋值在这个区间的时候，不会创建新的Integer对象，而是直接从缓存中获取已经创建好的Integer对象。而当大于这个区间的时候，会直接new Integer。
方拾二：直接在堆中创建，此方法在JKD9中已被弃用
所以 n1 != n2; n2 != n3;
	n1 == n4;
*/
Integer n1 = 47;
@Deprecated(since="9")
Integer n2 = new Integer(47);
Integer n3 = new Integer(47);
Integer n4 = Integer.valueOf(47);

```



Float是类，float不是类.  

​    查看JDK源码就可以发现Byte，Character，Short，Integer，Long，Float，Double，Boolean都在java.lang包中.  

   Float正确复制方式是Float f=1.0f,若不加f会被识别成double型,double无法向float隐式转换.  

   Float a= new    Float(1.0)是正确的赋值方法，但是在1.5及以上版本引入自动装箱拆箱后，会提示这是不必要的装箱的警告，通常直接使用Float    f=1.0f.



**Float a = new Float(1.0); 这个的东西能存在，是因为Float类中有形参是float和double的两个构造器。** 

  **Double d = new Double(1.0F);这个能成立的原因是float向上转型了。** 

  **Float a = 1.0；这个东西不成立是因为浮点型的默认类型是double，而double不会自动转成float，然后再装箱。** 

  **Double d = 1.0f；不成立的原因是因为Double类中的装箱方法，只有valueOf(String s)和valueOf(double d)；装箱本身可不会自动向上转型啊。**

## 9. 程序设计的六大原则

**1、开闭原则（Open Close Principle）**

开闭原则的意思是：**对扩展开放，对修改关闭**。在程序需要进行拓展的时候，不能去修改原有的代码，实现一个热插拔的效果。简言之，是为了使程序的扩展性好，易于维护和升级。

**2、里氏代换原则（Liskov Substitution Principle）**

里氏代换原则是面向对象设计的基本原则之一。 里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现。LSP 是继承复用的基石，只有当派生类可以替换掉基类，且软件单位的功能不受到影响时，基类才能真正被复用，而派生类也能够在基类的基础上增加新的行为。里氏代换原则是对开闭原则的补充。实现开闭原则的关键步骤就是抽象化，而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。

**3、依赖倒转原则（Dependence Inversion Principle）**

这个原则是开闭原则的基础，具体内容：针对接口编程，依赖于抽象而不依赖于具体。

**4、接口隔离原则（Interface Segregation Principle）**

这个原则的意思是：使用多个隔离的接口，比使用单个接口要好。它还有另外一个意思是：降低类之间的耦合度。由此可见，其实设计模式就是从大型软件架构出发、便于升级和维护的软件设计思想，它强调降低依赖，降低耦合。

**5、迪米特法则，又称最少知道原则（Demeter Principle）**

最少知道原则是指：一个实体应当尽量少地与其他实体之间发生相互作用，使得系统功能模块相对独立。

**6、合成复用原则（Composite Reuse Principle）**

合成复用原则是指：尽量使用合成/聚合的方式，而不是使用继承



## 10. 集合

![img](https://uploadfiles.nowcoder.com/images/20170304/837161_1488616442711_250E74268F38A4202D8C30E4329DEBCC)

线程安全的：Vector, HashTable

Vector 与 ArrayList 类似，只是Vector是线程安全的，它给几乎所有的public方法都加上了synchronized关键字。由于加锁导致性能降低，在不需要并发访问同一对象时，这种强制性的同步机制就显得多余，所以现在Vector已被弃用。

HashTable和HashMap类似，不同点是HashTable是线程安全的，它给几乎所有public方法都加上了synchronized关键字，还有一个不同点是HashTable的K，V都不能是null，但HashMap可以，它现在也因为性能原因被弃用了

## 11. 接口和抽象类

## interface & abstract 

​		在继承的时候，提到java是不支持多继承的，那如果一个类想要继承多个方法的时候，我们可以使用实现接口的方式，java是支持实现多接口的。这里先阐述一下接口和类的问题。下面我们看下接口和抽象类的基本概念。

### 1. 接口

接口是抽象方法和常量值的集合

接口的特点：

:one: 用interface定义

:two: 接口中的所有成员变量都默认是 public static final 修饰的

:three: 接口中的所有抽象方法都默认是 public abstract 修饰的

:four: 接口中没有构造器

:five: 接口采用多继承制

![image-20210831110127788](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20210831110127788.png)

:one: 如果是Java 7以及以前的版本，那么接口中可以包含的内容有：1. 常量；2. 抽象方法
:two: 如果是Java 8，还可以额外包含有：3. **默认方法**；4. **静态方法**（static和default只能存在一种，要不就是静态，要不就是default。）
:three: 如果是Java 9，还可以额外包含有：5. **私有方法**

### 2. 抽象类

​		在java语言中，使用 abstract 关键字来表示一个类是抽象类，当然 abstract 也可以修饰方法，代表抽象方法。抽象类是一个类中还存在不能实现的方法，我们通过把这些方法留给子类去定义，而父类提供方法名。值得注意的是：抽象类并不一定需要有抽象函数，但是有抽象方法的类一定是抽象类。

抽象类可以实现接口，可以继承具体类，可以继承抽象类，也可以继承有构造器的实体类。

抽先类的构成：

1. 各种类型的成员变量

2. 构造方法

   注：抽象类只是不能直接创建抽象类的实例对象而已。在继承了抽象类的子类中通过super(参数列表)调用抽象类中的构造方法，用于实例化抽象类的字段。

3. 各种权限的方法，包括 静态的main方法

### 3. 抽象方法

Abstract 关键字同样可以用来声明抽象方法，抽象方法只包含一个方法名，而没有方法体。

抽象方法没有定义，方法名后面直接跟一个分号，而不是花括号。

```java
public abstract class Employee {
    private String name;
    private String address;
    private int number;
    public abstract double computePay();
    //其余代码
}
```

**特点：**

1. 构造方法 不能声明为抽象方法。
2. 抽象方法要被实现，所以不能是静态static的，也不能是私有private的，也不能被final修饰（试想一下，静态方法可以被类名直接调用，而类名直接调用一个没有实现的抽象方法没有意义）。

### Q1: 接口 VS 抽象类

1）抽象类中的**成员变量**可以是各种类型的，而接口中的成员变量只能是 public static final 类型的。

2）接口中不能含有静态代码块，而抽象类是可以有静态代码块和静态方法。

3） 一个类只能继承一个抽象类，而一个类却可以实现多个接口。

其实上面的第三点是接口和抽象类的最大区别，这个也是接口和类的核心所在。

相同点在于：两种都不能直接被实例化，都需要被继承之后，通过子类实例。

**标记接口**：如果一个接口不包含任何的变量和函数定义的话，那我们就把该接口叫做标记接口，其实我们之前的序列化接口（Serializable、cloneable）就是一个标记接口。

标记接口在实际中有以下两个应用：

1）构建公共的父接口；

2）添加数据类型。

### Q2: 接口和实现类的关系

方法头指：修饰符+[返回类型](https://www.baidu.com/s?wd=返回类型&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao) +方法名（形参列表）

接口的访问权限：public，abstract

**两同两小一大原则:** 

1. 方法名和参数列表相同

2. 返回值类型小于等于父类的返回值类型

3. 异常小于等于父类抛出异常

4. 访问权限大于等于父类



1）接口可以继承接口，而且可以继承多个接口，但是不能实现接口，因为接口中的方法全部是抽象的，无法实现；

2）普通类可以实现接口，并且可以实现多个接口，但是只能继承一个类，这个类可以是抽象类也可以是普通类，如果继承抽象类，必须实现抽象类中的所有抽象方法，否则这个普通类必须设置为抽象类；

3）

下面总结常见的抽象类与接口的区别：

1）抽象类和接口都不能直接实例化，如果要实例化，抽象类变量必须指向实现所有抽象方法的子类对象，接口变量必须指向实现所有接口方法的类对象；

2）接口只能做方法申明，抽象类中可以做方法申明，也可以做方法实现（java8中 接口可以有实现方法 使用**default**修饰）；

4）抽象类里的抽象方法必须全部被子类所实现，如果子类不能全部实现父类抽象方法，那么该子类只能是抽象类。同样，一个类实现接口的时候，如不能全部实现接口方法，那么该类也只能为抽象类；

5）抽象方法要被实现，所以不能是静态static的，也不能是私有private的，也不能被final修饰（试想一下，静态方法可以被类名直接调用，而类名直接调用一个没有实现的抽象方法没有意义）。

## 12. 内部类

<img src="https://uploadfiles.nowcoder.com/images/20190221/242025553_1550728055483_BA9669C5826A238ACEC0BD86755FA5DB" alt="img" style="zoom:150%;" />

在Java中，可以将一个类定义在另一个类里面或者一个方法里边，这样的类称为内部类，广泛意义上的内部类一般包括四种：成员内部类，局部内部类，匿名内部类，静态内部类 。 

### 1.成员内部类 

  （1）该类像是外部类的一个成员，可以无条件的访问外部类的所有成员属性和成员方法（包括private成员和静态成员）； 

  （2）成员内部类拥有与外部类同名的成员变量时，会发生隐藏现象，即默认情况下访问的是成员内部类中的成员。如果要访问外部类中的成员，需要以下形式访问：【外部类.this.成员变量  或  外部类.this.成员方法】； 

  （3）在外部类中如果要访问成员内部类的成员，必须先创建一个成员内部类的对象，再通过指向这个对象的引用来访问； 

  （4）成员内部类是依附外部类而存在的，也就是说，如果要创建成员内部类的对象，前提是必须存在一个外部类的对象； 

  （5）内部类可以拥有private访问权限、protected访问权限、public访问权限及包访问权限。

​		如果成员内部类用private修饰，则只能在外部类的内部访问；如果用public修饰，则任何地方都能访问；如果用protected修饰，则只能在同一个包下或者继承外部类的情况下访问；如果是默认访问权限，则只能在同一个包下访问。

### 2.局部内部类 

  （1）局部内部类是定义在一个方法或者一个作用域里面的类，它和成员内部类的区别在于局部内部类的访问仅限于方法内或者该作用域内； 

  （2）局部内部类就像是方法里面的一个局部变量一样，是不能有public、protected、private以及static修饰符的。 

### 3.匿名内部类 

  （1）一般使用匿名内部类的方法来编写事件监听代码； 

  （2）匿名内部类是**不能有访问修饰符和static修饰符的**； 

  （3）匿名内部类是唯一一种**没有构造器的类**； 

  （4）匿名内部类用于继承其他类或是实现接口，并不需要增加额外的方法，只是对继承方法的实现或是重写。 

### 4.内部静态类 

  （1）静态内部类是不需要依赖于外部类的，这点和类的静态成员属性有点类似； 

  （2）不能使用外部类的非static成员变量或者方法。

## 13. 外部类

### 外部类的组成

类的成员变量包括**实例变量**和**类变量**（静态变量）,成员方法包括**实例方法**和**类方法**（**静态方法**）。

### 实例变量

1. 实例变量声明在一个类中，但在方法、构造方法和语句块之外；
2. 当一个对象被实例化之后，每个实例变量的值就跟着确定；
3. 实例变量在对象创建的时候创建，在对象被销毁的时候销毁；
4. 实例变量的值应该至少被一个方法、构造方法或者语句块引用，使得外部能够通过这些方式获取实例变量信息；
5. 实例变量对于类中的方法、构造方法或者语句块是可见的。一般情况下应该把实例变量设为私有。通过使用访问修饰符可以使实例变量对子类可见；
6. 实例变量具有默认值。数值型变量的默认值是0，布尔型变量的默认值是false，引用类型变量的默认值是null。变量的值可以在声明时指定，也可以在构造方法中指定；
7. 实例变量可以直接通过变量名访问。但在静态方法以及其他类中，就应该使用完全限定名：ObejectReference.VariableName。

### 类变量（静态变量）

1. 类变量也称为静态变量，在类中以static关键字声明，但必须在方法构造方法和语句块之外。
2. 无论一个类创建了多少个对象，类只拥有类变量的一份拷贝。
3. 静态变量除了被声明为常量外很少使用。常量是指声明为public/private，final和static类型的变量。常量初始化后不可改变。
4. 静态变量储存在静态存储区。经常被声明为常量，很少单独使用static声明变量。
5. 静态变量在程序开始时创建，在程序结束时销毁。
6. 与实例变量具有相似的可见性。但为了对类的使用者可见，大多数静态变量声明为public类型。
7. 默认值和实例变量相似。数值型变量默认值是0，布尔型默认值是false，引用类型默认值是null。变量的值可以在声明的时候指定，也可以在构造方法中指定。此外，静态变量还可以在静态语句块中初始化。
8. 静态变量可以通过：*ClassName.VariableName*的方式访问。
9. 类变量被声明为public static final类型时，类变量名称必须使用大写字母。如果静态变量不是public和final类型，其命名方式与实例变量以及局部变量的命名方式一致。

### 实例方法和类方法对实例变量和类变量的访问

实例方法可以对当前对象的实例变量进行操作，也可以对类变量进行操作。实例方法由实例对象调用。

类方法不能访问实例变量，只能访问类变量。类方法由类名或者实例对象调用。类方法中不能出现this或者super关键字

# 二. 多线程

## 1. 多线程的概念

​		在java中多线程的概念，一般是使用java.lang.Thread类或者java.lang.Runnable接口编写代码来定义、实例化和启动新线程。

​		一个Thread类实例只是一个对象，像Java中的任何其他对象一样，具有变量和方法，生死于堆上。Java中，每个线程都有一个调用栈，即使不在程序中创建任何新的线程，线程也在后台运行着。一个Java应用总是从main()方法开始运行，mian()方法运行在一个线程内，它被称为主线程。一旦创建一个新的线程，就产生一个新的调用栈。

线程总体分两类：用户线程和守候线程。

当所有用户线程执行完毕的时候，JVM自动关闭。但是守候线程却不独立于JVM，守候线程一般是由操作系统或者用户自己创建的。

## 1. 创建线程对象两种方式

1. Class A 继承java.lang.Thread类，并重写了run方法，则new A( ).start( )，就执行了线程。

2. Class A 实现 java.lang.Runnable 接口，并覆盖了run方法，new Thread( new A( )).start( )就执行了线程

以上两种run方法都是无返回值的，如果需要返回值，则需要 Callable接口

例题1：

以下多线程对int型变量x的操作，哪个不需要进行同步（  ）

- ```
  x=y;
  ```

- ```
  x++;
  ```

- ```
  ++x;
  ```

- ```
  x=1;
  ```

A.由于y的值不确定，所以要加锁； 

B,C 两个在多线程情况下是必须要加锁的，因为他们是先被读入寄存器，然后再进行+1操作，如果没有加锁，那么可能会出现数据异常； 

D 原子操作，所以不需要加锁

**原子性**：指该操作不能再继续划分为更小的操作。   

**Java中的原子操作包括：**

​        1、除long和double之外的基本类型的赋值操作   

​        2、所有引用reference的赋值操作   

​        3、java.concurrent.Atomic.* 包中所有类的一切操作

# 3.JVM

# 4. 网络

# 5.数据结构

# 6.算法

# 7.分布式系统

# 8.设计题

# 9.数据库

# 10.常用的中间件

# 11.zk

# 12.缓存redis

# 13.消息中间件





# 常用符号

:one: 

:two: 

:three: 

:four: 

