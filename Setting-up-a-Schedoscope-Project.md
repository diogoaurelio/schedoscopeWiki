Schedoscope is an internal Scala DSL for specifying views (Hive table partitions), their structure and dependencies, as well as the transformation logic required compute views from other views. As a consequence, setting up a Schedoscope project means setting up a Scala project that uses Schedoscope as a library. 

For this purpose, we provide a Maven POM template in this section. It is of course possible to use other build tools such as SBT or Ant/Ivy. You are also encouraged to look at the POM of the tutorial.  The template sets up mixed compilation of Scala and Java code. 

For running Schedoscope, the template utilizes the exec Maven plugin which assembles a classpath from the Maven dependencies and launches the Schedoscope REST service right out of the project folder.

In real-world production deployment scenarios, you should probably follow a different [deployment / bundling](Bundling and Deploying) / [launching strategy](Starting Schedoscope). 

# Maven POM Template

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <groupId>my-projects-group-id</groupId>
        <artifactId>my-project</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <name>Schedoscope POM template</name>
        <description>A template making it easier for you to set up a Schedoscope project</description>

		<dependencies>
			<dependency>
				<groupId>schedoscope</groupId>
				<artifactId>schedoscope-core</artifactId>
				<version>0.3.0</version>
			</dependency>
			<dependency>
				<groupId>schedoscope</groupId>
				<artifactId>schedoscope-tutorial-osm-data</artifactId>
				<version>0.1.1</version>
			</dependency>
			<dependency>
				<groupId>ch.hsr</groupId>
				<artifactId>geohash</artifactId>
				<version>1.0.13</version>
			</dependency>
			<dependency>
				<groupId>org.scalatest</groupId>
				<artifactId>scalatest_2.11</artifactId>
				<version>2.2.5</version>
				<scope>test</scope>
			</dependency>
			<dependency>
				<groupId>hadoop-launcher</groupId>
				<artifactId>hadoop-launcher</artifactId>
				<version>0.1.0</version>
				<scope>test</scope>
			</dependency>
			<dependency>
				<groupId>minioozie</groupId>
				<artifactId>minioozie</artifactId>
				<version>0.1.8</version>
				<scope>test</scope>
				<exclusions>
					<exclusion>
						<groupId> org.cloudera.logredactor</groupId>
						<artifactId>logredactor</artifactId>
					</exclusion>
				</exclusions>
			</dependency>
		</dependencies>

		<build>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<version>3.3</version>
					<configuration>
						<source>1.7</source>
						<target>1.7</target>
					</configuration>
					<executions>
						<execution>
							<id>default-compile</id>
							<phase>none</phase>
						</execution>
						<execution>
							<id>default-testCompile</id>
							<phase>none</phase>
						</execution>
					</executions>
				</plugin>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-resources-plugin</artifactId>
					<version>2.7</version>
					<configuration>
						<encoding>UTF-8</encoding>
					</configuration>
				</plugin>
				<plugin>
					<artifactId>maven-source-plugin</artifactId>
					<version>2.4</version>
					<executions>
						<execution>
							<id>attach-sources</id>
							<goals>
								<goal>jar</goal>
							</goals>
						</execution>
					</executions>
				</plugin>
				<plugin>
					<artifactId>maven-javadoc-plugin</artifactId>
					<version>2.10.3</version>
					<executions>
						<execution>
							<id>attach-javadocs</id>
							<goals>
								<goal>jar</goal>
							</goals>
						</execution>
					</executions>
				</plugin>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-surefire-plugin</artifactId>
					<version>2.19</version>
					<configuration>
						<skipTests>true</skipTests>
					</configuration>
				</plugin>
				<plugin>
					<groupId>net.alchim31.maven</groupId>
					<artifactId>scala-maven-plugin</artifactId>
					<version>3.2.2</version>
					<configuration>
						<recompileMode>incremental</recompileMode>
						<args>
							<arg>-target:jvm-1.7</arg>
						</args>
						<javacArgs>
							<javacArg>-source</javacArg>
							<javacArg>1.7</javacArg>
							<javacArg>-target</javacArg>
							<javacArg>1.7</javacArg>
						</javacArgs>
					</configuration>
					<executions>
						<execution>
							<id>scala-compile</id>
							<goals>
								<goal>compile</goal>
							</goals>
						</execution>
						<execution>
							<id>scala-test-compile</id>
							<goals>
								<goal>testCompile</goal>
							</goals>
						</execution>
					</executions>
				</plugin>
				<plugin>
					<groupId>org.scalatest</groupId>
					<artifactId>scalatest-maven-plugin</artifactId>
					<version>1.0</version>
					<configuration>
						<reportsDirectory>${project.build.directory}/surefire-reports</reportsDirectory>
						<junitxml>.</junitxml>
						<filereports>WDF TestSuite.txt</filereports>
						<argLine>-Xmx1024m -XX:MaxPermSize=512M</argLine>
						<environmentVariables>
							<HADOOP_HOME>${project.build.directory}/hadoop</HADOOP_HOME>
						</environmentVariables>
					</configuration>
					<executions>
						<execution>
							<id>test</id>
							<goals>
								<goal>test</goal>
							</goals>
						</execution>
					</executions>
				</plugin>
				<plugin>
					<groupId>org.codehaus.mojo</groupId>
					<artifactId>exec-maven-plugin</artifactId>
					<version>1.4.0</version>
					<executions>
						<execution>
							<goals>
								<goal>java</goal>
							</goals>
						</execution>
					</executions>
					<configuration>
						<mainClass>org.schedoscope.scheduler.rest.server.SchedoscopeRestService</mainClass>
						<arguments>
							<argument>--shell</argument>
						</arguments>
						<systemProperties>
							<systemProperty>
								<key>config.file</key>
								<value>src/main/resources/schedoscope.conf</value>
							</systemProperty>
						</systemProperties>
						<additionalClasspathElements>
							<additionalClasspathElement>/etc/hadoop/conf</additionalClasspathElement>
							<additionalClasspathElement>/etc/hive/conf</additionalClasspathElement>
							<additionalClasspathElement>target/${project.build.finalName}-mapreduce.jar</additionalClasspathElement>
						</additionalClasspathElements>
					</configuration>
				</plugin>
			</plugins>
		</build>

		<repositories>
			<repository>
				<id>otto-bintray</id>
				<url>https://dl.bintray.com/ottogroup/maven</url>
			</repository>
		</repositories>
	</project> 


As a reminder, a Maven project folder structure looks like this:

    project
    |
    +-- src
    |   |
    |   +-- main
    |   |   |
    |   |   +-- scala
    |   |   |
    |   |   +-- resources
    |   |
    |   +-- test
    |       |
    |       +-- scala
    |       |
    |       +-- resources
    |   
    +-- pom.xml