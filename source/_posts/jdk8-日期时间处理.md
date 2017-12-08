---
title: jdk8 日期时间处理
date: 2017-12-08 15:52:19
categories:
- technology

tags:
- java
---

jdk8更新了日期时间处理类。值得注意的是新增的时间处理方法更加方便使用，而且是线程安全的，包括日期的格式化，再也不用担心多线程下日期格式化的麻烦了！
<!-- more -->
直接上代码：
```java
// 当时日期时间
System.out.println("LocalDate.now() = " + LocalDate.now());
System.out.println("LocalTime.now() = " + LocalTime.now());
System.out.println("LocalDateTime.now() = " + LocalDateTime.now());
System.out.println("Instant.now() = " + Instant.ofEpochMilli(System.currentTimeMillis()));
System.out.println("ZonedDateTime.now() = " + ZonedDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));

// 字符串解析成日期
System.out.println("Date parse 20170101 = " + LocalDate.parse("20170101", DateTimeFormatter.BASIC_ISO_DATE));
System.out.println("Date parse 2017/01/01 = " + LocalDate.parse("2017/01/01", DateTimeFormatter.ofPattern("yyyy/MM/dd")));

// 日期格式化显示字符串
System.out.println("Date to string = " + LocalDateTime.now().format(DateTimeFormatter.BASIC_ISO_DATE));
System.out.println("Date to string = " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH时mm分ss秒")));

/*
 * 按单位显示，相差 几个月几天
 */
Period period = Period.between(LocalDate.now(), LocalDate.of(2018, 1,7));
System.out.println(String.format("相差时间：%s 年 %s 月 %s 天" , period.getYears(), period.getMonths(), period.getDays()));

/*
 * 相差多少天或者多少小时/分钟等
 */
Duration duration = Duration.between(LocalDateTime.now(), LocalDateTime.of(2018, 1, 7, 0, 0, 0));
System.out.println("相差天数：" + duration.toDays());
System.out.println("相差分钟数：" + duration.toMinutes());

// 本月第一天
System.out.println("本月第一天：" + LocalDate.now().with(TemporalAdjusters.firstDayOfMonth()));
```
输出内容：
```
LocalDate.now() = 2017-12-08
LocalTime.now() = 15:51:38.833
LocalDateTime.now() = 2017-12-08T15:51:38.833
Instant.now() = 2017-12-08T07:51:38.833Z
ZonedDateTime.now() = 2017-12-08 15:51:38
Date parse 20170101 = 2017-01-01
Date parse 2017/01/01 = 2017-01-01
Date to string = 20171208
Date to string = 2017年12月08日 15时51分38秒
相差时间：0 年 0 月 30 天
相差天数：29
相差分钟数：42248
本月第一天：2017-12-01
```
