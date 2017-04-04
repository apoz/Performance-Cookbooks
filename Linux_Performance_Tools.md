# Linux Performance Tools (Work in progress)
## Introduction
This repo is a quick cookbook about the tools I find to check/investigate performance in Linux systems.  It will therefore contain mostly summaries of 
[Brendan's Gregg](https://brendangregg.com) awesome job.

## Performance investigation methodology
### Workload Characterization Method
1. *Who* is causing the load? PID, UID, IP addr...
2. *Why* is the load called? code path, stack trace
3. *What* is the load? IOPS, tput, type, r/w
4. *How* is the load changing over time?

### USE Method
* For every resource, check:
  1. Utilization
  2. Saturation
  3. Errors

Reference for USE method in Linux systems: http://brendangregg.com/USEmethod/use-linux.html

## Performance tools
[Reference](http://techblog.netflix.com/2015/08/netflix-at-velocity-2015-linux.html)
### Observability
Watch activity. Safe, usually, depending on resource overhead
![Observability tools](http://www.brendangregg.com/Perf/linux_observability_tools.png)
#### Basic Tools
##### Uptime
```
apoz@apoz-VirtualBox:~$ uptime
 16:33:49 up 0 min,  1 user,  load average: 1,73, 0,49, 0,17
 ```
 On Linux systems, these numbers include processes wanting to run on CPU, as well as processes blocked in uninterruptible I/O (usually disk I/O). This gives a high level idea of resource load (or demand), but can’t be properly understood without other tools. Worth a quick look only. 

* Measure of resource demand: CPU + disks
* Time constants of 1, 5 and 15 minutes
* If the load > # CPUs it may mean CPU saturation

##### top or htop
```
top - 16:38:21 up 5 min,  1 user,  load average: 0,17, 0,31, 0,18
Tasks: 186 total,   1 running, 185 sleeping,   0 stopped,   0 zombie
%Cpu(s):  9,1 us,  6,6 sy,  3,8 ni, 77,2 id,  2,5 wa,  0,0 hi,  0,8 si,  0,0 st
KiB Mem :   991696 total,   111460 free,   581848 used,   298388 buff/cache
KiB Swap:  1021948 total,   911256 free,   110692 used.   241016 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND      
 2728 apoz      20   0 1222528 120220  10956 S 18,8 12,1   0:10.41 compiz       
 1785 root      20   0  305360  39464   6884 S  6,2  4,0   0:02.92 Xorg         
    1 root      20   0  185456   3552   2232 S  0,0  0,4   0:01.47 systemd      
    2 root      20   0       0      0      0 S  0,0  0,0   0:00.00 kthreadd     
    3 root      20   0       0      0      0 S  0,0  0,0   0:00.22 ksoftirqd/0  
    4 root      20   0       0      0      0 S  0,0  0,0   0:00.06 kworker/0:0  
    5 root       0 -20       0      0      0 S  0,0  0,0   0:00.00 kworker/0:0H 
    6 root      20   0       0      0      0 S  0,0  0,0   0:00.00 kworker/u2:0 
    7 root      20   0       0      0      0 S  0,0  0,0   0:00.27 rcu_sched    
    8 root      20   0       0      0      0 S  0,0  0,0   0:00.00 rcu_bh       
    9 root      rt   0       0      0      0 S  0,0  0,0   0:00.00 migration/0  
   10 root      rt   0       0      0      0 S  0,0  0,0   0:00.00 watchdog/0   
   11 root      20   0       0      0      0 S  0,0  0,0   0:00.00 kdevtmpfs    
   12 root       0 -20       0      0      0 S  0,0  0,0   0:00.00 netns        
   13 root       0 -20       0      0      0 S  0,0  0,0   0:00.00 perf         
   14 root      20   0       0      0      0 S  0,0  0,0   0:00.00 khungtaskd   
   15 root       0 -20       0      0      0 S  0,0  0,0   0:00.00 writeback  
```
* Can miss short-lived processes
* Can consume noticeable CPU to read/process

The top command includes many of the metrics we checked earlier. It can be handy to run it to see if anything looks wildly different from the earlier commands, which would indicate that load is variable. 

A downside to top is that it is harder to see patterns over time, which may be more clear in tools like vmstat and pidstat, which provide rolling output. Evidence of intermittent issues can also be lost if you don’t pause the output quick enough (Ctrl-S to pause, Ctrl-Q to continue), and the screen clears. 
##### ps
* ps -ef -f shows the tree of processes and its status
* it can show custom fields ie: ps -eo user,sz,rss,minflt,majflt,pcpu,args,

##### vmstat
```apoz@apoz-VirtualBox:~$ vmstat -Sm 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0    107     99     16    304    0    0  2021   635  211 1840  8  4 86  1  0
 0  0    107     98     16    304    0    0     0     0   94  778  5  1 94  0  0
 0  0    107     98     16    304    0    0     0     0   96  843  8  0 92  0  0
 0  0    107     98     16    304    0    0     0     0   91  625  5  1 94  0  0
 0  0    107     98     16    304    0    0     0     0   70  543  3  0 97  0  0
 0  0    107     98     16    304    0    0     0     0   89  873  5  1 94  0  0
```
* Virtual memory statistics
* Updated every second with vmstat -Sm 1
* High level CPU summary
**  r is runnable tasks
Columns to check:
* *r*: Number of processes running on CPU and waiting for a turn. This provides a better signal than load averages for determining CPU saturation, as it does not include I/O. To interpret: an “r” value greater than the CPU count is saturation.
* *free*: Free memory in kilobytes. If there are too many digits to count, you have enough free memory. The “free -m” command, included as command 7, better explains the state of free memory.
* *si, so*: Swap-ins and swap-outs. If these are non-zero, you’re out of memory.
* *us, sy, id, wa, st*: These are breakdowns of CPU time, on average across all CPUs. They are user time, system time (kernel), idle, wait I/O, and stolen time (by other guests, or with Xen, the guest's own isolated driver domain).

The CPU time breakdowns will confirm if the CPUs are busy, by adding user + system time. A constant degree of wait I/O points to a disk bottleneck; this is where the CPUs are idle, because tasks are blocked waiting for pending disk I/O. You can treat wait I/O as another form of CPU idle, one that gives a clue as to why they are idle. 

System time is necessary for I/O processing. A high system time average, over 20%, can be interesting to explore further: perhaps the kernel is processing the I/O inefficiently. 

In the above example, CPU time is almost entirely in user-level, pointing to application level usage instead. The CPUs are also well over 90% utilized on average. This isn’t necessarily a problem; check for the degree of saturation using the “r” column.

##### iostat
```
poz@apoz-VirtualBox:~$ iostat -xdmz 1
Linux 4.4.0-31-generic (apoz-VirtualBox)     02/04/17     _x86_64_    (1 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
scd0              0,00     0,00    0,02    0,00     0,00     0,00     6,36     0,00    0,36    0,36    0,00   0,36   0,00
sda               0,39    53,89   56,27    6,38     1,25     0,49    56,67     0,07    1,04    0,77    3,43   0,47   2,97

Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0,00     0,00    8,08    0,00     0,21     0,00    52,00     0,00    0,00    0,00    0,00   0,00   0,00

Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util


```
* Block I/O (disk) stats
* Usage example: iostat -xmdz 1 or iostat -xz 1
This is a great tool for understanding block devices (disks), both the workload applied and the resulting performance. Look for: 
* *r/s, w/s, rkB/s, wkB/s*: These are the delivered reads, writes, read Kbytes, and write Kbytes per second to the device. Use these for workload characterization. A performance problem may simply be due to an excessive load applied.
* *await*: The average time for the I/O in milliseconds. This is the time that the application suffers, as it includes both time queued and time being serviced. Larger than expected average times can be an indicator of device saturation, or device problems.
* *avgqu-sz*: The average number of requests issued to the device. Values greater than 1 can be evidence of saturation (although devices can typically operate on requests in parallel, especially virtual devices which front multiple back-end disks.)
* *%util*: Device utilization. This is really a busy percent, showing the time each second that the device was doing work. Values greater than 60% typically lead to poor performance (which should be seen in await), although it depends on the device. Values close to 100% usually indicate saturation.
If the storage device is a logical disk device fronting many back-end disks, then 100% utilization may just mean that some I/O is being processed 100% of the time, however, the back-end disks may be far from saturated, and may be able to handle much more work. 

Bear in mind that poor performing disk I/O isn’t necessarily an application issue. Many techniques are typically used to perform I/O asynchronously, so that the application doesn’t block and suffer the latency directly (e.g., read-ahead for reads, and buffering for writes). 




##### mpstat
```
apoz@apoz-VirtualBox:~$ mpstat -P ALL 1
Linux 4.4.0-31-generic (apoz-VirtualBox)     02/04/17     _x86_64_    (1 CPU)

23:27:53     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
23:27:53     all    6,48    0,50    1,18    0,31    0,00    0,11    0,00    0,00    0,00   91,42
23:27:53       0    6,48    0,50    1,18    0,31    0,00    0,11    0,00    0,00    0,00   91,42
```
* Multi-processor statistics, per cpu
* Look for unbalanced workloads, hot CPUs.
This command prints CPU time breakdowns per CPU, which can be used to check for an imbalance. A single hot CPU can be evidence of a single-threaded application. 

##### free
```
apoz@apoz-VirtualBox:~$ free -m
              total        used        free      shared  buff/cache   available
Mem:            968         547         106           6         314         251
Swap:           997         142         855
```
* Main memory usage
* Buffers: block device I/O cache
* cached: virtual page cache

The right two columns show:
* *buffers*: For the buffer cache, used for block device I/O.
* *cached*: For the page cache, used by file systems.
We just want to check that these aren’t near-zero in size, which can lead to higher disk I/O (confirm using iostat), and worse performance. The above example looks fine, with many Mbytes in each. 

The “-/+ buffers/cache” provides less confusing values for used and free memory. Linux uses free memory for the caches, but can reclaim it quickly if applications need it. So in a way the cached memory should be included in the free memory column, which this line does. There’s even a website, linuxatemyram, about this confusion. 

It can be additionally confusing if ZFS on Linux is used, as we do for some services, as ZFS has its own file system cache that isn’t reflected properly by the free -m columns. It can appear that the system is low on free memory, when that memory is in fact available for use from the ZFS cache as needed. 


#### Intermediate Tools
##### strace
* System call tracer
```
strace -tttT -p PID
```
* -ttt: time (us) since epoch; -T: syscall time (s)
* Translates syscall args
* Currently has massive overhead (ptrace based)
** can slow the target by >100x. *Use extreme caution*
##### tcpdump
* Sniff network packets for post analysis
* CPU overhead optimized, but use with caution.
##### netstat
* Various network protocol statistics using -s
* A multi-tool:
** -i: interface states
** -r: route table
** default: list conns
* netstat -p: show process details!
* Per-second interval with -c
##### nicstat
* Network interfaces stats, iostat-like output
* Check network throughput and interface % util
##### pidstat
* very useful process stat. Eg: by thread, disk I/O...
Pidstat is a little like top’s per-process summary, but prints a rolling summary instead of clearing the screen. This can be useful for watching patterns over time, and also recording what you saw (copy-n-paste) into a record of your investigation. 

The above example identifies two java processes as responsible for consuming CPU. The %CPU column is the total across all CPUs; 1591% shows that that java processes is consuming almost 16 CPUs. 

##### swapon
* Shows swap device usage
##### lsof
* File descriptor usage
##### sar
* System activity reporter
* sar -n TCP,ETCP,DEV,EDEV 1
![SAR](http://www.brendangregg.com/Perf/linux_observability_sar)
*sar -n DEV 1*
Use this tool to check network interface throughput: rxkB/s and txkB/s, as a measure of workload, and also to check if any limit has been reached. In the above example, eth0 receive is reaching 22 Mbytes/s, which is 176 Mbits/sec (well under, say, a 1 Gbit/sec limit). 

This version also has %ifutil for device utilization (max of both directions for full duplex), which is something we also use Brendan’s nicstat tool to measure. And like with nicstat, this is hard to get right, and seems to not be working in this example (0.00). 

*sar -n TCP,ETCP 1*
This is a summarized view of some key TCP metrics. These include:
* *active/s*: Number of locally-initiated TCP connections per second (e.g., via connect()).
* *passive/s*: Number of remotely-initiated TCP connections per second (e.g., via accept()).
* *retrans/s*: Number of TCP retransmits per second.
The active and passive counts are often useful as a rough measure of server load: number of new accepted connections (passive), and number of downstream connections (active). It might help to think of active as outbound, and passive as inbound, but this isn’t strictly true (e.g., consider a localhost to localhost connection). 

Retransmits are a sign of a network or server issue; it may be an unreliable network (e.g., the public Internet), or it may be due a server being overloaded and dropping packets. The example above shows just one new TCP connection per-second. 




#### Advanced Tools
##### ss
 More socket statistics.
##### iptraf
##### iotop
 * Block device I/O (disk) by process
 * Needs kernel support enabled (CONFIG_TASK_IO_ACCOUNTING)

##### slabtop
* Kernel slab allocator memory usage
##### pcstat
* Show page cache residency by file
* Example 'pcstat data0ASTERISK'
* Uses the mincore(2) syscall. Useful for database performance analysis.
##### tiptip
* IPC by process, %MISS, %BUS
##### rdmsr
* Model specific Registers (MSR), unlike PMCs can be read by default in Xen:
- Timestamp clock, temp, power...



### Benchmarking
Load test. Caution: production tests can cause issues due to contention
![Benchmarking tools](http://www.brendangregg.com/Perf/linux_benchmarking_tools.png)
### Tuning
Change. Danger: changes could hurt performance, now or later with load.
![Tuning tools](http://www.brendangregg.com/Perf/linux_tuning_tools.png)
### Static
Check configuration. Should be safe.
![Static tools](http://www.brendangregg.com/Perf/linux_static_tools.png)

## 10 first tools to use for any performance investigation

[This](http://techblog.netflix.com/2015/11/linux-performance-analysis-in-60s.html) is the blog post describing the first 60 seconds in a 
performance investigation in Netflix.

### Uptime

```
 apoz@apoz-VirtualBox:~$ uptime
 23:18:10 up 3 min,  1 user,  load average: 0,36, 0,46, 0,20
```

### Other Tools
#### top
#### iotop
#### iostat -x 1
#### netstat  (-i    or -s)
#### ss (socket stat)
#### dstat
#### sar -n DEV 1
#### vmstat 1
#### strace -p 'pgrep process_name'


## References
* http://techblog.netflix.com/2015/11/linux-performance-analysis-in-60s.html
* http://techblog.netflix.com/2015/08/netflix-at-velocity-2015-linux.html
* http://brendangregg.com/USEmethod/use-linux.html

