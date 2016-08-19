# Summary

The shell transformation executes arbitrary executables on the machine the schedoscope scheduler runs on.


If you want to use shell transformations, you need to [add the artifact `schedoscope-transformation-shell` to your `pom.xml`](Setting up a Schedoscope Project).

# Syntax

    case class ShellTransformation(script: String = "", scriptFile: String="", shell: String = "/bin/bash")

# Description

Execute the script either stored in an external `scriptFile` or directly passed as a string to `script` using `shell`.

The transformation fails if the script exits with an error code.

Please note that configuration properties you attach to the transformation are passed as environment variables to the shell script.

# Example

     transformVia(
        () => ShellTransformation("hive -e \"select * from ${viewTable}\" > ${target}")
     ).configureWith(
        Map(
          "target" -> s"/tmp/${view.n}.tsv",
          "viewTable" -> view.tableName
        )
     )  

# Packaging and Deployment

Schedoscope offers no special support for packaging or deploying shell scripts. They are assumed to exist on the machine Schedoscope runs on.

# Change Detection

Schedoscope considers the logic of a shell transformation to have changed if either the passed script string or the name of the script file has changed.