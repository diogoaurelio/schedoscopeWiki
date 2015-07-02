# Schedoscope Architecture

Schedoscope consists of three main components:

* A specification language (DSL) for defining data views
* An execution or scheduling environment based on Akka
* A test framework for unit testing view specifications

The scheduling framework can be controlled by a shell or using a rest 

## Scheduling

Schedoscope represents each view as an AKKA actor that can be directly referenced using its parametrization. Each actor encapsulates the current state of this.

### ViewActors


### Actions

### Metadata



## DSL
### Transformations
### Drivers