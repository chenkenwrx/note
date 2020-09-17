#### 内存模型以及分区

- 堆

  初始化的对象，成员变量 （那种非 static 的变量），所有的对象实例和数组都要
  在堆上分配

- 栈

  存储局部变量表(指针)，操作数栈，方法出口等信息

- 方法区

  类信息，常量池（static 常量和 static 变量），编译后的代码（字节码）等数据

- 本地方法栈

  主要为 Native 方法服务

- 程序计数器

  记录当前线程执行的行号

#### 堆内存

- **JAVA堆内存是如何划分**

  划分为堆内存和非堆内存，堆内存分为年轻代（Young Generation）、老年代（Old Generation），非堆内存就一个永久代（Permanent Generation）、JDK1.8版本废弃了永久代、替代的是元空间（放在本地内存中）、避免永久代出现的 OOM 问题

  年轻代又分为 Eden 和 Survivor 区。Survivor 区由 FromSpace 和 ToSpace 组成。Eden区占大容量，Survivor两个区占小容量，默认比例是8:1:1

- **分代概念**

  新生成的对象首先放到年轻代Eden区，当Eden空间满了，触发Minor GC、存活下来的对象移动到Survivor0（from）区、Survivor0区满后触发执行Minor GC、Survivor0区存活对象移动到Suvivor1区、保证了一段时间内总有一个survivor区为空、多次Minor GC仍然存活的对象移动到老年代、老年代存储长期存活的对象，占满时会触发**Major GC=Full GC**，GC期间会停止所有线程等待GC完成

- **为什么survivor分为两块相等大小的幸存空间**

  为了解决碎片化。如果内存碎片化严重，也就是两个对象占用不连续的内存，已有的连续内存不够新对象存放，就会触发GC

- **JVM堆内存常用参数**

  |          参数          |                           描述                            |
  | :--------------------: | :-------------------------------------------------------: |
  |          -Xms          |                 堆内存初始大小，单位m、g                  |
  |  -Xmx（MaxHeapSize）   |       堆内存最大允许大小，一般不要大于物理内存的80%       |
  |      -XX:PermSize      | 非堆内存初始大小，一般应用设置初始化200m，最大1024m就够了 |
  |    -XX:MaxPermSize     |                   非堆内存最大允许大小                    |
  |  -XX:NewSize（-Xns）   |                    年轻代内存初始大小                     |
  | -XX:MaxNewSize（-Xmn） |            年轻代内存最大允许大小，也可以缩写             |
  |  -XX:SurvivorRatio=8   |  年轻代中Eden区与Survivor区的容量比例值，默认为8，即8:1   |
  |          -Xss          |                       堆栈内存大小                        |

#### 垃圾回收算法

- **标记-清除**

  - 标记阶段：从根集合出发，将所有活动对象及其子对象打上标记

  - 清除阶段：遍历堆，将非活动对象（未打上标记）的连接到空闲链表上

  - 优点：

    实现简单

  - 缺点：

    碎片化；分配速度不理想，每次分配都需要遍历空闲列表找到足够大的分块

- **标记-压缩**

  - 在标记阶段后它将所有活动对象紧密的排在堆的一侧（压缩），消除了内存碎片

  - 优点：

    有效利用了堆，不会出现内存碎片 也不会像复制算法那样只能利用堆的一部分

  - 缺点：

    压缩过程的开销，需要多次搜索堆

- **引用计数**

  - 记录每个对象被引用的次数、新建对象、赋值引用和删除引用的同时更新计数器，如果计数器值为0则直接回收内存

  - 优点：

    可即刻回收垃圾、最大暂停时间短、不需要开始查找

  - 缺点：

    计数器的增减处理繁重、计数器需要占用很多位、循环引用无法回收

- **复制算法**

  - 为两个大小相同的空间 From 和 To， 利用 From 空间进行分配，当 From 空间满的时候，GC将其中的活动对象复制到 To 空间，之后将两个空间互换即完成GC

  - 优点：

    高速分配； 因为分块是连续的、不会发生碎片化

  - 缺点：

    堆使用率低、与保守式GC不兼容

- **分代收集算法**

#### 垃圾收集器

- 串行收集器

  单线程、收集时，必须暂停应用的工作线程，直到收集结束

- 并行收集器

  多条垃圾收集线程并行工作，在多核CPU下效率更高，应用线程仍然处于等待状态

- CMS收集器

  缩短暂停应用时间为目标而设计的，是基于标记-清除算法实现，整个过程分为4个步骤

  - 初始标记
  - 并发标记
  - 重新标记
  - 并发清除

  初始标记、重新标记这两个步骤仍然需要暂停应用线程、初始标记只是标记一下GC Roots能直接关联到的对象，速度很快，并发标记阶段是标记可回收对象、重新标记阶段则是为了修正并发标记期间因用户程序继续运作导致标记产生变动的那一部分对象的标记记录，这个阶段暂停时间比初始标记阶段稍长一点，但远比并发标记时间短、整个过程中消耗最长的并发标记和并发清除过程收集器线程都可以与用户线程一起工作，所以，CMS收集器内存回收与用户一起并发执行的，大大减少了暂停时间、G1跟踪各个Region里的垃圾堆积价值大小（所获得空间大小以及回收所需时间），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region，从而保证了再有限时间内获得更高的收集效率

  - 初始标记
  - 并发标记
  - 最终标记
  - 筛选回收

- G1收集器

  将堆内存划分多个大小相等的独立区域、预测暂停时间、避免对整个堆进行全区收集

- 垃圾收集器参数

  | -XX:+UseSerialGC                       | 串行收集器                                                   |
  | -------------------------------------- | ------------------------------------------------------------ |
  | -XX:+UseParallelGC                     | 并行收集器                                                   |
  | -XX:+UseParallelGCThreads=8            | 并行收集器线程数，同时有多少个线程进行垃圾回收，一般与CPU数量相等 |
  | -XX:+UseParallelOldGC                  | 指定老年代为并行收集                                         |
  | -XX:+UseConcMarkSweepGC                | CMS收集器（并发收集器）                                      |
  | -XX:+UseCMSCompactAtFullCollection     | 开启内存空间压缩和整理，防止过多内存碎片                     |
  | -XX:CMSFullGCsBeforeCompaction=0       | 表示多少次Full GC后开始压缩和整理，0表示每次Full GC后立即执行压缩和整理 |
  | -XX:CMSInitiatingOccupancyFraction=80% | 表示老年代内存空间使用80%时开始执行CMS收集，防止过多的Full GC |
  | -XX:+UseG1GC                           | G1收集器                                                     |
  | -XX:MaxTenuringThreshold=0             | 在年轻代经过几次GC后还存活，就进入老年代，0表示直接进入老年代 |

- 为什么会堆内存溢出

  - 老年代内存不足：java.lang.OutOfMemoryError:Javaheapspace
  - 永久代内存不足：java.lang.OutOfMemoryError:PermGenspace
  - 代码bug，占用内存无法及时回收

  可以通过添加个参数-XX:+HeapDumpOnOutMemoryError，让虚拟机在出现内存溢出异常时Dump出当前的内存堆转储快照以便事后分析

  - 设置堆内存最小和最大值，最大值参考历史利用率设置
  - 设置GC垃圾收集器为G1
  - 启用GC日志，方便后期分析

#### java 类加载过程

- 加载
  - 通过一个类的全限定名来获取定义此类的二进制流
  - 将字节流所代表的静态存储结构转化为方法区的运行时数据结构
  - 内存中生成一个.class对象，作为方法区这个类的各种数据访问接口
- 验证

~~~java
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.gov.cei</groupId>
    <artifactId>sportsLotteryPlatform</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <jdk.version>1.8</jdk.version>
        <spring.version>4.3.26.RELEASE</spring.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jms</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-oxm</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-instrument-tomcat</artifactId>
            <version>4.3.26.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit</artifactId>
            <version>1.6.11.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-erlang</artifactId>
            <version>1.4.6.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-mongodb</artifactId>
            <version>2.2.5.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-frontend-jaxws</artifactId>
            <version>3.0.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-transports-http-jetty</artifactId>
            <version>3.0.1</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.12</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <dependency>
            <groupId>net.bull.javamelody</groupId>
            <artifactId>javamelody-core</artifactId>
            <version>1.67.0</version>

        </dependency>
        <!-- https://mvnrepository.com/artifact/javax.xml.ws/jaxws-api -->
        <dependency>
            <groupId>javax.xml.ws</groupId>
            <artifactId>jaxws-api</artifactId>
            <version>2.3.1</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish.metro</groupId>
            <artifactId>webservices-rt</artifactId>
            <version>2.4.3</version>
        </dependency>

        <dependency>
            <groupId>org.apache.taglibs</groupId>
            <artifactId>taglibs-standard-impl</artifactId>
            <version>1.2.5</version>
        </dependency>
        <dependency>
            <groupId>org.mongodb.morphia</groupId>
            <artifactId>morphia</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>com.sun.mail</groupId>
            <artifactId>javax.mail</artifactId>
            <version>1.6.2</version>
        </dependency>
        <dependency>
            <groupId>org.libreoffice</groupId>
            <artifactId>unoil</artifactId>
            <version>6.3.2</version>
        </dependency>
        <dependency>
            <groupId>org.mongodb.morphia</groupId>
            <artifactId>morphia-no-proxy-deps-tests</artifactId>
            <version>0.108</version>
        </dependency>
        <dependency>
            <groupId>eu.bitwalker</groupId>
            <artifactId>UserAgentUtils</artifactId>
            <version>1.21</version>
        </dependency>
        <dependency>
            <groupId>commons-beanutils</groupId>
            <artifactId>commons-beanutils</artifactId>
            <version>1.9.4</version>
        </dependency>
        <dependency>
            <groupId>commons-cli</groupId>
            <artifactId>commons-cli</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.13</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-collections4</artifactId>
            <version>4.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-compress</artifactId>
            <version>1.19</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-configuration2</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.9</version>
        </dependency>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging-api</artifactId>
            <version>1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-math3</artifactId>
            <version>3.6.1</version>
        </dependency>
        <dependency>
            <groupId>commons-net</groupId>
            <artifactId>commons-net</artifactId>
            <version>3.6</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.7.0</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-ehcache</artifactId>
            <version>5.4.12.Final</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.70</version>
        </dependency>
        <dependency>
            <groupId>com.whalin</groupId>
            <artifactId>Memcached-Java-Client</artifactId>
            <version>3.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz</artifactId>
            <version>2.3.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>tomcat-jdbc</artifactId>
            <version>9.0.31</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.19</version>
        </dependency>
        <dependency>
            <groupId>org.swinglabs</groupId>
            <artifactId>pdf-renderer</artifactId>
            <version>1.0.5</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/net.sourceforge.jexcelapi/jxl -->
        <dependency>
            <groupId>net.sourceforge.jexcelapi</groupId>
            <artifactId>jxl</artifactId>
            <version>2.6.12</version>
        </dependency>

        <dependency>
            <groupId>xalan</groupId>
            <artifactId>xalan</artifactId>
            <version>2.7.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/cn.jpush.api/jpush-client -->
        <dependency>
            <groupId>cn.jpush.api</groupId>
            <artifactId>jpush-client</artifactId>
            <version>3.3.13</version>
        </dependency>
        <dependency>
            <groupId>com.ceis</groupId>
            <artifactId>core-base</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.apache.ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.10.7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.solr</groupId>
            <artifactId>solr-solrj</artifactId>
            <version>4.3.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.redisson/redisson -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-redis</artifactId>
            <version>1.8.2.RELEASE</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>jcl-over-slf4j</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring-context-support</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring-oxm</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.jdom2/jdom -->
        <dependency>
            <groupId>org.jdom</groupId>
            <artifactId>jdom2</artifactId>
            <version>2.0.6</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.htmlparser</groupId>
            <artifactId>htmlparser</artifactId>
            <version>2.1</version>
        </dependency>
        <dependency>
            <groupId>net.htmlparser.jericho</groupId>
            <artifactId>jericho-html</artifactId>
            <version>3.4</version>
        </dependency>
        <dependency>
            <groupId>joda-time</groupId>
            <artifactId>joda-time</artifactId>
            <version>2.10.3</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/javax.servlet.jsp/javax.servlet.jsp-api -->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.3</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>xerces</groupId>
            <artifactId>xercesImpl</artifactId>
            <version>2.12.0</version>
        </dependency>

        <!--自定义坐标-->
        <dependency>
            <groupId>org.mozilla.intl</groupId>
            <artifactId>chardet</artifactId>
            <version>1.0</version>
        </dependency>
        <dependency>
            <groupId>com.baidu</groupId>
            <artifactId>ueditor</artifactId>
            <version>1.1.2</version>
        </dependency>
        <dependency>
            <groupId>com.weld</groupId>
            <artifactId>osgi</artifactId>
            <version>bundle</version>
        </dependency>
        <dependency>
            <groupId>javax.xml.rpc</groupId>
            <artifactId>javax.xml.rpc-api</artifactId>
            <version>1.1.1</version>
        </dependency>
        <!-- axis 1.4 jar start -->
        <dependency>
            <groupId>org.apache.axis</groupId>
            <artifactId>axis</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>commons-discovery</groupId>
            <artifactId>commons-discovery</artifactId>
            <version>0.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.axis</groupId>
            <artifactId>axis-jaxrpc</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.axis</groupId>
            <artifactId>axis-saaj</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>wsdl4j</groupId>
            <artifactId>wsdl4j</artifactId>
            <version>1.6.3</version>
        </dependency>
        <!-- axis 1.4 jar end -->
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.6</version>
        </dependency>
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.12</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5.1</version>
                <configuration>
                    <source>${jdk.version}</source>
                    <target>${jdk.version}</target>
                    <encoding>${project.build.sourceEncoding}</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-release-plugin</artifactId>
                <version>2.5.3</version>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>${basedir}/src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
        </resources>
    </build>


</project>

~~~



####  java 类加载机制