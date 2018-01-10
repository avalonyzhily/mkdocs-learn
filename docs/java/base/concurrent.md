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
    public static ExecutorService pool = Executors.newCachedThreadPool();
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