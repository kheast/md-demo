# `mitto.iov2.tocsv`

## General

### The `path` parameter

The `path` parameter specifies the path, including filename, in which the CSV data will
be placed.

Example path specification:
```
'path': '/var/mitto/data/daily-results.csv'
```

## Date and Time Substitution in `path`

A number of *datetime variables* are available for use in dynamically creating paths and
filenames at runtime.  These can be useful when creating date-based CSV exports from jobs
that are regularly run on a schedule.

### Current Datetime Substitution

One group of datetime variables represent the time at which the export job was started.
For example, in a job that is run on a daily basis to export CSV data, the following
path:
```
'path': '/var/mitto/data/daily-results-{year}-{month}-{day}.csv'
```
might create the following files over time:
```
/var/mitto/data/daily-results_2019-01-29.csv
/var/mitto/data/daily-results_2019-01-30.csv
/var/mitto/data/daily-results_2019-01-31.csv
/var/mitto/data/daily-results_2019-02-01.csv
```

In the above `path`, `year`, `month`, and `day` are datetime variables.  When any of
these variables are enclosed between `'{'` and `'}` and present in the `path`, they are
replaced with actual the year, month, and day of the point in time that the job was
started.

The datetime variables `week`, `hour`, `minute`, `second`, and `start_time` are also
available.  All of the variables, except `start_time`, will be replaced with what one
would expect.  `start_time` is somewhat different; it is replaced with the date and time
at which the job was started, in ISO format, e.g.: `2019-01-01T00:30:00`.

If the export job is started at 00:30:30AM on January 1, 2019, current datetime
variables will be replaced with these values:

| Variable     | Value |
| ------------ | ----- |
| `year`  | `2019` |
| `month` | `01` |
| `week`  | `01` |
| `day`   | `01` |
| `hour`  | `00` |
| `start_time` | `01-01-2019T00:30:30` |

### Relative Datetime Substitution

A number of other datetime variables, all of which are relative to the start time, are
available: `last_year`, `last_month`, `last_week`, `last_day`, `last_hour`, `next_year`,
`next_month`, `next_week`, `next_day`, and `next_hour`.

Given the previous start time, the relative datetime variables will be replaced with
these values:


| Variable     | Value |
| ------------ | ----- |
| `last_year`  | `2018` |
| `last_month` | `12` |
| `last_week`  | `52` |
| `last_day`   | `31` |
| `last_hour`  | `23` |


| Variable     | Value |
| ------------ | ----- |
| `next_year`  | `2020` |
| `next_month` | `02` |
| `next_week`  | `02` |
| `next_day`   | `02` |
| `next_hour`  | `01` |


With relative datetimes, it is possible to specify a path for a file containing an export
of last year's data:
```
'path': '/var/mitto/data/yearly-results_{last_year}.csv'
```

They can also be used to organize files from a  daily export in a date-based directory
structure:
```
'path': '/var/mitto/data/daily/{year}/{month}/daily_{day}.csv'
```

## Formatting Datetime Variables

The replacement values for the datetime variables shown thus far default to a format
useful for common tasks.  However, there are situations in which other
representations may be desired. For example, replacing `month` with the full month name
instead of a two-digit number.

A *format directive* can be used to modify the format of the value that replaces a datetime
variable.  A path containing:
```
results_{year}-{month:%B}-{day}.csv
```
produces:
```
results_2019-January-01.csv
```
and:
```
results_{year}-{month:%B}-{day}-{day:%A}.csv
```
produces:
```
results_2019-January-01-Tuesday.csv
```

In these examples, a `':'` separates the datetime variable from its associated format
directive.  A format directive always begins with a `'%'` which is followed by one or
two characters.

A few format directives are listed in the following table:

| Code | Meaning | Example |
| ---- | ------- | ------- |
| `%a` | Weekday as locale's abbreviated name | `Mon` |
| `%w` | Weekday as a decimal number, Sunday = 0 | `1` |
| `%p` | Locale's equivalent of either AM or PM | `PM` |
| `%j` | Day of the year as zero padded decimal number | `273 ` |
| `%U` | Week number of the year, Sunday as first day | `39` |
| `%W` | Week number of the year, Monday as first day | `39` |

A complete list of format directives can be found at the end of this document.

## Datetime Variables Revisited

Assume that one wishes to schedule a job to run on the first of every month.  The job
will summarize data from the previous month and export it to a CSV file.  The name of
the CSV file is to be based on the data's year and month.  This isn't possible with the
functionality described thus far.  To be able to accomplish this task, we must revisit
datetime variables and describe them in more detail.

Each datetime variable contains complete information about a specific point in time --
the year, month, day, hour, minute, second, microsecond, and UTC offset.  When a
datetime variable is encountered in the path, it is replaced with a default
representation specific to that datetime variable.  As we have seen, `{year}` is
replaced with a four digit year, `{hour}` is replaced with a two digit zero padded hour
based on 24 hour clock, etc.

A format directives can be used to override the default replacements for a dattime
variable.  Format directives
access data stored in the datetime variable -- data that may not be present in the
default replacement.  For example `{hour:%j}` would be replaced with the day of the year
associated with the `hour` date time variable.

Assuming that `start_time` is `01-01-2019T00:30:30`, the following table illustrates the
entire value contained by each datetime variable, the format code used to create its
default replacement value, and an example of the variable's default replacement value:

| Variable     | Value                  | Code | Replacement |
| -----------  | ---------------------  | ---- | ------- |
| `last_year`  | `start_time` - 1 year  | `%Y` | `2018` |
| `year`       | `start_time`           | `%Y` | `2019` |
| `next_year`  | `start_time` + 1 year  | `%Y` | `2020` |
| `last_month` | `start_time` - 1 month | `%m` | `12` |
| `month`      | `start_time`           | `%m` | `01` |
| `next_month` | `start_time` + 1 month | `%m` | `02` |
| `last_week`  | `start_time` - 1 week  | \    | `52` |
| `week`       | `start_time`           | \    | `01` |
| `next_week`  | `start_time` + 1 week  | \    | `02` |
| `last_day`   | `start_time` - 1 day   | `%d` | `31` |
| `day`        | `start_time`           | `%d` | `01` |
| `next_day`   | `start_time` + 1 day   | `%d` | `02` |
| `last_hour`  | `start_time` - 1 hour  | `%H` | `23` |
| `hour`       | `start_time`           | `%H` | `00` |
| `next_hour`  | `start_time` + 1 hour  | `%H` | `01` |
| `minute`     | `start_time`           | `%M` | `30` |
| `second`     | `start_time`           | `%S` | `30` |

Armed with knowledge of the actual contents of datetime variables and their formatting,
it is now possible to specify the path described at the beginning of this section. The
key is knowing that `last_month` is exactly one month less than `start_time`.
```
'path': '/var/mitto/data/monthly_results_{last_month:%Y}-{last_month:%m}.csv'
```
If the job is run on 1/1/2019, the file will be correctly named:
`monthly_results_2018-12.csv`.  In fact, this job could be run at any time during
January 2019 and the file will be correctly named.

*Note*: the default replacements for `last_week`, `week`, and `next_week` (the two digit
week of the year) have no corresponding format directive.  These datetime variables,
however, can be used with all format directives described in this document.

## Appendix - Format Directives

All of these format directives can be used with any datetime variable:

| Code | Meaning | Example |
| ---- | ------- | ------- |
|`%a`| 	Weekday as locale’s abbreviated name. |`Mon`|
|`%A`| 	Weekday as locale’s full name. |`Monday`|
|`%w`| 	Weekday as a decimal number, where 0 is Sunday and 6 is Saturday. |`1`|
|`%d`| 	Day of the month as a zero-padded decimal number. |`30`|
|`%-d`| 	Day of the month as a decimal number. (Platform specific) |`30`|
|`%b`| 	Month as locale’s abbreviated name. |`Sep`|
|`%B`| 	Month as locale’s full name. |`September`|
|`%m`| 	Month as a zero-padded decimal number. |`09`|
|`%-m`| 	Month as a decimal number. (Platform specific) |`9`|
|`%y`| 	Year without century as a zero-padded decimal number. |`13`|
|`%Y`| 	Year with century as a decimal number. |`2013`|
|`%H`| 	Hour (24-hour clock) as a zero-padded decimal number. |`07`|
|`%-H`| 	Hour (24-hour clock) as a decimal number. (Platform specific) |`7`|
|`%I`| 	Hour (12-hour clock) as a zero-padded decimal number. |`07`|
|`%-I`| 	Hour (12-hour clock) as a decimal number. (Platform specific) |`7`|
|`%p`| 	Locale’s equivalent of either AM or PM. |`AM`|
|`%M`| 	Minute as a zero-padded decimal number. |`06`|
|`%-M`| 	Minute as a decimal number. (Platform specific) |`6`|
|`%S`| 	Second as a zero-padded decimal number. |`05`|
|`%-S`| 	Second as a decimal number. (Platform specific) |`5`|
|`%f`| 	Microsecond as a decimal number, zero-padded on the left. |`000000`|
|`%z`| 	UTC offset in the form +HHMM or -HHMM (empty string if the the object is naive). |  |
|`%Z`| 	Time zone name (empty string if the object is naive). |  |
|`%j`| 	Day of the year as a zero-padded decimal number. |`273`|
|`%-j`| 	Day of the year as a decimal number. (Platform specific) |`273`|
|`%U`| 	Week number of the year (Sunday as the first day of the week) as a zero padded decimal number. All days in a new year preceding the first Sunday are considered to be in week 0. |`39`|
|`%W`| 	Week number of the year (Monday as the first day of the week) as a decimal number. All days in a new year preceding the first Monday are considered to be in week 0. |`39`|
|`%c`| 	Locale’s appropriate date and time representation. 	| `Mon Sep 30 07:06:05 2013`|
|`%x`| 	Locale’s appropriate date representation. |`09/30/13`|
|`%X`| 	Locale’s appropriate time representation. |`07:06:05`|
|`%%`| 	A literal '%' character. |`%`|
