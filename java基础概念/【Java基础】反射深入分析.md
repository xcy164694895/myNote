### 概述 ###

Java反射机制是指在Java运行状态中，对于任何一个类，都可以获得这个类的所有属性和方法；对于给定的一个对象，都能够调用它的任意一个属性和方法。这种动态调用类的内容以及动态调用对象的方法称为反射机制。

反射机制(Reflection)是Java提供的一项较为高级的功能，它提供了一种动态功能，而此功能的体现在于通过反射机制相关的API就可以获取任何Java类的包括属性、方法、构造器、修饰符等信息。元素不必在JⅧ运行时进行确定，反射可以使得它们在运行时动态地进行创建或调用。反射技术在中间件领域应用得较多。

### RTTI和反射 ###

很多时候我们的程序可能需要在运行时识别对象和类的信息，比如多态就是基于运行时环境动态的判断实际引用的对象信息。在运行时识别对象和类的信息主要有两种方式：1、RTTI，它假定我们在编译期已经识别了所有对象类型。2、反射机制，它会在运行时发现和使用类的信息。

其实反射机制并没有什么神奇之处，当通过反射与一个位置对象打交道时，JVM只简单的检查这个对象，看它属于哪个特定的类。因此，那个类的`.class`对于JVM来说必须是可以获取的，要么在本地机器上，要么可以通过网络获取到。所以对于RTTI和反射之间的真正区别在于：
+ RTTI，编译器在编译时打开和检查`.class`文件
+ 反射，运行时打开和检查`.class` 文件

### 反射获取class对象三种方式 ###

#### 1、通过对象获取 #### 
```
        Student student = new Student();
        Class clz = student.getClass();
        System.out.println(clz.getName());

```

#### 2、通过类名获取 ####

```
    Class clz2 = Student.class;
    System.out.println(clz2.getName());
```

#### 3、通过全类名获取 ####

该方法通过类的全路径名获取class对象，如果通过全路径名找不到对应的类，那么系统会抛出ClassNotFoundException异常。

```
try {
            Class clz3 = Class.forName("com.xcy.javaBasic.Student");
            System.out.println(clz3.getName());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

```

三种创建class对象的方式，各有其使用场景，但是第三种使用更加普遍，因为第一种需要创建实例，再用反射就比较多余了。第二种，只适合在编译前就知道操作的 Class。
### 通过反射创建类对象的两种方式 ###

通过反射创建类对象主要有两种方式： 
    
 + 通过Class对象的newInstance()方法
 + 通过Constructor对象的newInstance()方法
 
 #### 通过Class对象的newInstance()方法 ####
    
```
try {
            Class clz3 = Class.forName("com.xcy.javaBasic.Student");
            System.out.println(clz3.getName());
            Object obj = clz3.newInstance();
            Student student1  = (Student) obj;
            System.out.println(student1);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        }

```

### 通过Constructor对象的newInstance()方法 ####

```
try {
            Class clz3 = Class.forName("com.xcy.javaBasic.Student");
            System.out.println(clz3.getName());
            Constructor constructor = clz3.getConstructor();
            Student student1 = (Student) constructor.newInstance();
            System.out.println(student1);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }  catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

```


通过Constructor对象创建类对象可以选择特定的构造器，而通过class创建对象则只能使用默认的无参构造方法。


### 通过反射获取类的属性 ###

我们先创建一个Student的类，其中包含各种属性如下：
```
public class Student implements Serializable {

    private int id;

    public String name;

    String sex;

    protected int age;

    private  String grade;

    public String getGrade() {
        return grade;
    }

    public void setGrade(String grade) {
        this.grade = grade;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                ", age=" + age +
                ", grade='" + grade + '\'' +
                '}';
    }


}


```
如下是通过反射获取实体，并使用其中的属性的测试用例：
```
Class clz4 = Student.class;
        System.out.println("*******************************获取所有公有字段************************");
        Field[] fields = clz4.getFields();
        for (Field field : fields){
            System.out.println(field);
        }
        System.out.println("*******************************获取所有字段*****************************");
        fields = clz4.getDeclaredFields();
        for (Field field:fields){
            System.out.println(field);
        }
        System.out.println("*******************************获取公有字段并调用*****************************");
        try {
            Field field = clz4.getField("name");
            System.out.println(field);
            Object obj = clz4.getConstructor().newInstance();
            field.set(obj,"xcy");
            Student student = (Student) obj;
            System.out.println(student);
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println("*******************************获取私有字段并调用*****************************");
        try {
            Field field = clz4.getDeclaredField("grade");
            System.out.println(field);
            Object obj = clz4.getConstructor().newInstance();
            //私有属性必须解除私有限定
            field.setAccessible(true);
            field.set(obj,"100");
            //关闭解除
            field.setAccessible(false);
            Student student = (Student) obj;
            System.out.println("验证成绩："+ student.getGrade());
        } catch (Exception e) {
            e.printStackTrace();
        }
```
得到的结果如下：
```
*******************************获取所有公有字段************************
public java.lang.String com.xcy.javaBasic.Student.name
*******************************获取所有字段*****************************
private int com.xcy.javaBasic.Student.id
public java.lang.String com.xcy.javaBasic.Student.name
java.lang.String com.xcy.javaBasic.Student.sex
protected int com.xcy.javaBasic.Student.age
private java.lang.String com.xcy.javaBasic.Student.grade
*******************************获取公有字段并调用*****************************
public java.lang.String com.xcy.javaBasic.Student.name
Student{id=0, name='xcy', sex='null', age=0, grade='null'}
*******************************获取私有字段并调用*****************************
private java.lang.String com.xcy.javaBasic.Student.grade
验证成绩：100

```


### 通过反射获取构造方法 ###
```
Class clz = Student.class;
        System.out.println("*******************************************获取所有公有构造器*************************************");
        Constructor[] constructors = clz.getConstructors();
        for (Constructor c : constructors){
            System.out.println(c);
        }
        System.out.println("*******************************************获取所有构造器(包含：私有，受保护，默认，公有)*************************************");
        constructors = clz.getDeclaredConstructors();
        for (Constructor c : constructors){
            System.out.println(c);
        }
        System.out.println("*******************************************获取公有无参构造器*************************************");
        try {
            Constructor constructor = clz.getConstructor(null);
            System.out.println("constructor : "+ constructor);
            Object obj = constructor.newInstance();
            System.out.println("obj:" + obj);
            Student student = (Student) obj;
            System.out.println("student:" + student);
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println("*******************************************获取私有构造器，并调用*************************************");
        try {
            Constructor constructor = clz.getDeclaredConstructor(int.class,String.class,String.class);
            System.out.println("constructor : " + constructor);
            constructor.setAccessible(true);
            Object obj = constructor.newInstance(1,"xcy","xxx");
            System.out.println("obj : "+ obj);
            Student student = (Student)obj;
            System.out.println("student :　" +student);
        } catch (Exception e) {
            e.printStackTrace();
        }

```

执行结果如下：
```
*******************************************获取所有公有构造器*************************************
public com.xcy.javaBasic.Student()
public com.xcy.javaBasic.Student(java.lang.String,int)
public com.xcy.javaBasic.Student(java.lang.String)
*******************************************获取所有构造器(包含：私有，受保护，默认，公有)*************************************
public com.xcy.javaBasic.Student()
private com.xcy.javaBasic.Student(int,java.lang.String,java.lang.String)
public com.xcy.javaBasic.Student(java.lang.String,int)
protected com.xcy.javaBasic.Student(int)
public com.xcy.javaBasic.Student(java.lang.String)
*******************************************获取公有无参构造器*************************************
constructor : public com.xcy.javaBasic.Student()
调用无参构造器！
obj:Student{id=0, name='null', sex='null', age=0, grade='null'}
student:Student{id=0, name='null', sex='null', age=0, grade='null'}
*******************************************获取私有构造器，并调用*************************************
constructor : private com.xcy.javaBasic.Student(int,java.lang.String,java.lang.String)
私有构造器，id:1name :xcysex : xxx
obj : Student{id=1, name='xcy', sex='xxx', age=0, grade='null'}
student :　Student{id=1, name='xcy', sex='xxx', age=0, grade='null'}

```


### 通过反射获取成员方法 ###

首先在student中创建各种不同的成员方法
```
public class Student implements Serializable {

    private int id;

    public String name;

    String sex;

    protected int age;

    private  String grade;

    public String getGrade() {
        return grade;
    }

    public void setGrade(String grade) {
        this.grade = grade;
    }

`   ……

    public void fun1(String s){
        System.out.println("调用了公有方法，String参数的fun1() s: " + s);
    }

    protected void fun2(){
        System.out.println("调用了：受保护的，无参的show2()");
    }
    void fun3(){
        System.out.println("调用了：默认的，无参的show3()");
    }
    private String fun4(int age){
        System.out.println("调用了，私有的，并且有返回值的，int参数的show4(): age = " + age);
        return "abcd";
    }
}


```

然后通过反射调用成员方法，如下：
```
//1.获取Class对象
        Class stuClass = Student.class;
        //2.获取所有公有方法
        System.out.println("***************获取所有的公有方法*******************");
        stuClass.getMethods();
        Method[] methodArray = stuClass.getMethods();
        for(Method m : methodArray){
            System.out.println(m);
        }
        System.out.println("***************获取所有的方法，包括私有的*******************");
        methodArray = stuClass.getDeclaredMethods();
        for(Method m : methodArray){
            System.out.println(m);
        }
        System.out.println("***************获取公有的show1()方法*******************");
        Method m  = stuClass.getMethod("fun1", String.class);
        System.out.println(m);
        //实例化一个Student对象
        Object obj = stuClass.getConstructor().newInstance();
        m.invoke(obj, "xcy");

        System.out.println("***************获取私有的fun4()方法******************");
        m = stuClass.getDeclaredMethod("fun4", int.class);
        System.out.println(m);
        m.setAccessible(true);//解除私有限定
        Object result = m.invoke(obj, 20);//需要两个参数，一个是要调用的对象（获取有反射），一个是实参
        System.out.println("返回值：" + result);
    }
```