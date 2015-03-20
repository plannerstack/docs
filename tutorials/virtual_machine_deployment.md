# Virtual machine deployment
At PlannerStack we are working hard towards real-time multimodal journey planners, available to anyone as a commodity. We envision that launching an affordable Software as a Service (SaaS) solution brings us closer to this goal. In the past year we collaborated with many skilled developers from a variety of solution providers to bring OpenTripPlanner to the masses. In this blog we layout the required effort to compress all that labor into a single deployable image. 

# Infrastructure as a service
Firstly we needed to know our numbers. At what cost would be able to launch a service, and what number of machines would be required for that? We did a benchmark on the currently OpenTripPlanner master and were satisfied with the results. Given we would be able to serve about 12 req/s from one single instance, a connected service provider would be able to spend its entire budget of 100k of requests in about two hours and twenty two minutes. Sounds reasonable. For a proper infrastructure we need an image that can be deployed easily, and for some reason we still believe that anything costing more than a few hundred megabytes is just too much. OpenTripPlanner is what we want, Java is what is required, hence we want a system that is connected to the network that can fulfil these constraints.

## Performance bottlenecks
Did we find any performance bottlenecks? Sure we did! The real-time deployment in The Netherlands and Belgium is part of the greatest deployments of OpenTripPlanner worldwide, resulting in a graph.obj of a whopping 4GB. The vanilla OpenTripPlanner system consumes about 19GB of memory for searching through the the transit and street network. We needed rock solid numbers: how many parallel searches would be possible in our infrastructure? Would we be able to recover the cost? For this benchmark we have have used a full blown 4x4 core Xeon E7330 system with 128GB of memory. Checking out NUMA was on our list, but this system couldn't allocate memory to specific CPUs. The output of one CPU from /proc/cpuinfo is below.

```
vendor_id       : GenuineIntel
cpu family      : 6
model           : 15
model name      : Intel(R) Xeon(R) CPU E7330  @ 2.40GHz
stepping        : 11
microcode       : 0xbb
cpu MHz         : 2393.848
cache size      : 3072 KB
physical id     : 6
siblings        : 4
core id         : 3
cpu cores       : 4
apicid          : 27
initial apicid  : 27
fpu             : yes
fpu_exception   : yes
cpuid level     : 10
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx lm constant_tsc arch_perfmon pebs bts rep_good nopl aperfmperf pni dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm dca lahf_lm dtherm tpr_shadow vnmi flexpriority
bogomips        : 4787.81
clflush size    : 64
cache_alignment : 64
address sizes   : 40 bits physical, 48 bits virtual
```

Given that memory wasn't really a problem, the CPU cache was pretty awesome, we were hoping for an epic linear scaling, as found by previous benchmarks.

## Method
We were interested in the performance of the planner-API output. Therefore we copied a single URL from the OTP leaflet client, resulting in a request to the OTP-API. In order to send request to the OpenTripPlanner instance we used [weighttp](https://github.com/lighttpd/weighttp) with a reasonable number of request, thread count and concurrent clients, with keep-alive. We were only interested in hot performance, hence the system received an initial request.

```
weighttp -n 2000 -t 32 -c 32 -k
```

## Results
While we have tested up to 32 threads, we did not observe any performance increase after 15 threads. As you can see in the graph below no single request is cached by any intermediate layer. Each request is recalculated each time it enters the system. Up to 4 CPUs we notice linear scaling, which becomes sub linear up to 15 CPUs. This may suggest the system is bound by the memory bandwidth. Is it worthwhile to invest in more CPU power above 4, or 8 CPUs with OpenTripPlanner? At this moment low end memory of 4x8GB would sell for about 280 euro, an Intel i7 goes for roughly the same price. Given the improvement from 13 req/s to 19 req/s it is not worthwhile to invest in the extra 8 cores in a single system. If our target would be 21 req/s we could implement that with 3 Intel i5 systems selling at 190 euro each, plus the 270 euro memory per system, this would bring us to to a three fold redundant setup of 1380 euro, which would be about 25% higher price in compare to a solution with two separate Intel i7 machines at 1120 euro combined. benchmark  

![](http://plannerstack.files.wordpress.com/2014/04/benchmark.png)

# The art of designing an embedded Linux Distribution
The best way to learn about Linux, is to actually learn what the GNU/Linux system is composed of. The key components that form the working bench that you are using every day to perform your magic, knowing what it requires and espects in full harmony. [LinuxFromScratch](http://www.linuxfromscratch.org/) was my choice of living, and I considered everyone running anything else just lazy. Talking about open source, zealoting about not open-not-close-enough-licenses, but never touching a compiler? It just doesn't add up. Given that experience it is quite trivial to find out what a Linux kernel configuration should look like for a system that is expected to boot on VMware, VirtualBox or Qemu. What is expected after that, and what properties are used when Java touches the otp.jar file.

## Key components
Every system boots in some sort of bootloader. For sake of simplicity I am currently sticking with grub 0.97, eventually replacing it by SYSLINUX, to perform booting over the network by [PXELINUX](http://www.syslinux.org/wiki/index.php/PXELINUX). The Linux 3.14 kernel I have configured is monolitic, no modules, no swap, just what we need to start up networking and Java. That includes POSIX FileLocking, /proc, /dev,/sys and tmpfs for /tmp. Obviously some drivers are required, I have picked the paravirtualised drivers identified by vmware and virtio. A special touch of magic is kernel network autoconfiguration. This allows me to specify ip=dhcp and the grub cmdline. The kernel handles IP address allocation, without any userspace tools. At this moment I am running ext2 filesystem, which might be replaced by root-over-NFS or Squashfs via initrd. Given the requirements to run Java and otp.jar are about 140MB, I am still considering my options. At the moment of writing OpenTripPlanner still requires a graph.obj. I have seen the new world where just gtfs.zip is enough, until then some sort of storage is required, just to store the graph.obj.

## The filesystem
Grub, stage1, stage2, e2fs_stage1_5 and  and vmlinuz are installed on a 8MB partition, they take about 2.1MB. The root filesystem hosts [busybox](http://www.busybox.net/) in /bin. Busybox is basically an all-in-one binary hosting a set of tools commonly used in an embedded context. Since everything is hosted in a single binary it doesn't eat up precious inodes per individual tool. The basics include a startup script that mounts /dev, /proc, /sys and /tmp and allows to start a terminal, just in case you want to interact with the machine. OpenTripPlanner has a hard dependency on Java. For this adventure I have used Oracle Java 1.7 (crippled it to 107MB), I might have used icedtea-7.2 (94MB stock), since it wouldn't have required me to throw away so many crap such as Java FX. Consider a future benchmark between the two as a todo, I am happy to migrate. otp.jar and a small graph.obj costs me 212MB. By default OpenTripPlanner floods the terminal, Thomas Koch [suggested](https://groups.google.com/forum/#!topic/opentripplanner-dev/uzZQD2DGrUg) a trivial way to disable this excessive output. Rerunning mvn clean package, gave me exactly what I wanted.

See the video tutorial [Booting Opentripplanner Virtual Machine](https://www.youtube.com/watch?v=8anjjCZFru4)
