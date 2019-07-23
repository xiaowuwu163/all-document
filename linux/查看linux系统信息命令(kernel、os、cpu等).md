# [查看linux系统信息命令(kernel、os、cpu等)](https://www.cnblogs.com/stark-summer/p/4829829.html)

### 1、查看当前操作系统内核信息

[spark@S1PA222 tomcat6]$ uname -a
Linux S1PA222 2.6.32-358.el6.x86_64 #1 SMP Fri Feb 22 00:31:26 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux

### 2、查看当前操作系统发行版信息

[spark@S1PA222 tomcat6]$ cat /etc/issue
CentOS release 6.4 (Final)
Kernel \r on an \m

### 3、查看cpu型号

[spark@S1PA222 tomcat6]$ cat /proc/cpuinfo |grep name|cut -d ':' -f 2|uniq -c
      4  Intel(R) Xeon(R) CPU           E5649  @ 2.53GHz

### 4、查看物理cpu颗数

[spark@S1PA222 tomcat6]$ cat /proc/cpuinfo |grep physical|uniq -c
      1 physical id     : 0
      1 address sizes   : 40 bits physical, 48 bits virtual
      1 physical id     : 0
      1 address sizes   : 40 bits physical, 48 bits virtual
      1 physical id     : 1
      1 address sizes   : 40 bits physical, 48 bits virtual
      1 physical id     : 1
      1 address sizes   : 40 bits physical, 48 bits virtual

### 5、查看cpu运行模式

[spark@S1PA222 tomcat6]$ getconf LONG_BIT
64

### 6、查看cpu是否支持64bit

[spark@S1PA222 tomcat6]$ cat /proc/cpuinfo |grep flags|grep 'lm'|wc -l
4

(结果大于0, 说明支持64bit计算. lm指long mode, 支持lm则是64bit)

### 7、查看cpu信息概要

[spark@S1PA222 tomcat6]$ lscpu
Architecture:          x86_64   # 
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    2
Socket(s):             2
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 44
Stepping:              2
CPU MHz:               2533.423
BogoMIPS:              5066.84
Hypervisor vendor:     VMware
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              12288K
NUMA node0 CPU(s):     0-3

### 8、查看cpu信息概要（比较全）

[spark@S1PA222 tomcat6]$ cat /proc/cpuinfo
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 44
model name      : Intel(R) Xeon(R) CPU           E5649  @ 2.53GHz
stepping        : 2
cpu MHz         : 2533.423
cache size      : 12288 KB
physical id     : 0
siblings        : 2
core id         : 0
cpu cores       : 2
apicid          : 0
initial apicid  : 0
fpu             : yes
fpu_exception   : yes
cpuid level     : 11
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts xtopology tsc_reliable nonstop_tsc aperfmperf unfair_spinlock pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 popcnt aes hypervisor lahf_lm arat epb dts
bogomips        : 5066.84
clflush size    : 64
cache_alignment : 64
address sizes   : 40 bits physical, 48 bits virtual
power management:
processor       : 1
vendor_id       : GenuineIntel
cpu family      : 6
model           : 44
model name      : Intel(R) Xeon(R) CPU           E5649  @ 2.53GHz
stepping        : 2
cpu MHz         : 2533.423
cache size      : 12288 KB
physical id     : 0
siblings        : 2
core id         : 1
cpu cores       : 2
apicid          : 1
initial apicid  : 1
fpu             : yes
fpu_exception   : yes
cpuid level     : 11
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts xtopology tsc_reliable nonstop_tsc aperfmperf unfair_spinlock pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 popcnt aes hypervisor lahf_lm arat epb dts
bogomips        : 5066.84
clflush size    : 64
cache_alignment : 64
address sizes   : 40 bits physical, 48 bits virtual
power management:
processor       : 2
vendor_id       : GenuineIntel
cpu family      : 6
model           : 44
model name      : Intel(R) Xeon(R) CPU           E5649  @ 2.53GHz
stepping        : 2
cpu MHz         : 2533.423
cache size      : 12288 KB
physical id     : 1
siblings        : 2
core id         : 0
cpu cores       : 2
apicid          : 2
initial apicid  : 2
fpu             : yes
fpu_exception   : yes
cpuid level     : 11
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts xtopology tsc_reliable nonstop_tsc aperfmperf unfair_spinlock pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 popcnt aes hypervisor lahf_lm arat epb dts
bogomips        : 5066.84
clflush size    : 64
cache_alignment : 64
address sizes   : 40 bits physical, 48 bits virtual
power management:
processor       : 3
vendor_id       : GenuineIntel
cpu family      : 6
model           : 44
model name      : Intel(R) Xeon(R) CPU           E5649  @ 2.53GHz
stepping        : 2
cpu MHz         : 2533.423
cache size      : 12288 KB
physical id     : 1
siblings        : 2
core id         : 1
cpu cores       : 2
apicid          : 3
initial apicid  : 3
fpu             : yes
fpu_exception   : yes
cpuid level     : 11
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts xtopology tsc_reliable nonstop_tsc aperfmperf unfair_spinlock pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 popcnt aes hypervisor lahf_lm arat epb dts
bogomips        : 5066.84
clflush size    : 64
cache_alignment : 64
address sizes   : 40 bits physical, 48 bits virtual
power management: