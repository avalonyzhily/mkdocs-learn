### ThreadPoolExecutor的扩展性
继承并覆盖以下三个方法来扩展
```
beforeExecute();
afterExecute();
terminated();
```
还可以覆盖execute方法,封装提交的Runnable任务,来扩展更多的功能,譬如增加堆栈信息的追踪等。


合理设置线程池线程数量,估计的公式:Nthreads = Ncpu*Ucpu*(1+W/C)
```
Nthreads = Ncpu*Ucpu*(1+W/C);
Ncpu = cpu数量
Ucpu = cpu使用率
W/C = 等待时间和计算时间的比率

--by 《Java Concurrency in Practice》
```