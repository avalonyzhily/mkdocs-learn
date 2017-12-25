### ThreaLocal 保存只有当前线程能访问的资源
但是,如果每个线程分配到的都是同一个对象,依然会有线程安全。所以,ThreaLocal适合可以为每一个线程分配独立副本资源的场景下使用(譬如同一个实例内的成员属性需要线程隔离的情况——自己推测的,未证实!)

其本质是每个线程实例内部持有一个ThreadLocal.ThreadLocalMap(这里有两种一个是threadLocals,一个是inheritableThreadLocals,后者可以获得继承效果获取父类的数据)的Map对象,以ThreadLocal实例引用作为key,实例的值作为value!
### 这里之前理解有误！

ThreadLocal中的ThreadLocalMap是一个类似于WeakHashMap的存在,内部的ThreadLoacal引用(即key)都是弱引用,由于弱引用的机制,当外部ThreadLoacal引用(强引用)回收时,内部的key就变成null了,所以这些线程持有的对象就都被回收了。

使用ThreadLocal之后,如果还有线程池的话,处理不当可能会有内存泄漏的情况,因为对象都是保存在线程中的,线程不退出,对象不会被回收。