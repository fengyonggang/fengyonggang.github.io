---
layout: post
title:  Centos7 crash caused by nouveau driver
category: centos
---
## crash dump backtrace: 
```
#0 [ffff88082f4459f0] machine_kexec at ffffffff81059cdb
#1 [ffff88082f445a50] __crash_kexec at ffffffff81105182
#2 [ffff88082f445b20] panic at ffffffff8167f3ba
#3 [ffff88082f445ba0] nmi_panic at ffffffff8108562f
#4 [ffff88082f445bb0] watchdog_overflow_callback at ffffffff8112f0e6
#5 [ffff88082f445bc8] __perf_event_overflow at ffffffff8117465e
#6 [ffff88082f445c00] perf_event_overflow at ffffffff811752a4
#7 [ffff88082f445c10] intel_pmu_handle_irq at ffffffff81009d88
#8 [ffff88082f445e48] perf_event_nmi_handler at ffffffff8168dbeb
#9 [ffff88082f445e68] nmi_handle at ffffffff8168f019
#10 [ffff88082f445eb0] do_nmi at ffffffff8168f233
#11 [ffff88082f445ef0] end_repeat_nmi at ffffffff8168e453
    [exception RIP: _raw_spin_lock_irqsave+71]
    RIP: ffffffff8168d9c7  RSP: ffff88082f443b70  RFLAGS: 00000002
    RAX: 0000000000007780  RBX: 0000000000000034  RCX: 0000000000008d44
    RDX: 0000000000008d42  RSI: 0000000000008d44  RDI: ffff8800c164b308
    RBP: ffff88082f443b70   R8: 0000000000000082   R9: 000015ae5127b240
    R10: 0000000000000034  R11: 0000000000000000  R12: ffff8800c164b308
    R13: ffff88080a276600  R14: ffff8800c164b200  R15: ffff8800c164b200
    ORIG_RAX: ffffffffffffffff  CS: 0010  SS: 0018
--- <NMI exception stack> ---
#12 [ffff88082f443b70] _raw_spin_lock_irqsave at ffffffff8168d9c7
#13 [ffff88082f443b78] nvkm_fantog_update at ffffffffa026ae23 [nouveau]
#14 [ffff88082f443bc0] nvkm_fantog_set at ffffffffa026af48 [nouveau]
#15 [ffff88082f443be0] nvkm_fan_update at ffffffffa026a378 [nouveau]
#16 [ffff88082f443c30] nvkm_therm_fan_set at ffffffffa026a519 [nouveau]
#17 [ffff88082f443c40] nvkm_therm_update at ffffffffa0269b87 [nouveau]
#18 [ffff88082f443c80] nvkm_therm_alarm at ffffffffa0269e17 [nouveau]
#19 [ffff88082f443c90] nvkm_timer_alarm_trigger at ffffffffa026d2d3 [nouveau]
#20 [ffff88082f443cf8] nvkm_timer_alarm at ffffffffa026d3d0 [nouveau]
#21 [ffff88082f443d30] nvkm_fantog_update at ffffffffa026aeea [nouveau]
#22 [ffff88082f443d78] nvkm_fantog_alarm at ffffffffa026af0a [nouveau]
#23 [ffff88082f443d88] nvkm_timer_alarm_trigger at ffffffffa026d2d3 [nouveau]
#24 [ffff88082f443df0] nv04_timer_intr at ffffffffa026d6fb [nouveau]
#25 [ffff88082f443e18] nvkm_timer_intr at ffffffffa026d174 [nouveau]
#26 [ffff88082f443e28] nvkm_subdev_intr at ffffffffa021da87 [nouveau]
#27 [ffff88082f443e38] nvkm_mc_intr at ffffffffa025f7f9 [nouveau]
#28 [ffff88082f443e80] nvkm_pci_intr at ffffffffa0264155 [nouveau]
#29 [ffff88082f443eb0] handle_irq_event_percpu at ffffffff8113015e
#30 [ffff88082f443ef8] handle_irq_event at ffffffff8113033d
#31 [ffff88082f443f20] handle_edge_irq at ffffffff81133007
#32 [ffff88082f443f40] handle_irq at ffffffff8102d26f
#33 [ffff88082f443f78] do_IRQ at ffffffff81698bef
```
<!--description-->

最近有一台centos7测试服务器经常崩溃重启， 进入/var/crash/目录发现有大量的crash dump文件。 于是根据网上的教程， 尝试去分析这些crash文件。

1. 安装内核调试包
下载地址：http://debuginfo.centos.org/7/x86_64/
```
kernel-debuginfo-`uname -r`.rpm
kernel-debuginfo-common-`uname -r`.rpm
```

安装： 
```
rpm -ivh kernel-debuginfo-`uname -r`.rpm
rpm -ivh kernel-debuginfo-common-`uname -r`
```

2. 进入调试
crash /usr/lib/debug/lib/modules/`uname -r`/vmlinux /var/crash/127.0.0.1-2018-04-25-01\:39\:27/vmcore

如果没有crash命令， 可通过yum安装： 
```
sudo yum -y install crash
```

进入crash调试控制台
```
crash> help

*              files          mach           repeat         timer          
alias          foreach        mod            runq           tree           
ascii          fuser          mount          search         union          
bt             gdb            net            set            vm             
btop           help           p              sig            vtop           
dev            ipcs           ps             struct         waitq          
dis            irq            pte            swap           whatis         
eval           kmem           ptob           sym            wr             
exit           list           ptov           sys            q              
extend         log            rd             task           

crash version: 7.1.9-2.el7   gdb version: 7.6
For help on any command above, enter "help <command>".
For help on input options, enter "help input".
For help on output options, enter "help output".

crash> bt
PID: 0      TASK: ffff88017caa9f60  CPU: 1   COMMAND: "swapper/1"
 #0 [ffff88082f4459f0] machine_kexec at ffffffff81059cdb
 #1 [ffff88082f445a50] __crash_kexec at ffffffff81105182
 #2 [ffff88082f445b20] panic at ffffffff8167f3ba
 #3 [ffff88082f445ba0] nmi_panic at ffffffff8108562f
 #4 [ffff88082f445bb0] watchdog_overflow_callback at ffffffff8112f0e6
 #5 [ffff88082f445bc8] __perf_event_overflow at ffffffff8117465e
 #6 [ffff88082f445c00] perf_event_overflow at ffffffff811752a4
 #7 [ffff88082f445c10] intel_pmu_handle_irq at ffffffff81009d88
 #8 [ffff88082f445e48] perf_event_nmi_handler at ffffffff8168dbeb
 #9 [ffff88082f445e68] nmi_handle at ffffffff8168f019
#10 [ffff88082f445eb0] do_nmi at ffffffff8168f233
#11 [ffff88082f445ef0] end_repeat_nmi at ffffffff8168e453
    [exception RIP: _raw_spin_lock_irqsave+71]
    RIP: ffffffff8168d9c7  RSP: ffff88082f443b70  RFLAGS: 00000002
    RAX: 00000000000074ba  RBX: 0000000000000032  RCX: 0000000000006e68
    RDX: 0000000000006e66  RSI: 0000000000006e68  RDI: ffff88080ac80908
    RBP: ffff88082f443b70   R8: 0000000000000082   R9: 00000b17a10e3200
    R10: 0000000000000032  R11: 0000000000000000  R12: ffff88080ac80908
    R13: ffff8800c15d1c80  R14: ffff88080ac80800  R15: ffff88080ac80800
    ORIG_RAX: ffffffffffffffff  CS: 0010  SS: 0018
--- <NMI exception stack> ---
#12 [ffff88082f443b70] _raw_spin_lock_irqsave at ffffffff8168d9c7
#13 [ffff88082f443b78] nvkm_fantog_update at ffffffffa0258e23 [nouveau]
#14 [ffff88082f443bc0] nvkm_fantog_set at ffffffffa0258f48 [nouveau]
#15 [ffff88082f443be0] nvkm_fan_update at ffffffffa0258378 [nouveau]
#16 [ffff88082f443c30] nvkm_therm_fan_set at ffffffffa0258519 [nouveau]
#17 [ffff88082f443c40] nvkm_therm_update at ffffffffa0257b87 [nouveau]
#18 [ffff88082f443c80] nvkm_therm_alarm at ffffffffa0257e17 [nouveau]
#19 [ffff88082f443c90] nvkm_timer_alarm_trigger at ffffffffa025b2d3 [nouveau]
#20 [ffff88082f443cf8] nvkm_timer_alarm at ffffffffa025b3d0 [nouveau]
#21 [ffff88082f443d30] nvkm_fantog_update at ffffffffa0258eea [nouveau]
#22 [ffff88082f443d78] nvkm_fantog_alarm at ffffffffa0258f0a [nouveau]
#23 [ffff88082f443d88] nvkm_timer_alarm_trigger at ffffffffa025b2d3 [nouveau]
#24 [ffff88082f443df0] nv04_timer_intr at ffffffffa025b6fb [nouveau]
#25 [ffff88082f443e18] nvkm_timer_intr at ffffffffa025b174 [nouveau]
#26 [ffff88082f443e28] nvkm_subdev_intr at ffffffffa020ba87 [nouveau]
#27 [ffff88082f443e38] nvkm_mc_intr at ffffffffa024d7f9 [nouveau]
#28 [ffff88082f443e80] nvkm_pci_intr at ffffffffa0252155 [nouveau]
#29 [ffff88082f443eb0] handle_irq_event_percpu at ffffffff8113015e
#30 [ffff88082f443ef8] handle_irq_event at ffffffff8113033d
#31 [ffff88082f443f20] handle_edge_irq at ffffffff81133007
#32 [ffff88082f443f40] handle_irq at ffffffff8102d26f
#33 [ffff88082f443f78] do_IRQ at ffffffff81698bef
--- <IRQ stack> ---
#34 [ffff88017cb1bda8] ret_from_intr at ffffffff8168dd6d
    [exception RIP: cpuidle_enter_state+82]
    RIP: ffffffff81514052  RSP: ffff88017cb1be50  RFLAGS: 00000206
    RAX: 00000b1763727d88  RBX: 000000000000f8a0  RCX: 0000000000000018
    RDX: 0000000225c17d03  RSI: ffff88017cb1bfd8  RDI: 00000b1763727d88
    RBP: ffff88017cb1be78   R8: 0000000000001d21   R9: 0000000000000018
    R10: 000000000000212a  R11: 0000000000000000  R12: ffff88017cb1be20
    R13: ffff88082f44f8e0  R14: 0000000000000082  R15: ffff88082f44f8e0
    ORIG_RAX: ffffffffffffff8d  CS: 0010  SS: 0018
#35 [ffff88017cb1be80] cpuidle_idle_call at ffffffff81514199
#36 [ffff88017cb1bec0] arch_cpu_idle at ffffffff8103516e
#37 [ffff88017cb1bed0] cpu_startup_entry at ffffffff810e7c95
#38 [ffff88017cb1bf28] start_secondary at ffffffff8104f12a
```

发现报错是跟nouveau有关，于是尝试去禁掉nouveau

重新安装Nvidia驱动可参考： 
https://zhuanlan.zhihu.com/p/31781281

3. 禁掉nouveau驱动

在/lib/modprobe.d/dist-blacklist.conf中，将nvidiafb注释掉：
```
#blacklist nvidiafb 
```

再在该文件中添加一下配置：
```
blacklist nouveau  
options nouveau modeset=0 
```

4. 重建initramfs image: 

如果/boot 分区大小不够，可以备份到其他目录
```
mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak  
dracut /boot/initramfs-$(uname -r).img $(uname -r) 
```

5. 重启系统
```
reboot
``` 

6. 系统重启后，查看nouveau驱动是否已经被禁止掉
```
[root@gemfield.org ~]# lsmod | grep nouveau 
[root@gemfield.org ~]#
```


nouveau驱动被禁掉后，没有再发生系统崩溃的情况。


