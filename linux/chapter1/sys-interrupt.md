# Linux的系统中断

中断分为硬中断和软中断，发生中断以后，中断处理程序会被调用。

中断处理程序运行期间，不能再次中断（设置了中断禁止）的中断是硬中断，硬中断处理期间发生的中断会丢失，所以硬中断的处理时间不能过程。中断处理程序运行期间，可以再次中断的中断是软中断。

Linux将中断处理过程分为两个阶段：

1. 第一阶段是上半部，在中断禁止模式下运行，用来快速处理硬件相关或者时间敏感的任务，触发上半部处理的中断是硬中断；

2. 第二阶段是下半部，允许继续中断，用于处理上半部未完成的工作，通常以内核线程的方式运行，触发下半部操作的中断是软中断。

上半部处理结束时，通常会发送一个软中断，由下半部处理时效性不那么紧迫的任务。上半部通常是硬件触发，处理硬件请求，下半部是内核触发。

下半部是软中断，软中断不全是下半部，一些内核自定义事件也是软中断，例如内核调度、RCU锁等。

每个CPU都有一个软中断内核线程，ksoftirqd/cpu编号：

```sh
$ps aux|grep ksoft
root         3  0.0  0.0      0     0 ?        S    Apr05   1:19 [ksoftirqd/0]
root        13  0.0  0.0      0     0 ?        S    Apr05   1:24 [ksoftirqd/1]
root        18  0.0  0.0      0     0 ?        S    Apr05   1:25 [ksoftirqd/2]
root        23  0.0  0.0      0     0 ?        S    Apr05   1:14 [ksoftirqd/3]
root        28  0.0  0.0      0     0 ?        S    Apr05   1:26 [ksoftirqd/4]
root        33  0.0  0.0      0     0 ?        S    Apr05   1:09 [ksoftirqd/5]
root        38  0.0  0.0      0     0 ?        S    Apr05   1:09 [ksoftirqd/6]
```

## 系统中断状态

`/proc/softirqs`中是软中断的运行情况，同一中断在不同CPU上的运行次数应当是接近的，如果明显不均衡，表明多核CPU没有被充分利用：

```sh
$ cat /proc/softirqs
                    CPU0       CPU1       CPU2       CPU3
          HI:          1          1          0          0
       TIMER:  607522918  541095809  566655073  545147469
      NET_TX:    1188125     727904    1105106    1090468
      NET_RX:  227103214 1942391353  244785330  214470397
       BLOCK:    9212292    9070209    6004124    9196281
    IRQ_POLL:          0          0          0          0
     TASKLET:        461        703        559        512
       SCHED:  262424378  225458960  231808842  212340029
     HRTIMER:       1598       1163       1373       1445
         RCU:  348165561  309546532  322077899  312389145
```

`/proc/interrupts`中是硬中断的运行情况：

```sh
$ cat /proc/interrupts
           CPU0       CPU1       CPU2       CPU3
  0:        114          0          0          0   IO-APIC   2-edge      timer
  1:          9          0          0          0   IO-APIC   1-edge      i8042
  6:          0          0          3          0   IO-APIC   6-edge      floppy
  8:          0          0          0          0   IO-APIC   8-edge      rtc0
  9:          0          0          0          0   IO-APIC   9-fasteoi   acpi
 10:          0          0          0          0   IO-APIC  10-fasteoi   virtio0
 11:          0          0         35          0   IO-APIC  11-fasteoi   uhci_hcd:usb1
 12:          0          0          0         15   IO-APIC  12-edge      i8042
 14:          0          0          0          0   IO-APIC  14-edge      ata_piix
 15:          0          0          0          0   IO-APIC  15-edge      ata_piix
 24:          0          0          0          0   PCI-MSI 81920-edge      virtio2-config
 25:          0          0          3          0   PCI-MSI 81921-edge      virtio2-virtqueues
 26:          0          0          0          0   PCI-MSI 65536-edge      virtio1-config
 27:         62 1012111617          0          0   PCI-MSI 65537-edge      virtio1-input.0
 28:         54          2          0      22109   PCI-MSI 65538-edge      virtio1-output.0
 29:          0          0          0          0   PCI-MSI 98304-edge      virtio3-config
 30:          0          0    9853249          0   PCI-MSI 98305-edge      virtio3-req.0
 31:          0          0          0          0   PCI-MSI 114688-edge      virtio4-config
 32:          0          0   12293598          0   PCI-MSI 114689-edge      virtio4-req.0
NMI:          0          0          0          0   Non-maskable interrupts
LOC: 1217427530 1144644230 1177257177 1149689000   Local timer interrupts
SPU:          0          0          0          0   Spurious interrupts
PMI:          0          0          0          0   Performance monitoring interrupts
IWI:        153        120        264        318   IRQ work interrupts
RTR:          0          0          0          0   APIC ICR read retries
RES: 1027217904  904556924  981991984  977800420   Rescheduling interrupts
CAL:  233863698   11399907  241091628  220970613   Function call interrupts
TLB:    1487790    1520042    1474210    1505102   TLB shootdowns
TRM:          0          0          0          0   Thermal event interrupts
THR:          0          0          0          0   Threshold APIC interrupts
DFR:          0          0          0          0   Deferred Error APIC interrupts
MCE:          0          0          0          0   Machine check exceptions
MCP:      14170      14170      14170      14170   Machine check polls
HYP:          0          0          0          0   Hypervisor callback interrupts
HRE:          0          0          0          0   Hyper-V reenlightenment interrupts
HVS:          0          0          0          0   Hyper-V stimer0 interrupts
ERR:          0
MIS:          0
PIN:          0          0          0          0   Posted-interrupt notification event
NPI:          0          0          0          0   Nested posted-interrupt event
PIW:          0          0          0          0   Posted-interrupt wakeup event
```
