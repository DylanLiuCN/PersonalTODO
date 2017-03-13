# junit学习笔记    
> SyinChwun Leo   
> 2017年3月12日     
> 
## 1. JUnit 安装
在Maven项目中创建工程，并在pom.xml文件中设置以下添加以下依赖：    
 
```xml
 <dependency>
   <greoupId>junit</greoupId>
   <artifactId>junit</artifactId>
   <version>4.12</version>
   <scope>test</scope>
 </dependency>
```

在测试过程中定义一个测试类的要求如下：    
* 该测试类必须是公共的，并且包含一个无参数的构造函数   
* 创建测试方法时，必须使用@Test注释
* 该测试方法必须是公共的，不带任何参数，并且返回void类型
### 2. Junit测试用例     
假设创建一个Calculator的类，具体如下：   

```java 
class Calculator {
    def add(a: Int, b: Int): Int = {
        a + b
    }
}
```
测试类如下：    

```java
import org.junit.Assert.assertEquals
import org.junit.Test

class CalculatorTest {

    @Test
    def testAdd(): Unit = {
        val calculator = new Calculator
        val res = calculator.add(3, 4)
        assertEquals(7, res)
    }
}
```
JUnit在调用（运行）每个@Test方法前，为测试类创建一个新的实例。这有助于提供测试方法之间的独立性，并且避免在测试代码中产生意外的副作用。因此，每个测试方法都运行在一个新的测试类实例上，所以不能在测试方法之间重用各个实例变量值。   
为了验证测试结果，可以使用JUnit的Assert类提供的方法。在以上测试中使用了`assertEquals()`方法，需要在文件中导入，还可以导入其他的Assert类的方法。最为流行的assert方法如下：    

| assertXXX方法 | 作用 |     
| -------------- | ---- |    
| assertArrayEquals("message", A, B) | 断言数组A和数组B相等 |    
| assertEquals("message", A, B) | 断言对象A和对象B相等。这个断言在比较两个对象时调用了`equals()`方法 |    
| assertSame("message", A, B) | 之前的assert方法是检查A和B是否有相同的值（使用了equals方法），而assertSame方法则是检查A与B是否是一个相同的对象(使用的是==操作符) |    
| assertTrue("message", A) | 断言A条件为真 |    
| assertNotNull("message", A) | 断言A对象不为null |    

注意： 计算机并不能精确地表示所有浮点书，通常都会有一些偏差。因此，如果想用断言来比较浮点数（在Java中，是类型为float或者double的数），则需要指定一个额外的误差。它表明用户需要多接近才能认为两数“相等”。

```java
assertEquals([String message],
    expected,
    actual.
    tolerance)
```

例如： 下面的断言将会检查实际的计算结果是否等于3.33，但是该检查只精确到小数点的后两位：    

```java
assertEquals("Should be 3 1/3", 3.33, 10.0/3.0, 0.01);
```