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

View URL paths can not only address individual views but also collections of views. To that end, they support range expressions:

- `rym(yyyyMM-yyyyMM)`: a range of `MonthlyParameterization`s between two months (inclusive)
- `rymd(yyyyMMdd-yyyyMMdd)`:a range of `DailyParameterization`s between two days (inclusive)

Examples:
    
    example.datamart/SearchExport/SHOP10/ym(201505-201410)
    example.stage/RawData/SHOP10/ymd(20150501-20141015)

Enumerations are also supported, even enumerations of ranges:
-  `e{constructor parameter value format}`({aValue},{anotherValue})`: enumerate multiple values for a given view parameter value format.

Examples:

    ei(1,2,3)                          => an enumeration of integer view parameters 
    e(aString, anotherString)          => an enumeration of string view parameters 
    eymd(yyyyMM,yyyMM)                 => an enumeration of MonthlyParameterizations
    erymd(yyyyMM-yyyyMM,yyyyMM-yyyyMM) => an enumeration of MonthlyParameterization ranges

Use backslashes to escape view URL pattern syntax given. The following characters need quotation: `\,(-)`.