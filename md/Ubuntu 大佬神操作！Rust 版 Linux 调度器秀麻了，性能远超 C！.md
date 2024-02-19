> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/IyQ3nvg17JLDGh4Iwv9aXg)

> **来自公众号：51CTO 技术栈**
> 
> 整理丨诺亚

近日，一位 Linux 内核工程师出于兴趣用 Rust 编写了一个 Linux 调度器。          

这位来自 Ubuntu 制造商 Canonical 的工程师名叫 Andrea Righi。他在 X（推特）上发文谈到，他利用圣诞假期进行了这项实验。没想到这个只是 “出于好玩” 而进行的项目却带来了意外惊喜。          

初步结果显示：通过 sched_ext 实现并基于 eBPF 技术、能够在运行时加载的 Rust 调度器具有很大的潜力和希望。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6SlG4gLUichgR8iaYjPK3ZttOJch09cCxSSgq7E1eLAlSa4af2VQFice2g/640?wx_fmt=png)

**令人意外的结果：Rust 版超越默认版**          

“结果让我很惊讶。它不仅能够正常工作，而且在某些负载（例如游戏）下甚至可以**超越 Linux 内核默认的 EEVDF 调度器**。”          

Righi 表示，尽管仍处于原型阶段，但它成功完成了使命，即 “证明在用户空间实现运行的工作调度器是可行的，并且在某些特定条件下甚至可以超越 Linux 默认调度器的性能”。          

他还分享了一段视频：一个简单的电脑游戏 Terraria 正常运行，同时该机器正在后台进行内核编译。

[视频详情](javascript:;)

基于广大吃瓜群众的要求，Righi 将 Linux 默认调度器和 Rust 版调度器的内核构建时间和游戏性能进行了比对。

![](https://mmbiz.qpic.cn/mmbiz_jpg/zYdZKiaLibic67YwHszAGogRRcy7Yc15kMaVZaVGAp5KQyPHrAyTcHVdEc4NFJ2pGwRN1sThh4NmhYib4Fb8doq6jw/640?wx_fmt=jpeg)

截图：来自 X@arighi          

可以看到，其中：

*   linux sched：内核构建 280s，Terraria ~ 每秒 30 帧
    
*   rustland：内核构建 361s，Terraria ~ 每秒 60 帧
    

也就是说，切换到 Rust 版调度器后，游戏画面提升到了每秒 60 帧，是之前的两倍，而同期内核编译只是多花了约 30% 的构建时间。由此可见，“scx_rustland” 调度器在游戏中表现的确优于 Linux 内核默认调度器。       

Righi 已将相关代码托管在了 GitHub 上。感兴趣的朋友可以继续关注其未来的发展方向。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6ojD8pHUQOsKpMfyhXp67maIRcFLyAdv0lgYYV3OI2g2zwpRHL6ljYg/640?wx_fmt=png)

**内核调度器之争：Linux 需要更多的调度器吗？**

Linux 内核调度器负责执行一项关键任务，即分配给应用程序一段段的 CPU 时间，旨在确保每个应用程序都能公平地获得执行时间。

通常，这一目标是通过使用完全公平调度（Completely Fair Scheduler, CFS）算法来实现的。（注：CFS 算法致力于平衡各个应用程序对处理器资源的需求，以达到整体系统性能最优化的效果。）          

**尽管 Linux 的 “一刀切” 式调度器在过去很长一段时间内表现出色，但在如今分布式计算环境日益复杂、IT 环境异构化加剧的背景下，可能需要对其进行更新升级。**          

当传统的单一调度策略难以满足现实需求时，人们在寻求更多破局的路径，比如，通过引入如 BPF 这样的技术来实现更加灵活、可扩展且针对性更强的调度机制。          

简单说明一下 BPF。所谓 BPF，是 Berkeley Packet Filter 的缩写。BPF 提供了一种当内核或应用特定事件发生时候，执行一段代码的能力。BPF 采用了虚拟机指令规范，所以也可以看作一种虚拟机实现，使我们可以在**不修改内核源码和重新编译**的情况下，提供一种**扩展内核**的能力的方法。          

2013 年由技术大牛 Alexei Starovoitov 向 Linux 社区提交了重新实现 BPF 的内核补丁，相关工作于 2014 年正式并入 Linux 内核主线。此举将其扩展成了通用的执行引擎，称为 eBPF，其可以完成多种任务，包括用来创建先进的性能分析工具。          

LWN.Net 首席编辑 Jonathan Corbett 在今年 2 月评论道：“将 BPF 引入内核 CPU 调度器只是时间问题。”          

Corbett 解释说，基于 BPF 的调度器之所以有意义，有多个原因：它使得尝试新的调度方法变得更加容易。如今的系统远比过去几十年要复杂得多，因此需要更多领域特定和针对性的调度解决方案（例如针对游戏和网络应用的调度器）。此外，它还能为开发者提供一种针对其自身应用程序优化 CPU 使用方式的方法。     

而 Righi 开发的 scx_rustland 是对 sched_ext 这一实验性 Linux 内核特性的实现，sched_ext 允许运行可在内核中创建并加载的基于 BPF 的内核线程调度器。该特性由来自 Meta 和 Google 的一组工程师以及其他内核社区成员共同研发，并希望有一天能将其合并到 Linux 内核的核心代码库中。          

值得注意的是，**并非 Linux 社区中的每个人都支持动态调度这一理念**，其中包括 Linus Torvalds 本人。Linux 调度器维护者 Peter Zijlstra 在评论 sched_ext 的首次发布时提到：“我讨厌这一切”，并补充道，**鉴于替代调度器引入的复杂性，Torvalds 曾否决了之前关于此类调度器的所有尝试**。          

此外，AMD 和 Google 两家公司也提出了各自的替代调度器方案。这些举措表明，在如何优化 Linux 内核调度以适应更复杂异构环境的问题上，业内依旧各执一词。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQoOJQQprmIGVMDdTdhZl4ib6O3T8iahkmTbcEjAuV6rXuzmq4SSibXZ8Tqk3TfvrIP54wMx2PtEYzraw/640?wx_fmt=png)

**使用 Rust，真的会比 C 更有优势吗？**

近年来，在编程语言界，Rust 的存在感越来越强。不少公司、个人都开始对使用内存安全的 Rust 进行关键任务开发产生了浓厚兴趣，以取代可能意外引入安全漏洞的 C 语言。          

此前，我们也曾在《[Linux 诞生 32 年：“暴君”Linus 平和了](http://mp.weixin.qq.com/s?__biz=MjM5ODI5Njc2MA==&mid=2655910459&idx=2&sn=cb1eb93b222cf15b9242b295395b9a3a&chksm=bd75b7ec8a023efa62ca32630a511d6c4fbad3b5ff313ba23e178d8ca147fe7ffa512cc7c96f&scene=21#wechat_redirect)》一文中报道过，Torvalds 对于在内核中使用 Rust 持开放态度。他曾谈到，从明年开始会着手将 Rust 引入驱动程序、甚至是某些主要子系统，总之 “Rust 确实有成为内核重要部分的趋势”。          

因此，在 Linux 新闻网站 Phoronix 报道 Righi 的工作时，将关注点放在了 Rust 的使用上。让这篇文章引起讨论的焦点从调度器本身挪开，集中到了关于使用 Rust 而非 C 语言是否更有优势上面。          

有人质疑：“这个调度器到底有何与众不同，使其表现出了实质性的差异？是因为它尚未完成还是确实因为它更优秀？因为我不相信 Rust 本身会比 C 语言更好。究竟有什么是在 Rust 中能做而在 C 中做不到的呢？”          

有人更是直接指出，**调度器的设计始终涉及权衡取舍**。显然，这款为游戏优化的调度器牺牲了一些其他功能以换取更好的游戏性能。          

当然也有人对此持更为积极的态度。在 Hacker News 讨论中，有网友写道：“这篇新闻告诉我们的是，Rust 实现的调度器在这一领域可以与 C 语言实现相媲美。这一消息为我们解锁了更多选择，无论是 C 语言调度器还是 Rust 调度器，意味着 Linux 社区在处理各种工作负载时可以获得更好的体验。”          

而对于 Righi 本人，他在接受媒体采访时表示，Rust 提供了极大的灵活性，这使他能够迅速启动并推进这个项目。          

“我不能说 Rust 在性能提升方面贡献巨大，但它让我能够在几周内编写出这个调度器，同时实现和重用了优雅的高级抽象，并在我需要的时候能够深入到底层细节。”          

参考链接：

https://www.phoronix.com/news/Rust-Linux-Scheduler-Experiment

https://thenewstack.io/bpf-opens-a-door-to-linux-dynamic-scheduling-maybe-with-rust/

---END---