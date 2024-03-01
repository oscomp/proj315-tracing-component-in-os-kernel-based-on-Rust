# proj315-tracing-component-in-os-kernel-based-on-Rust
# 基于Rust的操作系统内核跟踪组件

### 项目描述

跟踪（tracing）技术是观测内核行为的一种重要工具，我们可以将其简单理解为内核进行特定动作的时候记下的一种具有特定格式的日志。比如当某个cpu发生了调度、某个线程被阻塞、进入/离开某个内核函数、系统调用开始/返回这些事件发生的时候，均需要将这些事件记录下来。比如，可以记录为这样的文本格式：

```
    kworker/4:1H-556   (  556) [004] .... 261193.377394: sched_switch: prev_comm=kworker/4:1H prev_pid=556 prev_prio=100 prev_state=S ==> next_comm=jbd2/sda45-8 next_pid=682 next_prio=120
    jbd2/sda45_8-682   (  682) [004] .... 261193.377407: sched_waking: comm=kworker/u16:9 pid=13730 prio=120 success=1 target_cpu=5
    jbd2/sda45_8-682   (  682) [004] .... 261193.377410: sched_wakeup: comm=kworker/u16:9 pid=13730 prio=120 target_cpu=005
          <idle>-0     (-----) [005] .... 261193.377412: cpu_idle: state=4294967295 cpu_id=5
          <idle>-0     (-----) [005] .... 261193.377413: sched_switch: prev_comm=swapper/5 prev_pid=0 prev_prio=120 prev_state=R ==> next_comm=kworker/u16:9 next_pid=13730 next_prio=120
   kworker/u16:9-13730 (13730) [005] .... 261193.377419: sched_switch: prev_comm=kworker/u16:9 prev_pid=13730 prev_prio=120 prev_state=S ==> next_comm=swapper/5 next_pid=0 next_prio=120
    jbd2/sda45_8-682   (  682) [004] .... 261193.377419: sched_switch: prev_comm=jbd2/sda45-8 prev_pid=682 prev_prio=120 prev_state=D ==> next_comm=swapper/4 next_pid=0 next_prio=120
          <idle>-0     (-----) [005] .... 261193.377420: cpu_idle: state=0 cpu_id=5
          <idle>-0     (-----) [004] .... 261193.377426: cpu_idle: state=0 cpu_id=4
```

我们可以收集系统某段时间的trace，从中就可以明确的看出系统此段时间的运行状态，借助类似[perfetto ui]([Perfetto UI](https://www.ui.perfetto.dev/))这样的开源工具还可以将抓到的trace可视化，如图所示：

![perfetto-example](https://github.com/oscomp/proj315-tracing-component-in-os-kernel-based-on-Rust/assets/160444530/149628e6-8b6b-4641-9499-fde7b9e29eb7)


上图可以非常明确的看出每个cpu上面跑的线程的切换顺序，除此之外还包含了很多其他信息。

跟踪技术不仅可以用来在生产环境中定位系统的性能瓶颈问题，由于其可视化的潜力，预期其在教学中也会有很大价值，使得操作系统学习者可以更加容易观测内核的行为。Android上已有这样的atrace工具，但是其实现与linux内核绑定，难以移植到其他OS中；同时，对于锁冲突这种场景并未提供足够的信息，仅有线程间的唤醒关系，导致定位哪把锁成为了性能瓶颈需结合代码分析，比较困难。

因此，我们的项目目标便是基于灵活的Rust语言编写一个独立可复用的操作系统内核跟踪组件，从而赋予开源社区中的RustOS或者其他OS初步的可观测能力。

### 项目价值

* 项目解决现有技术痛点，可以输出A类顶会（ATC、ASPLOS、OSDI等）论文。即使完成部分功能也可以作为毕业设计。
* 项目面向系统核心功能，有机会成为开源社区重量级组件，为成千上万的实际产品所应用。
* 导师可以长期辅导。

### 参考资料

[perfetto]([Perfetto - System profiling, app tracing and trace analysis](https://perfetto.dev/))

### 支持单位 

华为

### 所属赛道
2024全国大学生操作系统比赛的“OS功能挑战”赛道

### 参赛要求

- 以小组为单位参赛，最多三人一个小组，且小组成员是来自同一所高校的本科生或研究生（2024年春季学期或之后毕业的大一~大四的本科生或研究生）
- 如学生参加了多个项目，参赛学生选择一个自己参加的项目参与评奖
- 请遵循“2024全国大学生操作系统比赛”的章程和技术方案要求

### 赛题分类
系统调试/支撑库的设计

### 项目导师

- 任玉鑫 email: renyuxin1AThuawei.com

- 吴一凡 github id: wyfcyx   email：shinbokuowAT163.com

### 难度

中等

### 特征

- 熟悉操作系统/计算机组成原理等基本概念
- 熟悉Rust语言
- 能通过[开源操作系统训练营](https://github.com/learningos)的训练


### License

- GPL-3.0 
- Apache-2.0

## 预期目标

### 注意：下面的内容是建议内容，不要求必须全部完成。选择本项目的同学也可与导师联系，提出自己的新想法，如导师认可，可加入预期目标

### 任务1

调研perfetto ui支持的trace的格式(或者自己定义trace格式并重新开发某种可视化工具，但还是更推荐基于开源工具)，对于某个开源RustOS实现可视化其某段时间内系统的执行状态。系统的执行状态包括每个cpu上面跑的任务的变化情况、每个线程的状态变化、线程之间的唤醒关系、以及其他与内核执行有关的信息。

实现有以下要点：

1. 要有地方保存trace。可以直接在物理内存中分配一个ring buffer，trace在里面滚动写入。也可以直接写入文件系统的一个文件中。
2. 有开关接口可以控制在事件触发的时候是否保留trace。
3. 要能够取出trace并进行可视化。

### 任务2

将跟踪功能作为一个独立的内核组件使得该组件可以被多个RustOS甚至基于C的OS使用。

在接口设计方面：该组件不可避免会依赖OS的一些功能，要考虑如何忽略不同OS的实现差异，甚至能够被跨平台使用；对于该组件提供给OS的接口，则需要屏蔽该组件内部的实现细节，为OS提供高效易用的接口。注意零成本抽象，接口使用者不应为用不到的功能付出额外开销。

### 任务3

前端优化：基于Rust的元编程特性（如过程宏）为用户提供更加易于使用的接口。允许用户自定义一些trace格式。

### 任务4

后端优化：在记录trace的时候不以文本格式记录，而是压缩为某种二进制格式，这样相同的ring buffer或者文件大小能够记录更多的trace。当取出trace的时候再解码转换回文本格式；基于无锁方式实现，使得多核可以并行写入trace，让跟踪功能不会成为系统的瓶颈。

### 任务5

对功能进行拓展，展示更多信息帮助开发者分析是（内核中）哪把锁冲突了。

### 任务6

从性能开销、接口易用性、代码可拓展性等多个角度对于组件的整体实现进行评估。
