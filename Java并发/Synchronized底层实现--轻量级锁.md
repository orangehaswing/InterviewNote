# Synchronized底层实现--轻量级锁

本文分为两个部分：

1.轻量级锁获取流程

2.轻量级锁释放流程

本人看的JVM版本是jdk8u，具体版本号以及代码可以在[这里](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/9ce27f0a4683/src)看到。

### 轻量级锁获取流程

下面开始轻量级锁获取流程分析，代码在[bytecodeInterpreter.cpp#1816](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9ce27f0a4683/src/share/vm/interpreter/bytecodeInterpreter.cpp#l1816)。

```
CASE(_monitorenter): {
  oop lockee = STACK_OBJECT(-1);
  ...
  if (entry != NULL) {
   ...
   // 上面省略的代码中如果CAS操作失败也会调用到InterpreterRuntime::monitorenter

    // traditional lightweight locking
    if (!success) {
      // 构建一个无锁状态的Displaced Mark Word
      markOop displaced = lockee->mark()->set_unlocked();
      // 设置到Lock Record中去
      entry->lock()->set_displaced_header(displaced);
      bool call_vm = UseHeavyMonitors;
      if (call_vm || Atomic::cmpxchg_ptr(entry, lockee->mark_addr(), displaced) != displaced) {
        // 如果CAS替换不成功，代表锁对象不是无锁状态，这时候判断下是不是锁重入
        // Is it simple recursive case?
        if (!call_vm && THREAD->is_lock_owned((address) displaced->clear_lock_bits())) {
          entry->lock()->set_displaced_header(NULL);
        } else {
          // CAS操作失败则调用monitorenter
          CALL_VM(InterpreterRuntime::monitorenter(THREAD, entry), handle_exception);
        }
      }
    }
    UPDATE_PC_AND_TOS_AND_CONTINUE(1, -1);
  } else {
    istate->set_msg(more_monitors);
    UPDATE_PC_AND_RETURN(0); // Re-execute
  }
}
```

如果锁对象不是偏向模式或已经偏向其他线程，则`success`为`false`。这时候会构建一个无锁状态的`mark word`设置到`Lock Record`中去，我们称`Lock Record`中存储对象`mark word`的字段叫`Displaced Mark Word`。

如果当前锁的状态不是无锁状态，则CAS失败。如果这是一次锁重入，那直接将`Lock Record`的 `Displaced Mark Word`设置为`null`。

我们看个demo，在该demo中重复3次获得锁，

```
synchronized(obj){
    synchronized(obj){
    	synchronized(obj){
    	}
    }
}
```

假设锁的状态是轻量级锁，下图反应了`mark word`和线程栈中`Lock Record`的状态，可以看到右边线程栈中包含3个指向当前锁对象的`Lock Record`。其中栈中最高位的`Lock Record`为第一次获取锁时分配的。其`Displaced Mark word`的值为锁对象的加锁前的`mark word`，之后的锁重入会在线程栈中分配一个`Displaced Mark word`为`null`的`Lock Record`。

[![img](https://camo.githubusercontent.com/70b25e334cf9ad63db378af530593a93efed85a9/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f31312f32392f313637356665643730303638623431323f773d36393426683d36343226663d706e6726733d3134373937)](https://camo.githubusercontent.com/70b25e334cf9ad63db378af530593a93efed85a9/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f31312f32392f313637356665643730303638623431323f773d36393426683d36343226663d706e6726733d3134373937)

为什么JVM选择在线程栈中添加`Displaced Mark word`为null的`Lock Record`来表示重入计数呢？首先锁重入次数是一定要记录下来的，因为每次解锁都需要对应一次加锁，解锁次数等于加锁次数时，该锁才真正的被释放，也就是在解锁时需要用到说锁重入次数的。一个简单的方案是将锁重入次数记录在对象头的`mark word`中，但`mark word`的大小是有限的，已经存放不下该信息了。另一个方案是只创建一个`Lock Record`并在其中记录重入次数，Hotspot没有这样做的原因我猜是考虑到效率有影响：每次重入获得锁都需要遍历该线程的栈找到对应的`Lock Record`，然后修改它的值。

所以最终Hotspot选择每次获得锁都添加一个`Lock Record`来表示锁的重入。

接下来看看`InterpreterRuntime::monitorenter`方法

```
IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorenter(JavaThread* thread, BasicObjectLock* elem))
  ...
  Handle h_obj(thread, elem->obj());
  assert(Universe::heap()->is_in_reserved_or_null(h_obj()),
         "must be NULL or an object");
  if (UseBiasedLocking) {
    // Retry fast entry if bias is revoked to avoid unnecessary inflation
    ObjectSynchronizer::fast_enter(h_obj, elem->lock(), true, CHECK);
  } else {
    ObjectSynchronizer::slow_enter(h_obj, elem->lock(), CHECK);
  }
  ...
IRT_END
```

`fast_enter`的流程在偏向锁一文已经分析过，如果当前是偏向模式且偏向的线程还在使用锁，那会将锁的`mark word`改为轻量级锁的状态，同时会将偏向的线程栈中的`Lock Record`修改为轻量级锁对应的形式。代码位置在[biasedLocking.cpp#212](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9ce27f0a4683/src/share/vm/runtime/biasedLocking.cpp#l212)。

```
 // 线程还存活则遍历线程栈中所有的Lock Record
  GrowableArray<MonitorInfo*>* cached_monitor_info = get_or_compute_monitor_info(biased_thread);
  BasicLock* highest_lock = NULL;
  for (int i = 0; i < cached_monitor_info->length(); i++) {
    MonitorInfo* mon_info = cached_monitor_info->at(i);
    // 如果能找到对应的Lock Record说明偏向的线程还在执行同步代码块中的代码
    if (mon_info->owner() == obj) {
      ...
      // 需要升级为轻量级锁，直接修改偏向线程栈中的Lock Record。为了处理锁重入的case，在这里将Lock Record的Displaced Mark Word设置为null，第一个Lock Record会在下面的代码中再处理
      markOop mark = markOopDesc::encode((BasicLock*) NULL);
      highest_lock = mon_info->lock();
      highest_lock->set_displaced_header(mark);
    } else {
      ...
    }
  }
  if (highest_lock != NULL) {
    // 修改第一个Lock Record为无锁状态，然后将obj的mark word设置为执行该Lock Record的指针
    highest_lock->set_displaced_header(unbiased_prototype);
    obj->release_set_mark(markOopDesc::encode(highest_lock));
    ...
  } else {
    ...
  }
```

我们看`slow_enter`的流程。

```
void ObjectSynchronizer::slow_enter(Handle obj, BasicLock* lock, TRAPS) {
  markOop mark = obj->mark();
  assert(!mark->has_bias_pattern(), "should not see bias pattern here");
  // 如果是无锁状态
  if (mark->is_neutral()) {
    //设置Displaced Mark Word并替换对象头的mark word
    lock->set_displaced_header(mark);
    if (mark == (markOop) Atomic::cmpxchg_ptr(lock, obj()->mark_addr(), mark)) {
      TEVENT (slow_enter: release stacklock) ;
      return ;
    }
  } else
  if (mark->has_locker() && THREAD->is_lock_owned((address)mark->locker())) {
    assert(lock != mark->locker(), "must not re-lock the same lock");
    assert(lock != (BasicLock*)obj->mark(), "don't relock with same BasicLock");
    // 如果是重入，则设置Displaced Mark Word为null
    lock->set_displaced_header(NULL);
    return;
  }

  ...
  // 走到这一步说明已经是存在多个线程竞争锁了 需要膨胀为重量级锁
  lock->set_displaced_header(markOopDesc::unused_mark());
  ObjectSynchronizer::inflate(THREAD, obj())->enter(THREAD);
}

```

### 轻量级锁释放流程

```
CASE(_monitorexit): {
  oop lockee = STACK_OBJECT(-1);
  CHECK_NULL(lockee);
  // derefing's lockee ought to provoke implicit null check
  // find our monitor slot
  BasicObjectLock* limit = istate->monitor_base();
  BasicObjectLock* most_recent = (BasicObjectLock*) istate->stack_base();
  // 从低往高遍历栈的Lock Record
  while (most_recent != limit ) {
    // 如果Lock Record关联的是该锁对象
    if ((most_recent)->obj() == lockee) {
      BasicLock* lock = most_recent->lock();
      markOop header = lock->displaced_header();
      // 释放Lock Record
      most_recent->set_obj(NULL);
      // 如果是偏向模式，仅仅释放Lock Record就好了。否则要走轻量级锁or重量级锁的释放流程
      if (!lockee->mark()->has_bias_pattern()) {
        bool call_vm = UseHeavyMonitors;
        // header!=NULL说明不是重入，则需要将Displaced Mark Word CAS到对象头的Mark Word
        if (header != NULL || call_vm) {
          if (call_vm || Atomic::cmpxchg_ptr(header, lockee->mark_addr(), lock) != lock) {
            // CAS失败或者是重量级锁则会走到这里，先将obj还原，然后调用monitorexit方法
            most_recent->set_obj(lockee);
            CALL_VM(InterpreterRuntime::monitorexit(THREAD, most_recent), handle_exception);
          }
        }
      }
      //执行下一条命令
      UPDATE_PC_AND_TOS_AND_CONTINUE(1, -1);
    }
    //处理下一条Lock Record
    most_recent++;
  }
  // Need to throw illegal monitor state exception
  CALL_VM(InterpreterRuntime::throw_illegal_monitor_state_exception(THREAD), handle_exception);
  ShouldNotReachHere();
}
```

轻量级锁释放时需要将`Displaced Mark Word`替换到对象头的`mark word`中。如果CAS失败或者是重量级锁则进入到`InterpreterRuntime::monitorexit`方法中。

```
//%note monitor_1
IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorexit(JavaThread* thread, BasicObjectLock* elem))
 
  Handle h_obj(thread, elem->obj());
  ...
  ObjectSynchronizer::slow_exit(h_obj(), elem->lock(), thread);
  // Free entry. This must be done here, since a pending exception might be installed on
  //释放Lock Record
  elem->set_obj(NULL);
  ...
IRT_END
```

`monitorexit`调用完`slow_exit`方法后,就释放`Lock Record`。

```
void ObjectSynchronizer::slow_exit(oop object, BasicLock* lock, TRAPS) {
  fast_exit (object, lock, THREAD) ;
}
void ObjectSynchronizer::fast_exit(oop object, BasicLock* lock, TRAPS) {
  ...
  markOop dhw = lock->displaced_header();
  markOop mark ;
  if (dhw == NULL) {
     // 重入锁，什么也不做
   	 ...
     return ;
  }

  mark = object->mark() ;

  // 如果是mark word==Displaced Mark Word即轻量级锁，CAS替换对象头的mark word
  if (mark == (markOop) lock) {
     assert (dhw->is_neutral(), "invariant") ;
     if ((markOop) Atomic::cmpxchg_ptr (dhw, object->mark_addr(), mark) == mark) {
        TEVENT (fast_exit: release stacklock) ;
        return;
     }
  }
  //走到这里说明是重量级锁或者解锁时发生了竞争，膨胀后调用重量级锁的exit方法。
  ObjectSynchronizer::inflate(THREAD, object)->exit (true, THREAD) ;
}
```

该方法中先判断是不是轻量级锁，如果是轻量级锁则将替换`mark word`，否则膨胀为重量级锁并调用`exit`方法，相关逻辑将在重量级锁的文章中讲解。



[https://github.com/farmerjohngit/myblog](https://github.com/farmerjohngit/myblog)

