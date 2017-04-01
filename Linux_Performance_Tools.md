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
### Benchmarking
Load test. Caution: production tests can cause issues due to contention
### Tuning
Change. Danger: changes could hurt performance, now or later with load.
### Static
Check configuration. Should be safe.

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

