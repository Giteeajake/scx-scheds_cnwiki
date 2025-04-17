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
}}