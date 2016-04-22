# Summary

The shell transformation executes arbitrary executables on the machine the schedoscope scheduler runs on.

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