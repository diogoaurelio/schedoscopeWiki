Schedoscope commands address views by URL paths. View URL paths have the following format:

    {package}/{view}(/{view parameter value})*

Thus, the first path segment is the Scala package the view resides in, the second segment the name of the view. 

Example:

    example.datamart/ShopLookup

For parameterized views, there is one additional path segment with the value for each parameter. Depending on the type of the respective parameter, values are encoded as follows:

- `i(aNumber)`: an integer parameter value
- `l(aNumber)`: a long parameter value
- `b(aNumber)`: a byte parameter value
- `t(true)`, `t(false)`: a boolean parameter value
- `f(aFloat)`: a float parameter value
- `d(aDouble)`: a double parameter value
- `ym(yyyyMM)`: a `MonthlyParameterization`
- `ymd(yyyyMMdd)`: a `DailyParameterization`
- `null()`: a null parameter value
- everything else: a string parameter value

Examples:

    example.datamart/SearchExport/SHOP10/2015/05
    example.datamart/ViewHavingAStringIntFloatBooleanAndSomeNullParameter/SHOP10/i(2015)/f(5.23)/t(true)/null()

View URL paths can not only address individual views but also collections of views. To that end, 

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