# spirngmvc学习笔记(2)_入门配置(pom)

## Spring配置

spring配置发展的过程

- xml配置：吧xml文件分放到不同的配置文件里
- 注解配置：申明Bean的注解（@Component,@Service）。应用的基本配置（如数据库配置）用xml，业务配置用注解
- Java配置：SpringBoot推荐使用Java配置，可以更好理解配置的Bean

## pom.xml配置

这是 [源码链接](https://github.com/spring-projects/spring-mvc-showcase/blob/master/pom.xml)

这是[DispatcherServlet说明](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc)

## 1、maven的协作相关属性

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>org.springframework.samples</groupId>
	<artifactId>spring-mvc-showcase</artifactId>
	<name>spring-mvc-showcase</name>
	<packaging>war</packaging>
	<version>1.0.0-BUILD-SNAPSHOT</version>
</project>
```

1. groupId : 组织标识，例如：org.codehaus.mojo，在M2_REPO目录下，将是: org/codehaus/mojo目录。
2. artifactId : 项目名称，例如：my-project，在M2_REPO目录下，将是：org/codehaus/mojo/my-project目录。
3. version : 版本号，例如：1.0，在M2_REPO目录下，将是：org/codehaus/mojo/my-project/1.0目录。
4. packaging : 打包的格式，可以为：pom , jar , maven-plugin , ejb , war , ear , rar , par
5. packaging:打包机制，如pom,jar,maven-plugin,ejb,war,ear,rar,par
6. url:应该是只是写明开发团队的网站，无关紧要，可选
7. classifer:分类

## 2、POM之间的关系

主要用于POM文件的复用。

### a）依赖关系：依赖关系列表（dependency list）是POM的重要部分

```
<dependencies>
		<!-- Spring -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${org.springframework-version}</version>
		</dependency>
</dependencies>
```

1. groupId , artifactId , version :这三个组合标示依赖的具体工程，而且 这个依赖工程必需是maven中心包管理范围内的，如果碰上非开源包，maven支持不了这个包，那么则有有三种 方法处理：

   a.本地安装这个插件install plugin

   例如：mvn install:intall-file -Dfile=non-maven-proj.jar -DgroupId=som.group -DartifactId=non-maven-proj -Dversion=1

   b.创建自己的repositories并且部署这个包，使用类似上面的deploy:deploy-file命令，

   c.设置scope为system,并且指定系统路径。

2. type：默认为jar类型，常用的类型有：jar,ejb-client,test-jar...,可设置plugins中的extensions值为true后在增加 新的类型，

3. scope：是用来指定当前包的依赖范围

   <dependency>中还引入了<scope>，它主要管理依赖的部署。目前<scope>可以使用5个值： 

   - compile，缺省值，适用于所有阶段，会随着项目一起发布。 
   - provided，类似compile，期望JDK、容器或使用者会提供这个依赖。如servlet.jar。 
   - runtime，只在运行时使用，如JDBC驱动，适用运行和测试阶段。 
   - test，只在测试时使用，用于编译和运行测试代码。不会随项目发布。 
   - system，类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它。

4. optional:设置指依赖是否可选，默认为false,即子项目默认都继承，为true,则子项目必需显示的引入，与dependencyManagement里定义的依赖类似 。

5. exclusions：如果X需要A,A包含B依赖，那么X可以声明不要B依赖，只要在exclusions中声明exclusion.

6. exclusion:是将B从依赖树中删除，如上配置，alibaba.apollo.webx不想使用com.alibaba.external  ,但是alibaba.apollo.webx是集成了com.alibaba.external,r所以就需要排除掉.

### b）继承关系：继承其他pom.xml配置的机制。

比如父pom.xml：

```
<project>
  [...]
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.4</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  [...]
</project>
```

在子pom.xml文件继承它的依赖（还可以继承其他的：developers and contributors、plugin lists、reports lists、plugin executions with matching ids、plugin configuration）：

```
[...]
<parent>
<groupId>com.devzuz.mvnbook.proficio</groupId>
  <artifactId>proficio</artifactId>
  <version>1.0-SNAPSHOT</version>
</parent>
[...]
```

### c）聚合关系：用于将多个maven项目聚合为一个大的项目。

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>my-parent</artifactId>
  <version>2.0</version>
  <modules>
    <module>my-project<module>
  </modules>
</project>
```

## 3、属性

maven的属性，是值的占位符，类似EL，类似ant的属性，比如${X}，可用于pom文件任何赋值的位置。有以下分类：

1. env.X：操作系统环境变量，比如${env.PATH}
2. project.x：pom文件中的属性，比如：<project><version>1.0</version></project>，引用方式：${project.version}
3. settings.x：settings.xml文件中的属性，比如：<settings><offline>false</offline></settings>，引用方式：${settings.offline}
4. Java System Properties：java.lang.System.getProperties()中的属性，比如java.home，引用方式：${java.home}
5. 自定义：在pom文件中可以：<properties><installDir>c:/apps/cargo-installs</installDir></properties>，引用方式：${installDir}

## 4、构建设置

build中的主要标签：resources和plugins。

resources：用于排除或包含某些资源文件

plugins：设置构建的插件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

...
	<build>
		<finalName>${project.artifactId}</finalName>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.7.0</version>
				<configuration>
					<source>${java-version}</source>
					<target>${java-version}</target>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-war-plugin</artifactId>
				<configuration>
					<failOnMissingWebXml>false</failOnMissingWebXml>
				</configuration>
			</plugin>
	</build>
</project>
```

1. defaultGoal:默认的目标，必须跟命令行上的参数相同，如：jar:jar,或者与时期parse相同,例如install
2. directory:指定build target目标的目录，默认为$(basedir}/target,即项目根目录下的target
3. finalName:指定去掉后缀的工程名字，例如：默认为${artifactId}-${version}
4. filters:用于定义指定filter属性的位置，例如filter元素赋值filters/filter1.properties,那么这个文件里面就可以定义name=value对，这个name=value对的值就可以在工程pom中通过${name}引用，默认的filter目录是${basedir}/src/main/fiters/
5. resources:描述工程中资源的位置 

### resource配置

1. targetPath:指定build资源到哪个目录，默认是base directory
2. filtering:指定是否将filter文件(即上面说的filters里定义的*.property文件)的变量值在这个resource文件有效,例如上面就指定那些变量值在configuration文件无效。
3. directory:指定属性文件的目录，build的过程需要找到它，并且将其放到targetPath下，默认的directory是${basedir}/src/main/resources
4. includes:指定包含文件的patterns,符合样式并且在directory目录下的文件将会包含进project的资源文件。
5. excludes:指定不包含在内的patterns,如果inclues与excludes有冲突，那么excludes胜利，那些符合冲突的样式的文件是不会包含进来的。
6. testResources:这个模块包含测试资源元素，其内容定义与resources类似，不同的一点是默认的测试资源路径是${basedir}/src/test/resources,测试资源是不部署的。

### plugins配置

1. extensions:true or false, 决定是否要load这个plugin的extensions，默认为true.
2. inherited:是否让子pom继承，ture or false 默认为true.
3. configuration:通常用于私有不开源的plugin,不能够详细了解plugin的内部工作原理，但使plugin满足的properties
4. dependencies:与pom基础的dependencies的结构和功能都相同，只是plugin的dependencies用于plugin,而pom的denpendencies用于项目本身。在plugin的dependencies主要用于改变plugin原来的dependencies，例如排除一些用不到的dependency或者修改dependency的版本等，详细请看pom的denpendencies.
5. executions:plugin也有很多个目标，每个目标具有不同的配置，executions就是设定plugin的目标，

更多详细[标签属性](https://www.cnblogs.com/qq78292959/p/3711501.html)

## 



















