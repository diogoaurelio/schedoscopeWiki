# Introduction

The Schedoscope DSL ships with a set of predefined traits with field and parameter declarations. They can be used to impose field ordering and naming conventions. Moreover, there are some utility classes alleviating the declaration of view dependencies when using those traits. Nevertheless, Schedoscope does not give those traits any first class support by any means - you are free to define your own trait system as desirable for your environment, data, and applications.

## Id

The trait `Id` simply takes care the first field of a view is named `id` and is of type string.

    package org.schedoscope.dsl.views

    import org.schedoscope.dsl.ViewDsl

    trait Id extends ViewDsl {
      val id = fieldOf[String](Int.MaxValue)
    }

## JobMetadata

The trait `JobMetadata` takes care that the last fields of a view are `createdAt` of type date and `createdBy` of type string as a way to attach job metadata to a record (record creation time and name of job that created the record). Note that these fields have to be filled explicitly by transformation rules - they are not filled in automatically:

    package org.schedoscope.dsl.views

    import java.util.Date
    import org.schedoscope.dsl.ViewDsl

    trait JobMetadata extends ViewDsl {
      val createdAt = fieldOf[Date](1)
      val createdBy = fieldOf[String](0)
    }

## PointOccurrence

The trait `PointOccurrence` takes care that a field `occurredAt` recording the occurrence of an event follows the ID of a view. Note that the field is typed as string to give applications room for choosing appropriate date-time formats.

    package org.schedoscope.dsl.views

    import org.schedoscope.dsl.ViewDsl

    trait PointOccurrence extends ViewDsl {
      val occurredAt = fieldOf[String](1000)
    }

## IntervalOccurrence

The trait `IntervalOccurrence` provides two fields `occurredFrom` and `occurredUntil` to denote the duration of events / activities. Again, both fields are merely typed as string to give applications room for choosing appropriate date-time formats.

    trait IntervalOccurrence extends ViewDsl {
      val occurredFrom = fieldOf[String](1000)
      val occurredUntil = fieldOf[String](999)
    }

## MonthlyParameterization

    package org.schedoscope.dsl.views

    import org.schedoscope.dsl.ViewDsl

    trait MonthlyParameterization {
      val year: Parameter[String]
      val month: Parameter[String]

      def prevMonth()

      def thisAndPrevMonths()

      def thisAndPrevDays()

      def thisAndPrevDaysUntil(thisDay: Calendar) 

      def allDays()

      def allMonths()

      def lastMonths(c: Int)

      def allDaysOfMonth()
    }