Path format: /{package}/{view}(/{view parameter value})*

View parameter value format:
  i(aNumber)                    => an integer
  l(aNumber)                    => a long
  b(aNumber)                    => a byte
  t(true)|t(false)              => a boolean
  f(aFloat)                     => a float
  d(aDouble)                    => a double
  ym(yyyyMM)                    => a MonthlyParameterization
  ymd(yyyyMMdd)                 => a DailyParameterization
  null()                        => null
  everything else               => a string

Ranges on view parameter values:
  rym(yyyyMM-yyyyMM)            => all MonthlyParameterizations between the first (earlier) and the latter (later)
  rymd(yyyyMMdd-yyyyMMdd)       => all DailyParameterizations between the first (earlier) and the latter (later)
  e{constructor parameter value format}({aValue},{anotherValue})
                                => enumerate multiple values for a given view parameter value format.
  For instance: 
    ei(1,2,3)                   => an enumeration of integer view parameters 
    e(aString, anotherString)   => an enumeration of string view parameters 
    eymd(yyyyMM,yyyMM)          => an enumeration of MonthlyParameterizations
    erymd(yyyyMM-yyyyMM,yyyyMM-yyyyMM) => an enumeration of MonthlyParameterization ranges

Quoting:
  Use backslashes to escape the syntax given above. The following characters need quotation: \,(-)