# Logback日志使用

## logback的介绍

Logback是由log4j创始人设计的另一个开源日志组件,官方网站：http://logback.qos.ch 。它当前分为下面下个模块：

- `logback-core`:其它两个模块的基础模块
- `logback-classic`:它是log4j的一个改良版本，同时它完整实现了slf4j API使你可以很方便地更换成其它日志系统如log4j或JDK14 Logging
- `logback-access`:访问模块与Servlet容器集成提供通过Http来访问日志的功能

## logback取代log4j的理由

1. 更快的实现：Logback的内核重写了，在一些关键执行路径上性能提升10倍以上。而且logback不仅性能提升了，初始化内存加载也更小了。
2. 非常充分的测试：Logback经过了几年，数不清小时的测试。Logback的测试完全不同级别的。
3. Logback-classic非常自然实现了SLF4j：Logback-classic实现了SLF4j。在使用SLF4j中，你都感觉不到logback-classic。而且因为logback-classic非常自然地实现了slf4j ， 所 以切换到log4j或者其他，非常容易，只需要提供成另一个jar包就OK，根本不需要去动那些通过SLF4JAPI实现的代码。
4. 非常充分的文档 官方网站有两百多页的文档。
5. 自动重新加载配置文件，当配置文件修改了，Logback-classic能自动重新加载配置文件。扫描过程快且安全，它并不需要另外创建一个扫描线程。这个技术充分保证了应用程序能跑得很欢在JEE环境里面。
6. Lilith是log事件的观察者，和log4j的chainsaw类似。而lilith还能处理大数量的log数据 。
7. 谨慎的模式和非常友好的恢复，在谨慎模式下，多个FileAppender实例跑在多个JVM下，能 够安全地写道同一个日志文件。RollingFileAppender会有些限制。Logback的FileAppender和它的子类包括 RollingFileAppender能够非常友好地从I/O异常中恢复。
8. 配置文件可以处理不同的情况，开发人员经常需要判断不同的Logback配置文件在不同的环境下（开发，测试，生产）。而这些配置文件仅仅只有一些很小的不同，可以通过,和来实现，这样一个配置文件就可以适应多个环境。
9. Filters（过滤器）有些时候，需要诊断一个问题，需要打出日志。在log4j，只有降低日志级别，不过这样会打出大量的日志，会影响应用性能。在Logback，你可以继续 保持那个日志级别而除掉某种特殊情况，如alice这个用户登录，她的日志将打在DEBUG级别而其他用户可以继续打在WARN级别。要实现这个功能只需加4行XML配置。可以参考MDCFIlter 。
10. SiftingAppender（一个非常多功能的Appender）：它可以用来分割日志文件根据任何一个给定的运行参数。如，SiftingAppender能够区别日志事件跟进用户的Session，然后每个用户会有一个日志文件。
11. 自动压缩已经打出来的log：RollingFileAppender在产生新文件的时候，会自动压缩已经打出来的日志文件。压缩是个异步过程，所以甚至对于大的日志文件，在压缩过程中应用不会受任何影响。
12. 堆栈树带有包版本：Logback在打出堆栈树日志时，会带上包的数据。
13. 自动去除旧的日志文件：通过设置TimeBasedRollingPolicy或者SizeAndTimeBasedFNATP的maxHistory属性，你可以控制已经产生日志文件的最大数量。如果设置maxHistory 12，那那些log文件超过12个月的都会被自动移除。

## logback使用

- maven依赖

  ```xm
  <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.21</version>
  </dependency>
  <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.1.7</version>
  </dependency>
  <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-access</artifactId>
      <version>1.1.7</version>
  </dependency>
  ```

  logback-classic包含了logback-core，不需要再单独引用了。

- 配置

  logback的配置文件都放在/src/main/resource/文件夹下的logback.xml文件中。其中logback.xml文件就是logback的配置文件。只要将这个文件放置好了之后，系统会自动找到这个配置文件。（配置见下文）

- 使用

  我们使用org.slf4j.LoggerFactory，就可以直接使用日志了。

  ```java
  private static final Logger logger = LoggerFactory.getLogger(this.getClass());
  ```

  使用：

  ```java
  logger.debug("DEBUG TEST 这个地方输出DEBUG级别的日志");  
  logger.info("INFO test 这个地方输出INFO级别的日志");
  logger.warn("INFO test 这个地方输出WARN级别的日志");
  logger.error("ERROR test 这个地方输出ERROR级别的日志");  
  ```

## logback的配置介绍

- `logger`、`appender`及`layout`

　　`Logger`作为日志的记录器，把它关联到应用的对应的context上后，主要用于存放日志对象，也可以定义日志类型、级别。
　　`Appender`主要用于指定日志输出的目的地，目的地可以是控制台、文件、远程套接字服务器、 MySQL、PostreSQL、 Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等。 
　　`Layout` 负责把事件转换成字符串，格式化的日志信息的输出。

- `logger context`

　　各个logger 都被关联到一个 LoggerContext，LoggerContext负责制造logger，也负责以树结构排列各logger。其他所有logger也通过org.slf4j.LoggerFactory 类的静态方法getLogger取得。 getLogger方法以 logger名称为参数。用同一名字调用LoggerFactory.getLogger 方法所得到的永远都是同一个logger对象的引用。

- 有效级别及级别的继承

　　Logger 可以被分配级别。级别包括：TRACE、DEBUG、INFO、WARN 和 ERROR，定义于ch.qos.logback.classic.Level类。如果 logger没有被分配级别，那么它将从有被分配级别的最近的祖先那里继承级别。root logger 默认级别是 DEBUG。

- 打印方法与基本的选择规则

　　打印方法决定记录请求的级别。例如，如果 L 是一个 logger 实例，那么，语句 L.info("..")是一条级别为 INFO的记录语句。记录请求的级别在高于或等于其 logger 的有效级别时被称为被启用，否则，称为被禁用。记录请求级别为 p，其 logger的有效级别为 q，只有则当 p>=q时，该请求才会被执行。
该规则是 logback 的核心。级别排序为： TRACE < DEBUG < INFO < WARN < ERROR

## logback的配置详解

### 根节点`Configuration`

- `scan`:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。

- `scanPeriod`:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。

- `debug`:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

  ```xml
  <configuration scan="true" scanPeriod="60 seconds" debug="false">  
  	<!-- 其他配置省略-->
  	...
  </configuration> 
  ```

### 子节点`contextName`

每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用`contextName`设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改。

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false">  
      <contextName>myAppName</contextName>  
      <!-- 其他配置省略-->  
</configuration>
```

### 子节点**`property`**

用来定义变量值的标签，`property` 有两个属性，`name`和`value`；其中name的值是变量的名称，value的值时变量定义的值。通过`property`定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。

> 例如使用`property`定义上下文名称，然后在`contextName`设置logger上下文时使用。

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false">  
      <property name="APP_Name" value="myAppName" />   
      <contextName>${APP_Name}</contextName>  
      <!-- 其他配置省略-->  
</configuration> 
```

### 子节点`timestamp`

获取时间戳字符串

两个属性 key：标识此`timestamp` 的名字；`datePattern`：设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循`java.txt.SimpleDateFormat`的格式。

例如将解析配置文件的时间作为上下文名称：

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false">  
      <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"/>   
      <contextName>${bySecond}</contextName>  
      <!-- 其他配置省略-->  
</configuration>
```

### 子节点`logger`和`root`

1. `logger`

   用来设置某一个包或者具体的某一个类的日志打印级别、以及指定<appender>。<logger>仅有一个name属性，一个可选的level和一个可选的addtivity属性。

   - `name`:

   用来指定受此loger约束的某一个包或者具体的某一个类。

   - `level`:

   用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。

   如果未设置此属性，那么当前loger将会继承上级的级别。

   - `addtivity`:

   是否向上级loger传递打印信息。默认是true。

   `logger`可以包含零个或多个`appender-ref`元素，标识这个appender将会添加到这个loger。

2. `root`

   也是`logger`元素，但是它是根**logger**。只有一个`level`属性，应为已经被命名为"root".

   - `level`:

     用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，不能设置为INHERITED或者同义词NULL。

     默认是DEBUG。

     `root`可以包含零个或多个`appender-ref`元素，标识这个appender将会添加到这个logger。

3. 案例：

   ```java
   package logback;  
     
   import org.slf4j.Logger;  
   import org.slf4j.LoggerFactory;  
     
   public class LogbackDemo {  
       private static Logger log = LoggerFactory.getLogger(LogbackDemo.class);  
       public static void main(String[] args) {  
           log.trace("======trace");  
           log.debug("======debug");  
           log.info("======info");  
           log.warn("======warn");  
           log.error("======error");  
       }  
   }
   ```

   logback.xml配置文件:

   1. 只配置`root`

      ```xml
      <configuration>   
         
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">   
          <!-- encoder 默认配置为PatternLayoutEncoder -->   
          <encoder>   
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>   
          </encoder>   
        </appender>   
         
        <root level="INFO">             
          <appender-ref ref="STDOUT" />   
        </root>  
       </configuration> 
      ```

      其中appender的配置表示打印到控制台(稍后详细讲解appender )；&lt;root level="INFO"&gt;将root的打印级别设置为“INFO”，指定了名字为“STDOUT”的appender。

      当执行logback.LogbackDemo类的main方法时，root将级别为“INFO”及大于“INFO”的日志信息交给已经配置好的名为“STDOUT”的appender处理，“STDOUT”appender将信息打印到控制台；

      运行结果：

      ```log
      13:30:38.484 [main] INFO  logback.LogbackDemo - ======info  
      13:30:38.500 [main] WARN  logback.LogbackDemo - ======warn  
      13:30:38.500 [main] ERROR logback.LogbackDemo - ======error 
      ```

   2. 带有logger的配置，不指定级别，不指定appender

      ```xml
      <configuration>   
         
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">   
          <!-- encoder 默认配置为PatternLayoutEncoder -->   
          <encoder>   
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>   
          </encoder>   
        </appender>   
         
        <!-- logback为java中的包 -->   
        <logger name="logback"/>   
         
        <root level="DEBUG">             
          <appender-ref ref="STDOUT" />   
        </root>     
           
       </configuration>
      ```

      其中appender的配置表示打印到控制台(稍后详细讲解appender )；

      &lt;logger name="logback" /&gt;将控制logback包下的所有类的日志的打印，但是并没用设置打印级别，所以继承他的上级&lt;root&gt;的日志级别“DEBUG”；

      没有设置addtivity，默认为true，将此loger的打印信息向上级传递；

      没有设置appender，此loger本身不打印任何信息。

      &lt;root level="DEBUG"&gt;将root的打印级别设置为“DEBUG”，指定了名字为“STDOUT”的appender。

      运行结果：

      ```log
      13:19:15.406 [main] DEBUG logback.LogbackDemo - ======debug  
      13:19:15.406 [main] INFO  logback.LogbackDemo - ======info  
      13:19:15.406 [main] WARN  logback.LogbackDemo - ======warn  
      13:19:15.406 [main] ERROR logback.LogbackDemo - ======error  
      ```

   3. 带有多个logger的配置，指定级别，指定appender

      ```xml
      <configuration>   
         <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">   
          <!-- encoder 默认配置为PatternLayoutEncoder -->   
          <encoder>   
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>   
          </encoder>   
        </appender>   
         
        <!-- logback为java中的包 -->   
        <logger name="logback"/>   
        <!--logback.LogbackDemo：类的全路径 -->   
        <logger name="logback.LogbackDemo" level="INFO" additivity="false">  
          <appender-ref ref="STDOUT"/>  
        </logger>   
          
        <root level="ERROR">             
          <appender-ref ref="STDOUT" />   
        </root>     
      </configuration>  
      ```

      其中appender的配置表示打印到控制台(稍后详细讲解appender )；

      &lt;logger name="logback"/&gt;将控制logback包下的所有类的日志的打印，但是并没用设置打印级别，所以继承他的上级&lt;root&gt;的日志级别"DEBUG"；

      没有设置addtivity，默认为true，将此loger的打印信息向上级传递；

      没有设置appender，此loger本身不打印任何信息。

       &lt;logger name="logback.LogbackDemo" level="INFO" additivity="false"&gt;控制logback.LogbackDemo类的日志打印，打印级别为“INFO”；

      additivity属性为false，表示此logger的打印信息不再向上级传递，

      指定了名字为"STDOUT"的appender。

      &lt;root level="DEBUG"&gt;将root的打印级别设置为"ERROR"，指定了名字为“STDOUT”的appender。

       当执行logback.LogbackDemo类的main方法时，先执行&lt;logger name="logback.LogbackDemo" level="INFO" additivity="false"&gt;，将级别大于等于"INFO"的日志信息交给此logger指定的名为"STDOUT"的appender处理，在控制台中打出日志，不再向次logger的上级 &lt;logger name="logback"/&gt;传递打印信息；

      &lt;logger name="logback"/&gt;未接到任何打印信息，当然也不会给它的上级root传递任何打印信息；

      运行结果：

      ```log
      14:05:35.937 [main] INFO  logback.LogbackDemo - ======info  
      14:05:35.937 [main] WARN  logback.LogbackDemo - ======warn  
      14:05:35.937 [main] ERROR logback.LogbackDemo - ======error
      ```

      如果将&lt;logger name="logback.LogbackDemo" level="INFO" additivity="false"&gt;修改为 &lt;logger name="logback.LogbackDemo" level="INFO" additivity="true"&gt;那打印结果将是什么呢？

      没错，日志打印了两次，想必大家都知道原因了，因为打印信息向上级传递，logger本身打印一次，root接到后又打印一次

      运行结果：

      ```xml
      14:09:01.531 [main] INFO  logback.LogbackDemo - ======info  
      14:09:01.531 [main] INFO  logback.LogbackDemo - ======info  
      14:09:01.531 [main] WARN  logback.LogbackDemo - ======warn  
      14:09:01.531 [main] WARN  logback.LogbackDemo - ======warn  
      14:09:01.531 [main] ERROR logback.LogbackDemo - ======error  
      14:09:01.531 [main] ERROR logback.LogbackDemo - ======error
      ```

### 子节点`appender`

&lt;appender&gt;是&lt;configuration&gt;的子节点，是负责写日志的组件。

&lt;appender&gt;有两个必要属性`name`和`class`。name指定appender名称，class指定appender的全限定名。

1. **`ConsoleAppender`**

   把日志添加到控制台，有以下子节点：

   - &lt;encoder&gt;：对日志进行格式化。（具体参数稍后讲解 ）

   - &lt;target&gt;：字符串 *System.out* 或者 *System.err* ，默认 *System.out* ；

   ```xml
   <configuration>  
     
     <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">  
       <encoder>  
         <pattern>%-4relative [%thread] %-5level %logger{35} - %msg %n</pattern>  
       </encoder>  
     </appender>  
     
     <root level="DEBUG">  
       <appender-ref ref="STDOUT" />  
     </root>  
   </configuration> 
   ```

2. `FileAppender`

   把日志添加到文件，有以下子节点：

   - &lt;file&gt;：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。

   - &lt;append&gt;：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。

   - &lt;encoder&gt;：对记录事件进行格式化。（具体参数稍后讲解 ）

   - &lt;prudent&gt;：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。

   ```xml
   <configuration>  
     
     <appender name="FILE" class="ch.qos.logback.core.FileAppender">  
       <file>testFile.log</file>  
       <append>true</append>  
       <encoder>  
         <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>  
       </encoder>  
     </appender>  
             
     <root level="DEBUG">  
       <appender-ref ref="FILE" />  
     </root>  
   </configuration>
   ```

3. `RollingFileAppender`

   滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：

   - &lt;file&gt;：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。

   - &lt;append&gt;：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。

   - &lt;encoder&gt;：对记录事件进行格式化。

     负责两件事，一是把日志信息转换成字节数组，二是把字节数组写入到输出流。

     目前**PatternLayoutEncoder** 是唯一有用的且默认的**encoder** ，有一个<pattern>节点，用来设置日志的输入格式。使用“%”加“转换符”方式，如果要输出“%”，则必须用“\”对“\%”进行转义。

   - &lt;rollingPolicy&gt;:当发生滚动时，决定 **RollingFileAppender** 的行为，涉及文件移动和重命名。

     - `TimeBasedRollingPolicy`

       最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。有以下子节点：

       - &lt;fileNamePattern&gt;:必要节点，包含文件名及“%d”转换符， “%d”可以包含一个`java.text.SimpleDateFormat指定的时间格式，如：%d{yyyy-MM}。如果直接使用 %d，默认格式是 yyyy-MM-dd。`**RollingFileAppender** 的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；如果没设置file，活动文件的名字会根据**fileNamePattern** 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。
       - &lt;maxHistory&gt;:可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且&lt;maxHistory&gt;是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除。

     - `FixedWindowRollingPolicy`

       根据固定窗口算法重命名文件的滚动策略。有以下子节点：

       + &lt;minIndex&gt;:窗口索引最小值
       + &lt;maxIndex&gt;:窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。
       + &lt;fileNamePattern &gt;:必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip

   - &lt;triggeringPolicy&gt;: 告知 **RollingFileAppender** 合适激活滚动。

     - `SizeBasedTriggeringPolicy`

       查看当前活动文件的大小，如果超过指定大小会告知**RollingFileAppender** 触发当前活动文件滚动。只有一个节点：

       - &lt;maxFileSize&gt;:这是活动文件的大小，默认值是10MB。

       例如：每天生成一个日志文件，保存30天的日志文件。

       ```xml
       <configuration>   
         <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">   
             
           <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">   
             <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>   
             <maxHistory>30</maxHistory>    
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

       例如：按照固定窗口模式生成日志文件，当文件大于20MB时，生成新的日志文件。窗口大小是1到3，当保存了3个归档文件后，将覆盖最早的日志。

       ```xml
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

   - &lt;prudent&gt;：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空。

   &lt;pattern&gt;里面的转换符说明：**



| 转换符                                                       | 作用                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| **c** {*length* } <br>**lo** {*length* }  <br>**logger** {*length* } | 输出日志的logger名，可有一个整形参数，功能是缩短logger名，设置为0表示只输入logger最右边点符号之后的字符串。<br>Conversion specifier      Logger name                 Result<br>%logger          mainPackage.sub.sample.Bar   mainPackage.sub.sample.Bar<br>%logger{0}    mainPackage.sub.sample.Bar    Bar<br/>%logger{5}    mainPackage.sub.sample.Bar    m.s.s.Bar<br/>%logger{10}  mainPackage.sub.sample.Bar    m.s.s.Bar<br>%logger{15}  mainPackage.sub.sample.Bar    m.s.sample.Bar<br>%logger{16}  mainPackage.sub.sample.Bar    m.sub.sample.Bar<br>%logger{26}  mainPackage.sub.sample.Bar    mainPackage.sub.sample.Bar |
| **C** {*length* }  <br>**class** {*length* }                 | 输出执行记录请求的调用者的全限定名。参数与上面的一样。尽量避免使用，除非执行速度不造成任何问题。 |
| **contextName** <br>**cn**                                   | 输出上下文名称。                                             |
| **d** {*pattern* }  <br/>**date** {*pattern* }               | 输出日志的打印日志，模式语法与`java.text.SimpleDateFormat` 兼容。Conversion Pattern                                            Result<br>%d                                                                             2006-10-20 14:06:49,812<br/>%date                                                                      2006-10-20 14:06:49,812<br/>%date{ISO8601}                                                  2006-10-20 14:06:49,812<br/>%date{HH:mm:ss.SSS}                                     14:06:49.812<br/>%date{dd MMM yyyy ;HH:mm:ss.SSS}       20 oct. 2006;14:06:49.812 |
| **F / file**                                                 | 输出执行记录请求的java源文件名。尽量避免使用，除非执行速度不造成任何问题。 |
| **caller{depth}caller{depth, evaluator-1, ... evaluator-n}** | 输出生成日志的调用者的位置信息，整数选项表示输出信息深度。<br>例如， **%caller{2}**   输出为：<br/>`0    [main] DEBUG - logging statement  Caller+0   at mainPackage.sub.sample.Bar.sampleMethodName(Bar.java:22) Caller+1   at mainPackage.sub.sample.Bar.createLoggingRequest(Bar.java:17)<br/>`例如， **%caller{3}**   输出为：<br/>`16   [main] DEBUG - logging statement  Caller+0   at mainPackage.sub.sample.Bar.sampleMethodName(Bar.java:22) Caller+1   at mainPackage.sub.sample.Bar.createLoggingRequest(Bar.java:17) Caller+2   at mainPackage.ConfigTester.main(ConfigTester.java:38)` |
| **L / line**                                                 | 输出执行日志请求的行号。尽量避免使用，除非执行速度不造成任何问题。 |
| **m / msg / message**                                        | 输出应用程序提供的信息。                                     |

  参考：https://logback.qos.ch/manual/layouts.html#conversionWord

 **格式修饰符，与转换符共同使用：**

   可选的格式修饰符位于“%”和转换符之间。

   第一个可选修饰符是**左对齐** 标志，符号是减号“-”；接着是可选的**最小宽度** 修饰符，用十进制数表示。如果字符小于最小宽度，则左填充或右填充，默认是左填充（即右对齐），填充符为空格。如果字符大于最小宽度，字符永远不会被截断。**最大宽度** 修饰符，符号是点号"."后面加十进制数。如果字符大于最大宽度，则从前面截断。点符号“.”后面加减号“-”在加数字，表示从尾部截断。

   例如：%-4relative 表示，将输出从程序启动到创建日志记录的时间 进行左对齐 且最小宽度为4。

4. 另外还有SocketAppender、SMTPAppender、DBAppender、SyslogAppender、SiftingAppender，并不常用，这些就不在这里讲解了，大家可以参考官方文档。当然大家可以编写自己的Appender。

## logback配置总览

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true-->
<!--scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。
    当scan为true时，此属性生效。默认的时间间隔为1分钟。-->
<!--debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。-->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!--每个logger都关联到logger上下文，默认上下文名称为“default”。
        但可以使用<contextName>设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改,可以通过%contextName来打印日志上下文名称-->
    <contextName>default</contextName>
    <!--用来定义变量值的标签，<property> 有两个属性，name和value；
        其中name的值是变量的名称，value的值时变量定义的值。
        通过<property>定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。-->
    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <property name="SYS_LOG_DIR" value="D:/Application/Logs/"/>
    <!--<property name="SYS_LOG_DIR" value="/usr/simtest/logs/"/>-->
    <property name="INFO_FILE" value="test_info.log"/>
    <property name="ERROR_FILE" value="test_error.log"/>
    <property name="fileLayoutPattern"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n"/>
    <property name="consoleLayoutPattern"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{50} - %msg%n"/>

    <!-- 彩色日志 -->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr"
                    converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wex"
                    converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx"
                    converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
	<!-- 彩色日志另外的写法 -->
    <!--
	<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <Pattern>%date{yyyy-MM-dd HH:mm:ss} | %highlight(%-5level) | %boldYellow(%thread) | %boldGreen(%logger)|%msg%n</Pattern>
        </encoder>
	-->

    <!--appender用来格式化日志输出节点，有俩个属性name和class，class用来指定哪种输出策略，常用就是控制台输出策略和文件输出策略。-->
    <!--<encoder>表示对日志进行编码：
            %d{HH: mm:ss.SSS}——日志输出时间
            %thread——输出日志的进程名字，这在Web应用以及异步任务处理中很有用
            %-5level——日志级别，并且使用5个字符靠左对齐(级别从左显示5个字符宽度)
            %logger{36}——日志输出者的名字
            %msg——日志消息
            %n——平台的换行符-->
    <appender name="INFO_LOG_ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${SYS_LOG_DIR}/${INFO_FILE}</file>
        <!-- 日志过滤器，只记录warn级别日志 ,若要配置其他级别，复制appender，修改level，并在root中配置-->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${SYS_LOG_DIR}/%d{yyyy-MM-dd}/${INFO_FILE}_%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <!--日志文件最大的大小-->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>10</maxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${fileLayoutPattern}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <layout>
            <pattern>${fileLayoutPattern}</pattern>
        </layout>
    </appender>

    <appender name="ERROR_LOG_ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${SYS_LOG_DIR}/${ERROR_FILE}</file>
        <!-- 日志过滤器，只记录warn级别日志 ,若要配置其他级别，复制appender，修改level，并在root中配置-->
        <!--比较日志记录请求的Level值和LevelFilter中配置的Level值，返回LevelFilter中配置的结果值。-->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <!-- deny all events with a level below INFO, that is TRACE and DEBUG -->
        <!--比较日志记录请求的Level值和ThresholdFilter中配置的Level值，当日志记录请求的Level值小于ThresholdFilter中配置的Level值，日志记录请求被判定为无效。-->
        <!--<filter class="ch.qos.logback.classic.filter.ThresholdFilter">-->
        <!--<level>INFO</level>-->
        <!--</filter>-->

        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${SYS_LOG_DIR}/%d{yyyy-MM-dd}/${ERROR_FILE}_%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>10</maxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${fileLayoutPattern}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <layout>
            <pattern>${fileLayoutPattern}</pattern>
        </layout>
    </appender>

    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <layout name="StandardFormat" class="ch.qos.logback.classic.PatternLayout">
            <pattern>${consoleLayoutPattern}</pattern>
        </layout>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!--关于过滤器onMatch表示匹配大于这个级别的日志和onMismatch表示匹配小于于这个级别的日志,可以选的有
                ACCEPT–打印
                DENY– 不打印
                NEUTRAL–中立-->
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 控制台彩色格式输出设置 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--<logger> 用来设置某一个包或者具体的某一个类的日志打印级别、以及指定<appender>。
        <logger>仅有一个name属性，一个可选的level和一个可选的additivity属性。
        name:用来指定受此logger约束的某一个包或者具体的某一个类。
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
              还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。如果未设置此属性，那么当前loger将会继承上级的级别。
        addtivity:是否向上级loger传递打印信息。默认是true。-->
    <!--<logger name="com.asiainfo" level="DEBUG" additivity="false">
      <appender-ref ref="CONSOLE"/>
    </logger>-->
    <logger name="org.apache.zookeeper.ClientCnxn" level="INFO" additivity="false">
        <appender-ref ref="CONSOLE"/>
    </logger>
    <logger name="java.sql.PreparedStatement" level="DEBUG"/>
    <logger name="java.sql.Connection" level="DEBUG"/>
    <logger name="java.sql.Statement" level="DEBUG"/>
    <logger name="com.ibatis" level="DEBUG"/>
    <logger name="com.ibatis.common.jdbc.SimpleDataSource" level="DEBUG"/>
    <logger name="com.ibatis.common.jdbc.ScriptRunner" level="DEBUG"/>
    <logger name="com.ibatis.sqlmap.engine.impl.SqlMapClientDelegate" level="DEBUG"/>

    <!--打印SQL-->
    <logger name="java.sql.Connection" level="DEBUG"/>
    <logger name="java.sql.Statement" level="DEBUG"/>
    <logger name="java.sql.PreparedStatement" level="DEBUG"/>
    <!-- 这一句至关重要如果没有，就无法输出sql语句 -->
    <logger name="com.vailter" level="DEBUG"></logger>
    <!-- <logger name="org.springframework" level="ERROR" />  -->
    <logger name="net.sf.ehcache" level="ERROR"/>
    <logger name="com.mchange" level="ERROR"/>
    <logger name="c3p0.provider" level="ERROR"/>
    <logger name="com.zaxxer" level="INFO"/>
    <logger name="com.alibaba" level="ERROR"/>

    <!--据不同环境（prod:生产环境，test:测试环境，dev:开发环境）来定义不同的日志输出，
        在 logback-spring.xml中使用 springProfile 节点来定义，方法如下：
        文件名称不是logback.xml，想使用spring扩展profile支持，要以logback-spring.xml命名-->
    <!--可以启动服务的时候指定 profile （如不指定使用默认），如指定prod 的方式为：
        java -jar xxx.jar -spring.profiles.active=prod-->
    <springProfile name="dev">
        <logger name="com.vailter" level="info"/>
    </springProfile>
    <springProfile name="prod">
        <logger name="com.vailter" level="debug"/>
    </springProfile>

    <!--<root level="INFO">-->
    <root>
        <!-- 打印debug级别日志及以上级别日志 -->
        <level value="DEBUG"/>
        <!-- 控制台输出 -->
        <!--<appender-ref ref="CONSOLE"/>-->
        <!-- 控制台彩色输出 -->
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="INFO_LOG_ROLLING"/> <!-- 文件输出 -->
        <appender-ref ref="ERROR_LOG_ROLLING"/> <!-- 文件输出 -->
    </root>
</configuration>
```

## Spring Boot配置

application.properties配置：

```properties
# 日志配置
# 日志配置文件的位置。 例如对于Logback的`classpath：logback.xml`
logging.config= 
# ％wEx#记录异常时使用的转换字。
logging.exception-conversion-word= 
# 日志文件名。 例如`myapp.log`
logging.file= 
# 日志级别严重性映射。 例如`logging.level.org.springframework =  DEBUG`
logging.level.*= 
# 日志文件的位置。 例如`/ var / log`
logging.path= 
# 用于输出到控制台的Appender模式。 只支持默认的logback设置。
logging.pattern.console= 
# 用于输出到文件的Appender模式。 只支持默认的logback设置。
logging.pattern.file= 
# 日志级别的Appender模式（默认％5p）。 只支持默认的logback设置。
logging.pattern.level= 
#注册日志记录系统的初始化挂钩。
logging.register-shutdown-hook= false
```



```xml
	<!--据不同环境（prod:生产环境，test:测试环境，dev:开发环境）来定义不同的日志输出，
        在 logback-spring.xml中使用 springProfile 节点来定义，方法如下：
        文件名称不是logback.xml，想使用spring扩展profile支持，要以logback-spring.xml命名-->
    <!--可以启动服务的时候指定 profile （如不指定使用默认），如指定prod 的方式为：
        java -jar xxx.jar -spring.profiles.active=prod-->
    <springProfile name="dev">
        <logger name="com.vailter" level="info"/>
    </springProfile>
    <springProfile name="prod">
        <logger name="com.vailter" level="debug"/>
    </springProfile>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出 -->
<!-- scan:当此属性设置为true时，配置文档如果发生改变，将会被重新加载，默认值为true -->
<!-- scanPeriod:设置监测配置文档是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。
                 当scan为true时，此属性生效。默认的时间间隔为1分钟。 -->
<!-- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。 -->
<configuration  scan="true" scanPeriod="10 seconds">
    <contextName>logback</contextName>

    <!-- name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入到logger上下文中。定义后，可以使“${}”来使用变量。 -->
    <property name="log.path" value="G:/logs/pmp" />

    <!--0. 日志格式和颜色渲染 -->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

    <!--1. 输出到控制台-->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>
        <encoder>
            <Pattern>${CONSOLE_LOG_PATTERN}</Pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--2. 输出到文档-->
    <!-- 2.1 level为 DEBUG 日志，时间滚动输出  -->
    <appender name="DEBUG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/web_debug.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志归档 -->
            <fileNamePattern>${log.path}/web-debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录debug级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>debug</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 2.2 level为 INFO 日志，时间滚动输出  -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/web_info.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天日志归档路径以及格式 -->
            <fileNamePattern>${log.path}/web-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录info级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 2.3 level为 WARN 日志，时间滚动输出  -->
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/web_warn.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/web-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录warn级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 2.4 level为 ERROR 日志，时间滚动输出  -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/web_error.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/web-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录ERROR级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--
        <logger>用来设置某一个包或者具体的某一个类的日志打印级别、
        以及指定<appender>。<logger>仅有一个name属性，
        一个可选的level和一个可选的addtivity属性。
        name:用来指定受此logger约束的某一个包或者具体的某一个类。
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
              还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。
              如果未设置此属性，那么当前logger将会继承上级的级别。
        addtivity:是否向上级logger传递打印信息。默认是true。
        <logger name="org.springframework.web" level="info"/>
        <logger name="org.springframework.scheduling.annotation.ScheduledAnnotationBeanPostProcessor" level="INFO"/>
    -->

    <!--
        使用mybatis的时候，sql语句是debug下才会打印，而这里我们只配置了info，所以想要查看sql语句的话，有以下两种操作：
        第一种把<root level="info">改成<root level="DEBUG">这样就会打印sql，不过这样日志那边会出现很多其他消息
        第二种就是单独给dao下目录配置debug模式，代码如下，这样配置sql语句会打印，其他还是正常info级别：
        【logging.level.org.mybatis=debug logging.level.dao=debug】
     -->

    <!--
        root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
        不能设置为INHERITED或者同义词NULL。默认是DEBUG
        可以包含零个或多个元素，标识这个appender将会添加到这个logger。
    -->

    <!-- 4. 最终的策略 -->
    <!-- 4.1 开发环境:打印控制台-->
    <springProfile name="dev">
        <logger name="com.sdcm.pmp" level="debug"/>
    </springProfile>

    <root level="info">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="DEBUG_FILE" />
        <appender-ref ref="INFO_FILE" />
        <appender-ref ref="WARN_FILE" />
        <appender-ref ref="ERROR_FILE" />
    </root>

    <!-- 4.2 生产环境:输出到文档
    <springProfile name="pro">
        <root level="info">
            <appender-ref ref="CONSOLE" />
            <appender-ref ref="DEBUG_FILE" />
            <appender-ref ref="INFO_FILE" />
            <appender-ref ref="ERROR_FILE" />
            <appender-ref ref="WARN_FILE" />
        </root>
    </springProfile> -->

</configuration>
```

