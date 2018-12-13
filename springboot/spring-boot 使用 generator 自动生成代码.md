# spring-boot 使用 generator 自动生成代码

## 1. 添加`pom.xml` 中`jar`包依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.mac</groupId>
	<artifactId>springboot-jdbc</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>springboot-jdbc</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<!-- mysql集成 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.6</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- mybatis集成 -->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.0.0</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>
				spring-boot-starter-data-elasticsearch
			</artifactId>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.sun.jna/jna -->
		<dependency>
			<groupId>com.sun.jna</groupId>
			<artifactId>jna</artifactId>
			<version>3.0.9</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-batch -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-batch</artifactId>
		</dependency>
		<dependency>
			<groupId>org.xmlunit</groupId>
			<artifactId>xmlunit-core</artifactId>
		</dependency>
		<dependency>
			<groupId>ant</groupId>
			<artifactId>ant</artifactId>
			<version>1.6.5</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>

            <!-- Mybatis自动生成代码插件 -->
			<plugin>
				<groupId>org.mybatis.generator</groupId>
				<artifactId>mybatis-generator-maven-plugin</artifactId>
				<version>1.3.1</version>
				<configuration>
					<!-- 这个xml为本项目中generatorConfig.xml的地址-->
					<configurationFile>${basedir}/src/main/resources/generator/generatorConfig.xml</configurationFile>
					<overwrite>true</overwrite>
					<verbose>true</verbose>
				</configuration>
			</plugin>

		</plugins>
	</build>


</project>
```

## 2. 配置`mybatis generator` 自动生成代码插件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
		PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
		"http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
	<!-- 数据库驱动:选择你的本地硬盘上面的数据库驱动包-->
	<classPathEntry  location="/users/singleton/.m2/repository/mysql/mysql-connector-java/5.1.6/mysql-connector-java-5.1.6.jar"/>
	<context id="DB2Tables"  targetRuntime="MyBatis3">
		<commentGenerator>
			<property name="suppressDate" value="true"/>
			<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
			<property name="suppressAllComments" value="true"/>
		</commentGenerator>
		<!--数据库连接驱动类,URL，用户名、密码 -->
		<jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://127.0.0.1/test" userId="root" password="root">
		</jdbcConnection>
		<javaTypeResolver>
			<property name="forceBigDecimals" value="false"/>
		</javaTypeResolver>
		<!-- 生成(实体)模型的包名和位置-->
		<javaModelGenerator targetPackage="com.mac.springbootjdbc.school.model.student" targetProject="src/main/java">
			<property name="enableSubPackages" value="true" />
			<property name="trimStrings" value="true"/>
		</javaModelGenerator>
		<!-- 生成XML映射文件的包名和位置-->
		<sqlMapGenerator targetPackage="resources.mappers.school.student" targetProject="src/main">
			<property name="enableSubPackages" value="true"/>
		</sqlMapGenerator>
		<!-- 生成DAO接口的包名和位置-->
		<javaClientGenerator type="XMLMAPPER" targetPackage="com.mac.springbootjdbc.school.mapper.student" targetProject="src/main/java">
			<property name="enableSubPackages" value="true"/>
		</javaClientGenerator>
		<!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName是实体类名-->
		<table tableName="student" domainObjectName="Student" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false">
            <!-- 如果设置为true，生成的model类会直接使用column本身的名字，而不会再使用驼峰命名方法，比如BORN_DATE，生成的属性名字就是BORN_DATE,而不会是bornDate -->
			<property name="useActualColumnNames" value="true" />
		</table>
	</context>
</generatorConfiguration>
```

> 注意：上述代码中的  `targetProject` 一定要写对，不然可能代码生成器即可能找不到路径也可能生成文件地址不对，还要手动移动太麻烦！！！

## 3. 自动生成代码操作步骤

1. 点击run-**--->** Edit Configurations
2. 点击弹框左上角的`+`号,  选择Maven
3. Working directory 文本框中是项目路径
4. Commond line 选择框填写 `mybatis-generator:generate -e`
5. 点击`OK`, 即可生成代码

## 4. 完成

[参考链接地址](https://blog.csdn.net/Winter_chen001/article/details/77249029)