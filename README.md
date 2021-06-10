## 0 解压到任意盘根目录

1. 将 nexus-private-service.zip 解压到任意盘的根目录

2. 解压SwitchHosts!_windows_portable_3.5.0(5486).zip到任意目录，以管理员权限运行exe文件，增加 **127.0.0.1 private.nexus.net**

## 1 修改默认端口:

```bash
cd nexus-3.19.1-01\bin\etc
打开 nexus-default.properties并编辑

application-port=3001
application-host=0.0.0.0
```

## 2 nexus安装与运行

```bash
# 1进入bin目录
cd nexus-3.19.1-01\bin
# 2安装服务
nexus.exe /install //安装nexus服务
net start nexus //启动nexus服务

# 3卸载服务
net stop nexus //关闭nexus服务
nexus.exe /uninstall //卸载nexus服务
```

## 3 浏览器访问

```bash
浏览器输入http://127.0.0.1:3001/，点击Sign in登录，nexus 默认的用户名是admin，密码为admin123
如果浏览器访问失败，在nexus-3.19.1-01\bin路径下尝试执行如下命令行操作
# 1重启
nexus restart
# 2强制重新刷新仓库
nexus force-reload
```

## 4 maven私服jar包默认存放目录

`sonatype-work\nexus3\blobs\default\content`

## 5 项目依赖启用私服

### pom文件增加

```xml
<repositories>
...
    <repository>
        <id>nexus-private-service</id>
        <name>nexus private service</name>
        <url>http://private.nexus.net:3001/repository/maven-public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
</pluginRepositories>
	...
    <pluginRepository>
        <id>nexus-private-service</id>
        <name>nexus private service</name>
        <url>http://private.nexus.net:3001/repository/maven-public/</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>
```

## 6 项目发布到私服仓库

### mvn配置文件修改
maven settings.xml配置文件加入server **nexus-private-service**

```xml
<servers>
    <server>
        <id>nexus-private-service</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
</servers>
```

### 发布私服地址配置
项目pom文件加入**distributionManagement**发布私服配置:

**repository id必须与mvn的settings.xml server id配置一致**

```xml
<project>
    ...
	<distributionManagement>
		<repository>
			<id>nexus-private-service</id>
			<url>http://private.nexus.net:3001/repository/maven-releases/</url>
		</repository>
		<snapshotRepository>
			<id>nexus-private-service</id>
			<url>http://private.nexus.net:3001/repository/maven-snapshots/</url>
		</snapshotRepository>
	</distributionManagement>
</project>
```

### 发布源码配置
项目pom文件加入**maven-source-plugin**发布源码配置:

```xml
<project>
    ...
    <build>
        <plugins>
            <!-- 要将源码放上去，需要加入这个插件 -->
            <plugin>
                <artifactId>maven-source-plugin</artifactId>
                <version>2.4</version>
                <configuration>
                    <attach>true</attach>
                </configuration>
                <executions>
                    <execution>
                        <phase>compile</phase>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    ...
</project>
```
### 发布文档配置
项目pom文件加入**maven-javadoc-plugin**发布文档配置:

```xml
<project>
    ...
    <build>
        <plugins>
			<!-- Javadoc -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-javadoc-plugin</artifactId>
                <version>2.9.1</version>
                <configuration>
                    <show>private</show>
                    <nohelp>true</nohelp>
                    <charset>UTF-8</charset>
                    <encoding>UTF-8</encoding>
                    <docencoding>UTF-8</docencoding>
                    <additionalparam>-Xdoclint:none</additionalparam>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    ...
</project>
```

### 发布命令说明

#### mvn deploy

`mvn deploy` 根据pom的`<version></version>`是否带**-SNAPSHOT**发布项目到私服的`maven-snapshots`还是`maven-releases`

#### mvn deploy -P release

`mvn deploy -P release`发布到 `maven-releases`,POM改动如下

 * `<version></version>`为变量`<version>${project.version}</version>`
 * 定义project.version为1.0.0--SNAPSHOT，`<project.version>1.0.0-SNAPSHOT</project.version>`
 * 
 * 增加profiles定义profile id=release的配置，并在下面定义<project.version>1.0.0</project.version>，这样此命令会用release的project.version替换原来的定义

```xml
</project>
	...
	<version>${project.version}</version>
	...
	<properties>
		<project.version>1.0.0-SNAPSHOT</project.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
	</properties>
	...

	<profiles>
		<profile>
			<id>release</id>
			<properties>
				<project.version>1.0.0</project.version>
			</properties>
		</profile>
	</profiles>
</project>
```

### POM Deploy示例

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.test.deploy</groupId>
	<artifactId>test-dependencies-parent</artifactId>
	<version>${project.version}</version>
	<packaging>pom</packaging>

	<name>test-dependencies-parent</name>
	<description>test maven 私服发布</description>

	<properties>
		<project.version>1.0.0-SNAPSHOT</project.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
	</properties>

	<profiles>
		<profile>
			<id>release</id>
			<properties>
				<project.version>1.0.0</project.version>
			</properties>
			<build>
				<plugins>
					<!-- GPG -->
<!--					<plugin>-->
<!--						<groupId>org.apache.maven.plugins</groupId>-->
<!--						<artifactId>maven-gpg-plugin</artifactId>-->
<!--						<version>1.5</version>-->
<!--						<executions>-->
<!--							<execution>-->
<!--								<phase>verify</phase>-->
<!--								<goals>-->
<!--									<goal>sign</goal>-->
<!--								</goals>-->
<!--							</execution>-->
<!--						</executions>-->
<!--					</plugin>-->
<!--					<plugin>-->
<!--						<groupId>org.sonatype.plugins</groupId>-->
<!--						<artifactId>nexus-staging-maven-plugin</artifactId>-->
<!--						<version>1.6.8</version>-->
<!--						<extensions>true</extensions>-->
<!--						<configuration>-->
<!--							<serverId>sonatype</serverId>-->
<!--							<nexusUrl>https://oss.sonatype.org/</nexusUrl>-->
<!--							<autoReleaseAfterClose>true</autoReleaseAfterClose>-->
<!--						</configuration>-->
<!--					</plugin>-->
				</plugins>
			</build>
		</profile>
	</profiles>
	
	<build>
		<plugins>
			<!-- Source -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-source-plugin</artifactId>
				<version>2.2.1</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>jar-no-fork</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<!-- Javadoc -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-javadoc-plugin</artifactId>
				<version>2.9.1</version>
				<configuration>
					<show>private</show>
					<nohelp>true</nohelp>
					<charset>UTF-8</charset>
					<encoding>UTF-8</encoding>
					<docencoding>UTF-8</docencoding>
					<additionalparam>-Xdoclint:none</additionalparam>
				</configuration>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>jar</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
	
	<distributionManagement>
		<repository>
			<id>nexus-private-service</id>
			<url>http://private.nexus.net:3001/repository/maven-releases/</url>
		</repository>
		<snapshotRepository>
			<id>nexus-private-service</id>
			<url>http://private.nexus.net:3001/repository/maven-snapshots/</url>
		</snapshotRepository>
	</distributionManagement>
	
</project>
```

## 7 制作私服库

### 通过本地空目录建立

1. 进入项目根目录，建立settings.xml

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository-->
  <localRepository>./.apache-maven-repo</localRepository>

  <!-- servers
   | This is a list of authentication profiles, keyed by the server-id used within the system.
   | Authentication profiles can be used whenever maven must make a connection to a remote server.
   |-->
  <servers>
	<server>
		<id>nexus-private-service</id>
		<username>admin</username>
		<password>admin123</password>
	</server>
  </servers>
 
</settings>

```

### 打开cmd程序，下载项目依赖包

#### 下载依赖jar

```bash
mvn package -Dmaven.tast.skip=true  -s settings.xml
```

#### 下载jar和source
```bash
mvn package -Dmaven.tast.skip=true  -s settings.xml dependency:sources
```

#### 下载jar和source和javadoc
```bash
mvn package -Dmaven.tast.skip=true -s settings.xml dependency:sources dependency:resolve -Dclassifier=javadoc
```

通过上述命令之一完成本地仓库和私服仓库的建立。