# Maven

## 基本概念

Maven是跨平台的项目管理工具。主要服务于基于Java平台的项目构建，依赖管理和项目信息管理。

### 项目构建

　　项目构建过程包括：清理项目→编译项目→测试项目→生成测试报告→打包项目→部署项目

　　理想的项目构建是高度自动化，跨平台，可重用的组件，标准化的。

### 依赖管理

　　依赖指的是jar包之间的相互依赖，Maven管理的方式就是“自动下载项目所需要的jar包，统一管理jar包之间的依赖关系”。

### 好处

- Maven中使用约定，约定java源代码代码必须放在哪个目录下，编译好的java代码又必须放到哪个目录下，这些目录都有明确的约定。
- Maven的每一个动作都拥有一个生命周期，例如执行 mvn install 就可以自动执行编译，测试，打包等构建过程
- 只需要定义一个pom.xml,然后把源码放到默认的目录，Maven帮我们处理其他事情
- 使用Maven可以进行项目高度自动化构建，依赖管理，仓库管理。

## 项目目录约定

MavenProjectRoot(项目根目录)
   |----src
   |     |----main
   |     |         |----java ——存放项目的.java文件
   |     |         |----resources ——存放项目资源文件，如spring, hibernate配置文件
   |     |----test
   |     |         |----java ——存放所有测试.java文件，如JUnit测试类
   |     |         |----resources ——存放项目资源文件，如spring, hibernate配置文件
   |----target ——项目输出位置
   |----pom.xml ----用于标识该项目是一个Maven项目

## 命令

- 编译：mvn clean compile
- 测试 ：mvn clean test
- 打包：mvn clean package
- 安装：mvn clean install 执行安装命令前，会先执行编译、测试、打包命令

常用参数项

- 设置系统属性 mvn -D，最常用的就是跳过test，该处定义的属性在Maven POM or Maven Plugin中同样生效

  mvn install -Dmaven.test.skip=true

- 启用profiles

  mvn package assembly:single -P profileid

- 离线模式，-o

- 针对failure的选项

  -fea 编译结束后显示错误

  -ff 错误后马上停止，默认应该是这个选项

  -fn 无视结果

- verbosity控制

  -e 会把maven执行时候的错误堆栈打出来，对于maven插件的开发者很有用


- -X debug：想要查看完整的依赖踪迹，包含那些因为冲突或者其它原因而被拒绝引入的构件，打开 Maven 的调试标记运行 

  -q quiet 只打印错误

- Dependencies策略

  -U 只是保证SNAPSHOT版本的依赖会更新到最新

  -C -c 对下载的依赖进行checksum

- 不对子工程递归执行，有时候只想install最外层的父pom至本地仓库，可使用-N参数

  mvn -N install

## 坐标

- groupId：组织标识（包名）
- artifactId：项目名称
- version：项目的当前版本
- packaging：项目的打包方式，最为常见的jar和war两种

拥有了统一规范，就可以把查找工作交给机器。











