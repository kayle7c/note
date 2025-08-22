# kdump

    kexec：快速重启，从一个内核快速重启到另一个内核不需要通过bios等

    kdump：指崩溃转储，当内核发生panic时，从生产内核（正常运行的内核）重启到捕获内核（kdump后，重启到的新内核），并生成vmcore文件    

    kdump服务：内核panic后，后进入kdump服务自动捕获内核，收集现场生成/proc/vmcore，通过makedumpfile，将vmcore文件自动转储压缩/var/vmcore，都完成后就会reboot到普通内核中

    系统需要提前开启kdump服务，linux内核在出现异常宕机时才能正常生成用于分析的内核coredump文件。一般coredump文件都存在/var/crash中

# 内核watchdog

    linux内核触发panic场景通常有访问异常地址或异常值以及内核各类watchdog主动探测到异常触发的panic。

## hungtask watchdog

    内核中 hung task watchdog通过创建一个内核线程"khungtaskd"周期性执行检查系统中是否有线程是状态为TASK_UNINTERRUPTIBLE且超过hung_task_timeout_secs秒没有发生过调度切换

## softlockup watchdog

    softlockup的检测在内核中交给watchdog_timer_fn函数实现，该函数由内核定时器每sample_period触发一次。在触发时会检查当前cpu的watchdog时间戳和上一次触发的时间是否超过了软锁死检测阈值，如果超过了，就说明发生了softlockup。

```c
static enum hrtimer_restart watchdog_timer_fn(struct hrtimer *hrtimer)
{
    //获取上一次更新的时间戳，这个是pcpu变量，每个cpu维护自己的
    unsigned long touch_ts = __this_cpu_read(watchdog_touch_ts);
    struct pt_regs *regs = get_irq_regs();
    int duration;
    int softlockup_all_cpu_backtrace = sysctl_softlockup_all_cpu_backtrace;

    if (!watchdog_enabled)
        return HRTIMER_NORESTART;

    /* kick the hardlockup detector */
    watchdog_interrupt_count();

    //在softlockup_fn中喂狗（更新时间戳）
    if (completion_done(this_cpu_ptr(&softlockup_completion))) {
        bool success;

        reinit_completion(this_cpu_ptr(&softlockup_completion));
        success = stop_one_cpu_nowait(smp_processor_id(), softlockup_fn, NULL,
                          this_cpu_ptr(&softlockup_stop_work));

        if (!success) {
            __touch_watchdog();
            complete(this_cpu_ptr(&softlockup_completion));
        }
    }
    ...

    //检查当前时间和上一次时间戳（touch_ts）+ 超时阈值谁大
    duration = is_softlockup(touch_ts);
    if (unlikely(duration)) {

        //...(打印堆栈和panic流程)

    return HRTIMER_RESTART;
}
```

    判断softlockup是否发生

```c
static int is_softlockup(unsigned long touch_ts)
{
    unsigned long now = get_timestamp();

    if ((watchdog_enabled & SOFT_WATCHDOG_ENABLED) && watchdog_thresh){
        /* Warn about unreasonable delays. */
        if (time_after(now, touch_ts + get_softlockup_thresh()))
            return now - touch_ts;
    }
    return 0;
}
```

这里要注意的是,watchdog_timer_fn的定时器时间为watchdog_thresh✖️2/5，

get_softlockup_thresh获取的是watchdog_thresh✖️2。也就是说，在整个softlockup狗叫过程中会喂5次狗，只要中间五次有一次喂了就不会狗叫。

![](assets/9b7721bcd9bba902e93cf22a5fde4f16dc53362d.png)
