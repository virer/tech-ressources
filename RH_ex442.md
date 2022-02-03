EX 442 Tips and things to know for the exam

<pre>
Get page size:
$ getconf PAGESIZE

Get readahead and blocksize value:
$ blockdev --report

Get current I/O scheduler for a disk
$ cat /sys/block/sda/queue/scheduler
</pre>

# FS
  be able to put filesystem journal on a separated drive/partition for xfs and ext4

# Tools :
<pre>
  pcp / pcp-chart / pmcd
  tuna
  numactl / numastats :
    numactl --interleave all -- <your cmd>
  pmval / pmdumplog
  sar
  valgrind and cachegrind module
  perf
  bcc-tools
  virsh schedinfo
</pre>

# dnf :
<pre>
  /etc/dnf/dnf.conf:
      fastestmirror=1
 </pre>

# Tuned :
<pre>
   tuned-adm list
   
   hugepage configuration (using tuned or something else like hugepagesz= at grubline)
   
   [main]
   include=override_me
   
   [vm]
   transparent_hugepages=never
   
   [sysctl]
   dirty page management
   vm.nr_hugepages=
   net.ipv4.tcp_window_scaling
   be able to calculate the buffer size based on desired bandwidth and latency:
    net.ipv4.tcp_rmem / net.ipv4.tcp_wmem 
    net.ipv4.udp_rmem / net.ipv4.udp_wmem
    net.core.rmem_max / net.core.wmem_max  (this value must be equal to the biggest value of tcp_[r|w]mem or udp_[r|w]mem)
   vm.dirty_ratio
   vm.dirty_background_ratio
   vm.swappiness
   vm.overcommit_memory=1
   
   [disk]
   devices=sda,sdb
   readahead=4096 sectors
   elevator=noop/none
   
   [my_disk_2]
   type=disk
   devices=sdc
   readahead=8192 sectors
   disable_barriers=false
   elevator=deadline
   
   [my_ssd_disk]
   type=disk
   devices=sdd
   elevator=kyber
   
   [script]
   script=${i:PROFILE_DIR}/script.sh
</pre>   

# Systemd:
<pre>
  be able to create a "cron" task using "OnCalendar" and .timer unit files with systemd  
  systemd slice and user related limitation and configuration user-.slice etc...
    [Unit]
    [Slice]
    MemoryMax=1G
    MemoryAccounting=yes
    CPUAccounting=yes
    CPUQuota=200% # for 2 CPU max

  man systemd.resource-control
  systemd-cgls
  systemctl edit servicename.service
  systemctl set-property servicename.service MemoryMax=1G
  systemctl show servicename.service
  systemctl daemon-reload
</pre>

# systemtap:
<pre>
     stap
     stap-prep
     stap -p 4 -v -m ...
</pre>

# Networking
<pre>
   ethtool -k <ifname>
   ## Teaming
   nmcli con add type team con_name Team0 ifname Team0 team.runner [loadbalance | roundrobin] 
</pre>
