# Java同步机制(二)-LockSupport

LockSupport是AQS的实现中一个非常重要的部分，LockSupport提供了阻塞线程和唤醒线程的功能。
说到阻塞和唤醒，我们一定会想到wait/notify这两个方法。
和wait/notify相比，LockSupport有以下优点
- 以thread为操作对象更符合阻塞线程的直观定义（wait和notify都是针对锁对象的操作）
- 操作更精准，可以准确地唤醒某一个线程（notify随机唤醒一个线程，notifyAll唤醒所有等待的线程），增加了灵活性。

