# Summary

The shell transformation executes arbitrary executables on the machine the schedoscope scheduler runs on.

# Syntax

    case class ShellTransformation(script: String = "",
                                      scriptFile :String="",
                                      shell: String = "/bin/bash")

# Description
