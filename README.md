## 0 ��ѹ�������̸�Ŀ¼

1. �� nexus-private-service.zip ��ѹ�������̵ĸ�Ŀ¼

2. ��ѹSwitchHosts!_windows_portable_3.5.0(5486).zip������Ŀ¼���Թ���ԱȨ������exe�ļ������� **127.0.0.1 private.nexus.net**

## 1 �޸�Ĭ�϶˿�:

```bash
cd nexus-3.19.1-01\bin\etc
�� nexus-default.properties���༭

application-port=3001
application-host=0.0.0.0
```

## 2 nexus��װ������

```bash
# 1����binĿ¼
cd nexus-3.19.1-01\bin
# 2��װ����
nexus.exe /install //��װnexus����
net start nexus //����nexus����

# 3ж�ط���
net stop nexus //�ر�nexus����
nexus.exe /uninstall //ж��nexus����
```

## 3 ���������

```bash
���������http://127.0.0.1:3001/�����Sign in��¼��nexus Ĭ�ϵ��û�����admin������Ϊadmin123
������������ʧ�ܣ���nexus-3.19.1-01\bin·���³���ִ�����������в���
# 1����
nexus restart
# 2ǿ������ˢ�²ֿ�
nexus force-reload
```

## 4 maven˽��jar��Ĭ�ϴ��Ŀ¼

`sonatype-work\nexus3\blobs\default\content`

## 5 ��Ŀ��������˽��

### pom�ļ�����

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

## 6 ��Ŀ������˽���ֿ�

### mvn�����ļ��޸�
maven settings.xml�����ļ�����server **nexus-private-service**

```xml
<servers>
    <server>
        <id>nexus-private-service</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
</servers>
```

### ����˽����ַ����
��Ŀpom�ļ�����**distributionManagement**����˽������:

**repository id������mvn��settings.xml server id����һ��**

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

### ����Դ������
��Ŀpom�ļ�����**maven-source-plugin**����Դ������:

```xml
<project>
    ...
    <build>
        <plugins>
            <!-- Ҫ��Դ�����ȥ����Ҫ���������� -->
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
### �����ĵ�����
��Ŀpom�ļ�����**maven-javadoc-plugin**�����ĵ�����:

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

### ��������˵��

#### mvn deploy

`mvn deploy` ����pom��`<version></version>`�Ƿ��**-SNAPSHOT**������Ŀ��˽����`maven-snapshots`����`maven-releases`

#### mvn deploy -P release

`mvn deploy -P release`������ `maven-releases`,POM�Ķ�����

 * `<version></version>`Ϊ����`<version>${project.version}</version>`
 * ����project.versionΪ1.0.0--SNAPSHOT��`<project.version>1.0.0-SNAPSHOT</project.version>`
 * 
 * ����profiles����profile id=release�����ã��������涨��<project.version>1.0.0</project.version>���������������release��project.version�滻ԭ���Ķ���

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

### POM Deployʾ��

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
	<description>test maven ˽������</description>

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

## 7 ����˽����

### ͨ�����ؿ�Ŀ¼����

1. ������Ŀ��Ŀ¼������settings.xml

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

### ��cmd����������Ŀ������

#### ��������jar

```bash
mvn package -Dmaven.tast.skip=true  -s settings.xml
```

#### ����jar��source
```bash
mvn package -Dmaven.tast.skip=true  -s settings.xml dependency:sources
```

#### ����jar��source��javadoc
```bash
mvn package -Dmaven.tast.skip=true -s settings.xml dependency:sources dependency:resolve -Dclassifier=javadoc
```

ͨ����������֮һ��ɱ��زֿ��˽���ֿ�Ľ�����