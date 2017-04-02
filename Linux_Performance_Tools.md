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
### Observability
Watch activity. Safe, usually, depending on resource overhead
![Observability tools](http://www.brendangregg.com/Perf/linux_observability_tools.png)
#### Basic Tools
##### Uptime
```
apoz@apoz-VirtualBox:~$ uptime
 16:33:49 up 0 min,  1 user,  load average: 1,73, 0,49, 0,17
 ```
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
##### ps
* ps -ef -f shows the tree of processes and its status
* it can show custom fields

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
* Usage example: iostat -xmdz 1


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

