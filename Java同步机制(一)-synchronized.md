# Java同步机制(一)-Synchronized

[toc]

## 相关知识点

### JVM对于不同级别锁的使用场景
1. 偏向锁适用于单线程访问
2. 轻量级锁适用于多个线程交替访问
3. 重量级锁适用于多个线程抢占式访问

### Java对象头
Synchronized的优化和对象头息息相关。

```
|--------------------------------------------------------------------------------------------------------------|
|                                              Object Header (128 bits)                                        |
|--------------------------------------------------------------------------------------------------------------|
|                        Mark Word (64 bits)                                    |      Klass Word (64 bits)    |       
|--------------------------------------------------------------------------------------------------------------|
|  unused:25 | identity_hashcode:31 | unused:1 | age:4 | biased_lock:1 | lock:2 |     OOP to metadata object   |  无锁
|----------------------------------------------------------------------|--------|------------------------------|
|  thread:54 |         epoch:2      | unused:1 | age:4 | biased_lock:1 | lock:2 |     OOP to metadata object   |  偏向锁
|----------------------------------------------------------------------|--------|------------------------------|
|                     ptr_to_lock_record:62                            | lock:2 |     OOP to metadata object   |  轻量锁
|----------------------------------------------------------------------|--------|------------------------------|
|                     ptr_to_heavyweight_monitor:62                    | lock:2 |     OOP to metadata object   |  重量锁
|----------------------------------------------------------------------|--------|------------------------------|
|                                                                      | lock:2 |     OOP to metadata object   |    GC
|--------------------------------------------------------------------------------------------------------------|

```

![](media/16170963850528.jpg)

对象头的格式在处于不同阶段的锁的时候，是不同的。
这里我们主要说一下`biased_lock`这个标志位
#### biased_lock
biased_lock这个标志位表示当前这个锁对象是否可以偏向。
在开启偏向锁，并且过了4秒钟的偏向锁生效时间后。
- 创建出来的实例的对象头的后三位为`1 01`，并且偏向的线程ID也为空,表示当前的实例状态是`可偏向、没有偏向线程、没有锁定`（也叫做`匿名偏向状态`）
- 当有线程对这个实例加锁，这时候线程会偏向这个线程，既对象头中的线程ID修改为这个线程的ID，后三位为`1 01`,这时候实例状态是，`可偏向，偏向某一线程，没有锁定`(这里为什么认为是没有锁定，我理解，就是把偏向锁也认为是无锁状态)
- 这时候有另一个线程，尝试锁定这个实例，这时候会触发偏向锁的撤销，也就是把对象头中的biased_lock标志位从1改为0，并且升级到轻量级锁。这时候如果之前偏向的线程已经执行完毕，这个线程就能顺利的使用轻量级获得锁。时候后三位是`0 00`。这时候意思是`不可偏向，已锁定`

#### epoch
epoch主要和批量重偏向次数有关，我们下面说明。

#### 匿名偏向状态
* 锁状态为偏向锁状态(后三位是101)，但是没有偏向任何一个线程

### 偏向锁撤销
如果一直只有一个线程持有这个锁，那么这个锁会一直保持偏向状态，并且获取所得操作只需要一次判断，效率最高。
如果一旦出现了非偏向的线程，尝试获取这个锁对象，就会触发偏向锁的撤销。偏向锁的撤销的开销比较大，需要遍历所有的线程栈，所以必须在安全点执行。
一旦发生了偏向锁的撤销，直到再次发生批量重偏向的时间内，这个锁对象会升级为轻量级锁。

### 批量重偏向，批量撤销
JVM在每个类中都记录了这个类型所有实例的偏向撤销次数，当达到20次的时候，会执行批量重偏向，达到40次的时候会执行批量撤销。
可以通过JVM参数来调整这两个数值

```
-XX:BiasedLockingBulkRebiasThreshold=20 偏向锁批量重偏向阈值
-XX:BiasedLockingBulkRevokeThreshold=40 偏向锁批量撤销阈值
```
- 批量重偏向，是因为JVM觉得，偏向这个线程导致的撤销太多，所以让这个。
- 批量撤销的意思是，JVM认为由于这个类型的实例发生了太多次的偏向撤销操作，所以这个类型不在适合适用偏向锁，所以会遍历当前所有线程的线程栈，找到所有这种类型的锁对象，撤销他们的偏向锁。

### 关于偏向撤销的思考
`其实这里我也一直有一个疑问，就是在很多情况下，直接重偏向应该是一个更好的选择，那为什么每次都要先进行偏向撤销？`
首先我们先明确一个问题，偏向锁比轻量级锁快在哪里？
快在了少一次CAS的操作，偏向锁在偏向之后，如果是同一个线程尝试加锁，只需要一次比较操作，就能加锁成功。

如果按照上面设想的，当一个线程尝试锁定一个已经偏向另一个线程的锁对象的时候，直接进行重偏向，那么这时候需要进行的操作就是
- 判断是不是偏向状态
- 判断是不是偏向的线程
- CAS替换偏向线程ID
这样的话，就损失掉了偏向锁相对于轻量级锁的优势。而且和轻量级锁没有什么太大的区别。甚至还要比直接使用轻量级锁还多了几次比较。

所以JVM在遇到有多个线程尝试锁定同一个对象的时候，会直接锁升级。
所以有就有说到了上面的提到的不同锁，使用场景问题。
让偏向锁工作在自己最优的没有线程竞争的环境下。


### 安全点
安全点的官方定义是在程序执行期间的所有GC Root已知并且所有堆对象的内容一致的点。
在这个期间，所有的用户线程都会停顿，然后由vm线程来执行一些操作
比如我们已经知道
- 偏向锁的撤销，
- 批量重偏向，
- 批量撤销
- CMS中的某些步骤


在JVM中，每种需要VMThread执行的操作都被封装成了单独的类型。然后把需要执行的逻辑放在doit方法中，比如批量偏向撤销操作就被封装成了`VM_RevokeBias`。

```
 // VM 线程执行偏向锁撤销？
VM_RevokeBias revoke(&obj, (JavaThread*) THREAD);
VMThread::execute(&revoke);
return revoke.status_code();
```

他在doit方法中执行的逻辑是

```c++
virtual void doit() {
    _status_code = bulk_revoke_or_rebias_at_safepoint((*_obj)(), _bulk_rebias, _attempt_rebias_of_object, _requesting_thread);
    clean_up_cached_monitor_info();
  }

```
他的继承关系是
`VM_BulkRevokeBias` -> `VM_RevokeBias` -> `VM_Operation`
VMThread在执行的时候，会检查每个操作对象的`evaluation_mode`,来判断这个操作能不能并发的执行。
`evaluation_mode`这个方法是是定义在父类`VM_Operation`，默认是`safepoint`也就是需要所有线程都进入安全点，也就是需要stw。
子类可以选择重写这个方法。有下面几种选择

```c++
enum Mode {
    _safepoint, // blocking, safepoint, vm_op C-heap allocated
    _no_safepoint,// blocking, no safepoint, vm_op C-Heap allocated
    _concurrent, // non-blocking,no safepoint, vm_op C-Heap allocated
    _async_safepoint // non-blocking,safepoint, vm_op C-Heap allocated
  };
```


具体什么时候会触发安全点，由JVM来决定。


### JVM中使用的CAS方法
在JVM代码中，实现CAS是借助于下面这个方法。
```
cmpxchg_ptr(intptr_t exchange_value, 
        volatile intptr_t* dest, 
        intptr_t compare_value)

```
* cmpxchg_ptr这个方法的三个参数分别是
    * 第一个 字段变化的值 exchange value
    * 第二个 需要修改的字段地址 address
    * 第三个 比较值， compare value，也就是只有当字段是这个值的时候，才能发生交换。
    * 方法的返回值，如果交换成功，返回compare value

下面借助一个例子来说明这个方法的作用
```c
//构建一个匿名偏向的对象头
markOop header = (markOop) ((uintptr_t) mark & ((uintptr_t)markOopDesc::biased_lock_mask_in_place |(uintptr_t)markOopDesc::age_mask_in_place |epoch_mask_in_place));
//构建一个偏向当前线程的对象头
markOop new_header = (markOop) ((uintptr_t) header | thread_ident);
//执行CAS操作
if (Atomic::cmpxchg_ptr((void*)new_header, lockee->mark_addr(), header) == header) {
    //成功
}else{
    //失败
}
```
* 根据上面的代码，的意思就是，如果锁对象的对象头的值为匿名偏向状态，那么把锁对象的对象头修改为偏向当前线程的对象头

## 升级过程
![](media/16163102457099.jpg)
首先看一下这张图，这是Hotspot官方给出的锁升级过程图，我们先简单分析一下这张图。

1. 开启了偏向锁
    1. 锁对象刚分配的时候默认锁状态是匿名偏向状态。
    2. 有一个线程抢占锁，则在对象头中存储当前线程ID，锁进入偏向状态。
    3. 如果发生了重偏向，则会重新回到匿名偏向状态
    4. 如果发生了偏向撤销
        1. 如果发生的时候是无锁状态，则会进入不可偏向的无锁状态
        2. 如果发生的时候是有线程正持有，则会进入轻量级加锁状态
2. 如果没有开启偏向锁，则默认分配的就是不可偏向的无锁状态
    1. 后面就是正常的锁升级流程


## 偏向锁过程
1. 首先根据对象头的后三位判断该是不是处于偏向锁状态
    1. 如果是不是偏向状态则走轻量级锁流程
    2. 如果是偏向状态。
        1. 则拿到锁对象头中偏向的线程ID，判断是不是当前的这个线程。
            1. 如果是则不需要任何处理，认为获取锁成功
        2. 判断锁对象所属的类中的偏向锁是否开启
            1. 如果没有开启，则执行偏向锁撤销
        3. 判断锁对象的epoch和所属类型的epoch是否相同 
            1. 如果不同，则尝试重偏向  （TODO 这里为什么）
                1. 如果偏向失败则进行锁升级
        4. 这时候通过CAS，把当前线程的ThreadID设置到锁对象头中。（走到这一步说明要么是匿名偏向状态，要么偏向其他线程）
            1. 如果设置成功，表示获取偏向锁成功
            2. 如果设置失败，表示已经排向了其他线程，则进行锁升级。

## 轻量级锁过程
当上面的偏向锁获取失败的时候，会升级到轻量级锁。
这时候锁对象的对象头格式也会发生相应的变化，在锁对象的对象头中，会存储一个指向持有锁的线程的线程栈中的一个Lock Record的指针。
被Lock Record指针锁顶替的对象头中其他字段的信息，为在Lock Record中存储。

```c++
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
      // 设置到Lock Record中去,对象头中的其他信息，存储到Lock record中了
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

1. 构建一个无锁状态的对象头，(也就是把对象头的后三位设置为001，其他位不变)
2. 把对象头设置到之前创建的Lock Record中，也就是把对象头中的信息保存到Lock Record中。
3. 判断是否有UseHeavyMonitors参数，如果有，则直接走重量级锁
4. 使用CAS操作，把Lock Record的地址，设置到锁对象的对象头中。
    1. 如果CAS成功，则表示获取成功，则结束
    2. 如果CAS失败，则表示当前已经有线程轻量级锁定了这个对象
        1. 需要判断是不是当前线程，如果是当前线程则表示是重入
        2. 