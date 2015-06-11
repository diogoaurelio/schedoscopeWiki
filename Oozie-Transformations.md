# Summary

With Oozie transformations, one can compute views with Oozie workflows. Oozie workflows can be parameterized with `${parameter}; the values for these parameters can be passed using the `.configureWith()` clause.

# Syntax

    case class OozieTransformation(bundle: String, workflow: String, workflowAppPath: String)

# Description

Oozie transformations take the following parameters:

* `bundle`: logical name for the bundle the workflow belongs to
* `workflow`: logical name of the workflow
* `workflowAppPath`: the HDFS folder where the workflow is deployed (i.e., `OozieClient.APP_PATH`)

# Helpers

## oozieWFAppPath

Schedoscope supports an autodeployment mechanism for Oozie bundles. If that mechanism is used, `workflowAppPath` should point to the HDFS path used for autodeployment (see below). `oozieWFPath` constructs this path from the `bundle` and `workflow` names of the transformation.

    def oozieWFPath(bundle: String, workflow: String): String

Parameters:

* `bundle`: logical name for the bundle the workflow belongs to
* `workflow`: logical name of the workflow

# Examples

The following is an example of an Oozie transformation referencing an already deployed workflow:

    transformVia(() =>
      OozieTransformation(
        "osm", "computeNearestTrainstations",
        "/hdp/dev/osm/oozie/workflows/osm/computeNearestTrainstations"
      ).configureWith(
        Map(
          "jobTracker" -> cluster.getJobTrackerUri(),
          "nameNode" -> cluster.getNameNodeUri(),
          "oozie.use.system.libpath" -> "false"
      ))

Should autodeployment be used - in the default case, this means deployment to `/tmp/schedoscope/oozie/dev/workflows/osm/computeNearestTrainstations` - the transformation can be changed to

    transformVia(() =>
      OozieTransformation(
        "osm", "computeNearestTrainstations",
        oozieWFPath("osm", "computeNearestTrainstations")
      ).configureWith(
        Map(
          "jobTracker" -> cluster.getJobTrackerUri(),
          "nameNode" -> cluster.getNameNodeUri(),
          "oozie.use.system.libpath" -> "false"
      ))

# Packaging and Deployment

There are two ways of deploying Oozie workflows triggered by Oozie transformations: automatic and external.

## Automatic deployment

Based on the `schedoscope.transformation.oozie.location` and `schedoscope.app.env` configuration properties, Schedoscope automatically uploads jar files on the classpath that end with `-oozie.jar` to the HDFS folder constructed as `${schedoscope.transformation.oozie.location}/${schedoscope.app.env}` and unjars it. 

Within the jar, it is expected that the `workflow.xml` of the Oozie workflow resides in:

    workflows/${bundle}$/${workflow}/

Any resources for that workflow - such as library jar files - should reside in:

    workflows/${bundle}$/${workflow}/lib

Such an `-oozie.jar` bundle could be created using Maven with the assembly plugin with a plugin descriptor like this

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>2.4</version>
        <execution>
            <id>oozie</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
            <configuration>
                <descriptor>src/main/assemble/oozie.xml</descriptor>
            </configuration>
        </execution>
    </plugin>

and an assembly descriptor in `src/main/assemble/oozie.xml` like that

    <assembly
        xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">
        <id>oozie</id>
        <includeBaseDirectory>false</includeBaseDirectory>
        <formats>
            <format>jar</format>
        </formats>
        <fileSets>
            <fileSet>
                <directory>${basedir}/src/main/<path_to_oozie_workflow_xml_files></directory>
                <outputDirectory>/workflows/<bundle>/<workflow></outputDirectory>
                <includes>
                    <include>*/</include>
                </includes>
                <filtered>true</filtered>
            </fileSet>
            <fileSet>
                <directory>${basedir}/<path_to_workflow_lib_resources></directory>
                <outputDirectory>/workflows/<bundle>/<workflow></outputDirectory>
                <includes>
                    <include><resource_pattern></include>
                </includes>
            </fileSet>
        </fileSets>
    </assembly>

# Change detection