[TOC]
# 什么是反射

在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

# 反射有什么用

1. 在运行时判断任意一个对象所属的类；

2. 在运行时构造任意一个类的对象；

3. 在运行时判断任意一个类所具有的成员变量和方法；

4. 在运行时调用任意一个对象的方法；

5. 生成动态代理。

# Reflection API

|Member| Class API| Return type | Inherited(继承) members | Private members|
| --- | --- | --- | --- | --- |
|Class            | getDeclaredClasses()         | Array    | N        | Y|
|                 | getClasses()                       | Array   | Y          | N|
|Field             | getDeclaredField()              | Single  | N         | Y|
|                    | getField()                            | Single  | Y         | N|
|                    | getDeclaredFields()            | Array    | N         | Y|
|                    | getFields()                          | Array    | Y         | N|
|Method         | getDeclaredMethod()         | Single   | N        | Y|
|                    | getMethod()                       | Single   | Y         | N|
|                    | getDeclaredMethods()        | Array    | N        | Y|
|                    | getMethods()                      | Array    | Y        | N|
|Constructor  | getDeclaredConstructor()   | Single  |   N/A   | Y|
|                    | getConstructor()                  | Single |  N/A    | N|
|                    | getDeclaredConstructors()  | Array   |  N/A    | Y|
|                    | getConstructors()                | Array  |   N/A    | N|

如表所示，getClasses()拥有继承的特点，可以获取父亲级定义的内部类，却不能访问定义为private的内部类；而getDeclaredClasses()刚好相反，可以访问定义为private的内部类，却无法获取父亲级定义的内部类。

# setAccessible()方法的作用
## 1. 提高java反射速度
```java
package com.chenshuyi.test;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws Exception {
        Method m = A.class.getDeclaredMethod("getName", new Class[] {});
        System.out.println(m.isAccessible());
        // getName是public的,猜猜输出是true还是false

        A a = new A();
        a.setName("Mr Lee");
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000000; i++) {
            m.invoke(a, new Object[] {});
        }
        System.out.println("Simple:" + (System.currentTimeMillis() - start));

        m.setAccessible(true); // 注意此处不同
        long start1 = System.currentTimeMillis();
        for (int i = 0; i < 10000000; i++) {
            m.invoke(a, new Object[] {});
        }
        System.out.println("setAccessible(true) :" + (System.currentTimeMillis() - start1));
    }
}

class A {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
测试结果 
```
false 
Simple :4969 
setAccessible(true) :250 
```
### 解释

1. 明显 Accessible并不是标识方法能否访问的. public的方法 Accessible仍为false 

2. 使用了method.setAccessible(true)后 性能有了20倍的提升 

3. Accessable属性是继承自AccessibleObject 类. 功能是启用或禁用安全检查 

4. JDK API中的解释：
    * AccessibleObject 类是 Field、Method 和 Constructor 对象的基类。它提供了将反射的对象标记为在使用时取消默认 Java 语言访问控制检查的能力。对于公共成员、默认（打包）访问成员、受保护成员和私有成员，在分别使用 Field、Method 或 Constructor 对象来设置或获得字段、调用方法，或者创建和初始化类的新实例的时候，会执行访问检查。 
    * 在反射对象中设置 accessible 标志允许具有足够特权的复杂应用程序（比如 Java Object Serialization 或其他持久性机制）以某种通常禁止使用的方式来操作对象。 
    * `public void setAccessible(boolean flag) throws SecurityException` 将此对象的 accessible 标志设置为指示的布尔值。值为 true 则指示反射的对象在使用时应该取消 Java 语言访问检查。值为 false 则指示反射的对象应该实施 Java 语言访问检查。
    *  实际上setAccessible是启用和禁用访问安全检查的开关,并不是为true就能访问为false就不能访问 
    *  由于JDK的安全检查耗时较多.所以通过setAccessible(true)的方式关闭安全检查就可以达到提升反射速度的目的 

## 2. 获取与修改private的`属性`的值

因为`setAccessible(true)` 指取消Java 语言访问检查，没有语言访问检查，所以private的属性就可以访问了。

**对于private的方法类似操作。**

```java
package org.iteye.bbjava.runtimeinformation;   
 
public class PrivateObject {   
 
   @SuppressWarnings("unused")
   private String privateString = null;   
 
   public PrivateObject(String privateString) {   
        this.privateString = privateString;   
    }   
}  
```

可以看到PrivateObject.java 里有一个private 的属性String型的 privateString,没有为其定义getter,setter方法。



### 1. 对PrivateObject类进行反射取值，是一种“暴力操作”

```java
package org.iteye.bbjava.runtimeinformation;
 
import java.lang.reflect.Field;
 
public class Test {
	 public static void main(String []args) throws Exception{
		  PrivateObject privateObject = new PrivateObject("The Private Value");
 
		  Field privateStringField = PrivateObject.class.getDeclaredField("privateString");
 
		  privateStringField.setAccessible(true);
 
		  String fieldValue = (String) privateStringField.get(privateObject);
		  System.out.println("fieldValue = " + fieldValue);
	  }
}
```
运行结果：
> fieldValue = The Private Value

### 2. 对PrivateObject类进行暴力修改

其中  privateStringField.setAccessible(true); 就是对private 属性修改的“权限开关”，当设置为true时，可以修改，为false时会抛出异常，运行时将会抛出该异常.

```java
package org.iteye.bbjava.runtimeinformation;
 
import java.lang.reflect.Field;
 
public class Test {
	 public static void main(String []args) throws Exception{
		  PrivateObject privateObject = new PrivateObject("The Private Value");
 
		  Field privateStringField = PrivateObject.class.getDeclaredField("privateString");
 
		 
		  
		  privateStringField.setAccessible(true);
		  String fieldValue = (String) privateStringField.get(privateObject);
		  System.out.println("fieldValue = " + fieldValue);
 
		  
		  privateStringField.setAccessible(true);
		  privateStringField.set(privateObject, "As you see,privateString's value is changed!");
		  String fieldValue1 = (String) privateStringField.get(privateObject);
		  System.out.println("fieldValue = " + fieldValue1);
 
		  privateStringField.setAccessible(false);
		  privateStringField.set(privateObject, "As you see,privateString's value is changed!");
		  String fieldValue2 = (String) privateStringField.get(privateObject);
		  System.out.println("fieldValue = " + fieldValue2);
	  }
}
```
运行结果：
```
fieldValue = The Private Value
fieldValue = As you see,privateString 'value is changed!

Exception in thread "main" java.lang.IllegalAccessException: Class org.iteye.bbjava.runtimeinformation.Test can not access a member of class org.iteye.bbjava.runtimeinformation.PrivateObject with modifiers "private"at sun.reflect.Reflection.ensureMemberAccess(Reflection.java:65)at java.lang.reflect.Field.doSecurityCheck(Field.java:960)at java.lang.reflect.Field.getFieldAccessor(Field.java:896)at java.lang.reflect.Field.set(Field.java:657)at org.iteye.bbjava.runtimeinformation.Test.main(Test.java:25)

```




# 基础实例

## 实例一：获得完整的类名
```java
package reflection.getclassname;
 
//获得完整的类名
public class GetClassName {
 
	public String getNameByClass() {
		
		String name = "";
		System.out.println("通过类本身获得对象");
		Class UserClass = this.getClass();
		System.out.println("获得对象成功！\n");
		
		System.out.println("通过类对象获得类名");
		name = UserClass.getName();
		System.out.println("获得类名成功！");
		return name;
	}
	public static void main(String[] args) {
		
		GetClassName gcn = new GetClassName();
		System.out.println("类名为："+gcn.getNameByClass());
	}
 
}
```

运行结果：
```
通过类本身获得对象
获得对象成功！

通过类对象获得类名
获得类名成功！
类名为：reflection.getclassname.GetClass Name       
```

```java
System.out.println(Test.class.getClassLoader().getClass().getName());//load到内存中的class名字
System.out.println(Test.class.getName());//类本身的名字。
// 分别打印出
//sun.misc.Launcher$AppClassLoader
//com.test.Test
```
## 实例二：获得类的属性
```java
package reflection.getfields;
 
import java.lang.reflect.Field;
 
//获得类的属性
public class GetFields {
 
	public static void getFieldNames(String className) {
		
		try {
			//获得类名
			Class c = Class.forName(className);
			//获得所有属性
			Field[] fds = c.getFields();
			
			for (int i=0; i<fds.length; i++)
			{
				String fn = fds[i].getName();
				Class tc = fds[i].getType();
				String ft = tc.getName();
				System.out.println("该属性的名字为："+fn+"，该属性的类型为："+ft);
			}
		}catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		GetFields.getFieldNames("reflection.getfields.FieldInfo");
	}
 
}
```

运行结果：
```
该属性的名字为：id，该属性的类型为：java.lang.String
该属性的名字为：username，该属性的类型为：java.lang.String    
```

## 实例三：获得类实现的接口
```java
package reflection.getinterfaces;
 
//获得类实现的接口
public class GetInterfaces {
 
	public static void getInterfaces(String className) {
		try {
			//取得类
			Class cl = Class.forName(className);
			Class[] ifs = cl.getInterfaces();
			for (int i = 0; i<ifs.length;i++)
			{
				String IName = ifs[i].getName();
				System.out.println("该类实现的接口名字为："+IName);
			}
		}catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		GetInterfaces.getInterfaces("reflection.getinterfaces.Student");
	}
 
}
```

运行结果：
```
该类实现的接口名字为：reflection.getinterfaces.Person    
```
## 实例四：获得类及其属性的修饰符
```java
package reflection.getmodifiers;
 
import java.lang.reflect.Field;
import java.lang.reflect.Modifier;
 
import reflection.UserInfo;
 
//获得类及其属性的修饰符
public class GetModifiers {
 
	private String username = "liu shui jing";
	float f = Float.parseFloat("1.000");
	public static final int i = 37;
	
	//获得类的修饰符
	public static String useModifiers(UserInfo ui) {
		
		Class uiClass = ui.getClass();
		int m = uiClass.getModifiers();
		return Modifier.toString(m);
		
	}
	
	//获得本类属性的修饰符
	public void checkThisClassModifiers() {
		
		Class tc = this.getClass();		
		Field fl[] = tc.getDeclaredFields();
	
		for(int i=0;i<fl.length;i++)
		{
			System.out.println("第"+(i+1)+"个属性的修饰符为："+Modifier.toString(fl[i].getModifiers()));
		}
		
	}
	public static void main(String[] args) {
		
		//获得类的修饰符
		UserInfo ui =new UserInfo();
		System.out.println("获得这个类的修饰符："+GetModifiers.useModifiers(ui)+"\n");
		
		//获得本类属性的修饰符
		GetModifiers gm = new GetModifiers();
		gm.checkThisClassModifiers();
		
	}
 
}
```

运行结果：
```

获得这个类的修饰符：public 

第1个属性的修饰符为：private
第2个属性的修饰符为：
第3个属性的修饰符为：public static final      
```
## 实例五：获得类的构造函数
```java
package reflection.getconstructor;
 
import java.lang.reflect.Constructor;
 
//获得类的构造函数
public class GetConstructor {
 
	//构造函数一
	GetConstructor(int a) {
		
	}
	
	//构造函数二
	GetConstructor(int a, String b) {
		
	}
	
	public static void getConstructorInfo(String className) {
		try {
			//获得类的对象
			Class cl =Class.forName(className);
			System.out.println("获得类"+className+"所有的构造函数");
			Constructor ctorlist[] = cl.getDeclaredConstructors();
			System.out.println("遍历构造函数\n");
			for(int i =0 ; i<ctorlist.length; i++)
			{
				Constructor con = ctorlist[i];
				System.out.println("这个构造函数的名字为："+con.getName());
				System.out.println("通过构造函数获得这个类的名字为："+con.getDeclaringClass());
				
				Class cp[] = con.getParameterTypes();
				for (int j=0; j<cp.length; j++)
				{
					System.out.println("参数 "+j+" 为 "+cp[j]+"\n");
				}
			}
		}catch (Exception e) {
				System.err.println(e);			
		}
	}
	public static void main(String[] args) {
		GetConstructor.getConstructorInfo("reflection.getconstructor.GetConstructor");
	}
 
}
```

运行结果：
```
获得类reflection.getconstructor.GetConstructor所有的构造函数

遍历构造函数

 

这个构造函数的名字为：reflection.getconstructor.GetConstructor

通过构造函数获得这个类的名字为：class reflection.getconstructor.GetConstructor       

参数 0 为 int

 

这个构造函数的名字为：reflection.getconstructor.GetConstructor

通过构造函数获得这个类的名字为：class reflection.getconstructor.GetConstructor

参数 0 为 int

 

参数 1 为 class java.lang.String

```
## 实例六：获得父类
```java
package reflection.getparentclass;
 
//获得父类
public class GetParentClass {
 
	public static String getParentClass(UserInfoMore uim) {
		
		//获得父类
		Class uimc = uim.getClass().getSuperclass();
		System.out.println("获得父类的名字为："+uimc.getName());
		return uimc.getName();
		
	}
	
	public static void searchParentClass() {
		
	}
	
	public static void main(String[] args) {
		
		UserInfoMore uim = new UserInfoMore();
		System.out.println("成功获得UserInfoMore的父类："+GetParentClass.getParentClass(uim));
		
	}
 
}
```

运行结果：

```
获得父类的名字为：reflection.UserInfo
成功获得UserInfoMore的父类：reflection.UserInfo   
```
## 实例七：获得类的方法
```java
package reflection.getmethod;
 
import java.lang.reflect.Method;
 
//获得类的方法
public class GetMethod {
 
	public static void getMethods(String className) {
		try {
			System.out.println("开始遍历类"+className+".class");
			//获得类名
			Class cls = Class.forName(className);
			//利用方法获得所有该类的方法
			System.out.println("利用类的getDeclaredMethods获得类的所有方法");
			Method ml[] =cls.getDeclaredMethods(); //返回 Method 对象的一个数组，这些对象反映此 Class 对象表示的类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。
          Method[] methods = c.getMethods();//返回一个包含某些 Method 对象的数组，这些对象反映此 Class 对象所表示的类或接口（包括那些由该类或接口声明的以及从超类和超接口继承的那些的类或接口）的公共 member 方法。
          
getMethod(String name, Class<?>... parameterTypes)//  返回一个 Method 对象，它反映此 Class 对象所表示的类或接口的指定公共成员方法。

getDeclaredMethod(String name, Class<?>... parameterTypes)// 返回一个 Method 对象，该对象反映此 Class 对象所表示的类或接口的指定已声明方法。

			System.out.println("遍历获得的方法数组\n");
			for (int i = 0 ;i<ml.length;i++)
			{
				System.out.println("开始遍历第"+(i+1)+"个方法");
				Method m = ml[i];
				System.out.println("开始获取方法的变量类型");
				Class ptype[] = m.getParameterTypes();
				for (int j=0; j<ptype.length; j++)
				{
					System.out.println("方法参数"+j+"类型为"+ptype[j]);
				}
				Class gEx[] = m.getExceptionTypes();
				for (int j=0 ; j<gEx.length; j++)
				{
					System.out.println("异常"+j+"为"+ gEx[j]);
				}
				
				System.out.println("该方法的返回值类型为："+m.getReturnType()+"\n");
				
			}
		}catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		GetMethod.getMethods("reflection.UserInfo");
	}
 
}
```
运行结果：
```
开始遍历类reflection.UserInfo.class

利用类的getDeclaredMethods获得类的所有方法         

遍历获得的方法数组

 

开始遍历第1个方法

开始获取方法的变量类型

该方法的返回值类型为：class java.lang.String

 

开始遍历第2个方法

开始获取方法的变量类型

该方法的返回值类型为：class java.lang.Integer

 

开始遍历第3个方法

开始获取方法的变量类型

方法参数0类型为class java.lang.String

该方法的返回值类型为：void

 

开始遍历第4个方法

开始获取方法的变量类型

该方法的返回值类型为：class java.lang.String

 

开始遍历第5个方法

开始获取方法的变量类型

方法参数0类型为class java.lang.Integer

该方法的返回值类型为：void

 

开始遍历第6个方法

开始获取方法的变量类型

该方法的返回值类型为：class java.lang.String

 

开始遍历第7个方法

开始获取方法的变量类型

方法参数0类型为class java.lang.String

该方法的返回值类型为：void

```


