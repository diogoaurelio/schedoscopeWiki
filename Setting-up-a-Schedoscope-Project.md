Schedoscope is an internal Scala DSL for specifying views (Hive table partitions), their structure and dependencies, as well as the transformation logic required compute views from other views. As a consequence, setting up a Schedoscope project means setting up a Scala project that uses Schedoscope as a library. 

For this purpose, we provide a Maven POM template in this section. It is of course possible to use other build tools such as SBT or Ant/Ivy. You are also encouraged to look at the POM of the tutorial.

For running Schedoscope, the template utilizes the exec Maven plugin which assembles a classpath from the Maven dependencies and launches the Schedoscope REST service right out of the project folder. 

In real-world production deployment scenarios, you should probably follow a different deployment / bundling / launching strategy. 

Your basic options in this regard are:
- create and distribute a fat jar using the Maven assembly plugin;
- copy all jars into a separate deployment directory with the Maven dependency plugin, create a shell scripts that constructs the classpath and launches the application, and distribute that folder.


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
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-nop</artifactId>
                <version>1.7.5</version>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>org.scalatest</groupId>
                <artifactId>scalatest_2.10</artifactId>
                <version>2.2.0</version>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>com.ottogroup.bi</groupId>
                <artifactId>hadoop-launcher</artifactId>
                <version>0.0.6</version>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-core</artifactId>
                <version>1.1.2</version>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-classic</artifactId>
                <version>1.1.2</version>
            </dependency>
            <dependency>
                <groupId>schedoscope</groupId>
                <artifactId>schedoscope-core</artifactId>
                <version>0.1.0</version>
            </dependency>
        </dependencies>

        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.0</version>
                    <configuration>
                        <source>1.7</source>
                        <target>1.7</target>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>2.6</version>
                    <configuration>
                        <encoding>UTF-8</encoding>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.7</version>
                    <configuration>
                        <skipTests>true</skipTests>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>net.alchim31.maven</groupId>
                    <artifactId>scala-maven-plugin</artifactId>
                    <version>3.2.0</version>
                    <executions>
                        <execution>
                            <goals>
                                <goal>compile</goal>
                                <goal>testCompile</goal>
                            </goals>
                            <configuration>
                                <args>
                                    <arg>-make:transitive</arg>
                                    <arg>-dependencyfile</arg>
                                    <arg>${project.build.directory}/.scala_dependencies</arg>
                                </args>
                            </configuration>
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
                        <mainClass>org.schedoscope.scheduler.api.SchedoscopeRestService</mainClass>
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