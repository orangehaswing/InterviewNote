# 操作系统原理-进程调度策略

## 多任务

### 并发和并行

Linux作为一个多任务操作系统，必须支持程序的并发执行。

### 分类

1. 非抢占式多任务

   除非任务自己结束，否则将会一直执行。

2. 抢占式多任务（Linux）

   这种情况下，由调度程序来决定什么时候停止一个进程的运行，这个强制的挂起动作即为**“抢占”**。采用抢占式多任务的基础是使用**时间片轮转**机制来为每个进程分配可以运行的时间单位。

## 进程调度

不同环境的调度算法目标不同，因此需要针对不同环境来讨论调度算法。

### 1. 批处理系统

批处理系统没有太多的用户操作，在该系统中，调度算法目标是保证吞吐量和周转时间（从提交到终止的时间）。

#### 1.1 先来先服务

先来先服务 first-come first-serverd（FCFS）

按照请求的顺序进行调度。

有利于长作业，但不利于短作业，因为短作业必须一直等待前面的长作业执行完毕才能执行，而长作业又需要执行很长时间，造成了短作业等待时间过长。

#### 1.2 短作业优先

短作业优先 shortest job first（SJF）

按估计运行时间最短的顺序进行调度。

长作业有可能会饿死，处于一直等待短作业执行完毕的状态。因为如果一直有短作业到来，那么长作业永远得不到调度。

#### 1.3 最短剩余时间优先

最短剩余时间优先 shortest remaining time next（SRTN）

按估计剩余时间最短的顺序进行调度。

### 2. 交互式系统

交互式系统有大量的用户交互操作，在该系统中调度算法的目标是快速地进行响应。

#### 2.1 时间片轮转

将所有就绪进程按 FCFS （先来先服务） 的原则排成一个队列，每次调度时，把 CPU 时间分配给队首进程，该进程可以执行一个时间片。当时间片用完时，由计时器发出时钟中断，调度程序便停止该进程的执行，并将它送往就绪队列的末尾，同时继续把 CPU 时间分配给队首的进程。

时间片轮转算法的效率和时间片的大小有很大关系。因为进程切换都要保存进程的信息并且载入新进程的信息，如果时间片太小，会导致进程切换得太频繁，在进程切换上就会花过多时间。

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/assets/8c662999-c16c-481c-9f40-1fdba5bc9167.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/assets/8c662999-c16c-481c-9f40-1fdba5bc9167.png)

#### 2.2 优先级调度

为每个进程分配一个优先级，按优先级进行调度。

为了防止低优先级的进程永远等不到调度，可以随着时间的推移增加等待进程的优先级。

#### 2.3 多级反馈队列

如果一个进程需要执行 100 个时间片，如果采用时间片轮转调度算法，那么需要交换 100 次。

多级队列是为这种需要连续执行多个时间片的进程考虑，它设置了多个队列，每个队列时间片大小都不同，例如 1,2,4,8,..。进程在第一个队列没执行完，就会被移到下一个队列。这种方式下，之前的进程只需要交换 7 次。

每个队列优先权也不同，最上面的优先权最高。因此只有上一个队列没有进程在排队，才能调度当前队列上的进程。

可以将这种调度算法看成是**时间片轮转调度算法和优先级调度算法**的结合。

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/assets/1-140629155KQ11.jpg)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/assets/1-140629155KQ11.jpg)

### 3. 实时系统

实时系统要求一个请求在一个确定时间内得到响应。

分为**硬实时和软实时**，前者必须满足绝对的截止时间，后者可以容忍一定的超时。

## 策略

### I/O消耗型和处理器消耗型

I/O消耗型进程是指那些大部分时间都在等待I/O操作的进程，处理器耗费型的进程则是指把大多数时间用于执行代码的进程，除非被抢占，他们一般都一直在运行。

为了保证交互式应用和桌面系统的性能，一般Linux更倾向于优先调度I/O消耗型进程。

### 进程优先级

Linux采用了两种不同的优先级范围。

1. 使用nice值：越大的nice值意味着更低的优先级。 (-19 ~ 20之间)

2. 实时优先级：可配置，越高意味着进程优先级越高。

   任何实时的进程优先级都高于普通的进程，因此上面的两种优先级范围处于互不相交的范畴。

3. 时间片：Linux中并不是以固定的时间值(如10ms)来分配时间片的，而是将处理器的使用比作为“时间片”划分给进程。这样，进程所获得的实际CPU时间就和系统的负载密切相关。

Linux中的抢占时机取决于新的可运行进程消耗了多少处理器使用比，如果消耗的使用比当前进程小，则立刻投入运行，否则将推迟其运行。

**举例**

现在我们来看一个简单的例子，假设我们的系统只有两个进程在运行，一个是文本编辑器（I/O消耗型），另一个是视频解码器（处理器消耗型）。

理想的情况下，文本编辑器应该得到更多的处理器时间，至少当它需要处理器时，处理器应该立刻被分配给它（这样才能完成用户的交互），这也就意味着当文本编辑器被唤醒的时候，它应该抢占视频解码程序。

按照普通的情况，OS应该分配给文本编辑器更大的优先级和更多的时间片，但在Linux中，这两个进程都是普通进程，他们具有相同的nice值，因此它们将得到相同的处理器使用比（50%）。

但实际的运行过程中会发生什么呢？CFS将能够注意到，文本编辑器使用的处理器时间比分配给它的要少得多（因为大多时间在等待I/O），这种情况下，要实现所有进程“公平”地分享处理器，就会让文本编辑器在需要运行时立刻抢占视频解码器（每次都是如此）。

## Linux调度算法

### 调度器类

Linux的调度器是以模块的方式提供的，这样使得不同类型的进程按照自己的需要来选择不同的调度算法。

上面说讲到的CFS算法就是一个针对普通进程的调度器类，基础的调度器会按照优先级顺序遍历调度类，拥有一个可执行进程的最高优先级的调度器类胜出，由它来选择下一个要执行的进程。

#### Unix中的进程调度

存在的问题：

1. nice值必须映射到处理器的绝对时间上去，这意味着同样是瓜分100ms的两个同样优先级的进程，发生上下文切换的次数并不相同，可能会差别很大。优先级越低的进程分到的时间片单位越小，但是实际上他们往往是需要进行大量后台计算的，这样很不合理。
2. 相对的nice值引发的问题：两个nice值不同但差值相同的进程，分到的时间片的大小是受到其nice值大小影响的：比如nice值18和19的两个进程分到的时间片是10ms和5ms,nice值为0和1的两个进程分到的却是100ms和95ms,这样的映射并不合理。
3. 如果要进行nice值到时间片的映射，我们必须能够拥有一个可以测量的“绝对时间片”（这牵扯到定时器和节拍器的相关概念）。实际上，时间片是会随着定时器的节拍而改变的，同样的nice值最终映射到处理器时间时可能会存在差异。
4. 为了能够更快的唤醒进程，需要对新的要唤醒的进程提升优先级，但是这可能会打破“公平性”。

为了解决上述的问题，CFS对时间片的分配方式进行了根本性的重新设计，摒弃了时间片，用处理器使用比重来代替它。

### 完全公平调度器（CFS）

出发点：进程调度的效果应该如同系统具备一个理想的多任务处理器—我们可以给任何进程调度无限小的时间周期，所以在任何可测量范围内，可以给n个进程桐乡多的运行时间。

举个例子来区分Unix调度和CFS：有两个运行的优先级相同的进程，在Unix中可能是每个各执行5ms，执行期间完全占用处理器，但在“理想情况”下，应该是，能够在10ms内同时运行两个进程，每个占用处理器一半的能力。

CFS的做法是：在所有可运行进程的总数上计算出一个进程应该运行的时间，nice值不再作为时间片分配的标准，而是用于处理计算获得的处理器使用权重。

接下来我们考虑调度周期，理论上，调度周期越小，就越接近“完美调度”，但实际上这必然会带来严重的上下文切换消耗。在CFS中，为能够实现的最小调度周期设定了一个近似值目标，称为“目标延迟”，于此同时，为了避免不可接受的上下文切换消耗，为每个进程所能获得的时间片大小设置了一个底线—最小粒度（通常为1ms）。

在每个进程的平均运行时间大于最小粒度的情况下，CFS无疑是公平的，nice值用于计算一个进程在当前这个最小调度周期中所应获得的处理器时间占比，这样就算nice值不同，只要差值相同，总是能得到相同的时间片。我们假设一个最小调度周期为20ms，两个进程的nice值差值为5：

- 两进程的nice值分别为0和5，后者获得的时间片是前者的1/3，因此最终分别获得15ms和5ms
- 两进程的nice值分别为10和15，后者获得的时间片是前者的1/3，最终结果也是15ms和5ms

关于上面这个推论，可能有些难以理解，所以我们深入一下，看看在底层nice差值究竟是如何影响到处理区占比的。

首先，在底层，在实际计算一个进程的处理器占比之前，内核会先把nice值转换为一个权重值weight，这个转换的公式：

weight = 1024/(1.25^nice)

举个例子，默认nice值的进程得到的权重就是1024/(1.25^0) = 1024/1 = 1024。这个转换公式保证了我们可以得到非负的权重值，并且nice对权重的影响是在指数上的。

现在假设我们的可运行进程队列中有n个进程，他们的权重和`w(1)+w(2)+...+w(n)`记为`w(queue)`，那么任意一个进程i最终得到的处理器占比将是`w(i)/w(queue)`。

接着，我们不难推导出，任意两个进程i和j所分配的到的处理器占比的比例应该是`w(i)/w(j)`，经过简单的数学推导就可以得到最后的结果：`1.25^(nice(i)-nice(j))`，这意味着只要两个nice值的差值相同，两个进程所获得处理器占比永远是相同的比例，从而解决了上面的第3点问题。

**总结** ：任何进程所获得的处理器时间是**由它自己和所有其他可运行进程nice值的相对差值决定的**，因此我们可以说，CFS至少保证了给每个进程公平的处理器占用比，算是一种近乎完美的多任务调度方式了。

## CFS的实现

### 时间记账

所有的调度器都必须对进程的运行时间记账，换句话说就是要知道当前调度周期内，进程还剩下多少个时间片可用（这将会是抢占的一个重要标准）

#### 1. 调度器实体结构

CFS中用于记录进程运行时间的数据结构为调度实体

```
struct sched_entity {
	/* 用于进行调度均衡的相关变量，主要跟红黑树有关 */
	struct load_weight		load; // 权重，跟优先级有关
	unsigned long			runnable_weight; // 在所有可运行进程中所占的权重
	struct rb_node			run_node; // 红黑树的节点
	struct list_head		group_node; // 所在进程组
	unsigned int			on_rq; // 标记是否处于红黑树运行队列中

	u64				exec_start; // 进程开始执行的时间
	u64				sum_exec_runtime; // 进程总运行时间
	u64				vruntime; // 虚拟运行时间，下面会给出详细解释
	u64				prev_sum_exec_runtime; // 进程在切换CPU时的sum_exec_runtime，简单说就是上个调度周期中运行的总时间

	u64				nr_migrations;

	struct sched_statistics		statistics;
	
	// 以下省略了一些在特定宏条件下才会启用的变量
}
```

#### 2. 虚拟实时 (vruntime)

现在我们来谈谈上面结构体中的vruntime变量所表示的意义。我们称它为“虚拟运行时间”，该运行时间的计算是经过了所有可运行进程总数的标准化（简单说就是加权的）。它以ns为单位，与定时器节拍不再相关。

可以认为这是CFS为了能够实现理想多任务处理而不得不虚拟的一个新的时钟，具体地讲，一个进程的vruntime会随着运行时间的增加而增加，但这个增加的速度由它所占的权重`load`来决定。

结果就是权重越高，增长越慢：所得到的调度时间也就越小 — CFS用它来记录一个程序到底运行了多长时间以及还应该运行多久。

记账功能的实现源码

```
/*
 * Update the current task's runtime statistics.
 */
static void update_curr(struct cfs_rq *cfs_rq)
{
	struct sched_entity *curr = cfs_rq->curr;
	u64 now = rq_clock_task(rq_of(cfs_rq));
	u64 delta_exec;

	if (unlikely(!curr))
		return;
	
	// 获得从最后一次修改负载后当前任务所占用的运行总时间
	delta_exec = now - curr->exec_start;
	if (unlikely((s64)delta_exec <= 0))
		return;
		
	// 更新执行开始时间
	curr->exec_start = now;

	schedstat_set(curr->statistics.exec_max,
		      max(delta_exec, curr->statistics.exec_max));

	curr->sum_exec_runtime += delta_exec;
	schedstat_add(cfs_rq->exec_clock, delta_exec);

	// 计算虚拟时间，具体的转换算法写在clac_delta_fair函数中
	curr->vruntime += calc_delta_fair(delta_exec, curr);
	update_min_vruntime(cfs_rq);

	if (entity_is_task(curr)) {
		struct task_struct *curtask = task_of(curr);

		trace_sched_stat_runtime(curtask, delta_exec, curr->vruntime);
		cgroup_account_cputime(curtask, delta_exec);
		account_group_exec_runtime(curtask, delta_exec);
	}

	account_cfs_rq_runtime(cfs_rq, delta_exec);
}
```

该函数计算了当前进程的执行时间，将其存放在`delta_exec`变量中，然后使用`clac_delta_fair`函数计算对应的虚拟运行时间，并更新`vruntime`值。

这个函数是由系统定时器周期性调用的（无论进程的状态是什么），因此vruntime可以准确地测量给定进程的运行时间，并以此为依据推断出下一个要运行的进程是什么。

### 进程选择

这里是调度的核心部分，用一句话来梗概CFS算法的核心就是**选择具有最小vruntime的进程**作为下一个需要调度的进程。

为了实现选择，当然要维护一个可运行的进程队列，CFS使用了**红黑树**来组织这个队列。

记住：红黑树是一种**自平衡二叉树**，再简单一点，它是一种以树节点方式储存数据的结构，每个节点对应了一个键值，利用这个键值可以快速索引树上的数据，并且它可以按照一定的规则自动调整每个节点的位置，使得通过键值检索到对应节点的速度和整个树节点的规模呈指数比关系。

#### 1. 找到下一个任务节点

先假设一个红黑树储存了系统中所有的可运行进程，节点的键值就是它们的vruntime，CFS现在要找到下一个需要调度的进程，那么就是要找到这棵红黑树上键值最小的那个节点：就是最左叶子节点。

实现此过程的源码如下:

```
static struct sched_entity *
pick_next_entity(struct cfs_rq *cfs_rq, struct sched_entity *curr)
{
	struct sched_entity *left = __pick_first_entity(cfs_rq);
	struct sched_entity *se;

	/*
	 * If curr is set we have to see if its left of the leftmost entity
	 * still in the tree, provided there was anything in the tree at all.
	 */
	if (!left || (curr && entity_before(curr, left)))
		left = curr;

	se = left; /* ideally we run the leftmost entity */

	/*
	 * 下面的过程主要针对一些特殊情况，我们在此不做讨论
	 */
	if (cfs_rq->skip == se) {
		struct sched_entity *second;

		if (se == curr) {
			second = __pick_first_entity(cfs_rq);
		} else {
			second = __pick_next_entity(se);
			if (!second || (curr && entity_before(curr, second)))
				second = curr;
		}

		if (second && wakeup_preempt_entity(second, left) < 1)
			se = second;
	}

	if (cfs_rq->last && wakeup_preempt_entity(cfs_rq->last, left) < 1)
		se = cfs_rq->last;

	if (cfs_rq->next && wakeup_preempt_entity(cfs_rq->next, left) < 1)
		se = cfs_rq->next;

	clear_buddies(cfs_rq, se);

	return se;
}
```

#### 2. 向队列中加入新的进程

向可运行队列中插入一个新的节点，意味着有一个新的进程状态转换为可运行，这会发生在两种情况下：

1. 进程由阻塞态被唤醒，
2. fork产生新的进程时。

将其加入队列的过程本质上来说就是红黑树插入新节点的过程：

```
static void
enqueue_entity(struct cfs_rq *cfs_rq, struct sched_entity *se, int flags)
{
	bool renorm = !(flags & ENQUEUE_WAKEUP) || (flags & ENQUEUE_MIGRATED);
	bool curr = cfs_rq->curr == se;

	/*
	 * 如果要加入的进程就是当前正在运行的进程，重新规范化vruntime
	 * 然后更新当前任务的运行时统计数据
	 */
	if (renorm && curr)
		se->vruntime += cfs_rq->min_vruntime;

	update_curr(cfs_rq);

	/*
	 * Otherwise, renormalise after, such that we're placed at the current
	 * moment in time, instead of some random moment in the past. Being
	 * placed in the past could significantly boost this task to the
	 * fairness detriment of existing tasks.
	 */
	if (renorm && !curr)
		se->vruntime += cfs_rq->min_vruntime;

	/*
	 * 更新对应调度器实体的各种记录值
	 */
	 
	update_load_avg(cfs_rq, se, UPDATE_TG | DO_ATTACH);
	update_cfs_group(se);
	enqueue_runnable_load_avg(cfs_rq, se);
	account_entity_enqueue(cfs_rq, se);

	if (flags & ENQUEUE_WAKEUP)
		place_entity(cfs_rq, se, 0);

	check_schedstat_required();
	update_stats_enqueue(cfs_rq, se, flags);
	check_spread(cfs_rq, se);
	if (!curr)
		__enqueue_entity(cfs_rq, se); // 真正的插入过程
	se->on_rq = 1;

	if (cfs_rq->nr_running == 1) {
		list_add_leaf_cfs_rq(cfs_rq);
		check_enqueue_throttle(cfs_rq);
	}
}
```

上面的函数主要用来更新运行时间和各类统计数据，然后调用`__enqueue_entity()`来把数据真正插入红黑树中：

```
static void __enqueue_entity(struct cfs_rq *cfs_rq, struct sched_entity *se)
{
	struct rb_node **link = &cfs_rq->tasks_timeline.rb_root.rb_node;
	struct rb_node *parent = NULL;
	struct sched_entity *entry;
	bool leftmost = true;

	/*
	 * 在红黑树中搜索合适的位置
	 */
	while (*link) {
		parent = *link;
		entry = rb_entry(parent, struct sched_entity, run_node);
		/*
		 * 具有相同键值的节点会被放在一起
		 */
		if (entity_before(se, entry)) {
			link = &parent->rb_left;
		} else {
			link = &parent->rb_right;
			leftmost = false;
		}
	}

	rb_link_node(&se->run_node, parent, link);
	rb_insert_color_cached(&se->run_node,
			       &cfs_rq->tasks_timeline, leftmost);
}
```

while()循环是遍历树以寻找匹配键值的过程，也就是搜索一颗平衡树的过程。找到后我们对要插入位置的父节点执行`rb_link_node()`来将节点插入其中，然后更新红黑树的自平衡相关属性。

#### 3. 从队列中移除进程

从队列中删除一个节点有两种可能：一是进程执行完毕退出，而是进程受到了阻塞。

```
static void
dequeue_entity(struct cfs_rq *cfs_rq, struct sched_entity *se, int flags)
{
	/*
	 * 更新“当前进程”的运行统计数据
	 */
	update_curr(cfs_rq);

	/*
	 * When dequeuing a sched_entity, we must:
	 *   - Update loads to have both entity and cfs_rq synced with now.
	 *   - Substract its load from the cfs_rq->runnable_avg.
	 *   - Substract its previous weight from cfs_rq->load.weight.
	 *   - For group entity, update its weight to reflect the new share
	 *     of its group cfs_rq.
	 */
	update_load_avg(cfs_rq, se, UPDATE_TG);
	dequeue_runnable_load_avg(cfs_rq, se);

	update_stats_dequeue(cfs_rq, se, flags);

	clear_buddies(cfs_rq, se);

	if (se != cfs_rq->curr)
		__dequeue_entity(cfs_rq, se);
	se->on_rq = 0;
	account_entity_dequeue(cfs_rq, se);

	/*
	 * 重新规范化vruntime
	 */
	if (!(flags & DEQUEUE_SLEEP))
		se->vruntime -= cfs_rq->min_vruntime;

	/* return excess runtime on last dequeue */
	return_cfs_rq_runtime(cfs_rq);

	update_cfs_group(se);

	/*
	 * Now advance min_vruntime if @se was the entity holding it back,
	 * except when: DEQUEUE_SAVE && !DEQUEUE_MOVE, in this case we'll be
	 * put back on, and if we advance min_vruntime, we'll be placed back
	 * further than we started -- ie. we'll be penalized.
	 */
	if ((flags & (DEQUEUE_SAVE | DEQUEUE_MOVE)) == DEQUEUE_SAVE)
		update_min_vruntime(cfs_rq);
}
```

和插入一样，实际对树节点操作的工作由`__dequeue_entity()`实现：

```
static void __dequeue_entity(struct cfs_rq *cfs_rq, struct sched_entity *se)
{
	rb_erase_cached(&se->run_node, &cfs_rq->tasks_timeline);
}
```

可以看到删除一个节点要比插入简单的多，这得益于红黑树本身实现的`rb_erase()`函数。

### 调度器入口

每当要发生进程的调度时，是有一个统一的入口，从该入口选择真正需要调用的调度类。

这个入口是内核中一个名为`schedule()`的函数，它会找到一个最高优先级的调度类，这个调度类拥有自己的可运行队列，然后向其询问下一个要运行的进程是谁。

这个函数中唯一重要的事情是执行了`pick_next_task()`，它以优先级为顺序，依次检查每一个调度类，并且从最高优先级的调度类中选择最高优先级的进程。

```
static inline struct task_struct *
pick_next_task(struct rq *rq, struct task_struct *prev, struct rq_flags *rf)
{
	const struct sched_class *class;
	struct task_struct *p;

	/*
	 * 优化：如果当前所有要调度的进程都是普通进程，那么就直接采用普通进程的调度类（CFS）
	 */
	if (likely((prev->sched_class == &idle_sched_class ||
		    prev->sched_class == &fair_sched_class) &&
		   rq->nr_running == rq->cfs.h_nr_running)) {

		p = fair_sched_class.pick_next_task(rq, prev, rf);
		if (unlikely(p == RETRY_TASK))
			goto again;

		/* Assumes fair_sched_class->next == idle_sched_class */
		if (unlikely(!p))
			p = idle_sched_class.pick_next_task(rq, prev, rf);

		return p;
	}

// 遍历调度类
again:
	for_each_class(class) {
		p = class->pick_next_task(rq, prev, rf);
		if (p) {
			if (unlikely(p == RETRY_TASK))
				goto again;
			return p;
		}
	}

	/* The idle class should always have a runnable task: */
	BUG();
}
```

每个调度类都实现了`pick_next_task()`方法，它会返回下一个可运行进程的指针，没有则返回NULL。调度器入口从第一个返回非NULL的类中选择下一个可运行进程。

### 睡眠和唤醒

睡眠和唤醒的流程在linux中是这样的：

- 睡眠：进程将自己标记成休眠状态，然后从可执行红黑树中移除，放入等待队列，然后调用`schedule()`选择和执行一个其他进程。
- 唤醒：进程被设置为可执行状态，然后从等待队列移到可执行红黑树中去。

休眠在Linux中有两种状态，一种会忽略信号，一种则会在收到信号的时候被唤醒并响应。不过这两种状态的进程是处于同一个等待队列上的。

#### 1.等待队列

和可运行队列的复杂结构不同，等待队列在linux中的实现只是一个简单的链表。

内核使用`wait_queue_head_t`结构来表示一个等待队列，它其实就是一个链表的头节点，但是加入了一个自旋锁来保持一致性（等待队列在中断时可以被随时修改）

```
struct wait_queue_head {
	spinlock_t		lock;
	struct list_head	head;
};
typedef struct wait_queue_head wait_queue_head_t;
```

而休眠的过程需要进程自己把自己加入到一个等待队列中，这可以使用内核所提供的、推荐的函数来实现。

一个可能的流程如下：

1. 调用宏`DEFINE_WAIT()`创建一个等待队列的项（链表的节点）
2. 调用`add_wait_queue()`把自己加到队列中去。该队列会在进程等待的条件满足时唤醒它，当然唤醒的具体操作需要进程自己定义好（你可以理解为一个回调）
3. 调用`prepare_to_wait()`方法把自己的状态变更为上面说到的两种休眠状态中的其中一种。

下面是上述提到的方法的源码

```
void add_wait_queue(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry)
{
	unsigned long flags;

	wq_entry->flags &= ~WQ_FLAG_EXCLUSIVE;
	spin_lock_irqsave(&wq_head->lock, flags);
	__add_wait_queue(wq_head, wq_entry);
	spin_unlock_irqrestore(&wq_head->lock, flags);
}

static inline void __add_wait_queue(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry)
{
	list_add(&wq_entry->entry, &wq_head->head);
}
```

```
void
prepare_to_wait(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry, int state)
{
	unsigned long flags;

	wq_entry->flags &= ~WQ_FLAG_EXCLUSIVE;
	spin_lock_irqsave(&wq_head->lock, flags);
	if (list_empty(&wq_entry->entry))
		__add_wait_queue(wq_head, wq_entry);
	// 标记自己的进程状态
	set_current_state(state);
	spin_unlock_irqrestore(&wq_head->lock, flags);
}
```

#### 2.唤醒

唤醒操作主要通过`wake_up()`实现，它会唤醒指定等待队列上的所有进程。内部由`try_to_wake_up()`函数将对应的进程标记为`TASK_RUNNING`状态，接着调用`enqueue_task()`将进程加入红黑树中。

`wake_up()`系函数由宏定义，一般具体内部由下面这个函数实现：

```
/*
 * The core wakeup function. Non-exclusive wakeups (nr_exclusive == 0) just
 * wake everything up. If it's an exclusive wakeup (nr_exclusive == small +ve
 * number) then we wake all the non-exclusive tasks and one exclusive task.
 *
 * There are circumstances in which we can try to wake a task which has already
 * started to run but is not in state TASK_RUNNING. try_to_wake_up() returns
 * zero in this (rare) case, and we handle it by continuing to scan the queue.
 */
static int __wake_up_common(struct wait_queue_head *wq_head, unsigned int mode,
			int nr_exclusive, int wake_flags, void *key,
			wait_queue_entry_t *bookmark)
{
	wait_queue_entry_t *curr, *next;
	int cnt = 0;

	if (bookmark && (bookmark->flags & WQ_FLAG_BOOKMARK)) {
		curr = list_next_entry(bookmark, entry);

		list_del(&bookmark->entry);
		bookmark->flags = 0;
	} else
		curr = list_first_entry(&wq_head->head, wait_queue_entry_t, entry);

	if (&curr->entry == &wq_head->head)
		return nr_exclusive;

	list_for_each_entry_safe_from(curr, next, &wq_head->head, entry) {
		unsigned flags = curr->flags;
		int ret;

		if (flags & WQ_FLAG_BOOKMARK)
			continue;

		ret = curr->func(curr, mode, wake_flags, key);
		if (ret < 0)
			break;
		if (ret && (flags & WQ_FLAG_EXCLUSIVE) && !--nr_exclusive)
			break;

		if (bookmark && (++cnt > WAITQUEUE_WALK_BREAK_CNT) &&
				(&next->entry != &wq_head->head)) {
			bookmark->flags = WQ_FLAG_BOOKMARK;
			list_add_tail(&bookmark->entry, &next->entry);
			break;
		}
	}
	return nr_exclusive;
}
```

## 抢占与上下文切换

### 上下文切换

上下文切换是指从一个可执行进程切换到另一个可执行进程。

```
static __always_inline struct rq *
context_switch(struct rq *rq, struct task_struct *prev,
	       struct task_struct *next, struct rq_flags *rf)
{
	struct mm_struct *mm, *oldmm;

	prepare_task_switch(rq, prev, next);

	mm = next->mm;
	oldmm = prev->active_mm;
	/*
	 * For paravirt, this is coupled with an exit in switch_to to
	 * combine the page table reload and the switch backend into
	 * one hypercall.
	 */
	arch_start_context_switch(prev);
	
	// 把虚拟内存从上一个内存映射切换到新进程中
	if (!mm) {
		next->active_mm = oldmm;
		mmgrab(oldmm);
		enter_lazy_tlb(oldmm, next);
	} else
		switch_mm_irqs_off(oldmm, mm, next);

	if (!prev->mm) {
		prev->active_mm = NULL;
		rq->prev_mm = oldmm;
	}

	rq->clock_update_flags &= ~(RQCF_ACT_SKIP|RQCF_REQ_SKIP);

	/*
	 * Since the runqueue lock will be released by the next
	 * task (which is an invalid locking op but in the case
	 * of the scheduler it's an obvious special-case), so we
	 * do an early lockdep release here:
	 */
	rq_unpin_lock(rq, rf);
	spin_release(&rq->lock.dep_map, 1, _THIS_IP_);

	/* Here we just switch the register state and the stack. */
	// 切换处理器状态到新进程，这包括保存、恢复寄存器和栈的相关信息 
	switch_to(prev, next, prev);
	barrier();

	return finish_task_switch(prev);
}
```

上下文切换由`schedule()`函数在切换进程时调用。但是内核必须知道什么时候调用`schedule()`，如果只靠用户代码显式地调用，代码可能会永远地执行下去。

为此，内核为每个进程设置了一个`need_resched`标志来表明是否需要重新执行一次调度，当某个进程应该被抢占时，`scheduler_tick()`会设置这个标志，当一个优先级高的进程进入可执行状态的时候，`try_to_wake_up()`也会设置这个标志位，内核检查到此标志位就会调用`schedule()`重新进行调度。

### 用户抢占

内核即将返回用户空间的时候，如果`need_reshced`标志位被设置，会导致`schedule()`被调用，此时就发生了用户抢占。意思是说，既然要重新进行调度，那么可以继续执行进入内核态之前的那个进程，也完全可以重新选择另一个进程来运行，所以如果设置了`need_resched`，内核就会选择一个更合适的进程投入运行。

简单来说有以下两种情况会发生用户抢占

- 从系统调用返回用户空间
- 从中断处理程序返回用户空间

### 内核抢占

Linux和其他大部分的Unix变体操作系统不同的是，它支持完整的内核抢占。

不支持内核抢占的系统意味着：内核代码可以一直执行直到它完成为止，内核级的任务执行时无法重新调度，各个任务是以协作方式工作的，并不存在抢占的可能性。

在Linux中，只要重新调度是安全的，内核就可以在任何时间抢占正在执行的任务，这个安全是指，只要没有持有锁，就可以进行抢占。

为了支持内核抢占，Linux做出了如下的变动：

- 为每个进程的`thread_info`引入了`preempt_count`计数器，用于记录持有锁的数量，当它为0的时候就意味着这个进程是可以被抢占的。
- 从中断返回内核空间的时候，会检查`need_resched`和`preempt_count`的值，如果`need_resched`被标记，并且`preempt_count`为0，就意味着有一个更需要调度的进程需要被调度，而且当前情况是安全的，可以进行抢占，那么此时调度程序就会被调用。

除了响应中断后返回，还有一种情况会发生内核抢占，那就是内核中的进程由于阻塞等原因显式地调用`schedule()`来进行显式地内核抢占：当然，这个进程显式地调用调度进程，就意味着它明白自己是可以安全地被抢占的，因此我们不用任何额外的逻辑去检查安全性问题。

下面罗列可能的内核抢占情况：

- 中断处理正在执行，且返回内核空间之前
- 内核代码再一次具有可抢占性时
- 内核中的任务显式地调用`schedule()`
- 内核中的任务被阻塞

