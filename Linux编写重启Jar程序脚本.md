---
title: Linux编写重启Jar程序脚本、定时清除日志
date: 2022-03-23 15:13:47
categories:
- 运维
tags:
- 服务监控
---

# 一、Centos7安装crontab

```shell
# 安装
yum -y install crontabs
# 查看状态
systemctl status crond
# 启动
systemctl start crond
# 开机启动
systemctl enable crond.service
```

![image-20220915115005062](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220915115005062.png)

# 二、**编写自动重启脚本**

```
vi /opt/traffic-job.sh
```

```shell
#!/bin/sh
#traffic 相关任务 - 重启命令


date=`date`

# 案件分享
caseShare=`ps -ef|grep traffic-job-executor-case-share-1.0.0.jar|grep -v grep|wc -l`
if [ ${caseShare} -eq 0 ]
then
    echo  "时间：${date}，traffic-job-executor-case-share-1.0.0.jar 服务器已挂，重启中..."
    sleep 10s
    source /etc/profile;
    nohup java -jar /opt/traffic-job-executor-case-share-1.0.0.jar >>/opt/case-share.log 2>&1 &
    echo "时间：${date}，traffic-job-executor-case-share-1.0.0.jar 重启完成！"
else
    echo "时间：${date}，traffic-job-executor-case-share-1.0.0.jar 服务【正常】"
fi

# 获取12328数据
complaint=`ps -ef|grep traffic-job-executor-complaint-1.0.0.jar|grep -v grep|wc -l`
if [ ${complaint} -eq 0 ]
then
    echo  "时间：${date}，traffic-job-executor-complaint-1.0.0.jar 服务器已挂，重启中..."
    sleep 10s
    source /etc/profile;
    nohup java -jar /opt/traffic-job-executor-complaint-1.0.0.jar >>/opt/12328.log 2>&1 &
    echo "时间：${date}，traffic-job-executor-complaint-1.0.0.jar 重启完成！"
else
    echo "时间：${date}，traffic-job-executor-complaint-1.0.0.jar 服务【正常】"
fi

# 数据清洗
dataClean=`ps -ef|grep traffic-job-executor-data-clean-1.0.0.jar|grep -v grep|wc -l`
if [ ${dataClean} -eq 0 ]
then
    echo  "时间：${date}，traffic-job-executor-data-clean-1.0.0.jar 服务器已挂，重启中..."
    sleep 10s
    source /etc/profile;
    nohup java -jar /opt/traffic-job-executor-data-clean-1.0.0.jar >>/opt/tpsc-clean.log 2>&1 &
    echo "时间：${date}，traffic-job-executor-data-clean-1.0.0.jar 重启完成！"
else
    echo "时间：${date}，traffic-job-executor-data-clean-1.0.0.jar 服务【正常】"
fi

 echo "==============================================================================="
```

需要赋予该sh文件可执行权限

```shell
chmod a+x traffic-job.sh
```

# 三、编辑系统定时任务文件

```shell
crontab -e
```

# 四、**使用cron表达式，设置1分钟运行一次**

```shell
*/1 * * * * bash /opt/traffic-job.sh >> /opt/traffic-job-shell-restart.log
```

# 五、**查看定时任务日志，可以看到每隔1分钟执行了一次**

![image-20220915115623290](https://raw.githubusercontent.com/lukaixin0527/images/master/java-img/image-20220915115623290.png)



# 六、安装日志分割工具cronolog

```shell
1、下载
 wget http://cronolog.org/download/cronolog-1.6.2.tar.gz
 
2、解压缩
tar zxvf cronolog-1.6.2.tar.gz

3、进入cronolog安装文件所在目录
cd cronolog-1.6.2

4、运行安装
 ./configure
 make
 make install
 
5、查看cronolog安装后所在目录（验证安装是否成功）
 which cronolog
 一般情况下显示为：/usr/local/sbin/cronolog
```

# 七、按天输出日志

```shell
#!/bin/sh
#traffic 相关任务 - 重启命令


date=`date`

# 案件分享
caseShare=`ps -ef|grep traffic-job-executor-case-share-1.0.0.jar|grep -v grep|wc -l`
if [ ${caseShare} -eq 0 ]
then
    echo  "时间：${date}，traffic-job-executor-case-share-1.0.0.jar 服务器已挂，重启中..."
    sleep 10s
    source /etc/profile;
    nohup java -jar /opt/traffic-job-executor-case-share-1.0.0.jar | /usr/local/sbin/cronolog  /opt/log/case-share-%Y%m%d.out &
    echo "时间：${date}，traffic-job-executor-case-share-1.0.0.jar 重启完成！"
else
    echo "时间：${date}，traffic-job-executor-case-share-1.0.0.jar 服务【正常】"
fi

# 获取12328数据
complaint=`ps -ef|grep traffic-job-executor-complaint-1.0.0.jar|grep -v grep|wc -l`
if [ ${complaint} -eq 0 ]
then
    echo  "时间：${date}，traffic-job-executor-complaint-1.0.0.jar 服务器已挂，重启中..."
    sleep 10s
    source /etc/profile;
    nohup java -jar /opt/traffic-job-executor-complaint-1.0.0.jar | /usr/local/sbin/cronolog  /opt/log/12328-%Y%m%d.out &
    echo "时间：${date}，traffic-job-executor-complaint-1.0.0.jar 重启完成！"
else
    echo "时间：${date}，traffic-job-executor-complaint-1.0.0.jar 服务【正常】"
fi

# 数据清洗
dataClean=`ps -ef|grep traffic-job-executor-data-clean-1.0.0.jar|grep -v grep|wc -l`
if [ ${dataClean} -eq 0 ]
then
    echo  "时间：${date}，traffic-job-executor-data-clean-1.0.0.jar 服务器已挂，重启中..."
    sleep 10s
    source /etc/profile;
    nohup java -jar /opt/traffic-job-executor-data-clean-1.0.0.jar | /usr/local/sbin/cronolog  /opt/log/tpsc-data-clean-%Y%m%d.out &
    echo "时间：${date}，traffic-job-executor-data-clean-1.0.0.jar 重启完成！"
else
    echo "时间：${date}，traffic-job-executor-data-clean-1.0.0.jar 服务【正常】"
fi

 echo "==============================================================================="
```

# 八、定期删除日志

编写文件 log_clean.sh

```shell
# +30 删除30天以前的数据
save_day=30
#日志所在目录,多个目录以逗号分隔
log_dirs=/opt/log/
#日志后缀，多种后缀以\|分隔s
log_subfixs="log\|out\|gz"
log_prefixs="catalina"
echo 'start clear ...'
IFS=,
#按后缀删除
if [[ $log_subfixs ]]
then
echo '根据后缀消除'
for log_dir in ${log_dirs}; do
  find ${log_dir} -mtime +${save_day} -regex ".*\.\(${log_subfixs}\)"  -exec echo delete file:{} \;
  find ${log_dir} -mtime +${save_day} -regex ".*\.\(${log_subfixs}\)"  -exec rm -rf {} \;
   echo "scan dir:${log_dir}"
   
done
fi
#按前缀删除
if [[ $log_prefixs ]]
then
echo '根据前缀清除'
for log_dir in ${log_dirs}; do
  find ${log_dir} -mtime +${save_day} -name "*${log_prefixs}.*"  -exec echo delete file:{} \;
  find ${log_dir} -mtime +${save_day} -name "*${log_prefixs}.*"  -exec rm -rf {} \;
  echo "scan dir:${log_dir}"
done
fi
```

赋予文件权限

```shell
chmod a+x log_clean.sh
```

定时执行日志清除任务

```shell
# 编写crontab
crontab -e

# 添加任务
0 1 * * * bash /opt/log_clean.sh >> /opt/log/crontab/traffic-log-clean-shell.out

```

# 九、crontab表达式和cron表达式

## cron表达式

Cron表达式是一个字符串，字符串以5或6个空格隔开，**分为6或7个域**，每一个域代表一个含义，Cron有如下两种语法格式：
（1） Seconds Minutes Hours DayofMonth Month DayofWeek Year
（2） Seconds Minutes Hours DayofMonth Month DayofWeek

### 每个域允许的值

| 域         | 允许的数值      | 允许的特殊字符        | 备注                                 |
| :--------- | :-------------- | :-------------------- | :----------------------------------- |
| 秒         | 0~59            | - * /                 | -                                    |
| 分         | 0~59            | - * /                 | -                                    |
| 小时       | 0~23            | - * /                 | -                                    |
| 日期       | 1~31            | - * ? / L W C         | -                                    |
| 月份       | 1~12            | JAN-DEC - * /         | -                                    |
| 星期       | 1~7             | SUN-SAT - * ? / L C # | 1 表示星期天，2 表示星期一，依次类推 |
| 年（可选） | 留空，1970~2099 | , - * /               | 自动生成，工具不显示该值             |

### 特殊字符的含义

| 字符 | 含义                                                         | 示例                                                         |
| :--- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| *    | 表示匹配域的任意值                                           | 在分这个域使用 `*`，即表示每分钟都会触发事件。               |
| ？   | 表示匹配域的任意值，但**只能用在日期和星期两个域**，因为这两个域会相互影响。 | 要在每月的 20 号触发调度，不管每个月的 20 号是星期几，则只能使用如下写法：`13 13 15 20 * ?`。其中，因为日期域已经指定了 20 号，最后一位星期域只能用 `?`，不能使用 `*`。如果最后一位使用 `*`，则表示不管星期几都会触发，与日期域的 20 号相斥，此时表达式不正确。 |
| -    | 表示起止范围                                                 | 在分这个域使用 5-20，表示从 5 分到 20 分钟每分钟触发一次。   |
| /    | 表示起始时间开始触发，然后每隔固定时间触发一次               | 在分这个域使用 5/20，表示在第 5 分钟触发一次，之后每 20 分钟触发一次，即 5、 25、45 等分别触发一次。 |
| ,    | 表示列出枚举值                                               | 在分这个域使用 5,20，则意味着在 5 和 20 分每分钟触发一次。   |
| L    | 表示最后，只能出现在日和星期两个域                           | 在星期这个域使用 5L，意味着在最后的一个星期四触发。          |
| W    | 表示有效工作日（周一到周五），只能出现在日这个域，系统将在离指定日期最近的有效工作日触发事件。 | 在日这个域使用 5W，如果 5 号是星期六，则将在最近的工作日星期五，即 4 号触发。如果 5 号是星期天，则在 6 号（周一）触发；如果 5 号为工作日，则就在 5 号触发。另外，W 的最近寻找不会跨过月份。 |
| LW   | 这两个字符可以连用，表示在某个月最后一个工作日，即最后一个星期五。 |                                                              |
| #    | 表示每个月第几个星期几，**只能出现在星期这个域**             | 在星期这个域使用 4#2，表示某月的第二个星期三，4 表示星期三，2 表示第二个。 |

### 示例

- `*/5 * * * * ?`：每隔 5 秒执行一次
- `0 */1 * * * ?`：每隔 1 分钟执行一次
- `0 0 2 1 * ? *`：每月 1 日的凌晨 2 点执行一次
- `0 15 10 ? * MON-FRI`：周一到周五每天上午 10：15 执行作业
- `0 15 10 ? 6L 2002-2006`：2002 年至 2006 年的每个月的最后一个星期五上午 10:15 执行作业
- `0 0 23 * * ?`：每天 23 点执行一次
- `0 0 1 * * ?`：每天凌晨 1 点执行一次
- `0 0 1 1 * ?`：每月 1 日凌晨 1 点执行一次
- `0 0 23 L * ?`：每月最后一天 23 点执行一次
- `0 0 1 ? * L`：每周星期天凌晨 1 点执行一次
- `0 26,29,33 * * * ?`：在 26 分、29 分、33 分执行一次
- `0 0 0,13,18,21 * * ?`：每天的 0 点、13 点、18 点、21 点都执行一次
- `0 0 10,14,16 * * ?`：每天上午 10 点，下午 2 点，4 点执行一次
- `0 0/30 9-17 * * ?`：朝九晚五工作时间内每半小时执行一次
- `0 0 12 ? * WED`：每个星期三中午 12 点执行一次
- `0 0 12 * * ?`：每天中午 12 点触发
- `0 15 10 ? * *`：每天上午 10:15 触发
- `0 15 10 * * ?`：每天上午 10:15 触发
- `0 15 10 * * ? *`：每天上午 10:15 触发
- `0 15 10 * * ? 2005`：2005 年的每天上午 10:15 触发
- `0 * 14 * * ?`：每天下午 2 点到 2:59 期间的每 1 分钟触发
- `0 0/5 14 * * ?`：每天下午 2 点到 2:55 期间的每 5 分钟触发
- `0 0/5 14,18 * * ?`：每天下午 2 点到 2:55 期间和下午 6 点到 6:55 期间的每 5 分钟触发
- `0 0-5 14 * * ?`：每天下午 2 点到 2:05 期间的每 1 分钟触发
- `0 10,44 14 ? 3 WED`：每年三月的星期三的下午 2:10 和 2:44 触发
- `0 15 10 ? * MON-FRI`：周一至周五的上午 10:15 触发
- `0 15 10 15 * ?`：每月 15 日上午 10:15 触发
- `0 15 10 L * ?`：每月最后一日的上午 10:15 触发
- `0 15 10 ? * 6L`：每月的最后一个星期五上午 10:15 触发
- `0 15 10 ? * 6L 2002-2005`：2002 年至 2005 年的每月的最后一个星期五上午 10:15 触发
- `0 15 10 ? * 6#3`：每月的第三个星期五上午 10:15 触发

## Crontab表达式

Crontab表达式还是比较好区分的，它**只有五位**。

### Crontab介绍

crontab指令常见于Unix和类Unix的操作系统之中，用于设置周期性被履行的指令。该指令从规范输入设备读取指令，并将其存放于“crontab”文件中，以供之后读取和履行。

crontab贮存的指令被看护进程激活，crond常常在后台运转，每一分钟检查是否有预订的作业需求执行。

crontab表达式的每一行均严格遵守特定的表达式，由空格或tab分隔为数个领域，每个领域可以放置单一或多个表达式。

时程表的格式:z1 z2 z3 z4 z5 program，其中 z1 是分钟，z2 小时，z3 一个月份中的第几日，z4 月份，z5 表示一个星期中的第几天。program 表示要执行的shell或者命名。

```shell
0    2    *    *    6
*    *    *    *    *    *
-    -    -    -    -    -
|    |    |    |    |    |
|    |    |    |    |    + 年 [可选参数]
|    |    |    |    +----- 星期几 (0 - 7) (Sunday=0 or 7)
|    |    |    +---------- 月份 (1 - 12)
|    |    +--------------- 几号 (1 - 31)
|    +-------------------- 小时 (0 - 23)
+------------------------- 分钟 (0 - 59)
```

### Crontab使用

cron是一个linux下的定时执行工具，可以在无需人工干预的情况下运行作业。由于Cron是Linux的内置服务，但它不自动起来，可以用以下的方法启动、关闭这个服务。

cron服务提供crontab命令来设定cron服务的，以下是这个命令的一些参数与说明： crontab -u //设定某个用户的cron服务，一般root用户在执行这个命令的时候需要此参数；crontab -l //列出某个用户cron服务的详细内容；crontab -r //删除某个用户的cron服务；crontab -e //编辑某个用户的cron服务。

### Crontab例子

- `30 16 * * * /usr/local/etc/rc.d/lighttpd restart` 表示每晚天中午的16:30重启lighttpd
- `40 3 3,15,23 * * /usr/local/etc/rc.d/lighttpd restart` 表示每月3、15、23日的3 : 40重启lighttpd
- `30 3 * * 6,0 /usr/local/etc/rc.d/lighttpd restart` 表示每周六、周日的3 : 30重启lighttpd
- `0,30 20-22 * * * /usr/local/etc/rc.d/lighttpd restart` 表示在每天20 : 00至22 : 00之间每隔30分钟重启lighttpd
- `0 23 * * 6 /usr/local/etc/rc.d/lighttpd restart` 表示每星期六的11 : 00 pm重启lighttpd
- `0 */2 * * * /usr/local/etc/rc.d/lighttpd restart` 每2小时重启lighttpd
- `* 23-7/1 * * * /usr/local/etc/rc.d/lighttpd restart` 晚上11点到早上7点之间，每隔一小时重启lighttpd
- `0 11 5 * mon-wed /usr/local/etc/rc.d/lighttpd restart` 每月的5号与每周一到周三的11点重启lighttpd
- `0 5 1 jan * /usr/local/etc/rc.d/lighttpd restart` 一月一号的5点重启lighttpd

## 时区问题

在github的action中使用crontab表达式设置定时任务时，通常会出现这样一个问题，我设置的是北京时间的每天8:00执行任务，结果在凌晨就执行了。

这个问题出现的原因是，crontab表达式的时间是受操作系统所设置的时区影响的，而如果在github action中使用ubuntu环境运行定时任务，里面的时区默认使用的是UTC，UTC时间比北京时间提前8小时。例如北京时间每天8:00调度函数，那么转化为UTC时间就是每天0:00调度函数，则可以使用`0 0 0 * * *`，而如果你想在北京时间每天20:00指定定时任务，则需要转换为UTC时间的12:00，cron表达式可以表示为：`0 0 12 * * *`。

### 解决方案

1. 上面也提到了，可以直接将北京时间转换为UTC时间，也就是将北京时间减去8小时再写入crontab表达式即可避免时区不一致的问题。

2. 如果是在github action中运行定时任务，也可以修改yml文件中的时区配置：

   ```yml
   env: # 设置环境变量
     TZ: Asia/Shanghai # 时区
   ```

3. 如果是在本地ubuntu系统中需要修改时区，