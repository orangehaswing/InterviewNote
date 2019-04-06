# shuffle基础

从这个例子的图中可以看出，每个 map function 会输出一组 key value pair, Shuffle 阶段需要从所有 map host 上把相同的 key 的 key value pair 组合在一起，组合后传给 reduce host, 作为输入进入 reduce function 里。

![img](https://pic2.zhimg.com/30be582c735f3cc83e840e63ba25a05d_b.jpg)



使用默认的 partitioner: HashPartitioner，它会把 key 放进一个 hash function 里，然后得到结果。如果两个 key 的 hashed result 一样，他们的 key value pairs 就被放到同一个 reduce function 里。我们也把分配到同一个 reduce function 里的 key value pairs 叫做一个 **reduce partition**.

## 在 Map 端的运作

Map function 的运行方式就是从 RecordReader 那边读出一个 input key value pair, 处理，然后把处理结果（通常也是 key value pair 形式）写到一个Hadoop maintained memory buffer 里，然后读取下一个 input key value pair.

Hadoop maintained memory buffer 里的 key value pair 按 key 值排序，并且按照 reduce partition 分到不同 partition 里（这就是 partitioner 被调用的时候）。一旦 memory buffer 满了，就会被 Hadoop 写到 file 里，这个过程叫 spill, 写出的 file 叫 spill file.

> 注意，这些 spill file 存在 map 所在 host 的 local disk 上，而不是我们之前介绍过的 HDFS.

随着 Map 不断运行，有可能有多个 spill file 被制造出来。当 Map 结束时，这些 spill file 会被 merge 起来——不是 merge 成一个 file，而是按 reduce partition 分成多个。

## 在 Reduce 端的运作

由于 Map tasks 有可能在不同时间结束，所以 reduce tasks 没必要等所有 map tasks 都结束才开始。事实上，每个 reduce task 有一些 threads 专门负责从 map host copy map output（默认是5个，可以通过 $mapred.reduce.parallel.copies 参数设置）；考虑到网络的延迟问题，并行处理可以在一定程度上提高效率。

通过前面的学习，我们知道 Hadoop JobTracker 会根据输入数据位置安排 map tasks，但 reduce tasks 是不知道这种安排的。那么当 reduce task 需要从map host copy map output 时，它怎么知道 map host 的位置呢（URL／IP）?

其实当 Map tasks 成功结束时，他们会通知负责的 tasktracker, 然后消息通过 jobtracker 的 heartbeat 传给 jobtracker. 这样，对于每一个 job, jobtracker 知道 map output 和 map tasks 的关联。Reducer 内部有一个 thread 负责定期向 jobtracker 询问 map output 的位置，直到 reducer 得到所有它需要处理的 map output 的位置。

Reducer 的另一个 thread 会把拷贝过来的 map output file merge 成更大的 file. 如果 map task 被 configure 成需要对 map output 进行压缩，那 reduce 还要对 map 结果进行解压缩。当一个 reduce task 所有的 map output 都被拷贝到一个它的 host上时，reduce 就要开始对他们排序了。

排序并不是一次把所有 file 都排序，而是分几轮。每轮过后产生一个结果，然后再对结果排序。最后一轮就不用产生排序结果了，而是直接向 reduce 提供输入。这时，用户提供的 reduce function 就可以被调用了。输入就是 map task 产生的 key value pairs.

## Hadoop Benefit

我们需要注意的是，Shuffle 阶段的功能是完完全全由 Hadoop framework 提供的，这里边没有任何用户的代码（即使我们有可能需要根据具体 Hadoop job 的特点配置一下这个阶段，但也非常方便）。

Shuffle 阶段应该说是整个 MapReduce Job 里需要处理问题最复杂，需要提高 performance 最多的地方。想象一下，公司里的 cluster 中可能有上百个 reducer 从上千个 mapper 那边高效率的拷贝处理数据，全靠 Hadoop 处理这个阶段，让我们可以轻轻松松就写写 map function 和 reduce function 就得到理想的效率，这是 Hadoop 广受欢迎的重要原因之一。











