## Java基础

- 基本类型
  - boolean 1bit 只有true / false 两个值
    - Boolean
  - char——字符/2个字节/无符号16位unicode/0 ~ 2^16 -1/0 ~ 65535
    - Character
  - byte——1个字节/8bit/-2^7 ~ 2^7/-128~127
  - short——2个字节/16bit/-2^15 ~ 2^15-1/-32768 ~32767
  - int——4个字节/32bit/-2^31 ~ 2^32-1/ -2,147,483,648 ~ 2,147,483,647
  - long——8个字节/64bit/-2^63 ~ 2^63-1
  - float——IEEE计数法/ 符号位 阶数位 尾数位
  - double——IEEE计数法/ 符号位 阶数位 尾数位
- String 字符串
  - 比较特殊的引用类型,内部是char数组
  - StringBuffer StringBuilde的比较
  - intern
    - jdk1.6
    - jdk1.7
  >##### 源码阅读
  > TODO
- Collection 集合类 Iterable
  - Set
    - HashSet
    - TreeSet
  - List
    - LinkedList
    - ArrayList
- Map 映射
  - HashMap
  - Hashtable
- juc 并发包
  - AtomXX 原子类
  - ConcurrentHashMap
    - jdk1.7
    - jdk1.8
- ThreadMXBean java虚拟机线程管理接口
  - dumpAllThreads 获取线程信息的方法