+++
author = "pikachu"
title = "SpringBoot定时任务及其redis的分布式锁创建"
date = "2019-02-23"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java",
	"spring-boot",
	"中间件"
]
categories = [
    "IT"
]
+++


## 分布式锁实现流程图

![default](https://user-images.githubusercontent.com/38284818/53300510-3a360200-3883-11e9-9465-ee22e2dabc21.JPG)

&nbsp;

## 一、@Scheduled的使用及其redis的分布式锁创建
- 创建新类，使用`@Scheduled`注解注释需要定时启动的方法

- 单机版的定时任务直接将业务代码写入`@Scheduled`注解的方法中即可，集群版的定时任务需要加**分布式锁**，防止同一时间所有集群都启动定时任务，造成资源的浪费。

- 此处使用分布式的**双重防死锁**，在使用redis实现分布式锁的同时，加入判断防止**系统在创建了分布式锁后，因为手动关闭或者意外关闭了服务器，造成没有成功执行释放锁的代码，从而形成死锁**。

- **双重防死锁**：使用redis实现分布式锁，并且为创建的键赋值为`System.currentTimeMillis() + RedisConst.APPLICATION.TASK_LOCK_TIME`，即当前系统时间加上有效时间，若分布式锁已经存在，则使用`if (time != null && System.currentTimeMillis() > Long.valueOf(time))`判断是否过期，若过期，则可以正常获取到分布式锁并使用。

```
@Component
public class ApplicationCloseTask {

    @Autowired
    FormExecuteService formExecuteService;

    private Logger log = LoggerFactory.getLogger(this.getClass());

    /**
     *  定时更新未处理且已经过期的个人申请单，将其设置为过期状态
     *  定时任务每七天执行一次
     *
     * @author 曾博佳
     * @since 2019-02-23
     */
    @Scheduled(cron = "0 0 0 */7 * ?")
    public void closeApplication(){

        log.info("定时任务正常启动：{}",new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));

        //设置分布式锁，若不存在，将其值设置为{当前时间 + 有限时间}，返回true
        boolean setnxResult = RedisPoolUtil.setnx(RedisConst.APPLICATION.TASK_CLOSE_LOCK,
                String.valueOf(System.currentTimeMillis() + RedisConst.APPLICATION.TASK_LOCK_TIME));

        if(setnxResult){
            updateApplicationState();
        }else{
            //如果分布式锁已过期或者为永久(出现死锁)，则可以获取分布式锁
            String time = RedisPoolUtil.get(RedisConst.APPLICATION.TASK_CLOSE_LOCK);

            if (time != null && System.currentTimeMillis() > Long.valueOf(time)){
                log.info("获取分布式锁{} 成功，ThreadName：{}",RedisConst.APPLICATION.TASK_CLOSE_LOCK,Thread.currentThread().getName());

                String time2 = RedisPoolUtil.getset(RedisConst.APPLICATION.TASK_CLOSE_LOCK,
                        String.valueOf(System.currentTimeMillis() + RedisConst.APPLICATION.TASK_LOCK_TIME));

                // time2为null表示集群应用的锁已经其他进程执行并删除，从正常逻辑上此时可以获取到锁
                // 如果time不等于time2并且time2不等于null，说明time2已经被其他进程获取并修改
                if (time2 == null || StringUtils.equals(time,time2)){
                    log.info("获取分布式锁{} 成功，ThreadName：{}",RedisConst.APPLICATION.TASK_CLOSE_LOCK,Thread.currentThread().getName());
                    updateApplicationState();
                }else{
                    log.info("分布式锁：{} 获取失败",RedisConst.APPLICATION.TASK_CLOSE_LOCK);
                }

            }else{
                log.info("分布式锁：{} 获取失败",RedisConst.APPLICATION.TASK_CLOSE_LOCK);
            }
        }
    }

    /**
     * 执行更新application状态的代码
     */
    private void updateApplicationState(){
        log.info("分布式锁获取成功，执行关闭订单程序");

        //将已经过期的申请更新为过期状态
        formExecuteService.updateAllApplicationState(new Date(), NormalConst.APPLICATION.EFFECTIVE_TIME);

        RedisPoolUtil.del(RedisConst.APPLICATION.TASK_CLOSE_LOCK);
        log.info("分布式锁删除成功");
    }

}
```

&nbsp;

## 二、SpringBoot Schedule的`cron`的使用
- 在线`corn`生成器：http://www.pppet.net/

- `@Scheduled(cron = "0 0 0 */7 * ?")`表示每七天执行一次，其中的`0 0 0 */7 * * ?`分别表示**秒、分、时、日、月、周、年**。

- 常见的corn表达式：

> 0 15 10 * * ? * | 每天10点15分触发
> -- | --
> 0 15 10 * * ? 2017 | 2017年每天10点15分触发
> 0 * 14 * * ? | 每天下午的 2点到2点59分每分触发
> 0 0/5 14 * * ? | 每天下午的 2点到2点59分(整点开始，每隔5分触发)
> 0 0/5 14,18 * * ? | 每天下午的 2点到2点59分、18点到18点59分(整点开始，每隔5分触发)
> 0 0-5 14 * * ? | 每天下午的 2点到2点05分每分触发
> 0 15 10 ? * 6L | 每月最后一周的星期五的10点15分触发
> 0 15 10 ? * 6#3 | 每月的第三周的星期五开始触发

&nbsp;

![image](https://user-images.githubusercontent.com/38284818/53106458-1797a680-356e-11e9-90d2-bf2df148703d.png)

![image](https://user-images.githubusercontent.com/38284818/53106491-23836880-356e-11e9-8fdd-7c39903a7222.png)

