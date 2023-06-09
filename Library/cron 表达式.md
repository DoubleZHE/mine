# cron 表达式

corn 表达式就是：**由若干数字、空格、符号按一定的规则，组成的一组字符串，从而表达时间的信息。**

cron 表达式是一个字符串，由 6 到 7 个字段组成，用空格分隔。其中前 6 个字段是必须的，最后一个是可选的【年的实用性不大】。每个字段的含义如图所示：

![cron 表达式含义图](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Library/cron%20%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%90%AB%E4%B9%89%E5%9B%BE.png)

## 99% 情况下会用到的字符

在大部分使用 cron 的场景下， `-`、` *`、 `/`、` ?`  这几个常用字符就可以满足我们的需求了。

- `*`：每的意思。在不同的字段上，就代表每秒，每分，每小时等。
- `-`：指定值的范围。比如 `1-10`，在秒字段里就是每分钟的第 1 到 10 秒，在分就是每小时的第 1 到 10 分钟，以此类推。
- `,`：指定某几个值。比如 `2,4,5`，在秒字段里就是每分钟的第 2，第 4，第 5 秒，以此类推。
- `/`：指定值的起始和增加幅度。比如 `3/5`，在秒字段就是每分钟的第 3 秒开始，每隔 5 秒生效一次，也就是第 3 秒、8 秒、13 秒，以此类推。
- `?`：仅用于【日】和【周】字段。因为在指定某日和周几的时候，这两个值实际上是冲突的，所以需要用 `?` 标识不生效的字段。比如 `0 1 * * * ?` 就代表每年每月每日每小时的 1 分 0 秒触发任务。这里的周就没有效果了。

## 极少能用到的字符

- `SUN` ：仅用于【周】字段，表示星期日。也可以用数字1设置。周日到周六分别为 SUN，MON，TUE，WED，THU，FRI 和 SAT，对应数字 1 (SUN)，2 (MON)，3 (TUE)，4 (WED)，5 (THU)，6 (FRI)，7 (SAT)。目前 Quartz 支持。

- `L` ：即 last，用于【日】【周】字段。这里需要注意的是，在不同的字段的不同使用方式，其含义有所差别。

- - 用于日字段：直接使用 `L` 代表每个月的最后一天。也支持偏移量的方式，配置 `L-1` 则代表每月的倒数第二天。
  - 用于周字段：直接使用 `L` 代表每周的最后一天，也就是等效于 `7` 或 `SAT`，但是如果配合上数字，比如 `7L`，则代表每个月最后一个周六，等效于 `SATL`。目前Quartz支持。

- ## 一些常见例子

- | cron 表达式                               | 含义                                                         | 常用场景                                             | 执行时间                                                     |
  | ----------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- | ------------------------------------------------------------ |
  | 5 * * * * ?                               | 每分钟的第 5 秒执行一次                                      | 常见的每分钟的定时任务，检查数据库和缓存数据是否一致 | 2021-04-11 13:10:05 2021-04-11 13:11:052021-04-11 13:12:052021-04-11 13:13:052021-04-11 13:14:052021-04-11 13:15:05 |
  | 5 * 10-22 * * ?                           | 从早上 10 点到晚上十点，每分钟的第 5 秒执行一次              | 将定时任务限制在每天的工作时间                       | 2021-04-11 13:10:05 2021-04-11 13:11:052021-04-11 13:12:052021-04-11 13:13:052021-04-11 13:14:052021-04-11 13:15:05 |
  | 5 0 0/6 * * ?  等效于 5 0 0,6,12,18 * * ? | 每天从 0 点开始，每隔 6 小时执行一次。执行时间为第 0 分 5 秒。 | 常用于每天较低频次的批量同步数据                     | 2021-04-12 00:00:05 2021-04-12 06:00:052021-04-12 12:00:052021-04-12 18:00:05 |