# Maven的使用

### Maven常用打包/编译插件

| plugin                   | function                                                     |
| :----------------------- | :----------------------------------------------------------- |
| maven-compiler-plugin    | 用于指定编译环境，默认使用JDK1.5进行编译                     |
| maven-surefire-plugin    | 用于指定单元测试类的打包方式，一般配置<skip>true</skip>跳过打包 |
| spring-boot-maven-plugin | 用于springboot项目打包，包含依赖jar                          |
| maven-jar-plugin         | maven 默认打包插件，不包含依赖jar（需要配合maven-dependency-plugin） |
| maven-shade-plugin       | 包含依赖jar，生产中常用这种                                  |
| maven-assembly-plugin    | 支持定制化打包方式，生产中常用这种                           |

### SpringBoot项目引入Maven插件

##### **第一步：在spring-boot的启动项目Module中增加以下配置**

```
        <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>${java.version}</source> <!-- 源代码使用的JDK版本 -->
                    <target>${java.version}</target> <!-- 需要生成的目标class文件的编译版本 -->
                    <encoding>UTF-8</encoding>  <!-- 字符集编码 -->
                    <compilerArguments>
                        <verbose/>
                        <bootclasspath>${java.home}/lib/rt.jar${path.separator}${java.home}/lib/jce.jar</bootclasspath>  <!-- 这个选项用来传递编译器自身不包含但是却支持的参数选项 -->
                    </compilerArguments>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <skip>true</skip> <!--跳过maven打包-->
                </configuration>
            </plugin>
        </plugins>
    </build>
```

第二步：在需要打包的成jar或者war（original）的项目Module的pom.xml中增加：

```
分两种情况，第一：基于spring-boot-starter-web的项目
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.2.0.RELEASE</version>
            </plugin>
        </plugins>
    </build>
第二：普通java项目，需要指定主启动类
	<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>线程通信.死锁</mainClass> <!--指定主启动类-->
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal> <!--去除重复依赖-->
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

### MVN的常用命令

```
mvn compile //编译源代码
mvn test-compile //编译测试代码
mvn test //运行应用程序中的单元测试
mvn site //生成项目相关信息的网站
mvn clean //清除目标目录中的生成结果
mvn package //依据项目生成 jar 文件
mvn install //在package的基础上，将打包的jar上传到本地仓库
mvn deploy //在install的基础上，将打包的jar上传到远程私服仓库
mvn dependency:tree //打印依赖树
```

### 单独打包Module

```
对于Maven工程下多个Module，有时候需要单独打包其中的某一个或多个Module，具体操作：
参数介绍：
-pl,--projects
        Build specified reactor projects instead of all projects
-am,--also-make
        If project list is specified, also build projects required by the list

如果工程P下有A/B/C三个Module
①只打包B，mvn package -pl B -am
③全部打包，mvn package
```

### scala的mvn命令

```
mvn scala:compile compile
```

#### 解决日志框架冲突

```
很多框架中都集成了日志框架，常见的日志框架有：
①log4j，没有实现slf4j接口，老的框架用的比较多
②slf4j-log4j12，实现了slf4j接口
③logback，实现了slf4j接口

新的日志框架大多实现了slf4j接口，经常会发生slf4j-log4j12与logback两种框架发生冲突
解决办法：
①查看项目依赖树：idea -> terminal -> mvn dependency:tree
②如果想统一使用logback框架，就找到引用slf4j-log4j12依赖有哪些
③在子模块module的pom中加上
    <exclusions>
        <exclusion>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
```

### 打包常见报错

```
错误一：
	 No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.
原因：
解决：
```

