# 前言
jdk8之前我们一直在用Date、Calender和SimpleDateFormat，但它的API不够强大，还存在线程安全问题，一直被人诟病。jdk8提供了新的时间API，在java.time包里的类是不可变且线程安全的，它和Joda是同一个作者实现的，因此借鉴了Joda很多特点，如果你用习惯Joda，可以很方便地切换到java.time类的使用

关注公众号，一起交流，微信搜一搜: 潜行前行
---

## java.time类的简单介绍
-	Date和time做下区分，Date（日期）的单位是年月日。time（时间）的单位是时分秒

类　 | 描述
--- | ---
Instant 　         |　时间戳（瞬时时间，带时区）
LocalDate　		  |　日期（比如：2018-09-24，不带时区）
LocalTime　	  	  |　时间（比如：10:32:10，不带时区）
LocalDateTime　　　 |　日期时间（比如：2018-09-24 10:32:10，不带时区）
Duration　		  |　两个时间的差，精确到秒或纳秒
Peroid 　　		  |　两个日期的差（精确到日）
DateTimeFormatter　|　日期时间格式化类
ZoneId 　　　　　　　|　时区
ZoneOffset         |　时区偏移量(比如：+8:00)
ZonedDateTime 　　　|　带时区的日期时间
ChronoUnit 　　　　 |　日期枚举类(在时间加减操作可用到)
MonthDay　         |　月日
YearMonth　        |　年月

## Clock时钟
-	Clock是关联上时区的时钟，Clock可以获取时间戳和时区ZoneId，用来代替System.currentTimeMillis()和TimeZone.getDefault()。它是个抽象类，一共有四个子类
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b021478e5a364cc2a98394bbe8489787~tplv-k3u1fbpfcp-watermark.image)
```java
public static Clock systemDefaultZone()
public static Clock offset(Clock baseClock, Duration offsetDuration)
public static Clock tick(Clock baseClock, Duration tickDuration)
public static Clock fixed(Instant fixedInstant, ZoneId zone) 
--------下面方法由上面四个子类实现---------
//获取时区
public abstract ZoneId getZone()
//指定时区
public abstract Clock withZone(ZoneId zone)
//获取时间戳
public abstract Instant instant()
```
-	系统默认本地时钟SystemClock
```java
Clock clock = Clock.systemDefaultZone();
System.out.println(clock.getZone());
Instant instant = clock.instant();
System.out.println(instant);
----------输出结果-----------
Asia/Shanghai
2021-01-03T05:05:31.791Z
1609650331791
```
-	偏移时钟OffsetClock
```java
Clock clock = Clock.systemDefaultZone();
Clock pastClock = Clock.offset(clock, Duration.ofMillis(-10000));
System.out.println(pastClock.getZone());
//当前时间和过去pastClock相差10000毫秒
System.out.println(clock.millis() - pastClock.millis());
----------输出结果-----------
10000
```
-	周期计时的TickDuration，截取时间到最接近的上个周期或下个周期的时间。注意：TickDuration不会把当前时间点作为周期的起始时间
```java
Clock clock = Clock.systemDefaultZone();
Clock nearestHourClock = Clock.tick(clock, Duration.ofMillis(10));
//当前时间是2021-01-03T05:36:54.088Z，周期是10毫秒，TickDuration自动
//选择2021-01-03T05:36:54.090Z作为起始时间
System.out.println(clock.instant());
System.out.println(nearestHourClock.instant());
Thread.sleep(10);
System.out.println(clock.instant());
System.out.println(nearestHourClock.instant());
----------输出结果-----------
2021-01-03T05:43:19.088Z
2021-01-03T05:43:19.090Z
2021-01-03T05:43:19.107Z
2021-01-03T05:43:19.100Z
```
-	时间不变的FixedInstant
```java
Clock clock = Clock.systemDefaultZone();
Clock fixedClock = Clock.fixed(clock.instant(), ZoneId.systemDefault());
System.out.println(fixedClock.instant());
Thread.sleep(1000);
System.out.println(fixedClock.instant());
----------输出结果-----------
2021-01-03T05:27:43.272Z
2021-01-03T05:27:43.272Z
```

## Temporal
-	时间类的统一接口，定义一些通用的方法操作，如：某时间单位的加减，设置为时间域为某一固定值
```java
public interface Temporal extends TemporalAccessor {
    //获取时间类能表示的TemporalField范围值
    public ValueRange range(TemporalField field)
    //设置TemporalField时间域的值
    public Temporal with(TemporalField field, long newValue)
    //根据TemporalAdjuster接口转化时间
    public Temporal with(TemporalAdjuster adjuster)
    //增加指定TemporalUnit单位的数量
    public Temporal plus(long amountToAdd, TemporalUnit unit)
    //减少指定TemporalUnit单位的数量
    public Temporal minus(long amountToSubtract, TemporalUnit unit)
```
-	Temporal的子类
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3b26f5bb7374205a41e721f79eacb94~tplv-k3u1fbpfcp-watermark.image)
## Instant
-	Instant是用来操作时间戳的，带时区，默认UTC的格林威治时区。因此其他时间类和Instant互转时，需要指定自己的时区
```java
public static Instant now()
public static Instant now(Clock clock)
//根据毫秒生成Instant
public static Instant ofEpochMilli(long epochMilli)
//根据秒生成Instant，外附加纳秒单位的数值
public static Instant ofEpochSecond(long epochSecond, long nanoAdjustment)
//text的格式：2021-01-04T06:37:53.743Z
public static Instant parse(CharSequence text) 
//计算与Temporal相隔的时间差，以TemporalUnit为单位度量
public long until(Temporal endExclusive, TemporalUnit unit) 
```
-	时间戳的创建
```java
Instant instant = Instant.now();
System.out.println(instant);
instant = Instant.now(Clock.systemDefaultZone());
System.out.println(instant);
//和Date的相互转换
instant = Instant.ofEpochMilli(new Date().getTime());
System.out.println(instant);
//根据字符串生成时间，支持到纳秒级别
instant = Instant.parse("1995-10-23T10:12:35.999999999Z");
System.out.println(instant);
//根据秒和偏差的纳秒 创建时间戳
Instant preInstant = Instant.ofEpochSecond(1609741558,1);
//过去时间和当前时间的相差值,可以指定单位TemporalUnit
System.out.println(preInstant.until(Instant.now(), ChronoUnit.MINUTES));
---------------输出结果------------------------
2021-01-04T06:37:53.743Z
2021-01-04T06:37:53.795Z
2021-01-04T06:37:53.795Z
1995-10-23T10:12:35.999999999Z
2021-01-04T06:25:58.000000001Z
2021-01-04T06:37:53.798Z
11
```
-	Instant的使用
```java
Instant instant = Instant.now();
//设置时区为America/El_Salvador(美国)
ZonedDateTime zonedDateTime = instant.atZone(ZoneId.of("America/El_Salvador"));
System.out.println(zonedDateTime);
//设置时区为偏移-6的时区（美国）
OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(-6));
System.out.println(offsetDateTime);
//天数加2，分钟数减1。月数设定为12月
Instant fixInstant = instant.minus(1, ChronoUnit.MINUTES)
        .plus(2, ChronoUnit.DAYS);
System.out.println(instant);
System.out.println(fixInstant);
---------------输出结果-----------------
2021-01-04T00:53:01.895-06:00[America/El_Salvador]
2021-01-04T00:53:01.895-06:00
2021-01-04T06:53:01.895Z
2021-01-06T06:52:01.895Z
```

## LocalTime 
-	LocalTime是用来操作时分秒的类，外加精确到纳秒级别；无时区概念，转Instant需要先设置时区
-	LocalTime的构造方法
```java
public static LocalTime now() 
public static LocalTime of(int hour, int minute, int second, int nanoOfSecond)
//距离凌晨的秒数
public static LocalTime ofSecondOfDay(long secondOfDay)
//距离凌晨的纳秒数
public static LocalTime ofNanoOfDay(long nanoOfDay)
public static LocalTime parse(CharSequence text, DateTimeFormatter formatter) 
//jdk9
public static LocalTime ofInstant(Instant instant, ZoneId zone)
```
-	LocalTime与其他时间类的转换
```java
public LocalDateTime atDate(LocalDate date)
public OffsetTime atOffset(ZoneOffset offset) 
public long toEpochSecond(LocalDate date, ZoneOffset offset)
```
-	LocalTime创建示例
```java
//  1的单位是纳秒
LocalTime localTime = LocalTime.of(12, 12, 12, 1);
System.out.println(localTime);
localTime = LocalTime.ofSecondOfDay(60 * 60 * 12 + 60 * 12);
System.out.println(localTime);
localTime = LocalTime.parse("12:12:12", DateTimeFormatter.ISO_TIME);
System.out.println(localTime);
---------------输出结果-----------------
12:12:12.000000001
12:12
12:12:12
```
-	LocalTime的常用处理方法
```java
LocalTime localTime = LocalTime.of(12, 12, 12, 1);
System.out.println(localTime);
//拼接日期，生成日期时间
LocalDateTime dateTime = localTime.atDate(LocalDate.now());
System.out.println(dateTime);
//设定时区，时间量不变
OffsetTime offsetTime = LocalTime.now().atOffset(ZoneOffset.ofHours(-6));
System.out.println(offsetTime);
//当前时间加日期,并设置时区为offset
long seconds = LocalTime.now().toEpochSecond(LocalDate.now(), ZoneOffset.ofHours(-6));
System.out.println(seconds);
---------------输出结果-----------------
12:12:12.000000001
2021-01-04T12:12:12.000000001
16:29:33.917387700-06:00
1609799373
```
## LocalDate 
-	LocalDate是用来操作年月日的类；表示的时间单位是截止到日，不包括小时及后面的单位
```java
public static LocalDate now()
public static LocalDate of(int year, int month, int dayOfMonth)
public static LocalDate ofInstant(Instant instant, ZoneId zone)
```
```java
//当天是当月的第几天
public int getDayOfMonth()
//当天是当年的第几天
public int getDayOfYear()
//当天是当周的第几天（1~7）
public DayOfWeek getDayOfWeek() 
//是否闰年
public boolean isLeapYear() 
public LocalDateTime atTime(LocalTime time)
//设置时间为今天的凌晨
public LocalDateTime atStartOfDay()
//以下两个方法是jdk9,获取当前时间到endExclusive的每一个日期
public Stream<LocalDate> datesUntil(LocalDate endExclusive)
public Stream<LocalDate> datesUntil(LocalDate endExclusive, Period step) 
```
-	LocalDate的创建示例
```java
LocalDate localDate = LocalDate.now();
System.out.println(localDate);
localDate = LocalDate.of(2020,3,19);
System.out.println(localDate);
localDate = LocalDate.parse("20210319", DateTimeFormatter.BASIC_ISO_DATE);
System.out.println(localDate);
---------------输出结果-----------------
2021-01-04
2020-03-19
2021-03-19
```
-	LocalDate的操作
```java
LocalDate localDate = LocalDate.parse("20210319", DateTimeFormatter.BASIC_ISO_DATE);
System.out.println("当前年的第几天:" + localDate.getDayOfYear() + "; 当前月的第几天" + localDate.getDayOfMonth()+"; 当前星期几:"+localDate.getDayOfWeek());
//加LocalTime转变为LocalDateTime
LocalDateTime dateTime = localDate.atTime(LocalTime.now());
System.out.println(dateTime);
---------------输出结果-----------------
当前年的第几天:78; 当前月的第几天19; 当前星期几:FRIDAY
2021-03-19T15:37:54.713
```
## LocalDateTime
-	相当于LocalDate 和 LocalTime的结合，用来表示年月日时分秒的类，外加精确到纳秒级别；无时区概念，转Instant需要先设置时区
```java
public static LocalDateTime now()
public static LocalDateTime of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond)
public static LocalDateTime of(LocalDate date, LocalTime time) 
public static LocalDateTime ofInstant(Instant instant, ZoneId zone)
```
```java
//设置偏移时区
public OffsetDateTime atOffset(ZoneOffset offset) 
//设置时区,ZonedDateTime会根据夏令时调的，纯粹是配合美国政策的东西
public ZonedDateTime atZone(ZoneId zone) 
//截取时间到TemporalUnit单位
public Instant truncatedTo(TemporalUnit unit)
default Instant toInstant(ZoneOffset offset) 
default long toEpochSecond(ZoneOffset offset) 
```
-	LocalDateTime构造示例
```java
LocalDateTime dateTime = LocalDateTime.of(2021,3,19,12,12,12,01);
System.out.println(dateTime);
dateTime = LocalDateTime.of(LocalDate.now(), LocalTime.now());
System.out.println(dateTime);
dateTime = LocalDateTime.parse("2021-03-19 12:12:12", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
System.out.println(dateTime);
---------------输出结果-----------------
2021-03-19T12:12:12.000000001
2021-01-04T16:36:40.193887
2021-03-19T12:12:12
```
-	LocalDateTime操作示例
```java
LocalDateTime dateTime = LocalDateTime.now();
//设置偏移时区为-6
OffsetDateTime offsetDateTime = dateTime.atOffset(ZoneOffset.ofHours(-6));
System.out.println(offsetDateTime);
//设置时区为美国时区
ZonedDateTime zonedDateTime = dateTime.atZone(ZoneId.of("America/El_Salvador"));
System.out.println(zonedDateTime);
System.out.println(zonedDateTime.toEpochSecond());
//输出LocalDateTime的时间戳，因为LocalDateTime是不带时区的，需要指定时区
System.out.println(dateTime.toEpochSecond(ZoneOffset.ofHours(0)));
```
## Period和Duration
-	Period操作的时间跨度是年月日
```java
public static Period of(int years, int months, int days)
public static Period between(LocalDate startDateInclusive, LocalDate endDateExclusive)
//获取当前Period实例的单位
public List<TemporalUnit> getUnits()
//获取TemporalUnit的数量
public long get(TemporalUnit unit)
public int getYears() 
public int getMonths() 
public int getDays() 
```
-	Duration操作的时间跨度是时分秒，外加纳秒
```java
//间隔days天的时间差(单位秒)
public static Duration ofDays(long days)
//间隔hours小时的时间差(单位秒)
public static Duration ofHours(long hours)
public static Duration ofSeconds(long seconds, long nanoAdjustment)
public static Duration parse(CharSequence text) 
public static Duration between(Temporal startInclusive, Temporal endExclusive)
public List<TemporalUnit> getUnits()
//时间差的间隔天数，天数单位部分，没有则是０
public long toDaysPart()
//时间差的小时跨度数，小时单位部分，没有则是０
public int toHoursPart()
//时间差的分钟跨度数，分钟单位部分，没有则是０
public int toMinutesPart()
//时间差的秒跨度数，秒单位分，没有则是０
public int toSecondsPart()
```
-	使用示例
```java
Duration duration = Duration.ofHours(1+24);
System.out.println("相隔天数："+duration.toDaysPart());
System.out.println("外加"+duration.toHoursPart()+"小时");
---------------输出结果-----------------
相隔天数：1
外加1小时
```

## DateTimeFormatter
```java
//构造指定模式的时间格式类
public static DateTimeFormatter ofPattern(String pattern)
//构造指定模式的时间格式类，并指定Locale
public static DateTimeFormatter ofPattern(String pattern, Locale locale)
public DateTimeFormatter withLocale(Locale locale) 
//解析出来的时间是带时区的
public DateTimeFormatter withZone(ZoneId zone)
//获取当前时区
public ZoneId getZone() 
public String format(TemporalAccessor temporal)
//解析字符串为TemporalAccessor
public TemporalAccessor parse(CharSequence text) 
```
-	使用示例
```java
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
dateTimeFormatter = dateTimeFormatter.withZone(ZoneId.of("America/El_Salvador"));
System.out.println(dateTimeFormatter.parse("2021-03-19 12:12:12"));
System.out.println(dateTimeFormatter.parse("2021-03-19 12:12:12").getLong(ChronoField.INSTANT_SECONDS));
---------------输出结果-----------------
{InstantSeconds=1616177532},ISO,America/El_Salvador resolved to 2021-03-19T12:12:12
1616177532
```

## ZoneId和ZoneOffset
-	ZoneId表示时区，会根据夏令时调整。在不同年份时间，该ZoneId对应的ZonedDateTime会变动
```java
public static ZoneId systemDefault()
//获取jdk支持的ZoneId的集合
public static Set<String> getAvailableZoneIds()
public static ZoneId of(String zoneId)
//根据一个TemporalAccessor的实体获取，如:ZonedDateTime
public static ZoneId from(TemporalAccessor temporal)
```
-	ZoneOffset也表示时区，固定时间的偏移。不会根据夏令时调整，固定不变的
```java
public static ZoneOffset of(String offsetId)
public static ZoneOffset ofHours(int hours) 
public static ZoneOffset ofHoursMinutesSeconds(int hours, int minutes, int seconds)
```
## ZonedDateTime和OffsetDateTime
-	ZonedDateTime，对应ZoneId加LocalDateTime的结合
```java
public static ZonedDateTime now()
public static ZonedDateTime ofInstant(Instant instant, ZoneId zone)
public static ZonedDateTime ofInstant(LocalDateTime localDateTime, ZoneOffset offset, ZoneId zone)
public static ZonedDateTime parse(CharSequence text, DateTimeFormatter formatter) 
public LocalDateTime toLocalDateTime()
public LocalDate toLocalDate() 
//转化为OffsetDateTime（时区变成固定的，不再根据夏令时动态调整）
public OffsetDateTime toOffsetDateTime() 
```
-	OffsetDateTime，对应ZoneOffset加LocalDateTime的结合
```java
public static OffsetDateTime of(LocalDate date, LocalTime time, ZoneOffset offset) 
public static OffsetDateTime of(LocalDateTime dateTime, ZoneOffset offset) 
public static OffsetDateTime ofInstant(Instant instant, ZoneId zone) 
public static OffsetDateTime parse(CharSequence text, DateTimeFormatter formatter)
//当前时间转成offset时区表示的时间
public OffsetDateTime withOffsetSameInstant(ZoneOffset offset) 
```
-	ZonedDateTime和OffsetDateTime使用示例
```java
System.out.println(LocalDateTime.now());
//当前时间并转为America/Toronto时区的时间（带America/Toronto时区，会根据夏令时调整时间）
ZonedDateTime zonedDateTime = ZonedDateTime.ofInstant(Instant.now(), ZoneId.of("America/Toronto"));
System.out.println(zonedDateTime);
//当前时间并转为+0时区的时间
OffsetDateTime offsetDateTime = OffsetDateTime.ofInstant(Instant.now(), ZoneOffset.of("+0"));
System.out.println(offsetDateTime);
---------------输出结果---------------------
2021-01-10T15:07:08.072968
2021-01-10T02:07:08.073187-05:00[America/Toronto]
2021-01-10T07:07:08.075340Z
```
## MonthDay和YearMonth
-	月日的操作类MonthDay
```java
public static MonthDay now()
public static MonthDay of(int month, int dayOfMonth)
public static MonthDay parse(CharSequence text, DateTimeFormatter formatter)
public LocalDate atYear(int year)
public Month getMonth()
//当前月的总天数
public int getDayOfMonth()
```
-	年月的操作类YearMonth
```java
public static YearMonth now()
public static YearMonth of(int year, int month) 
//根据一个时间获取YearMonth，如果LocalDate
public static YearMonth from(TemporalAccessor temporal)
//是否闰年
public boolean isLeapYear()
public Month getMonth()
public int getYear()
//当前年的总天数
public int lengthOfYear()
//此天数是否在这个月是有效的
public boolean isValidDay(int dayOfMonth)
public LocalDate atDay(int dayOfMonth)
```
-	使用示例
```java
MonthDay monthDay = MonthDay.of(3,19);
System.out.println(MonthDay.from(LocalDate.now()).isAfter(monthDay));
YearMonth yearMonth = YearMonth.of(2021,03);
System.out.println("2021 是否是闰年:"+yearMonth.isLeapYear());
------------输出结果--------------
false
2021 是否是闰年:false
```
## ChronoUnit
-	时间度量单位，枚举类，继承TemporalUnit。它表示的是一个时间间隔用什么单位度量，比如两天的时间间隔可以用48个小时代替表示。一般用于某时间单位的设置，加减操作
```java
public enum ChronoUnit implements TemporalUnit {
    NANOS("Nanos", Duration.ofNanos(1)),
    MICROS("Micros", Duration.ofNanos(1000)),
    MILLIS("Millis", Duration.ofNanos(1000_000)),
    SECONDS("Seconds", Duration.ofSeconds(1)),
    MINUTES("Minutes", Duration.ofSeconds(60)),
    HOURS("Hours", Duration.ofSeconds(3600)),
    HALF_DAYS("HalfDays", Duration.ofSeconds(43200)),
    DAYS("Days", Duration.ofSeconds(86400)),
    WEEKS("Weeks", Duration.ofSeconds(7 * 86400L)),
    MONTHS("Months", Duration.ofSeconds(31556952L / 12)),
    YEARS("Years", Duration.ofSeconds(31556952L)),
    DECADES("Decades", Duration.ofSeconds(31556952L * 10L)),
    CENTURIES("Centuries", Duration.ofSeconds(31556952L * 100L)),
    MILLENNIA("Millennia", Duration.ofSeconds(31556952L * 1000L)),
    ERAS("Eras", Duration.ofSeconds(31556952L * 1000_000_000L)),
    FOREVER("Forever", Duration.ofSeconds(Long.MAX_VALUE, 999_999_999));
	//ChronoUnit的定义
    private final String name;
    private final Duration duration;
    private ChronoUnit(String name, Duration estimatedDuration) {
        this.name = name;
        this.duration = estimatedDuration;
    }
}
```

## ChronoField
-	继承TemporalField，枚举类。和ChronoUnit功能类似，基于TemporalUnit实现，一般用于获取不同时间域的值
```java
public enum ChronoField implements TemporalField {
    //一秒钟的纳秒数
    NANO_OF_SECOND("NanoOfSecond", NANOS, SECONDS, ValueRange.of(0, 999_999_999))
    //一分钟的秒数
    SECOND_OF_MINUTE("SecondOfMinute", SECONDS, MINUTES, ValueRange.of(0, 59), "second")
    //一个小时的分钟数
    MINUTE_OF_HOUR("MinuteOfHour", MINUTES, HOURS, ValueRange.of(0, 59), "minute")
    //一上午或者一下午有多少个小时
    CLOCK_HOUR_OF_AMPM("ClockHourOfAmPm", HOURS, HALF_DAYS, ValueRange.of(1, 12))
    //一天的小时数
    CLOCK_HOUR_OF_DAY("ClockHourOfDay", HOURS, DAYS, ValueRange.of(1, 24))
    //上午还是下午
    AMPM_OF_DAY("AmPmOfDay", HALF_DAYS, DAYS, ValueRange.of(0, 1), "dayperiod")
    //一周的第几天
    DAY_OF_WEEK("DayOfWeek", DAYS, WEEKS, ValueRange.of(1, 7), "weekday")
    //当前月的天数
    DAY_OF_MONTH("DayOfMonth", DAYS, MONTHS, ValueRange.of(1, 28, 31), "day")
    //当前年的天数
    DAY_OF_YEAR("DayOfYear", DAYS, YEARS, ValueRange.of(1, 365, 366))
    //当前月的周数
    ALIGNED_WEEK_OF_MONTH("AlignedWeekOfMonth", WEEKS, MONTHS, ValueRange.of(1, 4, 5))
    //当前年的周数
    ALIGNED_WEEK_OF_YEAR("AlignedWeekOfYear", WEEKS, YEARS, ValueRange.of(1, 53))
    //以每月的第一天为星期一，然后计算当天是一周的第几天
    ALIGNED_DAY_OF_WEEK_IN_MONTH("AlignedDayOfWeekInMonth", DAYS, WEEKS, ValueRange.of(1, 7))
    //以每月的第一天为星期一，然后计算当天是一周的第几天
    ALIGNED_DAY_OF_WEEK_IN_YEAR("AlignedDayOfWeekInYear", DAYS, WEEKS, ValueRange.of(1, 7))
    //当前年的月数
    MONTH_OF_YEAR("MonthOfYear", MONTHS, YEARS, ValueRange.of(1, 12), "month")
    private final TemporalUnit baseUnit;
    private final String name;
    private final TemporalUnit rangeUnit;
    private final ValueRange range;
    private final String displayNameKey;
    private ChronoField(String name, TemporalUnit baseUnit, TemporalUnit rangeUnit, ValueRange range) {
        this.name = name;
        this.baseUnit = baseUnit;
        this.rangeUnit = rangeUnit;
        this.range = range;
        this.displayNameKey = null;
    }
```
-	ALIGNED_WEEK_OF_MONTH　和　ALIGNED_DAY_OF_WEEK_IN_MONTH　使用示例
```java
//以１号为周一，每七天一周，10号就是一周的第三天
int num = LocalDateTime.now().get(ChronoField.ALIGNED_DAY_OF_WEEK_IN_MONTH);
System.out.println(num);
//这个月的第二周
num = LocalDateTime.now().get(ChronoField.ALIGNED_WEEK_OF_MONTH);
System.out.println(num);
------------输出结果--------------
3
2
```
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbf1374ccf334a11b68b917e8febfef2~tplv-k3u1fbpfcp-watermark.image)

## 时间转换调整接口:TemporalAdjuster
-	因为java.time的时间类都是不可变，因此需要调整时间时，可调用该方法实现
-	jdk提供了一些默认调整方法的工具类TemporalAdjusters
```java
//由子类实现
Temporal adjustInto(Temporal temporal)
```

## TemporalAdjusters
-	时间调整工具类
```java
//设置天单位度量数为年的第一天
public static TemporalAdjuster firstDayOfYear()
//设置天单位度量数为年的最后一天
public static TemporalAdjuster lastDayOfYear()
//设置时间为下一年的第一天
public static TemporalAdjuster firstDayOfNextYear() 
//设置时间为当月的第一天
public static TemporalAdjuster firstInMonth(DayOfWeek dayOfWeek)
//设置时间为当月的最后一天
public static TemporalAdjuster lastInMonth(DayOfWeek dayOfWeek)
//设置时间为当月第ordinal个周的星期几－dayOfWeek
public static TemporalAdjuster dayOfWeekInMonth(int ordinal, DayOfWeek dayOfWeek)
//设置时间为下周星期几
public static TemporalAdjuster next(DayOfWeek dayOfWeek) 
//设置时间为上周星期几
public static TemporalAdjuster previous(DayOfWeek dayOfWeek)
//如果当前星期数和dayOfWeek不一样，则设置时间为上周的星期几－dayOfWeek
public static TemporalAdjuster previousOrSame(DayOfWeek dayOfWeek) 
```
-	示例
```java
LocalDateTime dateTime = LocalDateTime.now();
//设置天区域的值为当前月的第一天,其他域的值不变（现在时间是202101）
dateTime = dateTime.with(TemporalAdjusters.firstDayOfMonth());
System.out.println(dateTime);
------------输出结果--------------
2021-01-01T14:50:40.659823
```
## Date和LocalDateTime互转
-	JDK1.8之前用的时间类是Date，为了兼容，我们需要了解一下，Date和LocalDateTime之间的转换
-	示例
```java
//Date转LocalDateTime
Date date = new Date();
LocalDateTime dateTime = LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
System.out.println(dateTime);
dateTime = LocalDateTime.ofInstant(Instant.ofEpochMilli(date.getTime()), ZoneId.systemDefault());
System.out.println(dateTime);
------------输出结果------------
2021-01-10T14:25:08.649
2021-01-10T14:25:08.649
```
```java
//LocalDateTime转Date
LocalDateTime localDateTime = LocalDateTime.now();
Date date = Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
System.out.println(date);
------------输出结果------------
Sun Jan 10 14:29:04 CST 2021
```
欢迎指正文中错误
---

# 参考文章
-	[JAVA8时间类库与JodaTime](https://blog.csdn.net/zzz_zjz/article/details/93030049)
-	[JDK8 新特性 - 新的时间和日期 API](https://blog.csdn.net/qq_37581282/article/details/108610492)