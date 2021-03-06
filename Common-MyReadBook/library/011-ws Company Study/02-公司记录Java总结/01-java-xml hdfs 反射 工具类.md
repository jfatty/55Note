## xml 文件读取

### xml 解析xml文件

```Java
package cn.netcommander.config;

import org.apache.commons.configuration.XMLConfiguration;
import org.apache.log4j.Logger;

import java.io.File;

/***
 * 解析配置
 *
 * @author xufanqi
 */
public class ParseConfig {

    public static final Logger logger = Logger.getLogger(ParseConfig.class);

    private static String XML_PATH = "config/config.xml";

    public static void parseXML(String filename) {

        try {
            XMLConfiguration parser = new XMLConfiguration();
            if (filename == null) {
                filename = XML_PATH;
            }
            parser.setFile(new File(filename));
            parser.load();

            Config.url = parser.getString("sys_setting.url");
            Config.username = parser.getString("sys_setting.username");
            Config.password = parser.getString("sys_setting.password");
            Config.intervalTime = parser.getString("sys_setting.intervalTime");
            Config.output = parser.getString("sys_setting.output");
            Config.hdfs = parser.getString("sys_setting.hdfs");
            Config.input = parser.getString("sys_setting.input");
            Config.local = parser.getString("sys_setting.local");

        } catch (Exception e) {
            logger.error("parse userinfo config file error : ", e);
        }
    }
}

```

## Java操作hdfs 文件

###
```Java
public static boolean writeJavaList(String path, List<String> results, String charsetName) {
        boolean flag = false;
        FileSystem fs = null;
        OutputStream os = null;
        BufferedWriter bw = null;
        OutputStreamWriter osWeiter = null;
        try {
            String target = path.substring(0, path.indexOf(".")) + ".tmp";
            logger.warn("target " + target);

            conf = new Configuration();
            fs = FileSystem.newInstance(URI.create(target), conf);
            os = fs.create(new Path(target)); //打开创建目标文件

            osWeiter = new OutputStreamWriter(os, charsetName);
            bw = new BufferedWriter(osWeiter);
            int num = 0;

            for (String list : results) {
                String line = list + "\n";
                bw.write(line);
                num++;
                if (num >= 500) {
                    bw.flush();
                    num = 0;
                }
            }
            bw.flush();

            flag = true;
        } catch (Exception e) {
            logger.error("", e);
        } finally {
            if (bw != null) {
                try {
                    bw.close();
                    bw = null;
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (osWeiter != null) {
                try {
                    osWeiter.close();
                    osWeiter = null;
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (os != null) {
                try {
                    os.close();
                    os = null;
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            if (fs != null) {
                try {
                    fs.close();
                    fs = null;
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return flag;
    }
```


## StringWriter

### 学习记录 StringReader StringWriter
[博主记录，StringReader、StringWriter的源码](https://blog.csdn.net/moonfish0607/article/details/78320363)

简单实用：
```Java
package StringIO;  

import java.io.StringReader;  
import java.io.StringWriter;  

public class StringIOTest {  
    public static void main(String[] args) {  
        try (StringReader sr = new StringReader("just a test~");  
                StringWriter sw = new StringWriter()) {  
            int c = -1;  
            while((c = sr.read()) != -1){  
                sw.write(c);  
            }  
            //这里展示了即使关闭了StringWriter流，但仍然能获取到数据，因为其close方法是一个空实现。  
            sw.close();  
            System.out.println(sw.getBuffer().toString());  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  

    }  
}  
```

## java 的反射
[简单博客1](https://blog.csdn.net/sinat_38259539/article/details/71799078)
反射的创建三种方式
```Java

/**
 * 获取Class对象的三种方式
 * 1 Object ——> getClass();
 * 2 任何数据类型（包括基本数据类型）都有一个“静态”的class属性
 * 3 通过Class类的静态方法：forName（String  className）(常用)
 *
 */  
public class Fanshe {  
    public static void main(String[] args) {  
        //第一种方式获取Class对象    
        Student stu1 = new Student();//这一new 产生一个Student对象，一个Class对象。  
        Class stuClass = stu1.getClass();//获取Class对象  
        System.out.println(stuClass.getName());  

        //第二种方式获取Class对象  
        Class stuClass2 = Student.class;  
        System.out.println(stuClass == stuClass2);//判断第一种方式获取的Class对象和第二种方式获取的是否是同一个  

        //第三种方式获取Class对象  
        try {  
            Class stuClass3 = Class.forName("fanshe.Student");//注意此字符串必须是真实路径，就是带包名的类路径，包名.类名  
            System.out.println(stuClass3 == stuClass2);//判断三种方式是否获取的是同一个Class对象  
        } catch (ClassNotFoundException e) {  
            e.printStackTrace();  
        }  

    }  
}

```

###  创建Student类

```Java
public class Student {  

    //---------------构造方法-------------------  
    //（默认的构造方法）  
    Student(String str){  
        System.out.println("(默认)的构造方法 s = " + str);  
    }  

    //无参构造方法  
    public Student(){  
        System.out.println("调用了公有、无参构造方法执行了。。。");  
    }  

    //有一个参数的构造方法  
    public Student(char name){  
        System.out.println("姓名：" + name);  
    }  

    //有多个参数的构造方法  
    public Student(String name ,int age){  
        System.out.println("姓名："+name+"年龄："+ age);//这的执行效率有问题，以后解决。  
    }  

    //受保护的构造方法  
    protected Student(boolean n){  
        System.out.println("受保护的构造方法 n = " + n);  
    }  

    //私有构造方法  
    private Student(int age){  
        System.out.println("私有的构造方法   年龄："+ age);  
    }  

}  
```

### 调用构造方法，调用的使用

```Java
import java.lang.reflect.Constructor;  


/*
 * 通过Class对象可以获取某个类中的：构造方法、成员变量、成员方法；并访问成员；
 *  
 * 1.获取构造方法：
 *      1).批量的方法：
 *          public Constructor[] getConstructors()：所有"公有的"构造方法
            public Constructor[] getDeclaredConstructors()：获取所有的构造方法(包括私有、受保护、默认、公有)

 *      2).获取单个的方法，并调用：
 *          public Constructor getConstructor(Class... parameterTypes):获取单个的"公有的"构造方法：
 *          public Constructor getDeclaredConstructor(Class... parameterTypes):获取"某个构造方法"可以是私有的，或受保护、默认、公有；
 *       
 *          调用构造方法：
 *          Constructor-->newInstance(Object... initargs)
 */  
public class Constructors {  

    public static void main(String[] args) throws Exception {  
        //1.加载Class对象  
        Class clazz = Class.forName("fanshe.Student");  


        //2.获取所有公有构造方法  
        System.out.println("**********************所有公有构造方法*********************************");  
        Constructor[] conArray = clazz.getConstructors();  
        for(Constructor c : conArray){  
            System.out.println(c);  
        }  

        System.out.println("************所有的构造方法(包括：私有、受保护、默认、公有)***************");  
        conArray = clazz.getDeclaredConstructors();  
        for(Constructor c : conArray){  
            System.out.println(c);  
        }  

        System.out.println("*****************获取公有、无参的构造方法*******************************");  
        Constructor con = clazz.getConstructor(null);  
        //1>、因为是无参的构造方法所以类型是一个null,不写也可以：这里需要的是一个参数的类型，切记是类型  
        //2>、返回的是描述这个无参构造函数的类对象。  

        System.out.println("con = " + con);  
        //调用构造方法  
        Object obj = con.newInstance();  
    //  System.out.println("obj = " + obj);  
    //  Student stu = (Student)obj;  

        System.out.println("******************获取私有构造方法，并调用*******************************");  
        con = clazz.getDeclaredConstructor(char.class);  
        System.out.println(con);  
        //调用构造方法  
        con.setAccessible(true);//暴力访问(忽略掉访问修饰符)  
        obj = con.newInstance('男');  
    }  

}  


```

### 获取反射对象中的字段

```Java

public class Student {  
  public Student(){  

  }  
  //**********字段*************//  
  public String name;  
  protected int age;  
  char sex;  
  private String phoneNum;  

  @Override  
  public String toString() {  
      return "Student [name=" + name + ", age=" + age + ", sex=" + sex  
              + ", phoneNum=" + phoneNum + "]";  
  }  


}
```

调用字段时：需要传递两个参数：
Object obj = stuClass.getConstructor().newInstance();//产生Student对象--》Student stu = new Student();
//为字段设置值
f.set(obj, "刘德华");//为Student对象中的name属性赋值--》stu.name = "刘德华"
第一个参数：要传入设置的对象，第二个参数：要传入实参

```Java



import java.lang.reflect.Field;  
/*
 * 获取成员变量并调用：
 *  
 * 1.批量的
 *      1).Field[] getFields():获取所有的"公有字段"
 *      2).Field[] getDeclaredFields():获取所有字段，包括：私有、受保护、默认、公有；
 * 2.获取单个的：
 *      1).public Field getField(String fieldName):获取某个"公有的"字段；
 *      2).public Field getDeclaredField(String fieldName):获取某个字段(可以是私有的)
 *  
 *   设置字段的值：
 *      Field --> public void set(Object obj,Object value):
 *                  参数说明：
 *                  1.obj:要设置的字段所在的对象；
 *                  2.value:要为字段设置的值；
 *  
 */  
public class Fields {  

        public static void main(String[] args) throws Exception {  
            //1.获取Class对象  
            Class stuClass = Class.forName("fanshe.field.Student");  
            //2.获取字段  
            System.out.println("************获取所有公有的字段********************");  
            Field[] fieldArray = stuClass.getFields();  
            for(Field f : fieldArray){  
                System.out.println(f);  
            }  
            System.out.println("************获取所有的字段(包括私有、受保护、默认的)********************");  
            fieldArray = stuClass.getDeclaredFields();  
            for(Field f : fieldArray){  
                System.out.println(f);  
            }  
            System.out.println("*************获取公有字段**并调用***********************************");  
            Field f = stuClass.getField("name");  
            System.out.println(f);  
            //获取一个对象  
            Object obj = stuClass.getConstructor().newInstance();//产生Student对象--》Student stu = new Student();  
            //为字段设置值  
            f.set(obj, "刘德华");//为Student对象中的name属性赋值--》stu.name = "刘德华"  
            //验证  
            Student stu = (Student)obj;  
            System.out.println("验证姓名：" + stu.name);  


            System.out.println("**************获取私有字段****并调用********************************");  
            f = stuClass.getDeclaredField("phoneNum");  
            System.out.println(f);  
            f.setAccessible(true);//暴力反射，解除私有限定  
            f.set(obj, "18888889999");  
            System.out.println("验证电话：" + stu);  

        }  
    }

```


### 调用方法

```Java
m = stuClass.getDeclaredMethod("show4", int.class);//调用制定方法（所有包括私有的），需要传入两个参数，第一个是调用的方法名称，第二个是方法的形参类型，切记是类型。
System.out.println(m);
m.setAccessible(true);//解除私有限定
Object result = m.invoke(obj, 20);//需要两个参数，一个是要调用的对象（获取有反射），一个是实参
System.out.println("返回值：" + result);//

```
* 反射方法调用
```Java

import java.lang.reflect.Method;  

/*
 * 获取成员方法并调用：
 *  
 * 1.批量的：
 *      public Method[] getMethods():获取所有"公有方法"；（包含了父类的方法也包含Object类）
 *      public Method[] getDeclaredMethods():获取所有的成员方法，包括私有的(不包括继承的)
 * 2.获取单个的：
 *      public Method getMethod(String name,Class<?>... parameterTypes):
 *                  参数：
 *                      name : 方法名；
 *                      Class ... : 形参的Class类型对象
 *      public Method getDeclaredMethod(String name,Class<?>... parameterTypes)
 *  
 *   调用方法：
 *      Method --> public Object invoke(Object obj,Object... args):
 *                  参数说明：
 *                  obj : 要调用方法的对象；
 *                  args:调用方式时所传递的实参；

):
 */  
public class MethodClass {  

    public static void main(String[] args) throws Exception {  
        //1.获取Class对象  
        Class stuClass = Class.forName("fanshe.method.Student");  
        //2.获取所有公有方法  
        System.out.println("***************获取所有的”公有“方法*******************");  
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
        Method m = stuClass.getMethod("show1", String.class);  
        System.out.println(m);  
        //实例化一个Student对象  
        Object obj = stuClass.getConstructor().newInstance();  
        m.invoke(obj, "刘德华");  

        System.out.println("***************获取私有的show4()方法******************");  
        m = stuClass.getDeclaredMethod("show4", int.class);  
        System.out.println(m);  
        m.setAccessible(true);//解除私有限定  
        Object result = m.invoke(obj, 20);//需要两个参数，一个是要调用的对象（获取有反射），一个是实参  
        System.out.println("返回值：" + result);  


    }  
}  


```

### 反射调用main方法 （内省）

#### 创建Student类

```Java
public class Student {  

    public static void main(String[] args) {  
        System.out.println("main方法执行了。。。");  
    }  
}  
```
* 调用main

```Java
import java.lang.reflect.Method;  

/**
 * 获取Student类的main方法、不要与当前的main方法搞混了
 */  
public class Main {  

    public static void main(String[] args) {  
        try {  
            //1、获取Student对象的字节码  
            Class clazz = Class.forName("fanshe.main.Student");  

            //2、获取main方法  
             Method methodMain = clazz.getMethod("main", String[].class);//第一个参数：方法名称，第二个参数：方法形参的类型，  
            //3、调用main方法  
            // methodMain.invoke(null, new String[]{"a","b","c"});  
             //第一个参数，对象类型，因为方法是static静态的，所以为null可以，第二个参数是String数组，这里要注意在jdk1.4时是数组，jdk1.5之后是可变参数  
             //这里拆的时候将  new String[]{"a","b","c"} 拆成3个对象。。。所以需要将它强转。  
             methodMain.invoke(null, (Object)new String[]{"a","b","c"});//方式一  
            // methodMain.invoke(null, new Object[]{new String[]{"a","b","c"}});//方式二  

        } catch (Exception e) {  
            e.printStackTrace();  
        }  


    }  
}
```

### 通过反射获取配置文件

* Student类

```Java
public class Student {  
    public void show(){  
        System.out.println("is show()");  
    }  
}  
```

* .txt 配置为例

pro.txt
```Java
className = cn.fanshe.Student  
methodName = show  
```

* 调用示例

```Java
import java.io.FileNotFoundException;  
import java.io.FileReader;  
import java.io.IOException;  
import java.lang.reflect.Method;  
import java.util.Properties;  

/*
 * 我们利用反射和配置文件，可以使：应用程序更新时，对源码无需进行任何修改
 * 我们只需要将新类发送给客户端，并修改配置文件即可
 */  
public class Demo {  
    public static void main(String[] args) throws Exception {  
        //通过反射获取Class对象  
        Class stuClass = Class.forName(getValue("className"));//"cn.fanshe.Student"  
        //2获取show()方法  
        Method m = stuClass.getMethod(getValue("methodName"));//show  
        //3.调用show()方法  
        m.invoke(stuClass.getConstructor().newInstance());  

    }  

    //此方法接收一个key，在配置文件中获取相应的value  
    public static String getValue(String key) throws IOException{  
        Properties pro = new Properties();//获取配置文件的对象  
        FileReader in = new FileReader("pro.txt");//获取输入流  
        pro.load(in);//将流加载到配置文件对象中  
        in.close();  
        return pro.getProperty(key);//返回根据key获取的value值  
    }  
}  
```

### 反射越过泛型擦除

```Java
import java.lang.reflect.Method;  
import java.util.ArrayList;  

/*
 * 通过反射越过泛型检查
 *  
 * 例如：有一个String泛型的集合，怎样能向这个集合中添加一个Integer类型的值？
 */  
public class Demo {  
    public static void main(String[] args) throws Exception{  
        ArrayList<String> strList = new ArrayList<>();  
        strList.add("aaa");  
        strList.add("bbb");  

    //  strList.add(100);  
        //获取ArrayList的Class对象，反向的调用add()方法，添加数据  
        Class listClass = strList.getClass(); //得到 strList 对象的字节码 对象  
        //获取add()方法  
        Method m = listClass.getMethod("add", Object.class);  
        //调用add()方法  
        m.invoke(strList, 100);  

        //遍历集合  
        for(Object obj : strList){  
            System.out.println(obj);  
        }  
    }  
}  
```

* 这里在注明一下摘自博客吧
[点击博主链接](https://blog.csdn.net/sinat_38259539/article/details/71799078)
