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

c

The trait `MonthlyParameterization` imposes a monthly partitioning scheme over a view. A month is given by the year (a string, 4 characters) and the month within the year (a string, two characters).

    package org.schedoscope.dsl.views

    import org.schedoscope.dsl.ViewDsl

    trait MonthlyParameterization {
      val year: Parameter[String]
      val month: Parameter[String]

      def prevMonth()
      def thisAndPrevMonths()
      def thisAndPrevDays()
      def allDays()
      def allMonths()
      def lastMonths(c: Int)
      def allDaysOfMonth()
    }

When using this trait, one benefits from the following methods when declaring a view's dependencies:

* `prevMonth()`: returns the previous month;
* `thisAndPrevMonths()`: returns all months from the given month down to the earliest month, as given by the configuration property `schedoscope.scheduler.earliestDay`;
* `thisAndPrevDays()`: returns all days of the given month down to the earliest day, as given by the configuration property `schedoscope.scheduler.earliestDay`;
* `allDays()`: returns all days from the current date down to the earliest day, as given by the configuration property `schedoscope.scheduler.earliestDay`;
* `allMonths()`: returns all months from the current month down to the earliest month, as given by the configuration property `schedoscope.scheduler.earliestDay`;
* `lastMonth(c)`: returns the last `c` months from the given month including the given month.
* `allDayOfMonth()`: returns all days of the given month.

