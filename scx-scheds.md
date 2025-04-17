[[en:Scx-scheds]]

{{Pkg|scx-scheds}}是scheds-ext的程序实现，'''scheds-ext'''（Scheduler Extensions）是Linux的一个可拓展调度器框架，允许在不修改内核代码的情况下通过BPF/eBPF来实现自定义调度策略。'''[[zhwp:BPF|BPF]]'''（Berkeley Packet Filter）

===安装===
<code>sudo&#32;pacman&#32;-S&#32;scx-scheds&#32;bpf</code>

===部署===
scx-scheds安装后会提供两个service文件，分别为scx_loader.service和scx.service：scx.service是scx调度程序，scx_loader.service则是通过[[Wikipedia:D-Bus|D-Bus]]来实现scheds-ext框架加载器和管理器的实用程序。

{{警告|现在不再推荐使用<code>scx.service</code>，推荐使用scx_loader.service}}

需要先检查内核是否支持bpf，执行指令<code>sudo&#32;zcat&#32;/proc/config.gz&#32;|grep&#32;-i&#32;BPF</code>。通常情况下会输出这些内容：{{hc|head=sudo&#32;zcat&#32;/proc/config.gz&#32;|grep&#32;-i&#32;BPF|output=
CONFIG_BPF=y
CONFIG_HAVE_EBPF_JIT=y
CONFIG_ARCH_WANT_DEFAULT_BPF_JIT=y
# BPF subsystem
CONFIG_BPF_SYSCALL=y
CONFIG_BPF_JIT=y
CONFIG_BPF_JIT_ALWAYS_ON=y
CONFIG_BPF_JIT_DEFAULT_ON=y
CONFIG_BPF_UNPRIV_DEFAULT_OFF=y
# CONFIG_BPF_PRELOAD is not set
CONFIG_BPF_LSM=y
# end of BPF subsystem
CONFIG_CGROUP_BPF=y
CONFIG_IPV6_SEG6_BPF=y
CONFIG_NETFILTER_BPF_LINK=y
CONFIG_NETFILTER_XT_MATCH_BPF=m
CONFIG_NET_CLS_BPF=m
CONFIG_NET_ACT_BPF=m
CONFIG_BPF_STREAM_PARSER=y
CONFIG_LWTUNNEL_BPF=y
CONFIG_BPF_LIRC_MODE2=y
# HID-BPF support
CONFIG_HID_BPF=y
# end of HID-BPF support
CONFIG_LSM="landlock,lockdown,yama,integrity,bpf"
CONFIG_BPF_EVENTS=y
CONFIG_BPF_KPROBE_OVERRIDE=y
# CONFIG_TEST_BPF is not set
}}