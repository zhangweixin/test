<?xml version="1.0" encoding="utf-8" ?>
<!--zhangweixin 2016-12-08-->
<Configuration status="INFO">
    <!--log4j2官方文档https://logging.apache.org/log4j/2.x/-->
    <Appenders>
        <!--定义常用三种appender-->
        <!--定义轮转日志appender-->
        <Console name="Console" target="SYSTEM_OUT" follow="true">
            <PatternLayout pattern="%d %p %c{1}.%M:%line [%t] %m%n"/>
        </Console>

        <RollingFile name="RollingFile" fileName="${sys:logPath}/log/app.log"
                     filePattern="${sys:logPath}/log/${date:yyyy-MM}/app-${date:dd-MM-yyyy}-%i.log.gz">
            <!--
                重要:在属性值里面可以使用log4j2提供的各种LookUp动态的构造属性值,比如fileName属性使用系统Lookup查找logPath(${sys:logPath})
                LookUp种类以及具体用法文档 http://logging.apache.org/log4j/2.x/manual/lookups.html
                为什么说查找属性重要而且强大那,因为在log4j2运行的时候当我们没有办法设置某个配置值的时候
                我们可以在程序启动的时候设置某些Lookup值,然后在配置中引用这些Lookup,在log4j2初始化的时候会自动获取我们配置的值
                这样就达到动态设置某些属性的目的;本例子中我们通过在web启动的时候设置系统属性logPath来设置日志路径(不要直接在配置文件写日志路径
                因为可能结果路径可能不是你想要的，比如设置成相对路径那么日志路径会跑到tomcat的bin目录下，这肯定不是你想要的)为web项目路径下
            -->
            <PatternLayout>
                <!--定义日志的输出模式
                参数解释
                d:日期等于date,默认的日期输出模式为yyyy-MM-dd HH:mm:ss ,SSS 可以在后面接大括号指定日期格式如:%d{yy-MM-ss HH:mm:ss}
                p:日志等级等于level
                c:类名字,默认输出类的完全限定名如com.test.XXXX;可以在后面跟大括号指定输出长短如:%c{1}输出XXXX,  
                  %c{2}test.XXXX,%c{3}com.test.XXXX %{-1} test.XXXX  %{-2}XXXX
                M:输出日志的方法名字
                line:行号
                t:线程名字,跟T不一样,T指的是输出线程的id
                m:输出的消息
                n:换行
                -->
                <Pattern>%d %p %c{1}.%M:%line [%t] %m%n</Pattern>
            </PatternLayout>

            <!--rolling类型的appender需要TriggeringPolicy和RolloverStrategy两个策略指导rolling-->
            <Policies>
                <!--TriggeringPolicy指导何时触发rolling动作,常用的有三类.
                    CronTriggeringPolicy:基于corn表达式触发rolling 两个属性schedule:指定的corn表达式
                        evaluateOnStartup:是否在启动的时候计算一次是否需要rolling,如果为true则会根据
                                          日志文件最后一次修改时间与当前时间计算是否需要立即rolling
                    SizeBasedTriggeringPolicy:基于日志文件大小是否达到指定的容量触发rolling,属性size指定容量比如2MB,2KB,2GB
                    TimeBasedTriggeringPolicy:基于时间触发rolling两个属性
                        interval:两次触发rolling间隔默认是1,单位与上面filePattern属性指定的打包日志文件时间有关,
                                 比如dd-MM-yyyy单位就是天,yyyy-MM-dd HH单位就是小时
                        modulate:是否对两次打包间隔时间进行调整,如果为true则以0点为起点进行调整，比如为true
                                 interval为4 单位是小时,当前时间是3点则第一次打包时间为4点,下次为8点，14点，16点等等.
                    一般指定几个触发策略组合提供触发动作，只要其中一个触发器触发，则触发rolling动作.
                -->
                <CronTriggeringPolicy schedule="0/3 * * * * ?" evaluateOnStartup="true"/>
                <SizeBasedTriggeringPolicy size="2MB"/>
                <TimeBasedTriggeringPolicy interval="4" modulate="true"/>
            </Policies>
            <!--rolling策略
                作用就是从到小rolling还是从小到大rollingy以及最大rolling数;假设日志初始文件为foo.log 打包文件模式为 foo-%i.log.gz
                max:最大rolling数,最大打包文件为foo-20.log.gz,这个数字与filePattern属性中是否使用%i(保存当前counter)有关
                min:最小rolling数,最小打包文件为foo-1.log.gz
                fileIndex:指定从min还是max开始新的rollover,max的话则index大的是新的打包文件比如foo-2.log.gz比foo-1.log.gz新;
                反之foo-1.log.gz比foo-2.log.gz新
            -->
            <DefaultRolloverStrategy max="20" min="1" fileIndex="min"/>
        </RollingFile>
        <!--定义随机访问轮转日志appender(性能比上面那个好，推荐使用这种)-->
        <RollingRandomAccessFile name="RandomAccessFile" bufferSize="8192" fileName="${sys:logPath}/log/app.log"
                                 filePattern="${sys:logPath}/log/${date:yyyy-MM}/app-${date:dd-MM-yyyy}-%i.log.gz">
            <Policies>
                <CronTriggeringPolicy schedule="0/3 * * * * ?" evaluateOnStartup="true"/>
                <SizeBasedTriggeringPolicy size="2MB"/>
            </Policies>

            <DefaultRolloverStrategy max="10" min="3" fileIndex="max"/>
        </RollingRandomAccessFile>
        <!--另外log4j还支持其他appender比如:kafka,jdbc,nosql...具体参考http://logging.apache.org/log4j/2.x/manual/appenders.html-->
    </Appenders>
    <Loggers>
        <!--定义日志记录器可以包含一个rootLogger和多个子Logger或者只有一个rootLogger所有的logging都委托给它-->
        <!--Logger指定具体包下面的logger配置,可以定义多个指定不同包下的日志配置-->
        <!--additivity如果为true(默认),子类的日志级别会传递到父类logger,比如本配置的root,同一条日志会输出两遍-->
        <Logger name="org.springframework" level="info" additivity="false">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="RandomAccessFile"/>
            <!--可以引用多个不同的appender-->
        </Logger>

        <Logger name="com.test" level="info" additivity="false">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="RandomAccessFile"/>
        </Logger>

        <Root level="DEBUG">
            <AppenderRef ref="RollingFile"/>
            <AppenderRef ref="RandomAccessFile"/>
        </Root>
    </Loggers>
</Configuration>
