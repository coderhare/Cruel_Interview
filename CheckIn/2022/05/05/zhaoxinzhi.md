### 一、写在前面

Java对象为啥要实现Serializable接口？这是个困扰了我很久的问题。

是不是在网络传输的时候，必须要实现Serializable接口？

什么时候是必须实现Serializable接口的？

不实现会咋样？

> 参考链接：https://blog.csdn.net/weixin_40482816/article/details/117459939
>
> 参考链接2：https://www.cnblogs.com/Alitac/p/12344370.html



## 二、Java-Serializable接口

### 0、序列化

可以使用表示对象及其数据的类型信息和字节在内存中重新创建对象。

序列化对于面向对象的编程语言来说是非常重要的，因为无论什么编程语言，其底层涉及IO操作的部分还是由操作系统帮其完成的，而底层IO操作都是以字节流的方式进行的，所以**写操作**都涉及将编程语言数据类型转换为字节流，而**读操作**则又涉及将字节流转化为编程语言类型的特定数据类型。

对于Java来说，需要一种方式来让JVM知道在进行IO操作时如何将对象数据转换为字节流（序列化），以及如何将字节流数据转换为特定的对象（反序列化），而Serializable接口就承担了这样一个角色。

**序列化的方式**：转成json也是一种序列化方式，缺点是比较占用空间，不是二进制的，是文本的。优点是易于理解，所见即所得。

**序列化的用途**：

- 想把的内存中的对象状态保存到一个文件中或者数据库中时候。
- 想把对象通过网络进行传播的时候。

**为什么要序列化？**

> 1、为了数据持久化。2、为了网络传输。（因为以上两个场景在设计的时候都只能通过字节流的方式传输）

1、将对象的状态保存在存储媒体中以便可以在以后重新创建出完全相同的副本；

2、按值将对象从一个应用程序域发送至另一个应用程序域。实现serializabel接口的作用是就是可以把对象存到字节流，然后可以恢复，所以你想如果你的对象没实现序列化怎么才能进行持久化和网络传输呢，要持久化和网络传输就得转为字节流，所以在分布式应用中及设计数据持久化的场景中，你就得实现序列化。

**是不是每个实体bean都要实现序列化？**

取决于你的bean是否需要持久化存储媒体中以及是否需要传输给另一个应用，没有的话就不需要，例如我们利用fastjson将实体类转化成json字符串时，并不涉及到转化为字节流，所以其实跟序列化没有关系。



### 1、Serializable接口概述

Serializable是java.io包中定义的，用于实现Java类的序列化操作而提供的一个**语义级别**的接口。Serializable序列化接口没有任何方法或者字段，只是用于标识可序列化的语义。实现了Serializable接口的类可以被ObjectOutputStream转换为字节流，同时也可以通过ObjectInputStream再将其解析为对象。

我们可以将**序列化**对象写入文件，再次从文件中读取它并**反序列化**成对象，也就是说，可以使用表示对象及其数据的类型信息和字节在内存中重新创建对象。

### 2、如何序列化

序列化都是对类对象来说的，只要一个类实现Serializable接口，那么这个类就可以序列化了。（==可以序列化类吗还是只能对象？==好像不能类，从 不能保存静态成员就可以看出）

通过ObjectOutputStream 的writeObject()方法把这个类的对象写到一个地方（文件），再通过ObjectInputStream 的readObject()方法把这个对象读出来。

如果把Person类中的implements Serializable 去掉，在运行上述过程就会报**java.io.NotSerializableException**异常。

```java
piblic class Person implements Serializable{   
    private static final long serialVersionUID = 1L; //默认是1
    String name;
    int age;
    public Person(String name,int age){
        this.name = name;
        this.age = age;
    }   
    public String toString(){
        return "name:"+name+"\tage:"+age;
    }
}

测试类
public class SerializableTest { 
 
    /** 
     * 将User对象作为文本写入磁盘 ,注意序列化是针对对象的！！不是类的！
     */ 
    public static void writeObj() { 
        Person user = new Person("1001", "Joe"); 
        try { 
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("/Users/guanliyuan/user.txt")); 
            objectOutputStream.writeObject(user); 
            objectOutputStream.close(); 
        } catch (IOException e) { 
            e.printStackTrace(); 
        } 
    } 
 
    public static void main(String args[]) { 
        writeObj(); 
    } 
}
```



### 3、关于serialVersionUID

注意到上面程序中有一个 serialVersionUID ，实现了Serializable接口之后，Eclipse就会提示你增加一个 serialVersionUID，虽然不加的话上述程序依然能够正常运行。

序列化 ID 在 Eclipse 下提供了两种生成策略

- 一个是固定的 1L
- 一个是随机生成一个不重复的 long 类型数据（实际上是使用 JDK 工具，根据类名、接口名、成员方法及属性等来生成）

如果是通过网络传输的话，如果Person类的serialVersionUID不一致，那么反序列化就不能正常进行。例如在客户端A中Person类的**serialVersionUID=1L**，而在客户端B中Person类的**serialVersionUID=2L** 那么就不能重构这个Person对象。试图重构就会报**java.io.InvalidClassException**异常，因为这两个类的版本不一致，local class incompatible，重构就会出现错误。

如果没有特殊需求的话，serialVersionUID使用默认的 1L 就可以，这样可以确保代码一致时反序列化成功。那么随机生成的序列化 ID 有什么作用呢，有些时候，通过改变序列化 ID 可以用来限制某些用户的使用。



### 4、静态变量无法序列化

> 成员方法，静态成员变量，变量的修饰符都不能保存！

串行化只能保存对象的非静态成员交量，不能保存任何的**成员方法**和**静态的成员变量**，而且串行化保存的只是变量的值，对于变量的任何**修饰符都不能保存**。

如果把Person类中的name定义为static类型的话，试图重构，就不能得到原来的值，只能得到null。说明对静态成员变量值是不保存的。

这其实比较容易理解，**序列化保存的是对象的状态**，静态变量属于类的状态，因此 序列化并不保存静态变量。



### 5、transient关键字

经常在实现了 Serializable接口的类中能看见transient关键字。这个关键字并不常见。

**transient关键字的作用是**：阻止实例中那些用此关键字声明的变量持久化；当对象被反序列化时（从源文件读取字节序列进行重构），这样的实例变量值不会被持久化和恢复。

当某些变量不想被序列化，同是又不适合使用static关键字声明，那么此时就需要用transient关键字来声明该变量。

例如用 transient关键字 修饰name变量

```java
public class Person implements Serializable{   
    private static final long serialVersionUID = 1L;
    transient String name;
    int age;
    public Person(String name,int age){
        this.name = name;
        this.age = age;
    }   
    public String toString(){
        return "name:"+name+"\tage:"+age;
    }
}
//在反序列化试图重构对象的时候，作用与static变量一样： 输出结果为：
//name:null   age:22
```

**在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。**（==Integer也是null？==）

注：对于某些类型的属性，其状态是瞬时的，这样的属性是无法保存其状态的。例如一个线程属性或需要访问IO、本地资源、网络资源等的属性，对于这些字段，我们必须用transient关键字标明，否则编译器将报措。

（==暂时还没法理解。。==）

### 6、序列化中的继承问题（隐藏的坑点）

- 当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口。
- 一个子类实现了 Serializable 接口，它的父类都没有实现 Serializable 接口，要想将父类对象也序列化，就需要让父类也实现Serializable 接口。

第二种情况中：如果父类不实现 Serializable接口的话，就必须需要有默认的无参的构造函数（有参的不行）。这是因为一个 Java 对象的构造必须先有父对象，才有子对象，反序列化也不例外。在反序列化时，为了构造父对象，只能调用父类的无参构造函数作为默认的父对象。因此当我们取父对象的变量值时，它的值是调用父类无参构造函数后的值。**在这种情况下，在序列化时根据需要在父类无参构造函数中对变量进行初始化，若在无参构造函数中没有对变量初始化，则父类变量值都是默认声明的值**，如 int 型的默认是 0，string 型的默认是 null。

例子：

```java
public class People{
    int num;
    public People(){}           //默认的无参构造函数，没有进行初始化
    public People(int num){     //有参构造函数
        this.num = num;
    }
    public String toString(){
        return "num:"+num;
    }
}
 
public Person extends People implements Serializable{    
    private static final long serialVersionUID = 1L;
    String name;
    int age;
    public Person(int num,String name,int age){
        super(num);             //调用父类中的构造函数
        this.name = name;
        this.age = age;
    }
    public String toString(){
        return super.toString()+"\tname:"+name+"\tage:"+age;
    }
}
//在一端写出对象：
    Person person = new Person(10,"tom", 22); //调用带参数的构造函数num=10,name = "tim",age =22
    System.out.println(person);
    oos.writeObject(person);                  //写出对象

//在另一端读出对象：
    Person person = (Person)ois.readObject(); //反序列化，调用父类中的无参构函数。
    System.out.println(person);
//输出：    num:0   name:tom    age:22
```

