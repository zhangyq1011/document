# 华禽网分布式配置管理服务

参考一下网址：http://192.168.1.85:2040/index.jsp，请按照一下步骤 配置使用。

  - 0、项目中添加 pom 引用
  - 1、创建配置文件类
  - 2、使用配置文件类
  - 3、添加Spring配置文件
  - 4、添加disconf配置文件
  - 5、验证使用 配置文件管理成功
  


## 0、项目中添加 pom 引用
在项目pom.xml文件中添加一下pom引用:

		<!-- 使用百度提供的 统一配置文件管理方案 -->
		<dependency>
		    <groupId>com.baidu.disconf</groupId>
		    <artifactId>disconf-client</artifactId>
		    <version>2.6.31</version>
		    <exclusions>
		    	<exclusion>
		    		<groupId>org.apache.httpcomponents</groupId>
            		<artifactId>httpclient</artifactId>
		    	</exclusion>
		    </exclusions>
		</dependency>
		<!-- END 使用百度提供的 统一配置文件管理方案 END-->
	注意：因为disconf-client 使用的httpclient版本为4.5.1 与本地项目使用版本有冲突因此请使用如下pom替换原来的引用:
		<dependency>
			<groupId>com.huaying.common</groupId>
			<artifactId>huaying-common</artifactId>
			<version>0.0.1-SNAPSHOT</version>
			 <exclusions>
		    	<exclusion>
		    		<groupId>org.apache.httpcomponents</groupId>
            		<artifactId>httpclient</artifactId>
		    	</exclusion>
		    	<exclusion>
		    		 <groupId>org.apache.httpcomponents</groupId>
            		 <artifactId>httpmime</artifactId>
		    	</exclusion>
		    	<exclusion>
		    		 <groupId>org.apache.httpcomponents</groupId>
            		<artifactId>httpcore-nio</artifactId>
		    	</exclusion>
		    	<exclusion>
		    		<groupId>org.apache.httpcomponents</groupId>
            		<artifactId>httpcore</artifactId>
		    	</exclusion>
		    </exclusions>
		</dependency>
## 1、创建配置文件类
创建配置文件类：


@Service
@Scope("singleton")
@DisconfFile(filename = "sys.properties")
public class SysConf {

	private static String system_type_list_map;
	private static String sys_menu_parent_id;
	
	
	@DisconfFileItem(name = "SYSTEM_TYPE_LIST_MAP", associateField = "system_type_list_map")
	public static String getSystem_type_list_map() {
		return system_type_list_map;
	}
	public static void setSystem_type_list_map(String system_type_list_map) {
		SysConf2.system_type_list_map = system_type_list_map;
	}
	
	@DisconfFileItem(name = "SYS_MENU_PARENT_ID", associateField = "sys_menu_parent_id")
	public static String getSys_menu_parent_id() {
		return sys_menu_parent_id;
	}
	public static void setSys_menu_parent_id(String sys_menu_parent_id) {
		SysConf2.sys_menu_parent_id = sys_menu_parent_id;
	}

}
## 2、使用配置文件类
使用方法比较简单：
SysConf.getSystem_type_name() 就可以使用了

## 3、添加Spring配置文件
引用或者添加一下配置 内容：

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
						http://www.springframework.org/schema/beans/spring-beans.xsd
						http://www.springframework.org/schema/aop 
						http://www.springframework.org/schema/aop/spring-aop.xsd
						http://www.springframework.org/schema/context 
						http://www.springframework.org/schema/context/spring-context.xsd
						http://www.springframework.org/schema/tx 
						http://www.springframework.org/schema/tx/spring-tx.xsd">

	<!-- 启用注解 -->
	<context:annotation-config />
	
	
	<aop:aspectj-autoproxy proxy-target-class="true"/>

	<!-- 启动组件扫描，排除@Controller组件，该组件由SpringMVC配置文件扫描 -->
	<context:component-scan base-package="com.huaying.sys.utils.open.sso.disconf,com.huaying.sys.auth.disconf">
		<context:exclude-filter type="annotation"
			expression="org.springframework.stereotype.Controller" />
	</context:component-scan>


	<!-- 使用disconf必须添加以下配置 -->
	<bean id="disconfMgrBean" class="com.baidu.disconf.client.DisconfMgrBean"
		destroy-method="destroy">
		<property name="scanPackage" 
				  value="com.huaying.sys.utils.open.sso.disconf" />
	</bean>
	<bean id="disconfMgrBean2" class="com.baidu.disconf.client.DisconfMgrBeanSecond"
		init-method="init" destroy-method="destroy">
	</bean>
</beans>

## 4、添加disconf配置文件
在项目类路径下添加 配置文件名称为：disconf.properties 内容如下：

enable.remote.conf=true

conf_server_host=192.168.1.85:2040

version=1_0_0_0

app=disconf_sys

env=rd

debug=true

ignore=

conf_server_url_retry_times=1
conf_server_url_retry_sleep_seconds=1


## 5、验证使用 配置文件管理成功
如何验证呢？




