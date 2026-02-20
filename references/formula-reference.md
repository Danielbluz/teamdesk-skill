# TeamDesk Formula Reference — Complete Function & Operator Guide

*Source: Official TeamDesk Documentation (https://www.teamdesk.net/help/working-with-formulas/formula-building-blocks/)*

---

## Syntax Basics

### Column References
```
[Column Name]           // Reference column value
[Column With Spaces]    // Spaces allowed
```

### Variables
```
Var[Variable Name]      // Database variable (always text, convert as needed)
```

### User Properties
```
User[Department]        // Current user's property value
User[Email]
```

### Related Columns (Summary filters only)
```
Related[Column Name]    // Access column on other side of relationship
```

---

## 1. TYPE CONVERSION FUNCTIONS

### Format
**Syntax:**
- `Format(column)`
- `Format(Date date, Text format)`
- `Format(Date date, Text format, Text culture)`
- `Format(Numeric number, Text format)`
- `Format(Numeric number, Text format, Text unit)`
- `Format(Numeric number, Text format, Text unit, Text culture)`
- `Format(Time time, Text format)`
- `Format(Time time, Text format, Text culture)`
- `Format(Timestamp timestamp, Text format)`
- `Format(Timestamp timestamp, Text format, Text culture)`

**Returns:** Text

**Description:** Converts Numeric, Date, Time, and Timestamp values to text using formatting options, user-specific locale, and timezone settings.

**Format strings for Date:** "d" = Short Date, "D" = Long Date, "M" = Month Day, "Y" = Year Month, plus custom formats like "MM/yy".

**Format strings for Numeric:** "$#" = unit left, no grouping; "#,#" = grouping, hide zeros; "#,0" = grouping, show zeros; "0" = no grouping (default); "0%" = unit right, multiplies by 100. Custom: "$#,#.####", "-$#,0.00".

**Format strings for Time:** "t" = Short Time, "T" = Long Time.

**Format strings for Timestamp:** "g" = Short Date/Time, "G" = Short Date/Long Time, "f" = Date/Short Time, "F" = Long Date/Long Time.

**Examples:**
- `Format(#2009-06-15#, "D")` --> "Monday, June 15, 2009" (en-US)
- `Format(#2009-06-15#, "D", "es-ES")` --> "lunes, 15 de junio de 2009"
- `Format(1500.63, "$#")` --> "$15001"
- `Format(1500.63, "#,0")` --> "1,501"
- `Format(1.63, "0%")` --> "163%"
- `Format(-5.204, "-$#,0.00")` --> "-$5.20"
- `Format(1500.63, "$#", "£")` --> "£15001"

---

### ToBoolean
**Syntax:**
- `ToBoolean(Numeric number)`
- `ToBoolean(Text text)`

**Returns:** Boolean

**Description:** Converts a numeric or text value to a boolean. Numeric: returns true if non-zero. Text: accepts "1", "true", or "yes" (case-insensitive) as true; all others as false.

**Examples:**
- `ToBoolean(1)` --> true
- `ToBoolean(0)` --> false
- `ToBoolean("YES")` --> true
- `ToBoolean("text")` --> false

---

### ToDate
**Syntax:**
- `ToDate(Text text)` — YYYY-MM-DD format
- `ToDate(Text text, Text culture)`
- `ToDate(Timestamp timestamp)`
- `ToDate(Timestamp timestamp, Text timezone)`

**Returns:** Date

**Description:** Converts a timestamp or text value to a date.

**Examples:**
- `ToDate("2012-01-15")` --> #2012-01-15#
- `ToDate("Monday, 13 December 2010", "en-US")` --> #2010-12-13#
- `ToDate("08/05/2024", "fr-FR")` --> #2024-05-08#
- `ToDate(#2013-03-25 11:28:57#)` --> 3/25/2013 (user timezone)
- `ToDate(#2013-03-25 03:28:57#, "America/New_York")` --> 3/24/2013

---

### ToNumber
**Syntax:**
- `ToNumber(Boolean boolean)`
- `ToNumber(Text text)`
- `ToNumber(Text text, Text culture)`

**Returns:** Numeric

**Description:** Converts a boolean or text value to a numeric. Boolean: false=0, true=1. Text: parses the numeric representation. Culture variant handles locale-specific decimal separators.

**Examples:**
- `ToNumber(true)` --> 1
- `ToNumber(false)` --> 0
- `ToNumber("123.56")` --> 123.56
- `ToNumber("345,98", "de-DE")` --> 345.98

---

### ToText
**Syntax:** `ToText(value)`

**Returns:** Text

**Description:** Returns a text value containing the printed representation of the input value. Null returns null. Numeric values produce 6 decimal places. Dates use YYYY-MM-DD. Timestamps use YYYY-MM-DD HH:MM:SS. Times/Durations use HH:MM:SS. Booleans produce "1" (true) or "0" (false). Users return screen name or email.

**Examples:**
- Date 25/3/2013 --> "2013-03-25"
- Timestamp 25/03/2013 10:51:34 AM --> "2013-03-25 10:51:34"
- Time 10:15 AM --> "10:15:00"
- Duration 30 minutes --> "00:30:00"
- true --> "1"
- Numeric 10 --> "10.000000"

---

### ToTimeOfDay
**Syntax:**
- `ToTimeOfDay(Text text)` — hh:mm or hh:mm:ss format
- `ToTimeOfDay(Text text, Text culture)`
- `ToTimeOfDay(Timestamp timestamp)`
- `ToTimeOfDay(Timestamp timestamp, Text timezone)`

**Returns:** Time

**Description:** Converts a timestamp or text value into a time.

**Examples:**
- `ToTimeOfDay("22:00")` --> #22:00#
- `ToTimeOfDay("12:03:29")` --> #12:03:29#
- `ToTimeOfDay("3:04 pm", "en-US")` --> #15:04#
- `ToTimeOfDay(#2012-11-05 09:21:23#)` --> 9:21 AM (user timezone)

---

### ToTimestamp
**Syntax:**
- `ToTimestamp(Date date)`
- `ToTimestamp(Date date, Time time)`
- `ToTimestamp(Date date, Time time, Text timezone)`

**Returns:** Timestamp

**Description:** Converts a date or a date and time value into a timestamp.

**Examples:**
- `ToTimestamp([Date Column])` --> midnight of date in user's timezone
- `ToTimestamp([Date Column], [Time Column])` --> combined date/time
- `ToTimestamp([Date Column], #09:00#)` --> date at 9:00 AM user timezone
- `ToTimestamp([Date Column], [Time Column], "America/New_York")` --> interpreted as Eastern time

---

## 2. TEXT FUNCTIONS

### All
**Syntax:** `All(Text values, Text list)`
**Returns:** Boolean
**Description:** Returns true if all the values specified in its list argument exist in the values argument. Both arguments are comma-separated lists.
**Examples:**
- `All("A,B,C,D", "D,A")` --> true
- `All("A,B,C,D", "D,E")` --> false

---

### Any
**Syntax:** `Any(Text values, Text list)`
**Returns:** Boolean
**Description:** Returns true if at least one value from the list exists in the values.
**Examples:**
- `Any("A,B,C,D", "D,A")` --> true
- `Any("A,B,C,D", "D,E")` --> true

---

### Begins
**Syntax:** `Begins(Text u, Text v)`
**Returns:** Boolean
**Description:** Returns true if the text u begins with the text v.
**Examples:**
- `Begins("test", "tes")` --> true
- `Begins("test", "st")` --> false

---

### Concat
**Syntax:** `Concat(Text value1, Text value2, ..., Text valueN)`
**Returns:** Text
**Description:** Concatenates two or more string values end-to-end. Implicitly converts null values to empty strings.
**Examples:**
- `Concat("Happy ", "Birthday ", "11", "/", "25")` --> "Happy Birthday 11/25"
- `Concat("John", null, "Doe")` --> "JohnDoe"

---

### Contains
**Syntax:** `Contains(Text u, Text v)`
**Returns:** Boolean
**Description:** Returns true if text u contains text v. Case-insensitive.
**Examples:**
- `Contains("test", "es")` --> true
- `Contains([Type], "new")` --> true if Type contains "new"
- `Contains("test", "et")` --> false

---

### Ends
**Syntax:** `Ends(Text u, Text v)`
**Returns:** Boolean
**Description:** Returns true if the text u ends with the text v.
**Examples:**
- `Ends("test", "st")` --> true
- `Ends("test", "te")` --> false

---

### Guid
**Syntax:** `Guid()`
**Returns:** Text
**Description:** Returns a unique string in XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX format.
**Example:** `Guid()` --> "3A376A6C-46BA-4D56-BAFF-BFD3A3EDCE2F"

---

### Left
**Syntax:**
- `Left(Text text, Numeric number)`
- `Left(Text text, Text delimiter)`

**Returns:** Text

**Description:** Returns the leftmost characters from text. With numeric: returns that many characters. With delimiter: returns text up to but not including the first occurrence of delimiter. If delimiter not found, returns entire text.

**Examples:**
- `Left("invoice", 3)` --> "inv"
- `Left("John Smith", " ")` --> "John"
- `Left("test/123", "/")` --> "test"

---

### Len
**Syntax:** `Len(Text text)`
**Returns:** Numeric
**Description:** Returns the number of characters in the text.
**Example:** `Len("xxx")` --> 3

---

### List
**Syntax:** `List(Text delimiter, Text value1, Text value2, ..., Text valueN)`
**Returns:** Text
**Description:** Concatenates all value arguments using the first argument as a delimiter. If any argument is blank, that argument and its corresponding delimiter are excluded.
**Examples:**
- `List("-", "model", "a", "2012")` --> "model-a-2012"
- `List(" ", [First Name], [Last Name])` --> "John Smith" (or just "John" if Last Name is empty)

---

### Lower
**Syntax:** `Lower(Text text)`
**Returns:** Text
**Description:** Returns text converted to lowercase.
**Example:** `Lower("XXX")` --> "xxx"

---

### Mid
**Syntax:** `Mid(Text text, Numeric position, Numeric number)`
**Returns:** Text
**Description:** Returns a specific number of characters from a text string, starting at the position you specify (position 1 is the first character).
**Examples:**
- `Mid("type20A", 5, 2)` --> "20"
- `Mid("type20A", 7, 3)` --> "A"

---

### NotLeft
**Syntax:**
- `NotLeft(Text text, Text delimiter)`
- `NotLeft(Text text, Numeric number)`

**Returns:** Text

**Description:** Returns what remains after excluding the left part of a text value. With delimiter: removes up to and including the first occurrence. If delimiter not found, returns blank. With number: removes that many characters from the left.

**Examples:**
- `NotLeft("Jane Smith", " ")` --> "Smith"
- `NotLeft("type-20A", "-")` --> "20A"
- `NotLeft("type-20A", "/")` --> blank
- `NotLeft("typeABC", 4)` --> "ABC"

---

### NotRight
**Syntax:**
- `NotRight(Text text, Numeric number)`
- `NotRight(Text text, Text delimiter)`

**Returns:** Text

**Description:** Returns what remains after excluding the right part of a text value. With delimiter: removes from the last occurrence to the end. If delimiter not found, returns blank.

**Examples:**
- `NotRight("typeABC", 3)` --> "type"
- `NotRight("Jane Smith", " ")` --> "Jane"
- `NotRight("type-20A", "-")` --> "type"

---

### PadLeft
**Syntax:**
- `PadLeft(Text text, Numeric length)`
- `PadLeft(Text text, Numeric length, Text filler)`

**Returns:** Text
**Description:** Right-aligns by padding on the left. Default filler is whitespace.
**Examples:**
- `PadLeft("ABC", 10)` --> "       ABC"
- `PadLeft("ABC", 10, "-")` --> "-------ABC"

---

### PadRight
**Syntax:**
- `PadRight(Text text, Numeric length)`
- `PadRight(Text text, Numeric length, Text filler)`

**Returns:** Text
**Description:** Left-aligns by padding on the right. Default filler is whitespace.
**Examples:**
- `PadRight("XYZ", 10)` --> "XYZ       "
- `PadRight("XYZ", 10, "-")` --> "XYZ-------"

---

### Part
**Syntax:** `Part(Text text, Numeric position, Text delimiter)`
**Returns:** Text
**Description:** Returns the specified part of a text value. Parts are separated by the delimiter. Position 1 is leftmost; negative numbers start from right.
**Examples:**
- `Part("model/year/car", 1, "/")` --> "model"
- `Part("model/year/car", -1, "/")` --> "car"
- `Part("David Neil Evans", 2, " ")` --> "Neil"

---

### Proper
**Syntax:** `Proper(Text text)`
**Returns:** Text
**Description:** Capitalizes the first letter and any letter following a non-letter character. Words entirely in uppercase are treated as acronyms and left unchanged.
**Examples:**
- `Proper("Quick bROWn FOX")` --> "Quick Brown FOX"
- `Proper(Lower("Quick bROWn FOX"))` --> "Quick Brown Fox"

---

### Replace
**Syntax:** `Replace(Text text, Text old, Text new)`
**Returns:** Text
**Description:** Replaces all occurrences of the old text with the new text.
**Examples:**
- `Replace("TypeAA28", "AA", "BB")` --> "TypeBB28"
- `Replace("Linda May", "May", "Evans")` --> "Linda Evans"

---

### Right
**Syntax:**
- `Right(Text text, Numeric number)`
- `Right(Text text, Text delimiter)`

**Returns:** Text

**Description:** Returns the right part of a text value. With numeric: returns that many rightmost characters. With delimiter: returns text after the last occurrence. If delimiter not found, returns entire text.

**Examples:**
- `Right("typeAA1", 3)` --> "AA1"
- `Right("John Smith", " ")` --> "Smith"
- `Right("test/AA1", "/")` --> "AA1"

---

### Trim
**Syntax:** `Trim(Text text)`
**Returns:** Text
**Description:** Removes leading and trailing whitespace (space, tab, carriage return, line feed).
**Example:** `Trim(" NY City ")` --> "NY City"

---

### Upper
**Syntax:** `Upper(Text text)`
**Returns:** Text
**Description:** Returns text converted to uppercase.
**Example:** `Upper("xxx")` --> "XXX"

---

### URLEncode
**Syntax:** `URLEncode(Text text)`
**Returns:** Text
**Description:** Encodes text for safe URL usage using UTF-8 percent-encoding.
**Example:** `URLEncode("test@email.com")` --> "test%40email.com"

---

### URLDecode
**Syntax:** `URLDecode(Text text)`
**Returns:** Text
**Description:** Converts URL-encoded strings into their decoded form. Inverse of URLEncode.
**Example:** `URLDecode("test%40email.com")` --> "test@email.com"

---

### URLParam
**Syntax:**
- `URLParam(Text url, Text param)`
- `URLParam(Text url, Text param, Numeric index)`

**Returns:** Text

**Description:** Extracts and decodes the value of a parameter from a URL's query string. When multiple parameters share the same name, the two-param version joins them as comma-separated; the three-param version uses 1-based index.

**Examples:**
- `URLParam("google.com?q=teamdesk", "q")` --> "teamdesk"
- `URLParam("google.com?q=teamdesk&q=dbflex", "q", 2)` --> "dbflex"

---

## 3. DATE FUNCTIONS

### AdjustMonth
**Syntax:** `AdjustMonth(Date date, Numeric months)`
**Returns:** Date
**Description:** Returns the date that is a number of months after the given date. If the resulting day doesn't exist, returns the last day of that month.
**Examples:**
- `AdjustMonth(#2013-03-20#, 5)` --> 8/20/2013
- `AdjustMonth(#2013-03-20#, -2)` --> 1/20/2013
- `AdjustMonth(#2013-01-30#, 1)` --> 2/28/2013

---

### AdjustYear
**Syntax:** `AdjustYear(Date date, Numeric years)`
**Returns:** Date
**Description:** Returns the date that is a number of years after the given date. If the resulting day doesn't exist, returns the last day of that month.
**Examples:**
- `AdjustYear(#2012-01-30#, 2)` --> 1/30/2014
- `AdjustYear(#2012-02-29#, 2)` --> 2/28/2014

---

### Date
**Syntax:** `Date(Numeric year, Numeric month, Numeric day)`
**Returns:** Date
**Description:** Creates a date from a given year, month, and day.
**Example:** `Date(2012, 8, 20)` --> 8/20/2012

---

### Day
**Syntax:** `Day(Date date)`
**Returns:** Numeric
**Description:** Returns the day of the month (1-31).
**Example:** `Day(#2013-03-20#)` --> 20

---

### DayOfWeek
**Syntax:** `DayOfWeek(Date date)`
**Returns:** Numeric (0-6)
**Description:** Sunday=0, Monday=1, Tuesday=2, Wednesday=3, Thursday=4, Friday=5, Saturday=6.
**Examples:**
- `DayOfWeek(#2013-03-21#)` --> 4 (Thursday)
- `DayOfWeek(#2013-03-24#)` --> 0 (Sunday)

---

### DayOfYear
**Syntax:** `DayOfYear(Date date)`
**Returns:** Numeric
**Description:** Returns the day number within the year (Jan 1 = 1).
**Examples:**
- `DayOfYear(#2013-01-01#)` --> 1
- `DayOfYear(#2013-03-20#)` --> 79

---

### FirstDayOfMonth
**Syntax:** `FirstDayOfMonth(Date date)`
**Returns:** Date
**Example:** `FirstDayOfMonth(#2013-03-20#)` --> 3/1/2013

---

### FirstDayOfQuarter
**Syntax:** `FirstDayOfQuarter(Date date)`
**Returns:** Date
**Example:** `FirstDayOfQuarter(#2013-03-20#)` --> 1/1/2013

---

### FirstDayOfPeriod
**Syntax:** `FirstDayOfPeriod(Date date, Duration period, Date reference)`
**Returns:** Date
**Description:** Returns the first day of the period in which the date falls. Periods repeat cyclically from the reference date.
**Examples:**
- `FirstDayOfPeriod(Today(), Days(7), Date(2006,1,1))` — first day of week (Sunday start)
- `FirstDayOfPeriod(Today(), Days(7), Date(2007,1,1))` — first day of week (Monday start)

---

### FirstDayOfWeek
**Syntax:** `FirstDayOfWeek(Date date)`
**Returns:** Date
**Description:** First day of the week. Depends on database location settings (Sunday in US/Canada/Japan; Monday in Europe).

---

### FirstDayOfYear
**Syntax:** `FirstDayOfYear(Date date)`
**Returns:** Date
**Example:** `FirstDayOfYear(Today())` --> Jan 1 of current year

---

### IsLeapDay
**Syntax:** `IsLeapDay(Date date)`
**Returns:** Boolean
**Description:** Returns true if the date is February 29.

---

### IsLeapYear
**Syntax:** `IsLeapYear(Date date)` or `IsLeapYear(Numeric year)`
**Returns:** Boolean

---

### ISOWeek
**Syntax:** `ISOWeek(Date date)`
**Returns:** Numeric
**Description:** ISO 8601 week number. All weeks start Monday; week one starts on Monday of the first week containing Thursday.
**Example:** `ISOWeek(#2016-07-25#)` --> 30

---

### LastDayOfMonth
**Syntax:** `LastDayOfMonth(Date date)`
**Returns:** Date
**Example:** `LastDayOfMonth(#2013-02-20#)` --> 2/28/2013

---

### LastDayOfQuarter
**Syntax:** `LastDayOfQuarter(Date date)`
**Returns:** Date
**Example:** `LastDayOfQuarter(#2013-03-20#)` --> 3/31/2013

---

### LastDayOfPeriod
**Syntax:** `LastDayOfPeriod(Date date, Duration period, Date reference)`
**Returns:** Date
**Description:** Returns the last day of the period in which the date falls.

---

### LastDayOfWeek
**Syntax:** `LastDayOfWeek(Date date)`
**Returns:** Date
**Description:** Last day of the week. Depends on database location settings.

---

### LastDayOfYear
**Syntax:** `LastDayOfYear(Date date)`
**Returns:** Date
**Example:** `LastDayOfYear(Today())` --> Dec 31 of current year

---

### Month
**Syntax:** `Month(Date date)`
**Returns:** Numeric
**Description:** Returns the month number (January = 1).
**Example:** `Month(#2013-03-20#)` --> 3

---

### MonthsBetween
**Syntax:** `MonthsBetween(Date from, Date to)`
**Returns:** Numeric
**Description:** Returns the number of month boundaries crossed between the dates.
**Examples:**
- `MonthsBetween(#2012-01-01#, #2011-12-31#)` --> 1
- `MonthsBetween(#2013-08-10#, #2013-01-14#)` --> 7

---

### NextDayOfWeek
**Syntax:** `NextDayOfWeek(Date date, Numeric weekday)`
**Returns:** Date
**Description:** Returns the next specified weekday (0=Sunday..6=Saturday). Returns same day if already that weekday.
**Example:** `NextDayOfWeek(#2013-03-22#, 3)` --> 03/27/2013

---

### PrevDayOfWeek
**Syntax:** `PrevDayOfWeek(Date date, Numeric weekday)`
**Returns:** Date
**Description:** Returns the previous specified weekday. Returns same day if already that weekday.
**Example:** `PrevDayOfWeek(#2013-03-20#, 5)` --> 03/15/2013

---

### Quarter
**Syntax:** `Quarter(Date date)`
**Returns:** Numeric (1-4)
**Example:** `Quarter(#2013-03-20#)` --> 1

---

### QuartersBetween
**Syntax:** `QuartersBetween(Date from, Date to)`
**Returns:** Numeric
**Description:** Returns the number of quarter boundaries crossed between the dates.

---

### Today
**Syntax:** `Today()`
**Returns:** Date
**Description:** Returns the current date in the local time zone.

---

### Week
**Syntax:** `Week(Date date)` or `Week(Date date, Numeric weekday)`
**Returns:** Numeric
**Description:** Returns the week number. Week one begins on January 1st. Optional weekday param specifies first day of week.
**Examples:**
- `Week(#2016-07-25#)` --> 31
- `Week(#2016-07-25#, 3)` --> 30

---

### Workday
**Syntax:**
- `Workday(Date start, Numeric days)`
- `Workday(Date start, Duration days)`
- `Workday(Date start, Numeric days, Text weekends)`
- `Workday(Date start, Duration days, Text weekends)`

**Returns:** Date

**Description:** Adds workdays to a date, excluding weekends. Default weekends from database properties. Custom: "67" = Sat/Sun or bitmap "0000011" (Mon=1..Sun=7).

**Example:** `Workday(#2015-08-12#, 5)` --> #2015-08-19#

---

### Workdays
**Syntax:** `Workdays(Date start, Date end)` or `Workdays(Date start, Date end, Text weekends)`
**Returns:** Duration
**Description:** Calculates duration in work days between start and end dates inclusive.
**Example:** `Workdays(#2015-08-12#, #2015-08-19#)` --> 6 days

---

### Year
**Syntax:** `Year(Date date)`
**Returns:** Numeric
**Example:** `Year(#2013-03-22#)` --> 2013

---

### YearsBetween
**Syntax:** `YearsBetween(Date from, Date to)`
**Returns:** Numeric
**Description:** Returns the number of year boundaries crossed between the dates.

---

## 4. TIME FUNCTIONS

### Hour
**Syntax:** `Hour(Time time)`
**Returns:** Numeric (0-23)
**Example:** `Hour(ToTimeOfDay("2:03:29 pm"))` --> 14

---

### Minute
**Syntax:** `Minute(Time time)`
**Returns:** Numeric (0-59)
**Example:** `Minute(ToTimeOfDay("2:03:29 pm"))` --> 3

---

### Second
**Syntax:** `Second(Time time)`
**Returns:** Numeric (0-59)
**Example:** `Second(ToTimeOfDay("2:03:29 pm"))` --> 29

---

### Time
**Syntax:**
- `Time(Numeric hour, Numeric minute)`
- `Time(Numeric hour, Numeric minute, Numeric second)`
- `Time(Numeric seconds)` — seconds since midnight

**Returns:** Time

**Examples:**
- `Time(22, 15)` --> 22:15
- `Time(22, 15, 5)` --> 22:15:05
- `Time(246)` --> 00:04:06

---

## 5. NUMERIC FUNCTIONS

### Acos
**Syntax:** `Acos(Numeric number)`
**Returns:** Numeric (radians)
**Description:** Arccosine. Input must be between -1 and 1.

### Asin
**Syntax:** `Asin(Numeric number)`
**Returns:** Numeric (radians)
**Description:** Arcsine. Input must be between -1 and 1.

### Atan
**Syntax:** `Atan(Numeric number)` or `Atan(Numeric x, Numeric y)`
**Returns:** Numeric (radians)
**Description:** Arctangent. Two params: angle between positive x-axis and ray to point (x, y).

### Cos
**Syntax:** `Cos(Numeric angle)`
**Returns:** Numeric
**Description:** Cosine of angle in radians.

### Cot
**Syntax:** `Cot(Numeric angle)`
**Returns:** Numeric
**Description:** Cotangent of angle in radians.

### Degrees
**Syntax:** `Degrees(Numeric angle)`
**Returns:** Numeric
**Description:** Converts radians to degrees.
**Example:** `Degrees(Pi())` --> 180

### Exp
**Syntax:** `Exp(Numeric number)`
**Returns:** Numeric
**Description:** Returns e raised to the power of the given number (e = 2.71828182845904).

### Frac
**Syntax:** `Frac(Numeric number)`
**Returns:** Numeric
**Description:** Returns the fractional part of the number. Preserves sign.
**Examples:**
- `Frac(2.148)` --> 0.148
- `Frac(-2.148)` --> -0.148

### Int
**Syntax:** `Int(Numeric number)`
**Returns:** Numeric
**Description:** Returns the integer part (truncates toward zero).
**Examples:**
- `Int(2.148)` --> 2
- `Int(-2.148)` --> -2

### Ln
**Syntax:** `Ln(Numeric number)`
**Returns:** Numeric
**Description:** Natural (base e) logarithm.

### Log
**Syntax:** `Log(Numeric number)`
**Returns:** Numeric
**Description:** Base 10 logarithm.
**Example:** `Log(100)` --> 2

### Pi
**Syntax:** `Pi()`
**Returns:** Numeric
**Description:** Returns 3.14159265358979...

### PV
**Syntax:**
- `PV(Numeric rate, Numeric skip, Numeric amount)`
- `PV(Numeric rate, Numeric skip, Numeric amount, Numeric pay)`

**Returns:** Numeric

**Description:** Calculates Present Value. `rate` = discount rate, `skip` = periods before payment, `amount` = payment amount, `pay` = number of payments.

**Examples:**
- `PV(0.10, 2, 222)` --> 183.47
- `PV(0.10, 1, 222, 2)` --> 385.29

### Radians
**Syntax:** `Radians(Numeric angle)`
**Returns:** Numeric
**Description:** Converts degrees to radians.

### Random
**Syntax:** `Random()` or `Random(Numeric from, Numeric to)`
**Returns:** Numeric
**Description:** No-arg: random number between 0 and 1 (exclusive). Two-arg: random in [from, to).

### Sin
**Syntax:** `Sin(Numeric angle)`
**Returns:** Numeric
**Description:** Sine of angle in radians.

### Sqrt
**Syntax:** `Sqrt(Numeric number)`
**Returns:** Numeric
**Example:** `Sqrt(4)` --> 2

### Tan
**Syntax:** `Tan(Numeric angle)`
**Returns:** Numeric
**Description:** Tangent of angle in radians.

---

## 6. ROUNDING AND TRUNCATING FUNCTIONS

### Ceil
**Syntax:**
- `Ceil(Numeric number)`
- `Ceil(Numeric number, Numeric significance)`
- `Ceil(Duration duration, Duration significance)`
- `Ceil(Time time, Duration significance)`

**Returns:** Same type as input

**Description:** Rounds up. With significance: smallest multiple of significance >= the value.

**Examples:**
- `Ceil(4.9)` --> 5
- `Ceil(-4.9)` --> -4
- `Ceil(7, 5)` --> 10
- `Ceil(-7, 5)` --> -5
- `Ceil(Hours(7), Hours(5))` --> 10 hrs

---

### Floor
**Syntax:**
- `Floor(Numeric number)`
- `Floor(Numeric number, Numeric significance)`
- `Floor(Duration duration, Duration significance)`
- `Floor(Time time, Duration significance)`

**Returns:** Same type as input

**Description:** Rounds down. With significance: largest multiple of significance <= the value.

**Examples:**
- `Floor(4.9)` --> 4
- `Floor(-4.9)` --> -5
- `Floor(7, 5)` --> 5
- `Floor(Hours(7), Hours(5))` --> 5 hrs

---

### Round
**Syntax:**
- `Round(Numeric number)`
- `Round(Numeric number, Numeric significance)`
- `Round(Duration duration, Duration significance)`
- `Round(Time time, Duration significance)`

**Returns:** Same type as input

**Description:** Rounds to nearest. With significance: nearest multiple.

**Examples:**
- `Round(4.9)` --> 5
- `Round(5.5)` --> 6
- `Round(7.63456, 0.01)` --> 7.63
- `Round(7.63, 5)` --> 10

---

## 7. AGGREGATION FUNCTIONS

### Count
**Syntax:** `Count(value1, value2, ..., valueN)`
**Returns:** Numeric
**Description:** Returns the number of non-null arguments. Text: counts non-blank. Boolean: counts true only. Numeric zero IS counted.
**Examples:**
- `Count("", "xyz", "zzz")` --> 2
- `Count(28, 50, 0)` --> 3
- `Count(true, false)` --> 1

---

### Max
**Syntax:** `Max(value1, value2, ..., valueN)`
**Returns:** Same type as input
**Description:** Returns the maximum value. Null values are ignored. Works with Numbers, Text, Durations, Dates, Timestamps, Times, Booleans.
**Examples:**
- `Max(50, 60, 70, 100)` --> 100
- `Max("Jane", "Rita", "Linda")` --> "Rita"

---

### Min
**Syntax:** `Min(value1, value2, ..., valueN)`
**Returns:** Same type as input
**Description:** Returns the minimum value. Null values are ignored.
**Examples:**
- `Min(50, 60, 70, 100)` --> 50
- `Min("Jane", "Rita", "Linda")` --> "Jane"

---

### Sum
**Syntax:** `Sum(Numeric n1, Numeric n2, ...)` or `Sum(Duration d1, Duration d2, ...)`
**Returns:** Numeric / Duration
**Description:** Returns the sum of the non-null arguments.
**Examples:**
- `Sum(20, 0, 30)` --> 50
- `Sum(3d, 22h)` --> 94 hours

---

## 8. NULL HANDLING FUNCTIONS

### IsNull
**Syntax:** `IsNull(value)`
**Returns:** Boolean
**Description:** Returns true if value is empty or undefined. Zero returns false. Boolean types always return false.

---

### Nz
**Syntax:**
- `Nz(value1, value2, ..., valueN)` — returns first non-null value
- `Nz(Text text)` — returns text or empty string if null

**Returns:** Same type as input / Text

**Description:** Multi-param: returns the first not-null value. Single text param: returns the text or empty string.

**Examples:**
- `Nz([Delivery Date], [Shipment Date])` — returns first non-null
- `Nz([Last Name])` — returns Last Name or empty string

---

## 9. DURATION FUNCTIONS

### Abs
**Syntax:** `Abs(Duration duration)` or `Abs(Numeric number)`
**Returns:** Duration / Numeric
**Description:** Returns the absolute value.

### Weeks
**Syntax:** `Weeks(Numeric weeks)`
**Returns:** Duration

### Days
**Syntax:** `Days(Numeric days)`
**Returns:** Duration

### Hours
**Syntax:** `Hours(Numeric hours)`
**Returns:** Duration

### Minutes
**Syntax:** `Minutes(Numeric minutes)`
**Returns:** Duration

### Seconds
**Syntax:** `Seconds(Numeric seconds)`
**Returns:** Duration

### Mod
**Syntax:** `Mod(Numeric dividend, Numeric divisor)` or `Mod(Duration, Duration)`
**Returns:** Numeric / Duration
**Description:** Modulus. Result has the same sign as the divisor.
**Examples:**
- `Mod(7, 3)` --> 1
- `Mod(-7, 3)` --> 2
- `Mod(7, -3)` --> -2

### Rem
**Syntax:** `Rem(Numeric dividend, Numeric divisor)` or `Rem(Duration, Duration)`
**Returns:** Numeric / Duration
**Description:** Remainder. Result has the same sign as the dividend.
**Examples:**
- `Rem(7, 3)` --> 1
- `Rem(-7, 3)` --> -1
- `Rem(7, -3)` --> 1

### ToWeeks
**Syntax:** `ToWeeks(Duration duration)`
**Returns:** Numeric

### ToDays
**Syntax:** `ToDays(Duration duration)`
**Returns:** Numeric

### ToHours
**Syntax:** `ToHours(Duration duration)`
**Returns:** Numeric

### ToMinutes
**Syntax:** `ToMinutes(Duration duration)`
**Returns:** Numeric

### ToSeconds
**Syntax:** `ToSeconds(Duration duration)`
**Returns:** Numeric
**Example:** `ToSeconds(Minutes(25))` --> 1500

---

## 10. TIMESTAMP FUNCTIONS

### Now
**Syntax:** `Now()`
**Returns:** Timestamp
**Description:** Returns a timestamp representing the current moment.

### Timestamp
**Syntax:** `Timestamp(Numeric year, Numeric month, Numeric day, Numeric hour, Numeric minute, Numeric second)`
**Returns:** Timestamp
**Description:** Returns a timestamp specified in UTC.
**Example:** `Timestamp(2013, 03, 25, 16, 10, 11)` --> 3/25/2013 4:10:11 PM UTC

---

## 11. LOCATION FUNCTIONS

### Distance
**Syntax:** `Distance(Location from, Location to)` or `Distance(Location from, Location to, Text unit)`
**Returns:** Numeric
**Description:** Distance between two locations on WGS ellipsoid. Unit: "m" (default), "km", or "mi".
**Example:** `Distance(ToLocation(35.658068, 139.751599), ToLocation(41.836944, -87.684722), "km")` --> 10,162

### Latitude
**Syntax:** `Latitude(Location location)`
**Returns:** Numeric

### Longitude
**Syntax:** `Longitude(Location location)`
**Returns:** Numeric

### ToLocation
**Syntax:** `ToLocation(Numeric latitude, Numeric longitude)` or `ToLocation(Text location)`
**Returns:** Location
**Description:** Constructs a location from coordinates. Text accepts comma-separated pair.

---

## 12. SPECIAL FUNCTIONS

### Between
**Syntax:** `Between(value, min, max)`
**Returns:** Boolean
**Description:** Returns true if value >= min and value <= max (inclusive on both ends).

### Case
**Syntax:** `Case(expression, value1, result1, ..., valueN, resultN, else-result)`
**Returns:** Same type as result1
**Description:** Compares expression to each value sequentially, returns corresponding result. If no match, returns else-result (null if omitted).
**Example:** `Case([Priority], "High", 100, "Medium", 70, "Low", 40)`

### If
**Syntax:** `If(condition1, result1, condition2, result2, ..., else-result)`
**Returns:** Same type as result1
**Description:** Returns result for first true condition. If none match, returns else-result (null if omitted).
**Examples:**
- `If([Priority] = "High", 100, [Priority] = "Medium", 70)`
- `If([Discount] = true, [SubTotal] - [Discount Value], [SubTotal])`

### In
**Syntax:** `In(expression, value1, value2, ..., valueN)` or `In(single-ref, multi-ref)`
**Returns:** Boolean
**Description:** Returns true if expression equals one of the listed values.
**Example:** `In([State], "CA", "IL")`

### DeviceLatitude
**Syntax:** `DeviceLatitude()`
**Returns:** Numeric
**Description:** GPS latitude of mobile device. Mobile Device actions only.

### DeviceLongitude
**Syntax:** `DeviceLongitude()`
**Returns:** Numeric
**Description:** GPS longitude of mobile device. Mobile Device actions only.

### DeviceLocation
**Syntax:** `DeviceLocation()`
**Returns:** Location
**Description:** GPS coordinates of mobile device. Mobile Device actions only.

### DeviceTimestamp
**Syntax:** `DeviceTimestamp()`
**Returns:** Timestamp
**Description:** Timestamp from mobile device. Mobile Device actions only.

### FileName
**Syntax:** `FileName(file_attachment_column)`
**Returns:** Text
**Description:** Returns the file name. Call-URL action body only.

### FileType
**Syntax:** `FileType(file_attachment_column)`
**Returns:** Text
**Description:** Returns the MIME type. Call-URL action body only.

### Hash
**Syntax:** `Hash(Text algorithm, Text data, Text encoding)`
**Returns:** Text
**Description:** Calculates hash. Algorithm: "MD5", "SHA", "SHA1", "SHA2_256", "SHA2_512". Encoding: "base64" or "binhex".

### HMAC
**Syntax:** `HMAC(Text algorithm, Text key, Text data, Text encoding)`
**Returns:** Text
**Description:** Calculates HMAC. Same algorithm/encoding options as Hash.

### ParentKey
**Syntax:** `ParentKey()`
**Returns:** Key value
**Description:** Returns key value for populating reference columns. Workflow action assignments only.

### Response
**Syntax:**
- `Response()` — raw response content
- `Response(Text path)` — extract named data from XML/JSON
- `Response(Text path, type)` — extract and convert

**Returns:** Text / specified type
**Description:** Retrieves server response data. Call-URL actions and Webhooks only. JSON path: "property.property[index].property".

### ResponseHeader
**Syntax:** `ResponseHeader(Text name)`
**Returns:** Text
**Description:** Retrieves HTTP header value. Call-URL actions and Webhooks only.

### ResponseStatus
**Syntax:** `ResponseStatus()`
**Returns:** Numeric
**Description:** Returns response status code. Call-URL Error Message formula only.

---

## 13. USER/SESSION SPECIFIC FUNCTIONS

### AppId
**Syntax:** `AppId()`
**Returns:** Text
**Description:** Returns the ID of the current database.

### BackURL
**Syntax:** `BackURL()`
**Returns:** Text
**Description:** Returns the current URL location for navigation/back links.

### Browser
**Syntax:** `Browser()`
**Returns:** Text
**Description:** Returns device type: "Desktop", "Tablet", "Mobile", or "TV".

### ColumnId
**Syntax:** `ColumnId(column)`
**Returns:** Text
**Description:** Returns a unique column ID number as text.

### Exists
**Syntax:** `Exists(reference-column)`
**Returns:** Boolean
**Description:** Returns true if the referred record exists and is accessible.

### IsUserEmail
**Syntax:** `IsUserEmail(Text email)`
**Returns:** Boolean
**Description:** Returns true if the email belongs to a database user.

### RecordId
**Syntax:** `RecordId()`
**Returns:** Text
**Description:** Returns the internal ID of the current record (distinct from the "Id" column).

### Role
**Syntax:** `Role()` or `Role(User user)`
**Returns:** Text
**Description:** Returns the role name assigned to the user.
**Example:** `Role() = "Admin"`

### TableId
**Syntax:** `TableId()`
**Returns:** Text
**Description:** Returns the ID of the current table.

### ToUser
**Syntax:** `ToUser(Text email)`
**Returns:** User
**Description:** Converts email to user object. Returns null if user doesn't exist.

### URL
**Syntax:** `URL()`
**Returns:** Text
**Description:** Returns the full URL of the current page.

### URLRoot
**Syntax:** `URLRoot()`
**Returns:** Text
**Description:** Returns the base URL.
**Example:** `URLRoot()` --> "https://www.teamdesk.net/secure/db/26134"

### User
**Syntax:** `User()`
**Returns:** User
**Description:** Returns the current user.
**Example:** `[Created By] = User()`

### UserToEmail
**Syntax:** `UserToEmail(User user)`
**Returns:** Text
**Description:** Returns the user's email address.

### UserToName
**Syntax:** `UserToName(User user)` or `UserToName(User user, Text format)`
**Returns:** Text
**Description:** Returns the user's full name. Format: "FF" = first-name-first, "LF" = last-name-first.

---

## 14. PARAMETERS FOR THE ASK

### Ask
**Syntax:** `Ask(condition)`
**Returns:** Boolean
**Description:** Evaluates condition only after user supplies parameter values. Used in view filter formulas with "Ask the User" option.
**Example:** `Ask([State] = [?State])`

### Parameter
**Syntax:**
- `Parameter(column)`
- `Parameter(column, Boolean required)`
- `Parameter(column, Text label)`
- `Parameter(column, Text label, Boolean required)`
- `Parameter(Text label, type)` — type: Text, Bool, Date, Time, Timestamp, Numeric, Duration, User, Location
- `Parameter(Text label, type, Boolean required)`

**Returns:** The entered value
**Description:** Creates a parameter for user input in conjunction with Ask function.

---

## 15. OPERATORS

### Logical Operators

| Operator | Syntax | Description |
|----------|--------|-------------|
| `and` | `a and b` | True if both operands are true |
| `or` | `a or b` | True if either operand is true |
| `not` | `not a` | True if operand is false |

### Comparison Operators

| Operator | Syntax | Description |
|----------|--------|-------------|
| `=` | `a = b` | Equal |
| `<>` | `a <> b` | Not equal |
| `>` | `a > b` | Greater than |
| `>=` | `a >= b` | Greater than or equal |
| `<` | `a < b` | Less than |
| `<=` | `a <= b` | Less than or equal |

**Note:** Null values produce null results in comparisons. Use `IsNull()` to test for null.

### Arithmetical Operators

| Operator | Syntax | Description |
|----------|--------|-------------|
| `+` | `a + b` | Addition (also Date+Duration, Time+Duration, Timestamp+Duration) |
| `-` | `a - b` | Subtraction (also Date-Date=Duration, Time-Time=Duration) |
| `*` | `a * b` | Multiplication (also Numeric*Duration) |
| `/` | `a / b` | Division (also Duration/Numeric, Duration/Duration=Numeric) |
| `&` | `a & b` | Text concatenation (null produces null) |
| `^` | `a ^ b` | Power (exponentiation) |

### Type Combination Rules for + and -

| Expression | Result |
|---|---|
| Date + Duration | Date |
| Date - Date | Duration |
| Duration + Duration | Duration |
| Numeric + Numeric | Numeric |
| Time + Duration | Time |
| Time - Time | Duration |
| Timestamp + Duration | Timestamp |
| Timestamp - Timestamp | Duration |

---

## Summary Columns (used in table relationships)

Summary columns aggregate data from related (child) records:

| Function | Description |
|----------|-------------|
| `Total([Amount])` | Sum of values |
| `Count([Id])` | Count of non-null values |
| `Average([Value])` | Average of values |
| `Min([Date])` / `Max([Date])` | Minimum / Maximum |
| `Concatenate(separator, [Column])` | Join text values |
| `First([Column])` / `Last([Column])` | First/last by sort order |

### With Filter
```
Total([Amount]) where [Status]="Paid"
```

---

## Common Formula Patterns

### Record Age
```
Today() - ToDate([Date Created])
```

### Status Colorization
```
If([Age] > Days(20), "RED", [Age] > Days(10), "YELLOW", "GREEN")
```

### Completeness Score
```
(If(IsNull([Phone]), 0, 1) + If(IsNull([Email]), 0, 1)) / 2
```

### Round Robin Assignment
```
Mod([Id] - 1, 3) + 1
```

### Due Date by Priority
```
ToDate([Date Created]) + Case([Priority], "High", Days(2), "Medium", Days(5), Days(7))
```

### Google Search Link
```
"https://www.google.com/search?q=" & URLEncode([Name])
```

### Inline Chart (QuickChart.io)
Formula-URL column that renders as a chart image in XHTML:
```
"https://quickchart.io/chart?c=" & URLEncode("{type:'bar',data:{labels:['" & [Month1] & "','" & [Month2] & "','" & [Month3] & "'],datasets:[{label:'kWh',data:[" & ToText([Val1]) & "," & ToText([Val2]) & "," & ToText([Val3]) & "],backgroundColor:'rgba(76,175,80,0.7)'}]}}")
```

### Progress Bar (Formula-XHTML)
```
"<div class='pb'><progress value='" & ToText(Int([Percentage])) & "' max='100'></progress> " & Format([Percentage], "0") & "%</div>"
```
Requires CSS in `dbstyles-V3.css`:
```css
.pb progress { width: 200px; height: 20px; }
.pb progress::-webkit-progress-value { background: #4CAF50; }
.pb progress::-moz-progress-bar { background: #4CAF50; }
```

### Pre-filtered View Link (Formula-URL)
```
URLRoot() & "/table/view?filter=" & URLEncode("[Status]=""Overdue""")
```

### Custom Auto-Number (Per-Category)
For sequences like `INV-2026-001` per client:
```
[Category Prefix] & "-" & ToText(Year(Today())) & "-" & PadLeft(ToText([Sequence]), 3, "0")
```
Where `[Sequence]` is set via Summary Max() + 1 on create, with Unique constraint.

### JSON Fragment for API Batching
Child record formula that builds its JSON representation:
```
"{""id"":""" & [External ID] & """,""value"":" & ToText([Amount]) & "}"
```
Parent Summary: `"[" & Concatenate(",", [JSON Fragment]) & "]"` → ready for a single API call.

### File Rename via Formula-URL
Assign a Formula-URL to a File Attachment column to force a rename:
```
Replace([File URL], ".jfif", ".jpg")
```
When a URL is assigned to a File Attachment, TeamDesk downloads the file using the URL's filename.

---

## Formula Column Types

| Type | Returns | Example |
|------|---------|---------|
| Formula-Text | String | `List(" ", [First], [Last])` |
| Formula-Number | Number | `[Price] * [Quantity]` |
| Formula-Date | Date | `Today() + Days(30)` |
| Formula-Time | Time | `Time(9, 0, 0)` |
| Formula-Timestamp | Timestamp | `Now()` |
| Formula-Duration | Duration | `[End Date] - [Start Date]` |
| Formula-Checkbox | Boolean | `[Amount] > 0` |
| Formula-URL | URL | `"https://..." & URLEncode([Id])` |
| Formula-Email | Email | `[Username] & "@" & [Domain]` |
| Formula-Phone | Phone | Phone number construction |

---

## Common Errors

- **"Type mismatch"**: Converting wrong types (use ToNumber, ToDate, etc.)
- **"Cyclic reference"**: Formula references itself directly/indirectly
- **"Prefix Related not allowed"**: Using Related[] outside summary filter
- **Position 0 error**: Formula returns wrong type for column
