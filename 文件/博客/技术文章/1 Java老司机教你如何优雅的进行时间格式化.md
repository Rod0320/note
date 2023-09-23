##  Java老司机教你如何优雅的进行时间格式化

### 前言

在日常项目开发过程中，相信大家一定都经常遇到时间格式化的场景。很多人可能都感觉非常简单，但是你的时间格式化方法真的优雅高效吗？

### 一、常见时间格式化方式

```java
public static void main(String[] args) {
    Date now = new Date(); // 创建一个Date对象，获取当前时间
    String strDateFormat = "yyyy-MM-dd HH:mm:ss";

    //新人菜鸟实现
    SimpleDateFormat f = new SimpleDateFormat(strDateFormat);
    System.out.println("SimpleDateFormat:" + f.format(now)); // 将当前时间袼式化为指定的格式

    //java8进阶实现
    LocalDateTime localDateTime = now.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
    String result =  localDateTime.format(DateTimeFormatter.ofPattern(strDateFormat));
    System.out.println("DateTimeFormatter:"+result);

    //common-lang3老鸟实现
    result  = DateFormatUtils.format(now,strDateFormat);
    System.out.println("DateFormatUtils:"+result);

}
```

### 二、分析

#### 方式一：新人菜鸟实现

很多新人喜欢采用`SimpleDateForma`t进行时间格式化，这也是java8之前，jdk默认提供的时间格式化实现方式，它最大的问题是非线程安全的。

##### 原因：

在多线程环境下，当多个线程同时使用相同的`SimpleDateFormat`对象（如static修饰）的话，如调用format方法时，多个线程会同时调用`calender.setTime`方法，导致time被别的线程修改，因此线程是不安全的。

```java
/**
 * SimpleDateFormat线程安全测试
 */
public class SimpleDateFormatTest {
    private SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(10, 100, 1, TimeUnit.MINUTES, new LinkedBlockingQueue<>(1000), new MyThreadFactory("SimpleDateFormatTest"));


    public void test() {
        while (true) {
            poolExecutor.execute(new Runnable() {
                @Override
                public void run() {
                    String dateString = simpleDateFormat.format(new Date());
                    try {
                        Date parseDate = simpleDateFormat.parse(dateString);
                        String dateString2 = simpleDateFormat.format(parseDate);
                        System.out.println(dateString.equals(dateString2));
                    } catch (ParseException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
    }
```

输出：

```java
true
false
true
true
false
```

出现了false，说明线程不安全

format方法源码分析：



```java
protected Calendar calendar;

 // Called from Format after creating a FieldDelegate
    private StringBuffer format(Date date, StringBuffer toAppendTo,
                                FieldDelegate delegate) {
        // Convert input date to time field list
        calendar.setTime(date);

        boolean useDateFormatSymbols = useDateFormatSymbols();

        for (int i = 0; i < compiledPattern.length; ) {
            int tag = compiledPattern[i] >>> 8;
            int count = compiledPattern[i++] & 0xff;
            if (count == 255) {
                count = compiledPattern[i++] << 16;
                count |= compiledPattern[i++];
            }

            switch (tag) {
            case TAG_QUOTE_ASCII_CHAR:
                toAppendTo.append((char)count);
                break;

            case TAG_QUOTE_CHARS:
                toAppendTo.append(compiledPattern, i, count);
                i += count;
                break;

            default:
                subFormat(tag, count, delegate, toAppendTo, useDateFormatSymbols);
                break;
            }
        }
        return toAppendTo;
    }
```

可以看到，多个线程之间共享变量calendar，并修改calendar。因此在多线程环境下，当多个线程同时使用相同的`SimpleDateFormat`对象（如static修饰）的话，如调用format方法时，多个线程会同时调用`calender.setTime`方法，导致time被别的线程修改，因此线程是不安全的。

此外，parse方法也是线程不安全的，parse方法实际调用的是`CalenderBuilder`的establish来进行解析，其方法中主要步骤不是原子操作。

解决方案：

- 将`SimpleDateFormat`定义成局部变量
- 加一把线程同步锁：`synchronized(lock)`
- 使用`ThreadLocal`，每个线程都拥有自己的`SimpleDateFormat`对象副本。

#### 方式二：java8进阶实现

java8后，推荐使用`DateTimeFormatter`代替`SimpleDateFormat`。

`DateTimeFormatter`是线程安全的，默认提供了很多格式化方法，也可以通过`ofPattern`方法创建自定义格式化方法。

```java
//java8进阶实现
LocalDateTime localDateTime = LocalDateTime.now();
String result =  localDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
System.out.println("DateTimeFormatter:"+result);
```

需要注意的是，java8的时候格式化都是针对`LocalDate`和`LocalDateTime`对象，其中`LocalDateTime`包含时分秒，而`LocalDate`只包含年月日，没有时分秒信息。

Java8 日期时间API，新增了`LocalDate`、`LocalDateTime`、`LocalTime`等线程安全类：

- **LocalDate**：只有日期，诸如：`2019-07-13`
- **LocalTime**：只有时间，诸如：`08:30`
- **LocalDateTime**：日期+时间，诸如：`2019-07-13 08:30`

由于java8的时候格式化API都是针对LocalDate和`LocalDateTime`对象，所以date对象和`LocalDate`、`LocalDateTime`对象的相互转化就非常重要。

##### 1、Date转换成LocalDate

```java
public static LocalDate date2LocalDate(Date date) {
    if(null == date) {
        return null;
    }
    return date.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();
}
```

##### 2、Date转换成LocalDateTime

```java
public static LocalDateTime date2LocalDateTime(Date date) {
    if(null == date) {
        return null;
    }
    return date.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
}
```

##### 3、LocalDate转换成Date

```java
public static Date localDate2Date(LocalDate localDate) {
    if(null == localDate) {
        return null;
    }
    ZonedDateTime zonedDateTime = localDate.atStartOfDay(ZoneId.systemDefault());
    return Date.from(zonedDateTime.toInstant());
}
```

##### 4、LocalDateTime转换成Date

```java
public static Date localDateTime2Date(LocalDateTime localDateTime) {
     return Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
 }
```

##### 方式三：common-lang3老鸟实现

相信大家在项目开发过程中都多少使用过`common-lang3`中的一些API，都非常简单实用。在时间格式化方面，也给我们提供了非常好用的工具类`DateFormatUtils`。

这也是我最推荐的方式。

需要在项目中引入`common-lang3`的依赖。

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.11</version>
</dependency>
```

`DateFormatUtils`使用示例：

```java
Date now = new Date(); // 创建一个Date对象，获取当前时间
String strDateFormat = "yyyy-MM-dd HH:mm:ss";
String result  = DateFormatUtils.format(now,strDateFormat);
System.out.println("DateFormatUtils:"+result);
```

好处：

- 线程安全
- 简单高效
- 占用更小的内存

`DateFormatUtils.format`的内部实现中，是通过`FastDateFormat`进行时间格式化，而且对`FastDateFormat`对象进行了缓存处理，保证相同模式的格式化类型下不用重复生成`FastDateFormat`对象。

```java
FastDateFormat df = FastDateFormat.getInstance(pattern, timeZone, locale);
```

推荐使用方式：

```java
/**
 * @Description   时间格式化工具类
 * @Version 1.0
 * @Copyright 2019-2021
 */
public class DateUtils extends DateFormatUtils {
      //继承DateFormatUtils

      //自定义的时间相关方法
}
```

##### 总结

1、介绍了java时间格式化的3种常用方式。

2、`SimpleDateFormat`时间格式化主要的问题是非线程安全，多线程情况下会出现问题，通过跟踪源码说明了`SimpleDateFormat`非线程安全的原因，并提供了相应的解决方案。

3、介绍了java8下推荐的采用`DateTimeFormatter`进行时间格式化的使用方式，并提供了date到`LocalDateTime`、`LocalDate`的转换方式。

4、采用`common-lang3`中的`DateFormatUtils`实现时间格式化是我最推荐的，线程安全、API简单高效、占用内存低。并推荐了通过继承`DateFormatUtils`对象封装自己的时间工具类DateUtils方式。