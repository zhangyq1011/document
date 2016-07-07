#开发服务器、测试服务器、正式服务器各个项目日志打印及保存路径

#1.添加应用服务器过滤
为配合日志记录明细及内容需要在web项目中的web.xml中添加一下过滤器：

       <filter>
		<description>设置日志上下文过滤器</description>
		<filter-name>slcf</filter-name>
		<filter-class>com.huaying.common.web.filter.log.SetLoggingContextFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>slcf</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

    
该过滤器抄袭阿里日志框架记录详细的访问URL有重要价值
	
	
# 2. 声明日志存储路径
日志保存根目录：/program/logs/[mall|pay|sys]

# 3.日志配置基础配置文件
日志基础配置文件(logback.properties) 内容如下:

	#日志存储路径
	LOG_PATH=D/program/logs/mall/mall-sso
	smtpHost=smtp.163.com  
	smtpPort=25  
	username=13103712371@163.com  
	password=XXXXXX  
	SSL=false  
	email_to=759534585@qq.com,zyq@huaqinwang.com,dgf@huaqinwang.com,sdd@huaqinwang.com,wwg@huaqinwang.com,jdd@huaqinwang.com,ztd@huaqinwang.com,zxj@huaqinwang.com,yxy@huaqinwang.com  
	email_from=13103712371@163.com  
	email_subject=\u3010Error\u3011\: %logger  
	localhost=www.huaqinwang.com
	#日志项目名称雷同项目名称 唯一
	contextName=mall-sso
    
#4.定义日志appender

## 4.1 定义出错发送邮件appender

<configuration>
	 <!--↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓ 以下内容直接添加 ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓ -->
	<property file="./logback.properties" />
	<!--<property name="LOG_PATH" value="./logs/mall/mall-sso" />-->
	<contextName>${contextName}</contextName>
	
	<appender name="EMAIL" class="ch.qos.logback.classic.net.SMTPAppender">  
        <smtpHost>${smtpHost}</smtpHost>  
        <smtpPort>${smtpPort}</smtpPort>  
        <username>${username}</username>  
        <password>${password}</password>
        <localhost>${localhost}</localhost>  
        <SSL>${SSL}</SSL>  
        <asynchronousSending>true</asynchronousSending>  
        <to>${email_to}</to>  
        <from>${email_from}</from>  
        <subject>${email_subject}</subject>  
        <layout class="ch.qos.logback.classic.html.HTMLLayout" >  
            <!-- <pattern>%date%level%thread%logger{0}%line%message</pattern>   -->
            <pattern>%date%level%thread%logger{0}%line%message%-4r [%d{yyyy-MM-dd HH:mm:ss}] - %X{remoteAddr} %X{requestURIWithQueryString} %X{referrer} %X{userAgent} %X{cookie.MALL_JSESSIONID_SID} %C:%L - %m%n
            </pattern>
            
        </layout>  
        <filter class="ch.qos.logback.core.filter.EvaluatorFilter">    
            <evaluator class="ch.qos.logback.classic.boolex.JaninoEventEvaluator">  
                <expression>  <!-- &amp;&amp; null != throwable -->
                    if(level > WARN ) {  
                        return true;  
                    }  
                    return false;  
                </expression>    
            </evaluator> 
            <onMatch>ACCEPT</onMatch>    
            <onMismatch>DENY</onMismatch>      
        </filter>  
	</appender>  
	
	<!--↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑ 以上内容直接添加  ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑ -->
    
## 4.2 定义不同包日志保存到不同的日志文件中

如：把所有的dubbo的日志存储到huaying-dubbo-info.log日志文件中：
### 4.2.1 定义特定的appender

	<appender name="huayingDubboAppender"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${LOG_PATH}/huaying-dubbo-info.log</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<fileNamePattern>${LOG_PATH}/huaying-dubbo-info-%d{yyyy-MM-dd}.%i.log
			</fileNamePattern>
			<timeBasedFileNamingAndTriggeringPolicy
				class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
				<maxFileSize>50MB</maxFileSize>
			</timeBasedFileNamingAndTriggeringPolicy>
		</rollingPolicy>
		<append>true</append>
		<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
			<pattern>%-5p [%d][%mdc{mdc_accNo}] %C:%L - %m %n</pattern>
			<charset>utf-8</charset>
		</encoder>
		<!-- 此日志文件只记录info级别，不记录大于info级别的日志 -->
		<filter class="ch.qos.logback.classic.filter.LevelFilter">
			<level>INFO</level>
			<onMatch>ACCEPT</onMatch>
			<onMismatch>DENY</onMismatch>
		</filter>
	</appender>
    
### 4.2.2 定义logger

	<logger name="com.huaying.common.web.dubbo" level="INFO" additivity="false">  
	    <appender-ref ref="huayingDubboAppender" />  
	</logger>
    

## 4.3 定义其他日志级别的appender
原来配置文件已经有无需再次添加

# 5.定义日志 logger
可以使用如下定义：

<logger name="com.huaying.mall.sso.web" level="INFO" />

# 6.将appender添加到 root配置项中

	<root   >
		<appender-ref ref="EMAIL" level="ERROR"/>  		
	</root>
	
	
