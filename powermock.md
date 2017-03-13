# PowerMockito学习笔记   
> 转自http://blog.csdn.net/knighttools/article/details/44630975  
> 2017年3月12日    

## 1. 为什么使用Mock工具     
在做单元测试的时候，有时候要测试的方法会引入很多外部依赖的对象，如： 发送邮件、网络通信、远程服务、文件系统等。而实际测试中没有办法控制这些外部依赖的对象。为了解决这个问题，需要使用Mock工具来模拟这些外部依赖的对象，来完成单元测试。    

## 2. 为什么使用PowerMock    
当前比较流行的Mock工具，有jMock、EasyMock、Mockito等，但它们都有一个共同的缺点： 不能mock静态、final、私有方法等，而PowerMock能够完美的弥补以上三个工具的不足。     

## 3. PowerMock简介    
PowerMock是一个扩展了其他如EasyMock等mock框架、功能更加强大的框架。PowerMock使用了一个自定义类加载器和字节码操作来模拟静态方法、构造函数、final类和方法、私有方法、去除静态初始化器等等。熟悉PowerMock支持的mock框架的开发人员会发现PowerMock很容易使用，因为对于静态方法和构造器来说，生个的期望API是一样的。目前PowerMock支持EasyMock和Mockito。    

## 4. PowerMock入门    
PowerMock有两个重要的注解：   
* @RunWith(PowerMockRunner.class)   
* @PrepareForTest({YourClassWithEgStaticMethod.class})

如果测试用例里没有使用注解@PrepareForTest，那么可以不用加注解@RunWith(PowerMockRunner.class)，反之依然。当需要使用PowerMock强大功能（Mock静态、final、私有方法等）的时候，就需要加注解@PrepareForTest.    

## 5. PowerMock基本用法    
### 5.1 普通Mock: 模拟参数传递的对象     
* 测试目标代码：    
```java
public class ClassUnderTest {
    public boolean callArgumentInstance(File file) {
        return file.exists();
    } 
}
```

* 测试用例代码：   
```java
@Test
public void testCallArgumentInstance() {
    File file = PowerMockito.mock(File.class);
    ClassUnderTest underTest = new ClassUnderTest();
    PowerMockito.when(file.exists()).thenReturn(true);
    Assert.assertTure(underTest.callArgumentInstance(file));
}
```
说明： 普通Mock不需要加@RunWith和@PrepareForTest注解   

### 5.2 模拟方法内部new出来的对象   
* 测试目标代码：  
```java
public class ClassUnderTest {
    public boolean callInternalInstance(String path) {
        File file = new File(path);
        return file.exists();
    }
}
```
* 测试用例代码：   

```java
@RunWith(PowerMockRunner.class)
public class TestClassUnderTest {

    @Test
    @PrepareForTest(ClassUnderTest.class)
    public void testCallInternalInstance() throws Exception {
        File file = PowerMockito.mock(File.class);
        ClassUnderTest underTest = new ClassUnderTest();
        PowerMockito.whenNew(File.class).withArguments("bbb".thenReturn(file))
        PowerMockito.when(file.exists()).thenReturn(true);
        Assert.assertTrue(underTest.callInternalInstance("bbb"));
    }
}
```
说明： 当使用PowerMockito.whenNew方法时，必须加注解@PrepareForTest和@RunWith。注解@PrepareForTest里写的类是需要mock的new对象代码所在的类。

### 5.3 模拟普通对象的final方法    
* 测试目标代码：   
```java
public class ClassUnderTest {
    public boolean callFinalMethod(ClassDependency refer) {
        return refer.isAlive();
    }
}

public class ClassDependency {
    public final boolean isAlive() {
        // do something
        return false;
    }
}
```
* 测试用例代码：   
```java
@RunWith(PowerMockRunner.class)
public class TestClassUnderTest {

    @Test
    @PrepareForTest(ClassDependency.class)
    public void testCallFinalMethod() {
        ClassDependency dependency = PowerMockito.mock(ClassDependency.class);
        ClassUnderTest underTest = new ClassUnderTest();
        PowerMockito.when(dependency.isAlive()).thenReturn(true);
        Assert.assertTrue(underTest.callFinalMethod(dependency));
    }
}
```
说明： 当需要mock final方法的时候，必须加注解@PrepareForTest和@RunWith。注解@PrepareForTest里写的类是final方法所在的类。    

### 5.4 模拟普通类的静态方法    
* 测试目标代码：    
```java
public class ClassUnderTest {
    public boolean callStaticMethod() {
        return ClassDependency.isExist();
    }
}
public class ClassDependency() {
    public static boolean isExist() {
        // do something
        return false;
    }
}
```

* 测试用例代码：    
```java
@RunWith(PowerMockRunner.class)
public class TestClassUnderTest {

    @Test
    @PrepareForTest(ClassDependency.class)
    ClassUnderTest underTest = new ClassUnderTest();
    PowerMockito.mockStaic(ClassDependency.class);
    PowerMockito.when(ClassDependency.isExist()).thenReturn(true);
    Assert.assertTrue(underTest.callStaticMethod());
}
```
说明：当需要mock静态方法的时候，必须加注解@PrepareForTest和@RunWith。注解@PrepareForTest里写的类是静态方法所在的类。    

### 5.5 模拟私有方法
* 测试目标代码：    
```java
public class ClassUnderTest {
    public boolean callPrivateMethod() {
        return isExist();
    }

    private boolean isExist() {
        return false;
    }
}
```
* 测试用例代码：    
```java
@RunWith(PowerMockRunner.class)
public class TestClassUnderTest {

    @Test
    @PrepareForTest(ClassUnderTest.class)
    public void testCallPrivateMethod() throws Exception {
        ClassUnderTest underTest = PowerMockito.mock(ClassUnderTest.class);
        PowerMockito.when(underTest.callPrivateMethod()).thenCallRealMethod();
        PowerMockito.when(underTest, "isExist").thenReturn(true);
        Assert.assertTrue(underTest.callPrivateMethod());
    }
}
```
说明：和Mock普通方法一样，只是需要加注解@PrepareForTest(ClassUnderTest.class)，注解里写的类是私有方法所在的类。 

### 5.6 模拟系统类的静态和final方法
* 测试目标代码：   
```java
public class ClassUnderTest {
    public boolean callSystemFinalMethod(String str) {
        return str.isEmpty();
    }

    public String callSystemStaticMethod(String str) {
        return System.getProperty(str);
    }
}
```
* 测试用例代码：    
```java
@RunWith(PowerMockRunner.class)
public class TestClassUnderTest {

    @Test
    @PrepareForTest(ClassUnderTest.class)
    public void testCallSystemStaticMethod() {
        ClassUnderTest underTest = PowerMockito.mock(ClassUnderClass.class);
        PowerMockito.mockStatic(System.class);
        PowerMockito.when(System.getProperty("aaa")).thenReturn("bbb");
        Assert.assertEquals("bbb", underTest.callJDKSTaticMethod("aaa"));
    }
}
```
说明：和Mock普通对象的静态方法、final方法一样，只不过注解@PrepareForTest里写的类不一样 ，注解里写的类是需要调用系统方法所在的类。

## 6. 无所不能的PowerMock
* 验证静态方法：   
```java
PowerMockito.verifyStatic();
Static.firstStaticMethod(param);
```
* 扩展验证：    
  * `PowerMockito.verifyStatic(Mockito.times(2));` // 被调用2次
  * `Static.thirdStaticMethod(Mockito.anyInt());` // 以任何整数值被调用
* 更多的Mock方法    
http://code.google.com/p/powermock/wiki/MockitoUsage13

## 7. PowerMock简单实现原理   
* 当某个测试方法被注解@PrepareForTest标注以后，在运行测试用例时，会创建一个新的org.powermock.core.classloader.MockClassLoader实例，然后加载该测试用例使用到的类（系统类除外）。
* PowerMock会根据你的mock要求，去修改写在注解@PrepareForTest里的class文件（当前测试类会自动加入注解中），以满足特殊的mock需求。例如：去除final方法的final标识，在静态方法的最前面加入自己的虚拟实现等。
* 如果需要mock的是系统类的final方法和静态方法，PowerMock不会直接修改系统类的class文件，而是修改调用系统类的class文件，以满足mock需求。