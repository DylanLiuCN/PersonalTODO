# logback笔记    
> SyinChwun Leo     
> 2017年3月11日
> 
## 1. logback依赖设置    
目前logback主要包括三个模块：    

* logback-core
* logback-classic
* logback-access 

其中logback-core是其他两个模块的基础，logback-classic可以认为是log4j的一个增强版本。另外，logback-classic也自然应用了SLF4J API，所以，用户可以轻而易举的在logback和其他日志框架（如，log4j或java.util.logging(JUL)）之间切换.

假设本文使用的logbak的版本为1.2.1, self4j的版本为1.7.22。

### 1.1 通过命令行添加logback相关jar包

```shell
java -cp slf4j-api-1.7.22.jar;logback-core-1.2.1.jar;\
logback-classic-1.2.1.jar XXXjava程序
```    

### 1.2 通过maven添加logback相关jar包

为了使用logback，pom文件中应添加一下配置：    

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.1</version>
</dependency>
```
以上依赖直接添加的是logback-classic.jar，maven会自动添加slf4j-api.jar和logback-core.jar两个依赖包.
如果在你的Maven工程中想添加logback-access，需要在pom文件中添加以下依赖：    
```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-access</artifactId>
    <version>1.2.1</version>
</dependency>
```

## 2. 一个简单的日志例子    
代码如下：    

```java
package cn.edu.xxtc.cloudlab.log

import org.slf4j.{logger, LoggerFactory}

object Main {
    
    private val logger = LoggerFactory.getLogger(getClass)

    def main(args: Array[String]): Unit = {
        logger.info("Hello logger!")
        logger.debug("This is a debug information.")
    }
}
```
Main定义在cn.edu.xxtc.cloudlab.log包中，该object引入了Logger和LoggerFactory两个SLF4J API，这些API在org.slf4j中定义。      
在main()函数的第一行，变量logger通过调用LoggerFactory的静态方法getLogger获取一个Logger的实例。该logger被命名为“cn.edu.xxtc.cloudlab.log.Main”。main()函数中调用logger的info方法和debug方法输出两条信息。输出信息如下：
```shell
14:33:22.738[main] INFO  cn.edu.ustc.cloudlab.log.Main$ - hello logger!
14:33:22.837[main] DEBUG cn.edu.ustc.cloudlab.log.Main$ - Entry number: This is a debug information.
```

## 3. logback配置文件设置    
用户使用logback时需要在classpath上设置XML或Groovy格式的配置文件。logback按照以下步骤初始化自己的配置： 

* (a) logback尝试在classpath中查找名为"logback-test.xml"的配置文件;
* (b) 如果找不到，则尝试在classpath中查找名为"logback.groovy"的配置文件;
* (c) 如果找不到，则尝试在classpath中查找名为"logback.xml"的配置文件；
* (d) 如果找不到，"service-provider loading facility(JDK 1.6引入)"通过在classpath查找"META-INFO/services/ch.qos.logback.classic.spi.Configurator"（实现"com.qos.logback.classic.spi.Configurator"接口）。
* (e) 如果以上还不成功，logback使用BasicConfigurator进行配置，该配置将log日志输出到控制台上。

### 3.1 使用logback-test.xml或logback.xml自动配置
BasicConfigurator仅提供基本的log配置信息。在intellij IDEA等如果创建maven工程，可以在目录src/main/resources中添加"logback-test.xml"配置文件。假设logback-test.xml或logback.xml文件不存在，logback将使用BasicConfigurator作为最小配置。该配置将ConsoleAppender绑定到root logger上。输出使用PatternLayoutEncoder作为格式化输出。该格式化模式如下：   

```
%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
```

另外，root logger的日志级别为DEBUG.
以下配置文件等价于BasicConfigurator的配置信息：    
```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- encoders are assigned the type
            ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

### 3.2 遇到warning或error时自动打印status信息    

如果在解析configuration信息时，遇到warnings或errors，logback将自动将其内部的状态数据打印到控制台上。注意：为了避免重复，如果用户不明确注册一个状态监听器（status listener），状态自动打印功能将被禁止。在缺少warnings或errors时，如果用户仍然想查看logback的内部状态，可以通过调用StatusPrinter类的print()接口查看。举例如下：    
```java
import ch.qos.logback.classic.LoggerContext
import ch.qos.logback.core.util.StatusPrinter
import org.slf4j.{Logger, LoggerFactory}

object Main {
  private val logger = LoggerFactory.getLogger(getClass)

  def main(args: Array[String]): Unit = {
    val lc = LoggerFactory.getILoggerFactory.asInstanceOf[LoggerContext]
    StatusPrinter.print(lc)
    logger.info("Hello logback!")
    logger.debug("This is a debug information.")
  }
}
```
打印信息如下：    
```
15:48:41,923 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Found resource [logback-test.xml] at [file:/home/peak/%e6%96%87%e6%a1%a3/bigdata/logback/target/classes/logback-test.xml]
15:48:42,009 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - debug attribute not set
15:48:42,009 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - About to instantiate appender of type [ch.qos.logback.core.ConsoleAppender]
15:48:42,022 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - Naming appender as [STDOUT]
15:48:42,030 |-INFO in ch.qos.logback.core.joran.action.NestedComplexPropertyIA - Assuming default type [ch.qos.logback.classic.encoder.PatternLayoutEncoder] for [encoder] property
15:48:42,079 |-INFO in ch.qos.logback.classic.joran.action.RootLoggerAction - Setting level of ROOT logger to DEBUG
15:48:42,079 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [STDOUT] to Logger[ROOT]
15:48:42,080 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - End of configuration.
15:48:42,080 |-INFO in ch.qos.logback.classic.joran.JoranConfigurator@46f5f779 - Registering current configuration as safe fallback point

15:48:42.084[main] INFO  cn.edu.ustc.cloudlab.log.Main$ - Hello logback!
15:48:42.086[main] DEBUG cn.edu.ustc.cloudlab.log.Main$ - This is a debug information.
```

### 3.3 logback状态数据（status data）    
除了用户在代码中明确调用StatusPrinter之外，用户可以在配置文件中通过配置信息丢弃状态数据，为了达到这个目的，用户可以在配置文件的最外层元素中配置debug属性。注意：debug属性仅与状态数据有关，并不影响logback的配置，尤其是日志的级别。例如以下配置信息：    
```xml
<configuration debug="true"> 

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> 
    <!-- encoders are  by default assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

设置"debug=true"等价与创建"OnConsoleStatusLister",即：    
```xml
<configuration>
  <statusListener class="ch.qos.logback.core.status.OnConsoleStatusListener" />  

  ... the rest of the configuration file  
</configuration>
```
### 3.4 通过系统属性指定一个配置文件作为默认配置

用户可以通过系统属性"logback.configurationFile"确定默认配置文件的位置， 该系统属性的值可以是一个URL，classpath上的资源或者应用之外的一个文件的路径。方法如下：    
```shell
java -Dlogback.configurationFile=/path/to/config.xml java程序
```
注意： 该文件必须以".xml"或者".groovy"结尾，否则，将被忽略。

### 3.5 配置文件修改后自动重新加载     
如果用户要求，那么logback-classic将在配置文件修改后能够重新加载新的配置文件并配置自己。为了时logback-classic开启此功能，需要在配置文件的configuration元素中设置"scan=true"属性。具体如下：     
```xml
<configuration scan="true"> 
  ... 
</configuration> 
```
默认情况下，logback-classic将1分钟扫描一下配置文件，用户可以通过设置configuration的scanPeriod来确定扫描周期，scanPeriod可以以milliseconds、seconds、minutes以及hours为单位。举例如下：    
```xml
<configuration scan="true" scanPeriod="30 seconds" > 
  ...
</configuration> 
```
如果不带单位，默认使用milliseconds，但是并不鼓励。

### 3.6 在堆栈轨迹（stack traces）中显示packaging数据     
如果用户要求，logback可以在输出的每条stack trace中带有包数据(packaging data),Packaging data包含包的名称、包的版本好。Packaging data对确定软件版本非常有用。如：
该功能可以通过将configuration元素的packagingData属性设置为true实现。具体如下： 
```xml
<configuration packagingData="true">
  ...
</configuration>
```   
同样，也可以通过代码来实现：    
```java
 val lc = LoggerFactory.getILoggerFactory().asInstanceOf[];
  lc.setPackagingDataEnabled(true);
```
## 4. 配置文件语法    
logback允许用户在不需要重新编译代码的情况下重新定义log的行为。用户可以通过简单的配置logback以便禁止应用的部分logging，或这将日志输出到一个UNIX Syslog daemon、数据库、日志可视化工具、或者远程logback服务器（需要根据本地的服务器策略，如将日志事件推送到第二个logback server）中。配置文件的基本格式如下：   
一个`<configuration>`元素，里面包含0到多个`<appender>`元素，后面跟着0到多个`<logger>`元素，紧跟着至多1个`<root>`元素。具体如下：    
```xml
<configuration>
    <appender>
        ....
    </appender>
    <logger>
       ....
    </logger>
    <root>
        ....
    </root>
</configuration>
```
### 4.1 tag名称大小写敏感问题
从logback version 0.9.17开始，tag的名字对大小写不敏感，例如：`<logger>`,`<Logger>`以及`<LOGGER>`在配置文件中都是可以使用的元素，将被认为是同一个元素。如果用户使用`<xyz>`打开一个tag，那么他必须使用`</xyz>`关闭该tag，`</XyZ>`并不起作用。潜在的规则是，tag名字对第一个字母大小写不敏感，即：`<xyz>`和`<Xyz>`是相等的，但不等价与`<xYz>`。

### 4.2 配置loggers或者<logger>元素

一个logger可以通过`<logger>`元素进行配置。一个`<logger>`元素中包含一个强制性的`name`属性，一个可选的`level`属性，一个可选的`additivity`属性，属性值为`true`或`false`。`level`属性的取值可以为大小写不敏感的字符串`TRACE`，`DEBUG`，`INFO`，`WARN`，`ERROR`，`ALL`或者`OFF`。特殊值`INHERITED`或者其同义词`NULL`，将迫使它继承上层的logger配置。    

<logger>元素可以包含0个或多个<appender-ref>元素，每个appender参阅一个已经命名的logger。注意： 与log4j不同，当配置一个给定的appender时，logback-classic并不关闭或移除之前的appender。

### 4.3 配置root的logger或者<root>元素
<root>元素用来配置root的logger.它仅支持一个属性，即`level`属性。他并不允许有其他属性，因为root logger并不支持其他的flag.另外，既然root logger已经被命名为"ROOT"，因此，也不允许使用`name`属性。`level`的值可以取大小写不敏感的字符串`TRACE`,`DEBUG`,`INFO`,`WARN`,`ERROR`,`ALL`或者`OFF`.注意，root的`level`级别不允许设为`INHERITED`或者`NULL`。

 与`<logger>`元素，类似，`<root>`元素也可以包含0个或多个`<appender-ref>`元素，每个appender军添加到root logger中。注意： 与log4j不同，当配置一个给定的appender时，logback-classic并不关闭或移除之前的appender。

 ### 4.4 举例   

简单设置一个root logger.假设，不在需要输出属于包"chapters.configuration"中部分的任何DEBUG信息，，具体设置如下：     
```xml
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <logger name="chapters.configuration" level="INFO"/>

  <!-- Strictly speaking, the level attribute is not necessary since -->
  <!-- the level of the root level is set to DEBUG by default.       -->
  <root level="DEBUG">          
    <appender-ref ref="STDOUT" />
  </root>  
  
</configuration>
```
这样，属于chapter.configuration的日志仅打印info信息。   
用户可以设置多个logger,具体如下：
```xml
<configuration>

  <appender name="STDOUT"
    class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>
        %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
     </pattern>
    </encoder>
  </appender>

  <logger name="chapters.configuration" level="INFO" />
  <logger name="chapters.configuration.Foo" level="DEBUG" />

  <root level="DEBUG">
    <appender-ref ref="STDOUT" />
  </root>

</configuration>
```
即，以上设置的日志级别如下所示：     

| Logger 名称 | 设置的日志级别 | 实际日志级别 |    
| ----------- | -------------- | ----------- |      
| root        | DEBUG         | DEBUG      |    
| chapters.configuration |  INFO | INFO  |
| chapters.configuration.MyApp3 | null | INFO |    
| chapters.configuration.Foo | DEBUG | DEBUG |

在本设置中，root的日志级别为DEBUG,而包"chapters.configuration"的日志级别为INFO, "chapters.configuration.Foo"的日志级别为DEBUG,因为"chapter.configuration.MyApp3"没有设置日志级别，且其所属的最内层已设置日志级别的包"chapter.configuration"的日志级别为INFO，因此，此包的日志级别也为INFO.再如以下设置：    

```xml
<configuration>

  <appender name="STDOUT"
   class="ch.qos.logback.core.ConsoleAppender">
   <encoder>
     <pattern>
        %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
      </pattern>
    </encoder>
  </appender>

  <logger name="chapters.configuration" level="INFO" />

  <!-- turn OFF all logging (children can override) -->
  <root level="OFF">
    <appender-ref ref="STDOUT" />
  </root>

</configurat>
其日志级别如下：   

| Logger名称 | 设置的日志级别 | 实际日志级别 |    
| ----------- | ------------- | ----------- |    
| root       | OFF           | OFF        |
| chapter.configuration | INFO | INFO   |    
| chapter.configuration.MyApp3 | null | INFO |    
| chapter.configuration.Foo | null | INFO |    

 ### 4.4 配置Appender
 一个Appender通过`<appender>`元素来配置，该元素有两个必须的属性`name`和`class`。其中，`name`确定该appender的名称，而`class`确定该appender类实例化具体的名称。`<appender>`元素包含0或1个`<layout>`元素，0到多个`<encoder>`元素以及0到多个`<filter>`元素.除了这三个常用元素，`<appender>`也可以包含JavaBean的属性.

 `<layout>`元素包含一个必须的`class`属性，来确定要实例化的layout的具体名称。
`<encoder>`元素包含一个必须的`class`属性，来确定要实例话的encoder的具体名称。

。。。。。

## 5. Appenders

logback将写日志事件的部件叫做Appender.Appender必须implements接口`ch.qos.logback.core.Appender`。接口如下：    
```java
package ch.qos.logback.core;

import ch.qos.logback.core.spi.ContextAware;
import ch.qos.logback.core.spi.FilterAttachable;
import ch.qos.logback.core.spi.LifeCycle;

public interface Appender<E> extends LifeCycle, ContextAware, FilterAttachable {

  public String getName();
  public void setName(String name);
  void doAppend(E event);

}
```

### 5.1 ConsoleAppender    
ConsoleAppender将输出定位到控制台上，更精确的说是System.out或System.err,默认是System.out。一个简单的ConsoleAppenser的配置文件如下：    
```xml
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg %n</pattern>
    </encoder>
  </appender>

  <root level="DEBUG">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```
该配置文件将root的日志级别设置为DEBUG，并打印到控制台上.

### 5.2 FileAppender
该Appender将日志打印到一个文件中，目标文件通过`File`选项来配置，如果该文件已经存在，要么追加到最后，要么覆盖，通过`append`的值来配置。设置如下：
```xml
<configuration>

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>testFile.log</file>
    <append>true</append>
    <!-- set immediateFlush to false for much higher logging throughput -->
    <immediateFlush>true</immediateFlush>
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender>
        
  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```
用户可以通过时间戳来定义唯一的日志文件，在某些应用的开发阶段或者一些运行时间比较短的应用，如：批处理应用，需要每次提交作业时都产生一个新的日志文件。可以通过`<timestamp>`元素很容易的实现。具体如下：    
```xml
<configuration>

  <!-- Insert the current time formatted as "yyyyMMdd'T'HHmmss" under
       the key "bySecond" into the logger context. This value will be
       available to all subsequent configuration elements. -->
  <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"/>

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <!-- use the previously created timestamp to create a uniquely
         named log file -->
    <file>log-${bySecond}.txt</file>
    <encoder>
      <pattern>%logger{35} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```
`<timestamp>`选项有两个必须的属性，`key`和`datePattern`，和一个可选的属性`timeReference`。其中`key`可以作为变量的名称使用，使用该`key`即引用该`<timestamp>`， `datePattern`说明该`<timestamp>`的时间格式，并转化成字符串，格式由Java的`SimpleDateFormat`定义.`timeReference`说明时间戳是何时，默认使用解析配置文件时时的时间，也可以使用内容开始产生的时间，即`timeReference="contextBirth"`，举例如下：    
```java
<configuration>
  <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss" 
             timeReference="contextBirth"/>
  ...
</configuration>
```

### 5.3 RollingFileAppender    
RollingFileAppender扩展了FileAppender,可以滚动生产日志文件。例如， RollingFileAppender创建了一个名为log.txt的日志文件，一旦达到某个条件，将它的日志输出到一个新的文件中。

RollingFileAppender具有两个子组成部分，第一个组成部分叫RollingPolicy,用来定义滚动策略，第二个部分叫触发策略，用来定义何时触发滚动。因此RollingPolicy用来定义what，而TriggerPolicy用来定义when。
#### 5.3.1 RollingPoilicy
* TimeBasedRollingPolicy, 是最受欢迎的滚动策略，滚动策略以时间为基础，如：按天或按月。该策略包括以下几项内容：    

|  属性名称  | 属性类型 | 说明 |    
| --------- | -------- | ---- |    
| fileNamePattern | String | 如： /var/log/myapplication.%d{yyyy-MM-dd}.log |    
| maxHistory | int | 设置最大保存日志的数量，如设置最大数量为6， 每个月产生一个日志文件，当日志数量大于6个时，6个月以上的日志将被异步删除 |    
| totalSizeCap | int | 设置日志的总量，如果日志总量大于设定值时，最旧的日志将被删除 |    
| cleanHistoryOnStart | boolean | 如果设置为true，日志删除将在appender启动时进行，默认为false |    
如果fileNamePattern以.gz或.zip结尾，日志将被压缩。如"/var/log/foo.%d.gz"。
```xml
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>logFile.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- daily rollover -->
      <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>

      <!-- keep 30 days' worth of history capped at 3GB total size -->
      <maxHistory>30</maxHistory>
      <totalSizeCap>3GB</totalSizeCap>

    </rollingPolicy>

    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender> 

  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```
* 以大小和时间为基础的滚动策略     
一些情况下，用户希望按日期同时按每个日志文件的大小进行日志归档，为了达到这个目标，logback使用SizeAndTimeBasedRollingPolicy.同样，也可以设置totalSizeCap，具体如下：   
```xml
<configuration>
  <appender name="ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>mylog.txt</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
      <!-- rollover daily -->
      <fileNamePattern>mylog-%d{yyyy-MM-dd}.%i.txt</fileNamePattern>
       <!-- each file should be at most 100MB, keep 60 days worth of history, but at most 20GB -->
       <maxFileSize>100MB</maxFileSize>    
       <maxHistory>60</maxHistory>
       <totalSizeCap>20GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>


  <root level="DEBUG">
    <appender-ref ref="ROLLING" />
  </root>

</configuration>
```

* FixedWindowRollingPolicy
FixedWindowRollingPolicy将根据以下算法产生日志文件,该策略有3个属性，具体如下：    

| 属性名称 | 类型 | 说明 |   
| ------- | ---- | ---- |     
| minIndex  | int | window的下限 |    
| maxIndex  | int | window的上限 |    
| fileNamePattern| String | 字符串中必须包含%i,如foo%i.log |    
假设将fileNamePattern设置为foo.log,将minIndex设置为1，maxIndex设置为3.步骤如下：  

| 滚动阶段 | 输出 | 日志 |  说明 |   
| ------- | ---- | ---- | ---- |        
| 0 | foo.log | - | 没有滚动发生 |    
| 1 | foo.log | foo1.log | 原来的foo.log将被转存为foo1.log,产生新的foo.log |    
| 2 | foo.log | foo1.log, foo2.log | 原来的foo1.log转存为foo2.log, foo.log转存为foo1.log,并产生新的foo.log |   
| 3 | foo.log | foo1.log, foo2.log, foo3.log | 原来的foo2.log被转存到foo3.log, foo1.log转存到foo2.log, foo.log转存到foo1.log, 并创建新的foo.log |   
| 4 | foo.log | foo1.log, foo2.log, foo3.log | 原来的foo3.log将被删除， foo2.log转存为foo3.log, foo1.log转存为foo2.log, foo.log转存为foo1.log, 并创建新的foo.log日志 |    

一个使用FixedWindowRollingPolicy的配置文件如下：    
```java
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>test.log</file>

    <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
      <fileNamePattern>tests.%i.log.zip</fileNamePattern>
      <minIndex>1</minIndex>
      <maxIndex>3</maxIndex>
    </rollingPolicy>

    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
      <maxFileSize>5MB</maxFileSize>
    </triggeringPolicy>
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender>
        
  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
``` 

#### 5.3.2 TriggeringPolicy
* SizeBasedTriggeringPolicy  
该策略将实时查看当前log文件的大小，如果它超过设定的大小，将触发RollingPolicy.SizeBasedTriggeringPolicy只有一个参数maxFileSize, maxFileSize的单位可以是bytes, kilobytes, megabytes, gigabytes, 缩写为KB, MB以及GB,如：5000000, 5000KB, 5MB, 2GB。配置举例：    
```java
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>test.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
      <fileNamePattern>test.%i.log.zip</fileNamePattern>
      <minIndex>1</minIndex>
      <maxIndex>3</maxIndex>
    </rollingPolicy>

    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
      <maxFileSize>5MB</maxFileSize>
    </triggeringPolicy>
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender>
        
  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```
### 5.4 其他Appender请参阅logback文档    

## 6. Encoders    
Encoder负责将一个日志事件转换成byte数组并将其写到输出流`OutputStream`中。
### 6.1 Pattern
Pattern通过`<pattern>`元素进行设置，使用"%变量"的形式获取具体的信息，常用设置如下：    

| 变量名称 | 说明 |    
| -------- | --- |    
| %d/%date | 获取该日志事件产生的时间，可以通过{}设置日期显示格式，如：%d{HH:mm:ss.SSS} |    
| %thread | 获取该日志事件产生的Thread的名称 |    
| %level | 获取该日志事件的级别 |    
| %logger | 获取该日志事件产生的logger的名称, 后可以跟{number},其中number为显示该logger的字符数量，具体原则可以自行阅读logback的手册 |    
| % msg | 获取该日志事件的具体内容 |    
| %n | 产生回车换行 |    
| -number | 左对齐，number为向后对齐的字符个数，如%-5level, 即level后的信息要左对齐，5为对齐的字符个数 |     

* 使用context name   
  该变量通过`<contextName>`设置，举例： 

```xml
<configuration>
  <contextName>myAppName</contextName>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d %contextName [%t] %level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```
* 定义变量   
 用户可以使用`<property>`对变量进行定义，`<property>`具有两个属性，分别是`name`和`value`具体如下：    

```xml
<configuration>

  <property name="USER_HOME" value="/home/sebastien" />

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>${USER_HOME}/myApp.log</file>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

用户也可以通过`java -DUSER_HOME="XXXX" java程序`对变量进行设置。





