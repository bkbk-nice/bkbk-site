---
sidebar_position: 1
--- 
 
### 基本数据类型
```
基本数据类型的变量保存原始值，即变量代表值本身，存储在Java栈的栈帧中，直接在栈中参与运算。  
●  byte/8   
●  char/16  
●  short/16   
●  int/32   //在定义int a 的时候就已经给a在栈中分配了数据空间  
●  float/32  
●  long/64  
●  double/64  
●  boolean/~  
boolean在编译的时候  
如果是单个boolean变量，会被转化为int  
如果是boolean数组，会被转化为byte


```


### 引用类型
``` 
包装类型（属于引用类型）  
基本类型都有对应的包装类型，他们之间的相互转化通过自动装箱与自动拆箱完成。   
Integer x = 2; //自动装箱 Integer x = Integer.valueOf(2)  
int y = x ;  //自动拆箱 y = x.intValue()     

引用类型的变量保存引用值，存储在Java栈空间中。引用值指向具体对象的地址，对象存储在Java堆中。 
基础数据类型缓存池
	在一定范围内的基本类型会将其包装对象存储在缓冲池里面，等待复用。
●  new Integer(123)  每次使用都会在堆中创建一个新的对象 
●  Integer.valueOf(123) 每次会使用Integer常量池中的对象，所有引用都指向这一个对象。 

	在Java8中，Integer缓冲池的范围是-128-127。但是IntegerCache的上限是可以调整的，可以在JVM初始化的时候通过java.lang.IntegerCache.high来设置。
基本类型对应的缓冲池如下： 
  ○ boolean:  false和true
  ○ byte: 所有的字节数值对象
  ○ short:  -128 and 127
  ○ int: -128 and 127
  ○ char: \u0000 to \u007F


String
String被声明为final，即每一个String都是一个常量，不可变 。 在JDK1.8中，String通过char[]存储数据。  而在JDK9中通过byte[]存储数据。

public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    //可以看到 value[]就是静态的，所以String也是静态的
    private final char value[]; 
}

String常量化的好处
String常量化的一大原因是String被经常使用，例如用作HashMap中的key
1. String变为常量，使得哈希值可以存储 ，不用多次计算
2. 常量使得可以存在String Pool,通过String Pool实现字符串常量的复用。
3. 线程安全，常量属性导致String不可被修改，所以是线程安全的。

String/StringBuffer/StringBuilder
● String不可变，而StringBuffer和StringBuilder可以改变（通过改变自身char[]）
● String和StringBuffer是线程安全的，StringBuffer内部通过synchronized实现同步。而StringBuilder线程不安全。


StringPool 字符串常量池
字符串常量池保存所有的字符串字面量literal strings。
这些字面量在编译的时候就确定下来。字符串有两种方式添加到StringPool中：
● String str = "abc"; 通过字面量初始化的形式
● string.intern()方法可以在运行过程中添加字符串到StringPool
调用string.intern()的时候，如果StringPool中存在一个字符串对象和这个字符串的value相等，就会返回StringPool中对应字符串对象的引用。 如果不存在这个字符串就添加，然后返回。

String s1 = "abc"; //字面量s1在编译的时候就会被放入StringPool
String s2 = "abc"; //由于是字面量，发现StringPool有了，直接传给引用
System.out.println(s1 == s2); //true
String s3 = new String("abc"); //创建两个对象：常量池、堆
String s4 = new String("abc"); 
System.out.println(s3 == s4); //false
String s5 = s3.intern(); //把s3放到StringPool,这个时候字符串常量池中有new String("abc")带来的字面量存档，所以直接返回了
String s6 = s4.intern(); //添加的时候发现有了，直接返回
System.out.println(s5 == s6); //true
System.out.println(s1 == s5); //true

在JDK1.7之前，所有常量池（包括StringPool）都存储在永久代 。
但JDK1.7+中StringPool被转移到了Java Heap中。
new String("abc")
new String("abc")会同时创建两个对象，分别在字符串常量池和Java堆 ，
但是这两个String对象的char[] value指向同一个对象 。
1. 由于"abc"是字面量，所以在编译的时候，"abc"会被存储到StringPool中
2. 而new String()会创建一个对象在Java堆中

public class NewStringTest {
    public static void main(String[] args) {
        String s = new String("abc");
    }
}
使用    javap -verbose 进行反编译，得到以下内容：
 
Constant pool:
 
   #2 = Class              #18            // java/lang/String
   #3 = String             #19            // abc
 
  #18 = Utf8               java/lang/String
  #19 = Utf8               abc
 

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: new           #2                  // class java/lang/String
         3: dup
         4: ldc           #3                  // String abc
         6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
         9: astore_1
 

在Constant Pool 中，#19 存储着字符串字面量"abc"，#3 是 String Pool的字符串对象，它指向 #19 这个字符串字面量。在 main 方法中，0: 行使用 new #2 在堆中创建一个字符串对象，并且使用 ldc #3 将 String Pool 中的字符串对象作为 String 构造函数的参数。
以下是String 构造函数的源码，可以看到，在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 数组内容，而是都会指向同一个 value 数组。

public String(String original) {
    this.value = original.value; //value数组直接指过去
    this.hash = original.hash;  //hash也直接指过去
}


隐式类型转化:

可以发现精确度小的基本类型可以隐式地转化为大精度类型。
但是高精度不能隐式转换为低精度类型 ，因为这会使得精度降低 。
    int i = 1;
    long l = i ;
    System.out.println(l);
    float f = 1.0f;
    double d = f;
    System.out.println(d);
    1
    1.0
 
```

### 运算

```

Java二元运算隐式提升规则
● 整数运算
  a. 两个操作数至少有一个为long , 结果为long
  b. 除此之外，结果一定为int
● 浮点数计算
  a. 两个操作数至少一个为double，结果为double
  b. 除此之外，结果一定为float

参数传递
	Java方法的参数传递是值传递 。Java没有引用传递。


在Java中，运算符的优先级顺序是：()> 单目运算符 > 算数运算符 > 逻辑运算符 > 条件运算符 > 赋值运算符
1	（）
2	+（正） -（负） ++ -- ！ 
3	* / % 
4	+ -
5	< <= > >=
6	== !=
7	& | ^ 逻辑运算符
8	&& ||  条件运算符
10	? :
11	= += -= *= /= %=

```
### 关键字
```

final —— 常量
位于常量池（常量池在方法区中）中，在JDK1.7位于永久代，JDK1.8之后位于元空间（方法区新的实现）。 

修饰变量
● 对于基本类型变量（值变量），final使得值不变
● 对于引用类型变量，final使得引用值不变 ，但是具体引用的对象是可以修改的。

修饰方法
	声明方法不能被子类重写。private方法会被隐式地指定为final，子类中与父类private方法重名的方法都是在子类中定义的新方法 ，而final指定的方法重写会报错
修饰类
	声明 类不允许被继承 。继承也会报错。

static —— 静态变量
●  静态变量（类变量） ：即类变量 。 这个变量属于类 ，类的所有实例都可以共享这个变量，可以直接通过类名来访问。static变量是类变量，所以位于类的Class对象中，在JDK1.6中,Class对象位于方法区（永久代），JDK1.7中随着Class对象一起转移到了堆中 。 

实例变量：实例对象属于实例（对象），每创建一个对象都会创建一个新的实例变量 ，生命周期与该实例相同，位于堆 。 
class Obj {
    public static int value = 0 ; //静态变量
    private int param; //实例变量
}

静态内部类
	非静态内部类依赖于外部类的实例，也就是说需要先创建外部类实例，才能用这个实例去创建非静态内部类。而静态内部类不需要。静态内部类和非静态内部类一样，都是只有在被调用的时候才正式加载。

public class OuterClass {

    class InnerClass {
    }

    static class StaticInnerClass {
    }

    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        StaticInnerClass staticInnerClass = new StaticInnerClass();
    }
}

语句初始化的顺序
public class Test {
    public static int staticVar = 100 ;
    private int objVar = 10;
    {
        System.out.println(objVar);
        System.out.println("实例语句块");
    }
    static {
        System.out.println(staticVar);
        System.out.println("静态语句块");
    }
    public Test() {
        System.out.println("构造函数");
    }
    public static void main(String[] args) {
        Test test = new Test();
    }
}
1. 静态变量和静态语句块(根据先后顺序)
2. 实例变量与实例语句块
3. 构造函数

继承条件下的初始化顺序
1. 父类的静态变量与静态语句块
2. 子类的静态变量与静态语句块
3. 父类的实例变量与实例语句块
4. 父类的构造函数
5. 子类的实例变量与实例语句块
6. 子类的构造函数
实例变量和实例语句块在构造方法之前 ，因为构造方法可能会对实例变量和实例做处理。




```

### Object通用方法
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

clone()
	clone()是Object的protected方法，如果某类没有显式重写clone()方法，其他类就不能调用这个类实例的clone()。需要注意的是重写clone()方法需要实现Cloneable接口。
浅拷贝（Object的默认实现）
	被克隆对象o1和克隆对象o2是两个引用不同的对象（clone()新建了对象） ， 但是对象中引用类型的变量是同一个引用 。类似于public String(String original)。


```

### 继承
```
访问权限
	Java中有三个访问权限修饰符，public、protected、private。如果不使用修饰符，默认权限为包级可见 。
● 类可见 : 其他类可以用这个类创建实例对象
● 成员可见：可以用实例对象访问到这个成员属性

1. public: 整个项目都可以使用这个类的实例访问到
2. protected :  类本身、包、子类范围可以访问到
3. 没有修饰符（friendly）： 类本身和包级别可以访问到
4. private : 只有类本身可以访问


子类实例与父类的关系  
我们发现在创建子类实例的时候，会调用父类的构造方法，同时子类会继承父类的非私有属性。  
但是实质上创建子类的时候并没有创建父类实例，子类只是单独会有一个空间用来存储从父类继承来的属性，  
这些属性是属于子类实例的，而不是父类，父类实例并没有被生成。  

抽象类与接口   
抽象类  
	抽象类和抽象方法都用关键字abstract修饰，只要有一个方法是抽象方法，   
    这个类就是抽象类（并且必须声明为抽象类） 。 抽象类不能被实例化，只能被继承 。  
接口  
	接口是抽象类的延伸。  
● JDK1.8之前接口中的方法只能为抽象方法。不能有任何的方法实现 。   
● JDK1.8为接口添加了默认方法 ，  可以让实现类不用实现该方法 。  

接口的属性和方法默认为public，不许为protected和private，否则报错。
抽象类与接口的比较
● 从设计的角度看：抽象类是IS A的关系，与实现类是父子关系。接口是LIKE的关系，只限定了方法的实现。
● 使用上: 一个类可以实现多个接口，但只能继承一个父类
● 接口的成员修饰符有要求（默认或者为public），而抽象类没有
 
```

### 重写与重载

```
重写
	存在于继承体系中。 指子类实现了一个与父类在方法声明上完全相同的方法（final/private方法不可被重写） 。
	在调用一个类的方法的时候，主要有以下四个步骤：
1.  this.func(this)
	如果本类有实现，就调用本类的方法。 
2.  super.func(this)
	本类没有实现，看看是否是重写父类的方法 。 
3.  this.func(super)
	对参数进行转型，转成父类之后看是否有对应的方法。 
4.  super.func(super)
	操作类和参数类向上转型
即先后转型执行类和参数类。 重写主要通过动态单分派实现。


重载
	存在于同一个类中，指一个方法的名称与类中其他方法的名称相同，但是参数类型、参数个数、参数顺序等至少有一个不同。重载通过静态多分派实现。
	返回值不同而其他声明相同，不是重载 ，因为单纯从其他的因素来看无法分辨出方法是哪个，有歧义。而且这种声明是不被允许的 。


```

### 反射
```
每个类都有一个 Class对象，包含了与类有关的信息 。当编译一个新类的时候，会产生一个.class文件，该文件内容保存着Class对象。类加载即Class对象的加载，类加载在程序开始运行时进行  。可以通过Class.forName("com.mysql.jdbc.Driver")这种方式来加载一个类。

Class c = Main.class;
Class c1 = Class.forName("pac.Main");
System.out.println(c == c1);
//true

反射允许程序在执行期间借助于Reflection API取得任何类的内部信息，并能够直接操作任意对象的内部属性及方法。java.lang.reflect主要包含了以下三各类：
● Field ： 可以使用get()、set()读取和修改对象的字段
● Method:  可以使用invoke()方法调用类的方法
● Constructor: 可以使用Constructor的newInstance()创建新实例（对象）

反射的缺点
● 动态解析导致效率低，效率显著低于普通代码
● 不安全

Class类
1. Class本身是一个类
2. Class对象只能由系统建立
3. 一个加载的类在JVM中只会有一个Class实例
4. 一个Class对象对应的是一个加载到JVM中的class文件
5. 每个类的实例都会记得自己是由哪个Class实例所生成
6. 通过Class可以完整地得到一个类中所有被加载的结构
7. Class类是Reflection的根源，针对任何想象动态加载、运行的类，唯有先获得相应的Class对象


//1.若已知具体的类，通过类的Class属性获取。 该方法最安全可靠，程序性能最高
Class c = Person.class;
//2.已知某个类的实例，调用实例的getClass()方法获取
Class c = person.getClass();
//3.已知一个类的全名，通过Class类的静态法方法forName()获取，可能会抛出ClassNotFoundException
Class c = Class.forName("com.liu.User");
//4.内置基本数据类型可以直接用类名.Type
Class c = Integer.Type
//5.利用ClassLoader
public class Reflection {
    public static void main(String[] args) {
        //通过反射获取类的Class对象
        try {
            //通过完整包名获得
            Class c1 =  Class.forName("com.liu.User");
            //通过类的class属性获得
            Class c2 =  User.class;
            //通过实例的getClass（）方法获得
            User user = new User();
            Class c3 = user.getClass();
            //基本数据类型的TYPE
            Class c4 = Integer.TYPE;

            //获得父类Class
            Class father = c1.getSuperclass();
            System.out.println(father);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

public class Reflection1 {
    public static void main(String[] args) {
        Class c1 = Object.class; //类的Class
        Class c2 = Comparable.class; //接口的Class
        Class c3 = String[].class; // 一维数组的Class
        Class c4 = int[][].class ; // 二维数组的Class
        Class c5 = Override.class; // 注解的Class
        Class c6 = ElementType.class; // 枚举的Class
        Class c7 = Integer.class ; //基本数据类型
        Class c8 = void.class ; //void的Class
        Class c9 = Class.class ; //Class的Class

        System.out.println(c1);
        System.out.println(c2);
        System.out.println(c3);
        System.out.println(c4);
        System.out.println(c5);
        System.out.println(c6);
        System.out.println(c7);
        System.out.println(c8);
        System.out.println(c9);

    }
}

//输出结果 ：
class java.lang.Object
interface java.lang.Comparable
class [Ljava.lang.String;
class [[I
interface java.lang.Override
class java.lang.annotation.ElementType
class java.lang.Integer
void
class java.lang.Class


public class GetInfoOfClass {
    private int age ;
    private String name;
    public int field;

    public GetInfoOfClass(int age, String name, int field) {
        this.age = age;
        this.name = name;
        this.field = field;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getField() {
        return field;
    }

    public void setField(int field) {
        this.field = field;
    }

    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, NoSuchMethodException {
        Class c1 = GetInfoOfClass.class;
        //获得类的名字 包名 + 类名
        System.out.println(c1.getName());
        //获得类的简单名字  类名
        System.out.println(c1.getSimpleName());
        //获得类的属性
        Field[] fields = c1.getFields();  //getFields() 只能获得 public属性
        for(Field f : fields)
        {
            System.out.println(f);
        }
        Field[] fields1 = c1.getDeclaredFields(); //getDeclaredFields() 可以获得所有属性
        for(Field f : fields1)
        {
            System.out.println(f);
        }
        //获得指定属性的值
        Field name = c1.getDeclaredField("name");
        System.out.println(name);
        //获得类的方法
        Method[] method1 = c1.getMethods();
        for(Method method : method1)
        {
            System.out.println("1:"+method); //获得本类及其父类的所有public方法
        }
        Method[] methods = c1.getDeclaredMethods(); //获得本类的所有方法
        for(Method method : methods)
        {
            System.out.println("2:"+method);
        }
        //获得指定方法
        Method method = c1.getMethod("getName",null);
        Method methodd = c1.getMethod("setName",String.class);
        System.out.println(method);
        System.out.println(methodd);

        //获得构造器
        Constructor[] constructors = c1.getConstructors();
        for(Constructor c : constructors)
        {
            System.out.println(c);
        }
        Constructor[] constructors1 = c1.getDeclaredConstructors();
        for(Constructor c : constructors1)
        {
            System.out.println(c);
        }
        //获取指定的构造器
        Constructor c = c1.getConstructor(int.class,String.class,int.class);
        System.out.println(c);
    }
}

动态创建对象
1.  通过Class 对象的newInstance()方法 
  ○  类必须要有一个无参构造器 
  ○  类的访问权限需要足够 

  //1.获取一个Class对象
        Class c1 = Class.forName("com.liu.User");
 //2.构造一个对象
 	    User user = (User) c1.newInstance(); //调用无参构造器
        System.out.println(user);

2. 通过Class类的getDeclaredConstructor(Class...parameterTypes)取得有参构造器。向构造器传入一个数组，里面包含了构造实例的各个参数。最后通过Constructor实例化对象 

public class ClassCreateClass {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {
        //1.获取一个Class对象
        Class c1 = Class.forName("com.liu.User");
        //2.构造一个对象
        User user = (User) c1.newInstance(); //调用无参构造器
        System.out.println(user);
        //3.通过构造器创建对象
        Constructor constructor = c1.getDeclaredConstructor(String.class,int.class,int.class);
        User user2 = (User)constructor.newInstance("陆源东",10,20);
        System.out.println(user2);
        //4.通过反射获取一个方法
        Method setName = c1.getDeclaredMethod("setName",String.class);
        User user3 = (User) c1.newInstance();
        setName.invoke(user3,"名字");
        /*
        invoke() :
        激活函数
        {对象 ： “方法的参数”}
        
        private 修饰的方法不能够直接访问
        需要 setAccessible(true) 开启权限的安全检测
        
         */
        //5.通过反射操纵属性
        User user4 = (User)c1.newInstance();
        Field name = c1.getDeclaredField("name");
        /*
        private 属性不能够直接访问
        需要 setAccessible(true) 开启权限的安全检测
         */
        name.setAccessible(true);
        name.set(user,"东"); 
        /*
        set()方法
        属性设置方法
        {对象 ： “属性值”}
         */
        System.out.println(user4.getName());
    }
}

反射操作注解
ORM
Object relationship Mapping    ——> 对象关系映射

//反射操作注解
public class OperationAnnotation {

    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException {
        Class c1 = Class.forName("com.liu.Student");
        //通过反射获得注解
        Annotation[] annotations = c1.getAnnotations();
        for(Annotation a : annotations)
        {
            System.out.println(a);
        }
        //获得注解value的值
        //获取指定的注解
        Annotation annotation = c1.getAnnotation(Tableliu.class);
        Tableliu tableliu = (Tableliu)annotation;
        String value = tableliu.value();
        System.out.println(value);
        //获得属性指定的注解
        Field field = c1.getDeclaredField("name");
        Fieldliu fieldliu = field.getAnnotation(Fieldliu.class);
        System.out.println(fieldliu.columnName());
        System.out.println(fieldliu.type());
        System.out.println(fieldliu.length());
    }
}
@Tableliu("db_student")
class Student{
    @Fieldliu(columnName = "db_id",type="int",length = 10)
    private int id ;
    @Fieldliu(columnName = "db_id",type="int",length = 10)
    private int age;
    @Fieldliu(columnName = "db_id",type="varchar",length = 10)
    private String name;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", age=" + age +
                ", name='" + name + '\'' +
                '}';
    }

    public Student(int id, int age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }
}
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface Tableliu{
    String value();
}

//属性的注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface Fieldliu{
    String columnName();
    String type();
    int length();
}




```

### 注解
```
什么是注解（Annotation）？
● 注解不是程序，但可以对程序做出解释
● 可以被其他程序（如编译器）读取
Annotation的格式？
"@注释名"，例如 ：
@SuppressWarnings(value="unchecked")
Annotation 在哪里使用？
可以附加在 package、class、method、field等上面，通过反射机制编程实现元素的访问。

//重写注解
@Override
public String toString() {
    return super.toString();
}

内置注解
@Override : 重写注解
@Deprecated : 不鼓励使用注解
@SuppressWarnings : 用来抑制编译时的警告信息，需要参数如
   @SuppressWarnings("all")
   @SuppressWarning("unchecked")
   @SuppressWarning("unchecked","deprecation")
    ....
    
public class Test01 {
    //重写注解
    @Override
    public String toString() {
        return super.toString();
    }

    //不推荐使用，但是可以使用
    @Deprecated
    public void test(){}
    
    //镇压警告 ： 抑制编译警告信息
    @SuppressWarnings("all")
    public void warn(){}
    
}


元注解
作用
负责注解其他注解，Java定义了4个元Annotation
1. @Target: 描述注解的范围（用在什么地方）
2. @Retention : 表示需要在什么级别保存该注解信息，描述注解的生命周期（SOURCE<CLASS<RUNTIME）
3. @Document  : 说明该注解将被包含在javadoc中
4. @Inherited : 说明子类可以继承父类的该注解


通过元注解自定义一个注解
//定义一个注解
@Target(value = {ElementType.METHOD,ElementType.TYPE}) 
//Target注解表示注解可以用在什么地方，value传递一个数据。 可以通过 ElementType类查看有哪些可用参数
@Retention(value = RetentionPolicy.RUNTIME)
//Retention注解表示在什么地方有效,RUNTIME>CLASS>SOURCE ,可以通过 RetentionPolicy类查看有哪些可用参数
@Documented
@Inherited
@interface MyAnnotation{}


● @interface  用来声明一个注解，格式 @interface{定义内容}
● 其中的每一个方法实际上是声明了一个配置参数
● 方法的名称就是参数的名称
● 返回值类型就是参数的类型，返回值只能是基本类型Class、String、enum
● 可以通过default来声明参数的默认值
● 如果只有一个参数成员，一般参数名为value
● 注解元素必须要有值，定义注解元素的时候，经常使用空字符串，0作为默认值

//自定义注解
public class Test03 {
    @MyAnnotation1(age=1,name="213")
    public void test(){}
}

@Target({ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation1{
    //注解的参数 ： 参数类型 + 参数名();
    String name() default "";
    int age() default 0;
    int id() default -1; //如果默认值为-1，代表不存在
    String[] schoolds() default {"1","2","3"};
}




```
### Error与Exception
```

Error
Error是应用程序无法解决的问题，问题通常发生在JVM层面，例如StackOverFlowError、VirtualMachineError、OutOfMemoryError等。
Error无法被try-catch捕获，一旦这类程序发生，程序惠立刻终止，无法自行恢复。
Exception
Exception异常是程序本身可以解决或者忽略的问题，异常并不一定导致程序终止。
	Java的异常分为可检出异常checked exception和不可检出异常unchecked exception。
	可检出异常虽然是异常情况，但是在一定程度上它是可预计的，例如FileNotFoundException：只要有文件操作，那就有可能该文件不存在，所以就有可能可以预估出这个异常 。 因此Java编译器会强制要求处理此类异常，否则编译不通过。通过try-catch语句捕获或者throws抛出 。除了RuntimeException及其子类，其他的Exception及其子类都是可检出异常 。
	不可检出异常即编译器不要求强制处理的异常 ： 包括运行时异常RuntimeException和错误Error 。这一类异常之所以不可检出，是因为无法对这些错误进行预测，例如当程序中出现了1/0或者OOM，编译器在编译的时候是不会知道会发现这个异常的，只有在运行中这个异常才会暴露出来。
	需要注意的是ClassCastExceptison，这个是程序运行期间的强制代码转换 ，例如A a = (A)b。但是父子类的相互赋值是在编译期间检查的(来自《深入理解Java虚拟机》) 。 例如Java编译器不允许父类赋值子类 。

//如果直接抛出异常 throws ，程序会直接中断
public class Main3{
    public static void main(String[] args) throws IOException {
        //对文件的读写操作会引发IOException(FileNotFoundException)
        InputStream stream = new FileInputStream(new File("./213.txt"));
        stream.read();
        System.out.println("hello");
    }
}
//通过try-catch做处理的话，异常就不会中断程序，程序的后续代码依旧可以正常运行
public class Main3{
    public static void main(String[] args) {
        InputStream stream = null;
        try {
            stream = new FileInputStream(new File("./213.txt"));
            stream.read();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println("hello");
    }
}
```
### 泛型

```
泛型概述
	泛型，即参数化类型。顾名思义，就是将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。
	泛型只在编译期间有效，在编译期间泛型会被用作检查对应数据结构是否只包含泛型类型。在编译期间过了之后，泛型就会被移除 。

public class Main{
    public static void main(String[] args) throws ClassNotFoundException {
        List<String> stringList = new ArrayList();
        List<Integer> integerList = new ArrayList<>();
        stringList.add("string");
        integerList.add(1);
        System.out.println(stringList.getClass() == integerList.getClass());
    }
}
//true 可以发现在编译过之后，两个数据结构仍然是同类

泛型类
	将泛型用于类的定义 ，使得用同一套类实现操作不同的类 。典型为List、Set、Map等容器类。

package pac;
public class Main{
    public static void main(String[] args) throws ClassNotFoundException {
        //可以发现泛型类Generic可以用自己的一套定义 操作String、Integer等不同的类
        Generic<Integer> integerGeneric = new Generic<>(100);
        System.out.println(integerGeneric.getKey());
        Generic<String> stringGeneric = new Generic<>("string");
        System.out.println(stringGeneric.getKey());
        //不使用泛型限定
        Generic generic = new Generic(1.2);
        System.out.println(generic.getKey());
    }
}
class Generic<T>{
    //成员变量类型为T
    private T key;
    //构造函数
    public Generic(T key) {
        this.key = key;
    }
    //方法
    public T getKey() {
        return key;
    }
}

泛型接口
	作用与泛型类相似，可以以一套接口的定义使得实现类支持多种类的操作

public class Main implements Interfac<Integer>{
    public static void main(String[] args) throws ClassNotFoundException {
        Interfac<Integer> i = new Main();
        i.func(100);
    }
    @Override
    public void func(Integer key) {
        System.out.println(key);
    }
}
interface Interfac<T>{
    void func(T key);
}


泛型通配符
	在使用了泛型之后出现了一个问题：不同类型形参的泛型作为函数的参数是不等价的，尽管同一实现在编译之后是相同的，但是这样的使用方法会出现错误cannot be applied to 泛型<T>。

public class Main{
    public static void main(String[] args) throws ClassNotFoundException {
        Generic<Integer> integerGeneric = new Generic<>(1);
        Generic<String> stringGeneric = new Generic<>("string");
        func(integerGeneric);
        func(stringGeneric);
    }
    //使用Generic<?>就可以适配不同的类型参数 类似于使用父类来表示不同的子类
    public static void func(Generic<?> generic) {
        System.out.println(generic.getKey());
    }
}
/*
1
string
*/


泛型方法
	在调用方法的时候指定泛型的类型。泛型方法不是泛型类中的方法，泛型类中泛型方法的泛型实在实例化泛型类的时候指定的。在形参上直接使用泛型是不可以的，因为类不知道T是什么。
                                
	正确案例如下所示，需要在返回值之前声明泛型 。只要在方法中声明了泛型，这个方法就是个泛型方法，这个声明的泛型与类无关（即使与泛型类的泛型重名）。


public class Main{
    public static void main(String[] args) throws ClassNotFoundException {
        Generic<Integer> generic = new Generic<>(100);
        funcT(generic); //100
    }
    //在static 和 void 之间需要声明这个函数所使用的泛型 <T>
    public static <T> void funcT(Generic<T> container) {
        System.out.println(container.getKey());
    }
}
class Generic<T>{
    //成员变量类型为T
    private T key;
    //构造函数
    public Generic(T key) {
        this.key = key;
    }
    //方法
    public T getKey() {
        return key;
    }
}

泛型与静态方法
	静态方法无法访问类上的泛型，因为静态方法是类初始化的时候初始化的，而泛型的类型要到实例建立的时候才确定。

因此如果静态方法要使用泛型，就必须使用泛型方法 。 即将静态方法变为泛型方法 。

泛型限制条件
	在使用泛型的时候，可以为泛型设置限定条件

public class Main<T>{
    public static void main(String[] args) throws ClassNotFoundException {
    }
    //? extends Object Generic的类型参数必须继承自Object
    public static void func(Generic<? extends Object> generic) {
        System.out.println(generic.getKey());
    }
}
//Generic类可以操作的类必须继承自Object
class Generic<T extends Object>{
    //成员变量类型为T
    private T key;
    //构造函数
    public Generic(T key) {
        this.key = key;
    }
    //方法
    public T getKey() {
        return key;
    }
    public static <T> void func(T t) {
        
    }
}

泛型数组

List<Integer>[] lists = new ArrayList<Integer>[10]; //这种方式是不被允许的
List<Integer>[] lists = new ArrayList[10]; //这种方式是可以的
List<?>[] lists = new List<?>[10]; //不确定类型泛型可以通过通配符实现


```