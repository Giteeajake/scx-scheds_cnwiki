[[分类:系统管理]]
{{Pkg|scx-scheds}} 是 scheds-ext 的程序实现，'''scheds-ext'''（Scheduler Extensions）是 Linux 的一个可拓展调度器框架，允许在不修改内核代码的情况下通过 '''[[zhwp:BPF|BPF]]'''（Berkeley Packet Filter）或 eBPF 来实现自定义调度策略。

== 安装 ==

需要先检查内核是否支持 bpf，使用 [[root]] 权限执行指令：
{{bc|# zcat /proc/config.gz <nowiki>| </nowiki>grep -i BPF}}
通常情况下会输出这些内容：
{{hc|# zcat /proc/config.gz <nowiki>| </nowiki>grep -i BPF|output=CONFIG_BPF=y
CONFIG_HAVE_EBPF_JIT=y
CONFIG_ARCH_WANT_DEFAULT_BPF_JIT=y
.....
}}

如果得到如上结果，代表内核支持 BPF。
[[安装]] {{Pkg|scx-scheds}} 与 {{Pkg|bpf}}。

== 部署 ==

scx-scheds 安装后会提供两个 [[systemd]] 服务 {{ic|scx_loader.service}} 和 {{ic|scx.service}}：{{ic|scx.service}} 是 scx 调度程序，{{ic|scx_loader.service}} 则是通过 [[D-Bus]] 来实现 scheds-ext 框架加载器和管理器的实用程序。

{{警告|现在不再推荐使用 {{ic|scx.service}}，推荐使用 {{ic|scx_loader.service}} ，且不要同时运行，否则它们不会执行任何东西。}}
{{警告|在使用  scheds-ext  框架的任何一个调度器时强烈建议[[禁用]]并[[停止]]使用 {{AUR|ananicy-cpp-git}} 等相关的包，因为 {{AUR|anaicy-cpp-git}} 等相关的包会干扰系统优先级导致  scheds-ext  watchdog  超时，使调度程序被“杀死”。}}

[[启用]] scx_loader 服务。

{{Pkg|scx-scheds}} 在安装后会生成一个默认配置文件位于 {{ic|/etc/default/scx}} ，用于 {{ic|scx.service}}。

{{注意|{{ic|scx_loader.service}} 不会使用 {{ic|/etc/default/scx}} 配置文件。}}

=== 配置 scx服务 ===

在 {{ic|scx.service}} 中分为两个部分： {{ic|SCX_SCHEDULER}} 和 {{ic|SCX_FLAGS}} 。
{{hc|/etc/default/scx|# List of scx_schedulers: scx_bpfland scx_central scx_flash scx_lavd scx_layered scx_nest scx_qmap scx_rlfifo scx_rustland scx_rusty scx_simple scx_userland scx_p2dq scx_tickless
SCX_SCHEDULER<nowiki>=</nowiki>scx_bpfland

# Set custom flags for each scheduler, below is an example of how to use
#SCX_FLAGS<nowiki>=</nowiki>'-p -m performance'
}}
{{ic|SCX_SCHEDULER}}	为  scx  调度器，总共14个调度器，主要调度器有4个： {{ic|scx_bpfland}} ， {{ic|scx_rusty}} ， {{ic|scx_flash}} 和 {{ic|scx_lavd}} 。

{{ic|SCX_FLAGS}} 是调度器的启动参数，例如： {{bc|SCX_FLAGS<nowiki>=</nowiki>'-p -m performance'}}  {{ic|-p}} 参数表示 '''启用每颗CPU的任务优先级划分''' 。 {{ic|-m}} 表示使用模式， performance  代表  gaming  或  lowlatency  （低延迟）。

=== 配置 scx_loader服务 ===
{{注意|  scx_loader  服务的配置文件是不会自动生成的，需要用户手动配置。}}
scx_loader  的配置文件有两个：  {{ic|/etc/scx_loader.toml}} 和 {{ic|/etc/scx_loader/config.toml}} 。
文件结构为：{{hc|/etc/scx_loader/config.toml |output=default_sched = "scx_bpfland"
default_mode = "Auto"

[scheds.scx_bpfland]
auto_mode = ["-m","performance"]}}
<code>default_sched</code> 意思是默认使用的调度器，{{ic|scx}} 和 {{ic|scx_loader}} 的默认调度器都是 {{ic|scx_bpfland}} ，如果字段为空则什么都不启用。
<code>default_mode</code> 指默认启动模式，主要参数有： <code>"Auto"</code> ，<code>"LowLatency"</code> ，<code>"PowerSave"</code> ，<code>"Gaming"</code> 和 <code>"Server"</code> 。如果没有参数则为默认模式，即 <code>"Auto"</code> 。
{{ic|[scheds.scx_name]}} 指对调度器的自定义标志，把 {{ic|scx_name}} 修改为实际的调度程序。参数标志有： <code>auto_mode</code> , <code>gaming_mode</code> , <code>lowlatency_mode</code> , <code>powersave_mode</code> ，<code>server_mode</code> ；每个字段是一个字符串且每个字符串表示一个参数标志。如果没有参数标志则使用默认。

参考样本：{{hc|/etc/scx_loader.toml |output=default_sched = "scx_bpfland"
default_mode = "LowLatency"

[scheds.scx_bpfland]
auto_mode = []
gaming_mode = ["-m", "performance"]
lowlatency_mode = ["-k", "-s", "5000", "-l", "5000"]
powersave_mode = ["-m", "powersave"]
}}

=== 参数标签 ===
==== scx_bpfland ====
* Gaming Mode : <code>-m performance</code>
* Low Latency Mode : <code>-s 5000 -S 500 -l 5000</code>
* PowerSave Mode : <code>-m powersave</code>

==== scx_rusty ====
没有自定义参数标志，使用默认参数标志。

==== scx_lavd ====
* Gaming Mode : <code>--performance</code>
* Low Latency Mode : <code>--performance</code>
* AutoPower Mode : <code>--autopower</code>
* PowerSave Mode : <code>--powersave</code>

==== scx_flash ====
没有自定义参数标签，使用默认参数标签。

=== 启动和检查 ===
通过[[systemd]][[启动]]后，使用[[root]]权限进行检查。通过命令检查是否启用以及查看调度器：{{bc|# cat /sys/kernel/sched_ext/state /sys/kernel/sched_ext/*/ops 2>/dev/null}}

=== 回退模式 ===
如果配置文件中未定义特定参数标志， {{ic|scx_loader}} 会自动使用默认的参数标志。

== 参见 ==

* [https://wiki.cachyos.org/configuration/sched-ext/ CachyOS sched-ext教程]
* [https://github.com/sched-ext/scx/blob/main/rust/scx_loader/configuration.md sched-ext配置]
