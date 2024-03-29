# log4j2配置详解
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出-->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
<Configuration status="WARN" monitorInterval="60">
    <!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
    <Properties>
        <Property name="App">third-api</Property>
        <Property name="logDir">/home/migu/portal-third-api/logs</Property>
        <Property name="splitSize">100 MB</Property>
    </Properties>
 
    <Appenders>
        <!-- 输出控制台日志的配置 -->
        <Console name="console" target="SYSTEM_OUT">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="INFO" onMatch="ACCEPT" onMismatch="DENY"/>
            <!-- 输出日志的格式 -->
            <!-- 格式化输出：%date表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度 %msg：日志消息，%n是换行符-->
            <!-- %logger{36} 表示 Logger 名字最长36个字符 -->
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"/>
        </Console>
 
        <!-- 打印出所有的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档 -->
        <RollingFile name="infoLog" fileName="${logDir}/${App}-info.log"
                                 filePattern="${logDir}/${App}-info-%d{yyyy-MM-dd}-%i.log.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS z} [%thread] %-5level %logger{36} %L %M - %msg%xEx%n"/>
            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1， 单位到底是月 天 小时 分钟，根据filePattern配置的日期格式而定，本处的格式为天，则默认为1天-->
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <!--按大小分-->
                <SizeBasedTriggeringPolicy size="${splitSize}"/>
            </Policies>
            <Filters>
                <!-- 只记录info和warn级别信息 -->
                <!--<ThresholdFilter level="error" onMatch="DENY" onMismatch="NEUTRAL"/>-->
                <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
            <!-- 指定每天的最大压缩包个数，默认7个，超过了会覆盖之前的 -->
            <DefaultRolloverStrategy max="1000"/>
        </RollingFile>
 
        <!-- 存储所有error信息 -->
        <RollingFile name="errorLog" fileName="${logDir}/${App}-error.log"
                                 filePattern="${logDir}/${App}-error-%d{yyyy-MM-dd}-%i.log.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS z} [%thread] %-5level %logger{36} %L %M - %msg%xEx%n"/>
            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1， 单位到底是月 天 小时 分钟，根据filePattern配置的日期格式而定，本处的格式为天，则默认为1天-->
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <!--按大小分-->
                <SizeBasedTriggeringPolicy size="${splitSize}"/>
            </Policies>
            <Filters>
                <!-- 只记录error级别信息 -->
                <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
            <!-- 指定每天的最大压缩包个数，默认7个，超过了会覆盖之前的 -->
            <DefaultRolloverStrategy max="1000"/>
        </RollingFile>
 
        <!--大云5分钟宽带查询接口单独打印-->
        <RollingFile name="dayunLog" fileName="${logDir}/${App}-dayunInfo.log"
                     filePattern="${logDir}/${App}-dayunInfo-%d{yyyy-MM-dd}-%i.log.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS z} [%thread] %-5level %logger{36} %L %M - %msg%xEx%n"/>
            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1， 单位到底是月 天 小时 分钟，根据filePattern配置的日期格式而定，本处的格式为天，则默认为1天-->
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
                <!--按大小分-->
                <SizeBasedTriggeringPolicy size="${splitSize}"/>
            </Policies>
            <Filters>
                <!-- 只记录info和warn级别信息 -->
                <!--<ThresholdFilter level="error" onMatch="DENY" onMismatch="NEUTRAL"/>-->
                <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
            <!-- 指定每天的最大压缩包个数，默认7个，超过了会覆盖之前的 -->
            <DefaultRolloverStrategy max="1000"/>
        </RollingFile>
    </Appenders>
 
    <Loggers>
        <!-- root logger 配置,全局配置，默认所有的Logger都继承此配置 -->
        <!-- AsyncRoot - 异步记录日志 - 需要LMAX Disruptor的支持 -->
        <Root level="info">
            <AppenderRef ref="infoLog"/>
            <AppenderRef ref="errorLog"/>
            <AppenderRef ref="console"/>
        </Root>
 
        <!--将logger中的 additivity 属性配置为 false，则这个logger不会将日志流反馈到root中,默认为true。就是日志只会存在一个文件中，不会出现多个文件存在相同日志-->
        <Logger name="dayunLogger" additivity="true" level="INFO">
            <!--<appender-ref ref="sendCodeFile" level="INFO" />-->
            <appender-ref ref="dayunLog" level="INFO" />
        </Logger>
 
        <!--第三方的软件日志级别 -->
        <logger name="org.springframework" level="info" additivity="true">
        </logger>
    </Loggers>
</Configuration>
```
