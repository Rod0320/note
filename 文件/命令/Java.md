# 时间

```java
https://blog.csdn.net/gr912308719/article/details/80299898
String.format("%tF%n", new Date())//2022-03-31
String.format("%tF %tT%n", new Date(),new Date())//2022-03-31 00:00:00
SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
format.format(new Date());//2022-03-31 00:00:00

Date date = new Date();
//得到日历
Calendar calendar = Calendar.getInstance();
//把当前时间赋给日历
calendar.setTime(date);
//设置为前一天
calendar.add(Calendar.DAY_OF_MONTH, -1);
//得到前一天的时间
date = calendar.getTime();
LocalDateTime -> 现在用这个替换calendar
LocalDateTime beginTime = LocalDateTime.now();
String  date=DateTimeFormatter.ofPattern("yyyy-MM-dd 00:00:00").format(beginTime);
LocalDateTime now = LocalDateTime.now();
System.out.println("计算两个时间的差：");
LocalDateTime end = LocalDateTime.now();
Duration duration = Duration.between(now,end);
long days = duration.toDays(); //相差的天数
long hours = duration.toHours();//相差的小时数
long minutes = duration.toMinutes();//相差的分钟数
long millis = duration.toMillis();//相差毫秒数
long nanos = duration.toNanos();//相差的纳秒数
```

# 实体类 0=身份证 状态转换

- 使用set在实体类中转换

```Java
    /**
     * 证件类型
     */
    private String zjlx;

    public void setZjlx(String zjlx) {
        for (TypeOfCertificateEnum item : TypeOfCertificateEnum.values()) {
            if (item.getValue().equals(zjlx)) {
                this.zjlx = item.getName();
                break;
            }
        }
    }
```

# list分片

```
ListUtils.partition(listData, 500);
```



# threadlocal

![image-20220408231117309](../../图片保存\image-20220408231117309.png)

# 异步

```java
import java.util.concurrent.CompletableFuture;
CompletableFuture.runAsync(() -> {
    
}
```

```java
@Async
public class EnterpriseGroupService {}
```

# 定时任务

```java
-- 1
import net.javacrumbs.shedlock.spring.annotation.SchedulerLock;
import org.springframework.scheduling.annotation.Scheduled;
@Scheduled(cron = "0 0/1 * * * ? ")
@SchedulerLock(name = "GetGovSmsUpStreamScheduler", lockAtLeastFor = "PT30S", lockAtMostFor = "PT60S")
public void messageTaskTimer() {
    log.info("定时获取政务云上行短信任务开始，当前时间:{}", LocalDateTime.now());
    List<SmsExtendEntity> smsExtendEntityList = smsExtendRepository.findByExtendStatus(1);
    smsExtendEntityList.stream().forEach(smsExtendEntity -> {
        R r = govSmsService.getGovSmsUpStream(smsExtendEntity.getMessageTunnelId(), smsExtendEntity.getExtendPrefix());
        log.info("获取政务云上行短信任务：tunnelId={}, extend={}, result={}", smsExtendEntity.getMessageTunnelId(), smsExtendEntity.getExtendPrefix(), r.toString());
    });
}

-- 2
Spring Task 是 Spring 提供的轻量级定时任务工具
@SpringBootApplication
@EnableScheduling
public class CodingmoreSpringtaskApplication {

	public static void main(String[] args) {
		SpringApplication.run(CodingmoreSpringtaskApplication.class, args);
	}

}
新建定时任务类 CronTask，使用 @Scheduled 注解注册 Cron 表达式执行定时任务。
@Slf4j
@Component
public class CronTask {
    @Scheduled(cron = "0/1 * * ? * ?")
    public void cron() {
        log.info("定时执行，时间{}", DateUtil.now());
    }
}
https://www.cnblogs.com/qing-gee/p/16354218.html?utm_source=gold_browser_extension
```

```java
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;

@Configuration
@EnableScheduling
public class JobConfig extends BaseDao {
    @Scheduled(cron = "0 10 0 * * ?")
    public void configureTasks() {
        try {
            //orcle数据库备份
            oraBackup();
        } catch (IOException e) {
            log.error(e.toString());
        }
    }

    @Scheduled(cron = "0 0 0 1 * ?")
    public void clearBackup(){
        String dbBackupsPath = PropertiesUtil.getString("backupsPath");
        //定时删除历史备份数据
        File file = new File(dbBackupsPath+"/history");
        try {
            deleteFile(file);
        } catch (IOException e) {
            log.error(e.toString());
        }
    }
}
```



# object转其它类

```java
ObjectMapper objectMapper = new ObjectMapper();

List<ToUserEntity> entities = list.stream().map((item) -> {
return objectMapper.convertValue(item, ToUserEntity.class);
}).collect(Collectors.toList());
```

# 优秀的工具类

https://juejin.cn/post/6986795792883253278

# 三方jar包转成maven的jar包

```cmd
mvn install:install-file -Dfile=C:\Users\Administrator\Desktop\QRCode.jar -DgroupId=QRCode -DartifactId=QRCode -Dversion=3.0 -Dpackaging=jar -DgeneratePom=true

实例
mvn install:install-file -Dfile=G:\Maven\repository\e-iceblue\spire.pdf.free\5.1.0\spire.pdf.free-5.1.0.jar -DgroupId=eiceblue -DartifactId=pdf -Dversion=5.1.0 -Dpackaging=jar
```

- `-Dfile`的后面输入的为你下载的第三方jar包的本地文件路径。

- `-DgroupId`的后面输入的为你转maven jar包后groupId的标签内容QRCode。

- `-DartifactId`的后面输入的为你转maven jar包后artifactId的标签内容QRCode。

- `-Dversion`的后面输入的为你转maven jar包后version的标签内容版本号3.0。
