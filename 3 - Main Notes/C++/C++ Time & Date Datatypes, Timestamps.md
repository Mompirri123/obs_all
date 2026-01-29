**Created:** 02.12.25, 04:37

**Status:** #

- **Hashtags:**
	- #CPP 
	- #Time 
	- #datatypes


**Links / Tags:** 
- **Relevance Links:**

- **Topic Tags:**


# C++ Time & Date Datatypes , Timestamps

## Different time Datatypes

- There are 2 different data types that used to store the date and time
	- `time_t` $\Rightarrow$ **time stamps**
	- `struct tm` $\Rightarrow$ **date time structures**

- **Time stamps** 
	- represent a moment in time as a single number
	- makes it easier for the computer to do calculations

- **Date time structures**
	- Structures that represent different components of the date and time as members
	- This makes it easier for us to specify dates
	- Date time structures have the following members
		- `tm_sec` - The seconds within a minute
		- `tm_min` - The minutes within an hour
		- `tm_hour` - The hour within a day (from 0 to 23)
		- `tm_mday` - The day of the month
		- `tm_mon` - The month (from 0 to 11 starting with January)
		- `tm_year` - The number of years since 1900
		- `tm_wday` - The weekday (from 0 to 6 starting with Sunday)
		- `tm_yday` - The day of the year (from 0 to 365 with 0 being January 1)
		- `tm_isdst` - Positive when daylight saving time is in effect, zero when not in effect and negative when unknown

## Creating Time Stamps

- `time()` function can only create a timestamp for the current date

- `mktime()` function can create a timestamp for any date
	- `mktime()` function converts a date-time structure into a timestamp
	- The `mktime()` function needs these members to have a value: 
		- `tm_year`, `tm_mon`, `tm_mday`, `tm_hour`, `tm_min`, `tm_sec`, `tm_isdst`

#### Example

- Creating a timestamp using `mktime()` function
```cpp
struct tm datetime;
time_t timestamp;

tm datetime = {};
datetime.tm_year = 2025 - 1900; // Number of years since 1900
datetime.tm_mon = 12 - 1; // Number of months since January (months are 0-indexed)
datetime.tm_mday = 17;
datetime.tm_hour = 12;
datetime.tm_min = 30;
datetime.tm_sec = 1;
// Daylight Savings must be specified
// -1 uses the computer's timezone setting
datetime.tm_isdst = -1;

timestamp = mktime(&datetime);

cout << ctime(&timestamp);
```

## Creating Date Time Structures

- `mktime()` also fills in the `tm_wday` and `tm_yday` members of the date-time structure with the correct values
	- Completes structure and gives a valid date-time

#### Example
- Finding the weekday of a specified date:
```cpp
// Create datetime structure and use mktime to fill in the missing members
struct tm datetime;

datetime.tm_year = 2023 - 1900; // Number of years since 1900
datetime.tm_mon = 12 - 1; // Number of months since January
datetime.tm_mday = 17;
datetime.tm_hour = 0; datetime.tm_min = 0; datetime.tm_sec = 0;
datetime.tm_isdst = -1;
mktime(&datetime);

string weekdays[] = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"};

cout << "The date is on a " << weekdays[datetime.tm_wday];
```

### localtime() and gmtime() functions

- `localtime()`, `gmtime()` functions can convert timestamps into date-time structures
- `localtime()`
	- returns a pointer to a structure representing the time in computer's time zone
- `gmtime()`
	- returns a pointer to a structure representing the time in GMT time zone

- These function return a pointer to a date-time structure
- To make sure its value does not change unexpectedly
	- we should make a copy of it by dereferencing the pointer

```cpp
time_t timestamp = time(&timestamp);
struct tm datetime = *localtime(&timestamp);

cout << datetime.tm_hour;
```

## Display Dates

- `ctime()` function displays the date contained in a timestamp
- To display dates from a datetime structure we can use the `asctime()` function.

### Example

- Display the date represented by a datetime structure:
```cpp
time_t timestamp = time(NULL);
struct tm datetime = *localtime(&timestamp);

cout << asctime(&datetime);
```

## asctime() vs mktime()
- `asctime()` function does not correct invalid dates
	- Ex: if you set the day of the month to 32, it will display 32.
	- The `mktime()` function can correct these kinds of errors

#### Example

- Correct a date before displaying it
```cpp
// Create the datetime structure and use mktime to correct mistakes
struct tm datetime;
datetime.tm_year = 2022 - 1900; // Number of years since 1900
datetime.tm_mon = 0; // 0 is January
datetime.tm_mday = 32;
datetime.tm_hour = 0; datetime.tm_min = 0; datetime.tm_sec = 0;
datetime.tm_isdst = -1;
mktime(&datetime);

cout << asctime(&datetime);
```

## Choosing how time is displayed

- The `ctime()`  `asctime()` functions allow us to display the date 
	- but they do not allow us to choose how it's displayed.
- `strftime()` helps choose how a date is displayed

#### Example: 

- Representing the current date in different ways
```cpp

time_t timestamp = time(NULL);
struct tm datetime = *localtime(&timestamp);

char output[50];

strftime(output, 50, "%B %e, %Y", &datetime);
cout << output << "\n";

strftime(output, 50, "%I:%M:%S %p", &datetime);
cout << output << "\n";

strftime(output, 50, "%m/%d/%y", &datetime);
cout << output << "\n";

strftime(output, 50, "%a %b %e %H:%M:%S %Y", &datetime);
cout << output << "\n";
```

The `strftime()` function formats a date and writes it as a C-style string into a `char` array. It has four parameters:

1. The first parameter points to the char array where the formatted date will be written.
2. The second parameter specifies the space available in the array.
3. The third parameter allows us to choose how the date is formatted using format specifiers.
4. The last parameter is a pointer to the datetime structure which contains the date we want to display.

The following table has some useful format specifiers. For a more complete list, look at the [strftime() reference page](https://www.w3schools.com/cpp/ref_ctime_strftime.asp).

|Format Specifier|Description|Example|
|---|---|---|
|`%a`|Short representation of the weekday|`Fri`|
|`%b`|Short representation of the month name|`Dec`|
|`%B`|Full representation of the month name|`December`|
|`%d`|Day of the month with leading zero|`09`|
|`%e`|Day of the month with leading spaces|`9`|
|`%H`|24-hour format of an hour|`14`|
|`%I`|12-hour format of an hour|`02`|
|`%M`|Minutes within an hour|`30`|
|`%p`|AM or PM|`PM`|
|`%S`|Seconds within a minute|`01`|
|`%y`|2-digit year representation|`23`|
|`%Y`|4-digit year representation|`2023`|

## Measuring Time

There are two different functions that can be used to measure differences in time.

The `difftime()` function measures the number of seconds that passed between two different time stamps. This is useful when measuring time differences between dates.

#### Example

Measure the time difference between two timestamps

```java
time_t now;
time_t nextyear;
struct tm datetime;

now = time(NULL);
datetime = *localtime(&now);
datetime.tm_year = datetime.tm_year + 1;
datetime.tm_mon = 0;
datetime.tm_mday = 1;
datetime.tm_hour = 0; datetime.tm_min = 0; datetime.tm_sec = 0;
datetime.tm_isdst = -1;
nextyear = mktime(&datetime);

int diff = difftime(nextyear, now);

cout << diff << " seconds until next year";
```

- The `clock()` function is useful for measuring short intervals of time while the program is running
- It is more precise than the `difftime()` function.

- Each call to the clock function returns a special kind of timestamp measured in clocks (a unit of time that depends on how the library was implemented) which has a data type `clock_t`
- To measure a time difference, store a timestamp at two different moments in time and then subtract them.
- The time difference is measured in clocks, but you can convert it into seconds by dividing it by the `CLOCKS_PER_SEC` constant

#### Example

- Measure how long it takes for the program to run:

```java
clock_t before = clock();
int k = 0;
for(int i = 0; i < 100000; i++) {
  k += i;
}
clock_t duration = clock() - before;
cout << "Duration: " << (float)duration / CLOCKS_PER_SEC << " seconds";
```

**Note:** Make sure to cast the value to a `float` or `double` type before dividing, otherwise it may result in an integer division which would cause the decimal part to be cut off.

# References


## Closely Related Notes

### Next:

### Prev:
