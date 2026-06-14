## 配置

> 不要忘记永久禁用 zswap。Zswap 默认启用，与 zram 配合使用效果不佳。将 zswap.enabled=0 添加到内核参数中。https://linuxreviews.org/Zram#zram_and_zswap

因此经过更多的测试和观察，我有了一些非常有趣的发现。`DATA`确实是占用交换空间的**未压缩**`disksize`内存量。但乍一看，它非常具有欺骗性和令人困惑。当您设置 zram 并将其用作交换时，并不代表 zram 将为压缩数据消耗的内存总量。相反，它代表zram 将压缩的**未压缩**数据总量。因此，您可以创建一个大小为 2 GB 的 zram 设备，但实际上，当总压缩内存达到 500-1000 MB 左右时，zram 就会停止（当然取决于您的情况）。像`swapon -s`或 Gnome 的系统监视器这样的命令显示zram 设备的**未压缩**`DATA`数据大小，就像zramctl 一样。值得庆幸的是，实际上，zram 并没有真正用完报告的内存量。但这意味着在实践中，您实际上必须创建一个等于您拥有的 RAM + 50% 的 zram 磁盘大小才能真正利用它，而**不是**像 zram-config 错误地那样创建一个等于 RAM 大小一半的磁盘大小。但请继续阅读以了解更多信息。

以下是更深层次的背景：为什么我如此肯定？因为我也用 zswap 测试了这一点。我编译了一个自己的内核，其中我降低了 mm/vmscan.c 中的 file_prio 值，与 anon_prio 相比（在较新的 Linux 5.6 内核中，变量分别重命名为 fp 和 ap）。降低的 file_prio 值将使内核不再丢弃宝贵的缓存内存。默认情况下，即使设置为`vm.swappiness`100，内核也会丢弃大量缓存的 RAM 数据，无论是在待机内存中还是在活动程序中。当您真正想要使用 zram 时，默认配置对性能的影响在内存压力情况下是**极端的，因为那时您绝对\*\***希望**内核更频繁地交换很少使用**且\*\*高度可压缩的内存。有了更多的可用内存，您就有了更多的空间来存储缓存数据。然后缓存数据就不会以高得离谱的速度被丢弃，Linux 也不必反复重新读取某些已清除的程序文件缓存。在经典硬盘上测试时，您可以轻松验证性能影响。

回到我的 zswap 测试：使用我的自定义内核，一旦内存达到 50 - 70% 标记，zswap 就有足够的内存进行压缩。Gnome 的系统监视器立即显示页面分区的高交换数据使用率，但奇怪的是，根本没有硬盘分页！这当然是 zswap 的设计，它自己交换最近最少使用的内存。但有趣的是，系统无论如何都会报告**交换分区的**如此高的交换使用率，因此最终您会受到交换分区或交换文件大小的限制。即使所有内存都经过压缩，您也**必须**至少拥有未压缩数据的交换大小。因此，即使实际上 zswap 中 4 GB 的交换内存仅使用 1 - 2 GB，您的交换也需要具有未压缩数据大小的大小。zram 也是如此，但这里的内存至少没有实际保留。当然，除非您将 zswap 与动态增长的交换文件一起使用。

至于 zram，还有[一个非常有趣的细节](https://www.kernel.org/doc/Documentation/blockdev/zram.txt)支持了我的观察：

> **由于我们期望压缩率为 2:1，因此创建大于两倍内存大小**的 zram 没有什么意义 。请注意，不使用时，zram 会占用磁盘大小的约 0.1%，因此巨大的 zram 是一种浪费。

这意味着要有效使用 zram，您必须**至少**创建一个与已安装 RAM 大小相等的磁盘大小。由于压缩率高，我建议使用 GB 的 RAM + 50%，但上面的引用意味着如果超过 +100% 没有多大意义。此外，由于我们必须指定与未压缩数据大小匹配的磁盘大小，因此控制和预测实际内存使用情况要困难得多。从上面有用的官方来源，我们可以`TOTAL`使用以下命令限制实际内存使用量（等于 zramctl 的值）：。`echo 1G > /sys/block/zram0/mem_limit`但实际上，这样做会锁定机器。因为系统仍尝试交换到它，但 zram 施加了限制，并且机器因超高的 CPU 使用率而锁定。这种行为根本不是故意的，这加强了我对整个故事非常不稳定的印象。

总结一下：

- 您`disksize`在创建 zram 设备期间设置的基本上是一个虚拟磁盘大小，这并不**代表**真实的 RAM 使用情况。
- 您必须预测您的场景的实际 RAM 使用情况（压缩率），或者确保您永远不会创建过大的 zram 磁盘大小。在实践中，您当前的 RAM 大小 + 50% 应该几乎总是没问题的。
- 不幸的是，即使设置为 100，Linux 内核的默认配置也完全不适合 zram 压缩。`vm.swappiness`您需要制作自己的自定义内核才能真正**利用这个方便的功能，因为 Linux 会清除太多文件缓存，而不是通过\*\***更早\*\*地交换最易压缩的数据来释放内存。讽刺的是，一个有用的补丁[从未被接受](https://lore.kernel.org/patchwork/patch/685711/)来修复这种情况。
- 使用 zram 限制`echo 1G > /sys/block/zram0/mem_limit`会在压缩数据达到该阈值时锁定您的系统。您最好使用预测良好的 zram 磁盘大小来限制 zram 使用量，因为似乎没有其他限制选择。

## 参考

- https://chrisdown.name/2018/01/02/in-defence-of-swap.html
- https://linuxblog.io/linux-performance-almost-always-add-swap-part2-zram/
- https://wiki.archlinux.org/title/Improving_performance#RAM,_swap_and_OOM_handling
- https://github.com/sysstat/sysstat/
- https://lwn.net/Articles/454795/
- https://www.baeldung.com/linux/zram-zswap-zcache-comparison

---

- https://nvd.nist.gov/vuln/detail/CVE-2024-50064
