前言：本项目基于maven构建

spring-boot项目可以快速构建web应用，其内置的tomcat容器也十分方便我们的测试运行；


spring-boot项目需要部署在外部容器中的时候，spring-boot导出的war包无法再外部容器（tomcat）中运行或运行报错，本章就是详细讲解如何解决这个问题
1、pom.xml一览

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    	<groupId>项目名称自行自行定义</groupId>
    	<artifactId>项目名称自行定义</artifactId>
    	<version>1.1.2-SNAPSHOT</version>
    	<packaging>war</packaging>
    	<parent>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-parent</artifactId>
    		<version>1.4.0.RELEASE</version>
    	</parent>
    	<dependencies>
    		<!-- spring-boot web -->
    		<dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-web</artifactId>
    			<!-- 排除内置容器，排除内置容器导出成war包可以让外部容器运行spring-boot项目-->
    			<exclusions>
    				<exclusion>
    					<groupId>org.springframework.boot</groupId>
    					<artifactId>spring-boot-starter-tomcat</artifactId>
    				</exclusion>
    			</exclusions>
    			 
    		</dependency>
    		<dependency>
    			<groupId>javax.servlet</groupId>
    			<artifactId>javax.servlet-api</artifactId>
    		</dependency>
    		<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
    		<dependency>
    			<groupId>redis.clients</groupId>
    			<artifactId>jedis</artifactId>
    		</dependency>
     
    		<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-pool2 -->
    		<dependency>
    			<groupId>org.apache.commons</groupId>
    			<artifactId>commons-pool2</artifactId>
    		</dependency>
     
    		<dependency>
    			<groupId>org.json</groupId>
    			<artifactId>json</artifactId>
    		</dependency>
     
    		<dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-test</artifactId>
    			<scope>test</scope>
    		</dependency>
    		<dependency>
    			<groupId>com.jayway.jsonpath</groupId>
    			<artifactId>json-path</artifactId>
    			<scope>test</scope>
    		</dependency>
    	</dependencies>
    	
    	<!-- Package as an executable jar -->
    	<build>
    		<plugins>
    			<plugin>
    				<groupId>org.springframework.boot</groupId>
    				<artifactId>spring-boot-maven-plugin</artifactId>
    			</plugin>
    		</plugins>
    	</build>
    	<repositories>
    		<repository>
    			<id>spring-releases</id>
    			<url>https://repo.spring.io/libs-release</url>
    		</repository>
    	</repositories>
    	<pluginRepositories>
    		<pluginRepository>
    			<id>spring-releases</id>
    			<url>https://repo.spring.io/libs-release</url>
    		</pluginRepository>
    	</pluginRepositories>
    </project>


2、排除org.springframework.boot依赖中的tomcat内置容器

注意：只有排除内置容器，才能让外部容器运行spring-boot项目

org.springframework.boot依赖中的排除项，测试时不要排除内置容器，会导致springboot无法测试运行，导出war包时再加上该项即可

<!-- spring-boot web -->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
<!-- 排除内置容器，排除内置容器导出成war包可以让外部容器运行spring-boot项目-->
<exclusions>
<exclusion>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-tomcat</artifactId>
</exclusion>
</exclusions>
</dependency>
3、实现SpringBootServletInitializer接口

spring-boot入口类必须实现SpringBootServletInitializer接口的configure方法才能让外部容器运行spring-boot项目

注意：SpringBootServletInitializer接口需要依赖 javax.servlet

    package cn.eguid.Run;
     
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.builder.SpringApplicationBuilder;
    import org.springframework.boot.web.support.SpringBootServletInitializer;
    import org.springframework.context.annotation.ComponentScan;
     
    import cc.eguid.livepush.PushManager;
    import cc.eguid.livepush.PushManagerImpl;
    import cn.eguid.livePushServer.redisManager.RedisMQHandler;
     
    @SpringBootApplication
    // 开启通用注解扫描
    @ComponentScan
    public class Application extends SpringBootServletInitializer {
    	/**
    	 * 实现SpringBootServletInitializer可以让spring-boot项目在web容器中运行
    	 */
    	@Override
    	protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
    		builder.sources(this.getClass());
    		return super.configure(builder);
    	}
    	
    	public static void main(String[] args) {
    		SpringApplication.run(Application.class, args);
    		
    	}
    }


--------------------- 
作者：做好自己--eguid 
来源：CSDN 
原文：https://blog.csdn.net/eguid_1/article/details/52609600?utm_source=copy 
版权声明：本文为博主原创文章，转载请附上博文链接！
总结：

外部容器运行spring-boot项目，只需要在原项目上做两件事
1、在pom.xml中排除org.springframework.boot的内置tomcat容器

2、spring-boot入口实现SpringBootServletInitializer接口
补充：SpringBootServletInitializer接口依赖javax.servlet包，需要在pom.xml中引入该包
--------------------- 
作者：做好自己--eguid 
来源：CSDN 
原文：https://blog.csdn.net/eguid_1/article/details/52609600?utm_source=copy 
版权声明：本文为博主原创文章，转载请附上博文链接！
