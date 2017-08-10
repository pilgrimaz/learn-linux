#### XEN与KVM区别

##### xen

比如说在经典的虚拟机架构中，虚拟机软件运行于 Host System 之中，而 Guest System 运行于虚拟机软件之中。为了提高 Guest System 的运行速度，虚拟机软件一般会在 Host System 中使用内核模块开一个洞，将 Guest System 的运行指令直接映射到物理硬件上。但是在 Xen 中，则根本没有 Host System 的概念，传说它所有的虚拟机都直接运行于硬件之上，虚拟机运行的效率非常的高，虚拟机之间的隔离性非常的好。

　　当然，传说只是传说。我刚开始也是很纳闷，怎么可能让所有的虚拟机都直接运行于硬件之上。后来我终于知道，这只是一个噱头。虚拟机和硬件之间，还是有一个管理层的，那就是 Xen Hypervisor。当然 Xen Hypervisor 的功能毕竟是有限的，怎么样它也比不上一个操作系统，因此，在 Xen Hypervisor 上运行的虚拟机中，有一个虚拟机是具有特权的，它称之为 Domain 0，而其它的虚拟机都称之为 Domain U。

　　Xen的架构如下图：

![img](http://images.cnitblog.com/blog2015/16576/201503/072047590864636.png)

　　从图中可以看出，Xen 虚拟机架构中没有 Host System，在硬件层之上是薄薄的一层 Xen Hypervisor，在这之上就是各个虚拟机了，没有 Host System，只有 Domain 0，而 Guest System 都是 Domain U，不管是 Domain 0 还是 Domain U，都是虚拟机，都是被虚拟机软件管理的对象。

　　既然 Domain 0 也是一个虚拟机，也是被管理的对象，所以可以给它分配很少的资源，然后将其余的资源公平地分配到其它的 Domain。但是很奇怪的是，所有的虚拟机管理软件其实都是运行在这个 Domain 0 中的。同时，如果要连接到其它 Guest System 的控制台，而又不是使用远程桌面（VNC）的话，这些控制台也是显示在 Domian 0 中的。所以说，这是一个奇异的架构，是一个让人很不容易理解的架构。

　　这种架构桌面用户不喜欢，因为 Host System 变成了 Domain 0，本来应该掌控所有资源的主操作系统变成了一个受管理的虚拟机，本来用来打游戏、编程、聊天的主战场受到限制了，可能不能完全发挥硬件的性能了，还有可能运行不稳定了，自然会心里不爽。（Domain 0确实不能安装专用显卡驱动，确实会运行不稳定，这个后面会讲。）但是企业级用户喜欢，因为所有的 Domain 都是虚拟机，所以可以更加公平地分配资源，而且由于 Domain U 不再是运行于 Domian 0 里面的软件，而是和 Domain 0 平级的系统，这样即使 Domain 0 崩溃了，也不会影响到正在运行的 Domain U。（真的不会有丝毫影响吗？我表示怀疑。）

KVM

KVM是集成到[Linux](http://lib.csdn.net/base/linux)内核的Hypervisor，是X86[架构](http://lib.csdn.net/base/architecture)且硬件支持虚拟化技术（IntelVT或AMD-V）的[linux](http://lib.csdn.net/base/linux)的全虚拟化解决方案。它是Linux的一个很小的模块，利用Linux做大量的事，如任务调度、内存管理与硬件设备交互等。准确来说，KVM是Linuxkernel的一个模块。可以用命令modprobe去加载KVM模块。加载了模块后，才能进一步通过其他工具创建虚拟机。但仅有KVM模块是远远不够的，因为用户无法直接控制内核模块去作事情,你还必须有一个运行在用户空间的工具才行。这个用户空间的工具，kvm开发者选择了已经成型的开源虚拟化软件QEMU。说起来QEMU也是一个虚拟化软件。它的特点是可虚拟不同的CPU。比如说在x86的CPU上可虚拟一个Power的CPU，并可利用它编译出可运行在Power上的程序。KVM使用了QEMU的一部分，并稍加改造，就成了可控制KVM的用户空间工具了。所以你会看到，官方提供的KVM下载有两大部分(qemu和kvm)三个文件(KVM模块、QEMU工具以及二者的合集)。也就是说，你可以只升级KVM模块，也可以只升级QEMU工具。这就是KVM和QEMU的关系。