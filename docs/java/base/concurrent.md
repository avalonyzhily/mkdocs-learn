### 奇偶排序(串行->并行)改造
串行实现:
```
/**
 * 奇偶交换排序串行实现
 * 奇偶排序要成对出现
 */
public class TestMain {
    public static void oddEvenSort(int[] arr){
        //是否发生交换的标记,0:没有 1:有
        int exchFlag=1;
        //0偶排序 1奇排序 标记
        int start = 0;

        while(exchFlag == 1 || start == 1){
            exchFlag = 0;
            for(int i = start;i<arr.length-1;i+=2){
                if(arr[i] > arr[i+1]){
                    int temp = arr[i];
                    arr[i] = arr[i+1];
                    arr[i+1] = temp;
                    exchFlag = 1;
                }
            }
            if(start == 0){
                start = 1;
            }else {
                start = 0;
            }
        }

    }
    public static void main(String[] args){
        int[] arr = {65,2,9,5,7,1,13,54,3,42};
        oddEvenSort(arr);
        System.out.println(Arrays.toString(arr));
    }
}
```

并行排序
```
/**
 * 奇偶交换排序并行实现
 * 奇偶排序要成对出现
 * 排序循环止于 不再发生交换同时当前进行的是偶交换
 */
public class ParallelTestMain {
    //测试用线程池,执行完立即回收线程
    public static ExecutorService pool = new ThreadPoolExecutor(0, Integer.MAX_VALUE,
            0L, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>());
    public static int exchFlag = 1;
    public static int[] arr;
    public static int getExchFlag() {
        return exchFlag;
    }

    public static void setExchFlag(int v) {
        exchFlag = v;
    }
    public static class OddEventSortTask implements Runnable{
        int i;
        CountDownLatch latch;

        public OddEventSortTask(int i, CountDownLatch latch) {
            this.i = i;
            this.latch = latch;
        }

        @Override
        public void run() {
            if(arr[i] > arr[i+1]){
                int temp = arr[i];
                arr[i] = arr[i+1];
                arr[i+1] = temp;
                setExchFlag(1);
            }
            latch.countDown();
        }

        public static void pOddEvenSort(int[] arrWaitSort) throws InterruptedException{
            arr = arrWaitSort;
            int start = 0;
            while (getExchFlag() == 1 || start == 1){
                setExchFlag(0);
                CountDownLatch latch = new CountDownLatch(arr.length/2-(arr.length%2==0?start:0));
                for(int i= start;i<arr.length-1;i+=2){
                    pool.submit(new OddEventSortTask(i,latch));
                }
                latch.await();
                if(start==0){
                    start = 1;
                }else {
                    start = 0;
                }
            }
        }

        public static void main(String[] args) throws InterruptedException {
            int[] arr = {65,2,9,5,7,1,13,54,3,42};
            pOddEvenSort(arr);
            System.out.println(Arrays.toString(arr));
        }
    }

}
```

插入排序:从未排序部分选择一个元素插入到已排序部分,直到未排序部分元素个数为0
```
public class InsertSortMain {
    public static void insertSort(int[] arr){
        int length = arr.length;
        int i,j,key;
        for(i = 1;i<length;i++){
            //未排序部分选取的元素
            key = arr[i];
            //已排序部分的最后一个元素的下标(升序则为最大,降序则为最小,本例按升序排)
            j = i-1;
            while (j>=0&&arr[j]>key){
                //因为按升序排,则未排序部分的元素小时,则对比的已排序部分元素后移,如果是降序排,则条件为arr[i]<key
                arr[j+1] = arr[j];
                //key继续与前面的元素对比
                j--;
            }
            //j=-1说明已经到头了,key自然放在最前面下表为0(即j+1=-1+1=0)的位置,或是arr[j]<key了,也说明key要放在j+1的位置
            arr[j+1]=key;
        }
    }

    public static void main(String[] args){
        int[] arr = {65,2,9,5,7,1,13,54,3,42};
        shellSort(arr);
        System.out.println(Arrays.toString(arr));
    }
}
```

改进 插入排序-->希尔排序,按间隔h分割数组(插入排序的h是1)
```
/**
 * 希尔排序串行实现
 * 利用间隔值h,来跳跃比较交换数据
 * 此处都是升序排序
 */
public class shellSortMain {
    public static void shellSort(int[] arr){
        int h = 1;
        //计算最大的h
        while (h<=arr.length/3){
            h = h*3+1;
        }
        while (h>0){
            for(int i=h;i<arr.length;i++){
                //arr[i]是待插入的值,arr[i-h] / arr[j] 就是前面已经排序的元素
                if(arr[i]<arr[i-h]){
                    int temp = arr[i];
                    int j = i-h;
                    while (j>=0&&arr[j]>temp){
                        //这里的循环是为了找到arr[i],也就是temp的插入位置
                        arr[j+h] = arr[j];
                        j -= h;
                    }
                    arr[j+h] = temp;
                }
            }
            h=(h-1)/3;
        }
    }

    public static void main(String[] args){
        int[] arr = {65,2,9,5,7,1,13,54,3,42};
        shellSort(arr);
        System.out.println(Arrays.toString(arr));
    }
}
```

并行版希尔排序
```
/**
 * 奇偶交换排序并行实现
 * 奇偶排序要成对出现
 * 排序循环止于 不再发生交换同时当前进行的是偶交换
 */
public class ParallelTestMain {
    //测试用线程池,执行完立即回收线程
    public static ExecutorService pool = new ThreadPoolExecutor(0, Integer.MAX_VALUE,
            0L, TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>());
    public static int exchFlag = 1;
    public static int[] arr;
    public static int getExchFlag() {
        return exchFlag;
    }

    public static void setExchFlag(int v) {
        exchFlag = v;
    }
    public static class OddEventSortTask implements Runnable{
        int i;
        CountDownLatch latch;

        public OddEventSortTask(int i, CountDownLatch latch) {
            this.i = i;
            this.latch = latch;
        }

        @Override
        public void run() {
            if(arr[i] > arr[i+1]){
                int temp = arr[i];
                arr[i] = arr[i+1];
                arr[i+1] = temp;
                setExchFlag(1);
            }
            latch.countDown();
        }

        public static void pOddEvenSort(int[] arrWaitSort) throws InterruptedException{
            arr = arrWaitSort;
            int start = 0;
            while (getExchFlag() == 1 || start == 1){
                setExchFlag(0);
                CountDownLatch latch = new CountDownLatch(arr.length/2-(arr.length%2==0?start:0));
                for(int i= start;i<arr.length-1;i+=2){
                    //伪代码 TODO
                    pool.submit(new OddEventSortTask(i,latch));
                }
                latch.await();
                if(start==0){
                    start = 1;
                }else {
                    start = 0;
                }
            }
        }

        public static void main(String[] args) throws InterruptedException {
            int[] arr = {65,2,9,5,7,1,13,54,3,42};
            pOddEvenSort(arr);
            System.out.println(Arrays.toString(arr));

        }
    }

}
```