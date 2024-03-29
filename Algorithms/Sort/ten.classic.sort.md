[TOC]

# Ten classic sort algorithms

## Summary

参考： https://mp.weixin.qq.com/s/vn3KiV-ez79FmbZ36SX9lg

https://mp.weixin.qq.com/s/Qf416rfT4pwURpW3aDHuCg



![image](../pic/640.webp)

* 时间复杂度

  | 时间复杂度                     | 排序                                       |
  | ------------------------------ | ------------------------------------------ |
  | O(n^2)                         | 各类简单排序：直接插入，直接选择，冒泡排序 |
  | O(nlgn)                        | 快速排序，堆排序，归并排序                 |
  | O(n1+§)，§是介于0和1之间的常数 | 希尔排序                                   |
  | O(n)                           | 基数排序，桶排序，计数排序                 |

* 稳定性

| 稳定性 | 排序                                   |
| ------ | -------------------------------------- |
| 稳定   | 冒泡排序，插入排序，归并排序，基数排序 |
| 不稳定 | 选择排序，快速排序，希尔排序，堆排序   |



## Bubble sort

```java
import java.util.Arrays;
public class BubbleSort {
    public int[] sort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        boolean hasChanged;
        for (int i = arr.length - 1; i > 0; i --) {
            hasChanged = false;
            for (int j = 0; j < i; j ++) {
                if (arr[j] > arr[j + 1]) {
                    int tmp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = tmp;
                    hasChanged = true;
                }
            }
            if (!hasChanged) {
                break;
            }
        }
        return arr;
    }
}
```

* 重点

> for (int i = arr.length - 1; i > 0; i --) {

> for (int j = 0; j < i; j ++) {
>
> if (arr[j] > arr[j + 1] ) {

* 时间： 
  $$
  O(n^2)
  $$

* 空间：O(1)

* 稳定性：稳定



## Selection sort

```java
import java.util.Arrays;
public class SelectionSort {
    public int[] sort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int min, tmp;
        for (int i = 0; i < arr.length - 1; i++) {
            min = i;
            for (int j = i + 1; j < arr.length; j++) {
                if (arr[j] < arr[min]) {
                    min = j;
                }
            }
            if (min != i) {
                tmp = arr[i];
                arr[i] = arr[min];
                arr[min] = tmp;
            }
        }
        return arr;
    }
}
```

* 重点

> for (int i = 0; i < arr.length - 1; i++) {
>
> for (int j = i + 1; j < arr.length; j++) {

* 时间：
  $$
  O(n^2)
  $$

* 空间：O(1)

* 稳定性： 不稳定

> 选择排序不是稳定的排序算法，因为：
>
> 序列 5 8 5 2 9，第一遍选择第一个5会和2交换，那么原序列中两个5的相对前后顺序就被破坏了。



## Insert sort

```java
import java.util.Arrays;
public class InsertSort {
    public int[] sort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int tmp;
        for (int i = 1; i < arr.length; i++) {
            tmp = arr[i];
            int j = i;
            while (j > 0 && tmp < arr[j - 1]) {
                arr[j] = arr[j - 1];
                j--;
            }
            arr[j] = tmp;
        }
        return arr;
    }
}
```

* 重点

> ​        for (int i = 1; i < arr.length; i++) {
> ​            tmp = arr[i];
> ​            int j = i;
> ​            while (j > 0 && tmp < arr[j - 1]) {
> ​                arr[j] = arr[j - 1];
> ​                j--;
> ​            }
> ​            arr[i] = tmp;
> ​        }

* 时间：
  $$
  O(n^2)
  $$

* 空间：O(1)

* 稳定性： 稳定

  


## Shell sort

```java
import java.util.Arrays;

public class ShellSort {
    public int[] sort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int gap = 1;
        while (gap < arr.length) {
            gap = gap * 3 + 1;
        }
        while (gap > 0) {
            for (int i = gap; i < arr.length; i++) {
                int tmp = arr[i];
                int j = i;
                while (j >= gap && tmp < arr[j - gap]) {
                    arr[j] = arr[j - gap];
                    j -= gap;
                }
                arr[j] = tmp;
            }
            gap = (int) Math.floor(gap / 3);
        }
        return arr;
    }
}
```

* 重点

> ​        while (gap > 0) {
> ​            for (int i = gap; i < arr.length; i++) {
> ​                int tmp = arr[i];
> ​                int j = i;
> ​                while (j >= gap && tmp < arr[j - gap]) {
> ​                    arr[j] = arr[j - gap];
> ​                    j -= gap;
> ​                }
> ​                arr[j] = tmp;
> ​            }
> ​            gap = (int) Math.floor(gap / 3);
> ​        }

* 时间：
  $$
  O(N^{1.289}) 到 O(2.5NlgN)之间，精确的模型还没找到
  $$

* 空间：O(1)

* 稳定性： 不稳定

  

## Merge sort

```java
import java.util.Arrays;

public class MergeSort {
    public int[] sort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        if (arr.length < 2) {
            return arr;
        }
        int middle = (int) Math.floor(arr.length / 2);
        int[] left = Arrays.copyOfRange(arr, 0, middle);
        int[] right = Arrays.copyOfRange(arr, middle, arr.length);
        return merge(sort(left), sort(right));
    }
    
    private int[] merge(int[] left, int[] right) {
        int[] result = new int[left.length + right.length];
        int i = 0, j = 0, k = 0;
        while (j < left.length && k < right.length) {
            if (left[j] <= right[k]) {
                result[i++] = left[j++];
            } else {
                result[i++] = right[k++];
            }
        }
        while(j < left.length) {
            result[i++] = left[j++];
        }
        while(k < right.length) {
            result[i++] = right[k++];
        }
        return result;
    }
}
```

* 重点

> ​        int middle = (int) Math.floor(arr.length / 2);
> ​        int[] left = Arrays.copyOfRange(arr, 0, middle);
> ​        int[] right = Arrays.copyOfRange(arr, middle, arr.length);

* 时间：
  $$
  O(NlgN)
  $$

* 空间：O()

* 稳定性： 稳定

  

## Quick sort

```java
import java.util.Arrays;

public class QuickSort {
    public int[] sort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        return quickSort(arr, 0, arr.length - 1);
    }
    
    public int[] quickSort(int[] arr, int start, int end) {
        if (start < end) {
//            int pivotIndex = partitionSingleSide(arr, start, end);
            int pivotIndex = partitionDoubleSide(arr, start, end);
            quickSort(arr, start, pivotIndex - 1);
            quickSort(arr, pivotIndex + 1, end);
        }
        return arr;
    }
    
    /**
     * 单边扫描
     * @param arr
     * @param left
     * @param right
     * @return
     */
    private int partitionSingleSide(int[] arr, int start, int end) {
        int pivotValue = arr[start];
        int mark = start;
        int tmp;
        for (int i = mark + 1; i <= end; i++) {
            if (arr[i] < pivotValue) {
                mark++;
                if (mark != i) {
                    tmp = arr[i];
                    arr[i] = arr[mark];
                    arr[mark] = tmp;
                }
            }
        }
        if (mark != start) {
            arr[start] = arr[mark];
            arr[mark] = pivotValue;
        }
        return mark;
    }

    /**
     * 双边扫描
     * @param arr
     * @param left
     * @param right
     * @return
     */
    private int partitionDoubleSide(int[] arr, int start, int end) {
        int left = start;
        int right = end;
        int pivotValue = arr[start];
        int tmp;
        while (left < right) {
            // 顺序很重要，要先从右往左扫描
            while (arr[right] >= pivotValue && left < right) {
                right--;
            }
            // 从左往右扫描
            while (arr[left] <= pivotValue && left < right) {
                left++;
            }
            // 交换左右数据
            if (left < right) {
                tmp = arr[left];
                arr[left] = arr[right];
                arr[right] = tmp;
            }
        }
        // 此时left==right
        arr[start] = arr[left];
        arr[left] = pivotValue;
        return right;
    }
}
```

* 重点

> 双边扫描：先从右往左，最后对调left和start

* 时间：
  $$
  O(NlgN)
  $$

* 空间：

  首先就地快速排序使用的空间是O(1)的，也就是个常数级；

  而真正消耗空间的就是递归调用了，因为每次递归就要保持一些数据；

  * 最优的情况下空间复杂度为：O(logn)，每一次都平分数组的情况

  * 最差的情况下空间复杂度为：O(n)，退化为冒泡排序的情况, 优化点是怎么取哨兵元素才能不会让这个算法退化到冒泡排序

* 稳定性： 不稳定

  

## Heap sort

```java
package sort;

import java.util.Arrays;

public class HeapSort {
    public int[] sort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        for (int i = (int) Math.floor(arr.length / 2); i >= 0; i--) {
            heapify(arr, i, arr.length);
        }
        int length = arr.length;
        for (int i = length - 1; i > 0; i--) {
            //将堆顶元素与末位元素调换
            int tmp = arr[0];
            arr[0] = arr[i];
            arr[i] = tmp;
            length--; //数组长度-1 隐藏堆尾元素
            heapify(arr, 0, length);
        }
        return arr;
    }

    private void heapify(int[] arr, int index, int length) {
        int leftChild = 2 * index + 1;//左子节点下标
        int rightChild = 2 * index + 2;//右子节点下标
        int present = index;//要调整的节点下标
        //调整左边
        if (leftChild < length && arr[leftChild] > arr[present]) {
            present = leftChild;
        }
        //调整右边
        if (rightChild < length && arr[rightChild] > arr[present]) {
            present = rightChild;
        }
        //如果下标不相等 证明调换过了
        if (present != index) {
            //交换值
            int temp = arr[index];
            arr[index] = arr[present];
            arr[present] = temp;
            //继续调整
            heapify(arr, present, length);
        }
    }
}
```

* 重点

> ​        int leftChild = 2 * index + 1;//左子节点下标
> ​        int rightChild = 2 * index + 2;//右子节点下标
> ​        int present = index;//要调整的节点下标
> ​        //调整左边
> ​        if (leftChild < length && arr[leftChild] > arr[present]) {
> ​            present = leftChild;
> ​        }
> ​        //调整右边
> ​        if (rightChild < length && arr[rightChild] > arr[present]) {
> ​            present = rightChild;
> ​        }
> ​        //如果下标不相等 证明调换过了
> ​        if (present != index) {
> ​            //交换值
> ​            int temp = arr[index];
> ​            arr[index] = arr[present];
> ​            arr[present] = temp;
> ​            //继续调整
> ​            heapify(arr, present, length);
> ​        }

* 时间：
  $$
  O(NlgN)
  $$

* 空间：O(1)

* 稳定性： 不稳定

  

## Counting sort

```java
package sort;

import java.util.Arrays;

/**
 * 计数排序只适用于正整数, 并且取值范围相差不大的数组排序使用，它的排序的速度是非常可观的。
 * @author zhaofengyi
 *
 */
public class CoutingSort {
    public int[] sort(int[] srcArr) {
        if (srcArr.length == 0) {
            return srcArr;
        }
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int max = arr[0], min = arr[0];
        for (int value : arr) {
            if (value > max) {
                max = value;
            }
            if (value < min) {
                min = value;
            }
        }
        int[] bucket = new int[max - min + 1];
        for (int value : arr) {
            bucket[value - min]++;
        }
        int sortedIndex = 0;
        for (int i = 0; i < bucket.length; i++) {
            while (bucket[i] > 0) {
                arr[sortedIndex++] = i + min;
                bucket[i]--;
            }
        }
        return arr;
    }
}
```

* 重点

> int[] bucket = new int[maxValue - minValue + 1];
>
> for (int value : arr) {
>     bucket[value - minValue]++;
> }
>
> while (bucket[j] > 0) {
>     arr[sortedIndex++] = j + minValue;
>     bucket[j]--;
> }

* 时间：
  $$
  O(N+k)
  $$

* 空间：O()

* 稳定性： 稳定

  

## Bucket sort

```java
package sort;

import java.util.Arrays;

public class BucketSort {
    public int[] sort(int[] srcArr) {
        if (srcArr.length == 0) {
            return srcArr;
        }
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int max = arr[0], min = arr[0], bucketSize = 5;
        for (int value : arr) {
            if (value > max) {
                max = value;
            }
            if (value < min) {
                min = value;
            }
        }
        int bucketCount = (int) Math.floor((max - min) / bucketSize) + 1;
        int[][] buckets = new int[bucketCount][0];
        for (int value : arr) {
            int index = (int) Math.floor((value - min) / bucketSize);
            buckets[index] = Arrays.copyOf(buckets[index], buckets[index].length + 1);
            int i = buckets[index].length - 1;
            while (i > 0 && value < buckets[index][i - 1]) {
                buckets[index][i] = buckets[index][i - 1];
                i--;
            }
            buckets[index][i] = value;
        }
        int sortedIndex = 0;
        for (int[] bucket : buckets) {
            if (bucket.length == 0) {
                continue;
            }
            for (int value : bucket) {
                arr[sortedIndex++] = value;
            }
        }
        return arr;
    }
}
```

* 重点

> 在额外空间充足的情况下，尽量增大桶的数量，极限情况下每个桶只有一个数据时，或者是每只桶只装一个值时，完全避开了桶内排序的操作，桶排序的最好时间复杂度就能够达到 O(n)。

* 时间：
  $$
  O(N+k)
  $$

* 空间：O(N+k)

* 稳定性： 稳定

  

## Radix sort

```java
package sort;

import java.util.Arrays;

public class RadixSort {
    public int[] sort(int[] srcArr) {
        if (srcArr.length == 0) {
            return srcArr;
        }
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int max = arr[0], min = arr[0];
        for (int value : arr) {
            if (value > max) {
                max = value;
            }
            if (value < min) {
                min = value;
            }
        }
        if (min < 0) {
            min = - min;
            if (min > max) {
                max = min;
            }
        }
        int maxNumLen = 0;
        do {
            max = max / 10;
            maxNumLen ++;
        } while (max > 0);
        int mod = 10, dev = 1;
        for (int i = 0; i < maxNumLen; i++, dev *= 10) {
            int[][] buckets = new int[20][0];
            for (int value : arr) {
                int index = value / dev % mod + 10;
                buckets[index] = Arrays.copyOf(buckets[index], buckets[index].length + 1);
                buckets[index][buckets[index].length - 1] = value;
            }
            int pos = 0;
            for (int[] bucket : buckets) {
                for (int value : bucket) {
                    arr[pos++] = value;
                }
            }
        }
        return arr;
    }
}
```

* 重点

```java
            int[][] counter = new int[mod * 2][0];
            for (int j = 0; j < arr.length; j++) {
                int bucket = ((arr[j] % mod) / dev) + mod;
                counter[bucket] = Arrays.copyOf(counter[bucket], counter[bucket].length + 1);
                counter[bucket][counter[bucket].length - 1] = arr[j];
            }
            int pos = 0;
            for (int[] bucket : counter) {
                for (int value : bucket) {
                    arr[pos++] = value;
                }
            }
```

* 时间：
  $$
  O(N*k)
  $$
*k是最大值的位数*
  
* 空间：O(N+k)

* 稳定性： 稳定



## Summary

```java
package sort;

import java.util.Arrays;

public class TenClassicSorts {
    public int[] bubbleSort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int tmp;
        boolean hasChanged;
        for (int i = arr.length - 1; i > 0; i--) {
            hasChanged = false;
            for (int j = 0; j < i; j++) {
                if (arr[j] > arr[j + 1]) {
                    tmp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = tmp;
                    hasChanged = true;
                }
            }
            if (!hasChanged) {
                break;
            }
        }
        return arr;
    }

    public int[] selectionSort(int[] srcArr) {
        if (srcArr.length == 0) {
            return srcArr;
        }
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int tmp, min;
        for (int i = 0; i < arr.length; i++) {
            min = i;
            for (int j = i + 1; j < arr.length; j++) {
                if (arr[j] < arr[min]) {
                    min = j;
                }
            }
            if (min != i) {
                tmp = arr[i];
                arr[i] = arr[min];
                arr[min] = tmp;
            }
        }
        return arr;
    }

    public int[] insertSort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int tmp, j;
        for (int i = 1; i < arr.length; i++) {
            j = i;
            tmp = arr[i];
            while (j > 0 && tmp < arr[j - 1]) {
                arr[j] = arr[j - 1];
                j--;
            }
            arr[j] = tmp;
        }
        return arr;
    }

    public int[] shellSort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int gap = 1;
        while (gap < arr.length) {
            gap = gap * 3 + 1;
        }
        int tmp, j;
        while (gap > 0) {
            for (int i = gap; i < arr.length; i+=gap) {
                j = i;
                tmp = arr[i];
                while (j >= gap && tmp < arr[j - gap]) {
                    arr[j] = arr[j - gap];
                    j-=gap;
                }
                arr[j] = tmp;
            }
            gap = (int) Math.floor(gap / 3);
        }
        return arr;
    }

    public int[] quickSort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        return quick(arr, 0, arr.length - 1);
    }

    private int[] quick(int[] arr, int start, int end) {
        if (start < end) {
//            int pivot = partitionS(arr, start, end);
            int pivot = partitionD(arr, start, end);
            quick(arr, start, pivot - 1);
            quick(arr, pivot + 1, end);
        }
        return arr;
    }

    private int partitionS(int[] arr, int start, int end) {
        int mark = start;
        int pivotValue = arr[start];
        int tmp;
        for (int i = mark + 1; i <= end; i++) {
            if (arr[i] < pivotValue) {
                mark ++;
                if (mark != i) {
                    tmp = arr[mark];
                    arr[mark] = arr[i];
                    arr[i] = tmp;
                }
            }
        }
        if (mark != start) {
            arr[start] = arr[mark];
            arr[mark] = pivotValue;
        }
        return mark;
    }

    private int partitionD(int[] arr, int start, int end) {
        int left = start, right = end, pivotValue = arr[start], tmp;
        while (left < right) {
            while (left < right && arr[right] >= pivotValue) {
                right --;
            }
            while (left < right && arr[left] <= pivotValue) {
                left ++;
            }
            if (left < right) {
                tmp = arr[left];
                arr[left] = arr[right];
                arr[right] = tmp;
            }
        }
        arr[start] = arr[left];
        arr[left] = pivotValue;
        return left;
    }

    public int[] mergeSort(int[] srcArr) {
        if (srcArr.length < 2) {
            return srcArr;
        }
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int mid = (int) Math.floor(arr.length / 2);
        int[] left = Arrays.copyOfRange(arr, 0, mid);
        int[] right = Arrays.copyOfRange(arr, mid, arr.length);
        return merge(mergeSort(left), mergeSort(right));
    }

    private int[] merge(int[] left, int[] right) {
        int[] result =  new int[left.length + right.length];
        int i = 0, j = 0, k = 0;
        while (i < left.length && j < right.length) {
            if (left[i] < right[j]) {
                result[k++] = left[i++];
            } else {
                result[k++] = right[j++];
            }
        }
        while (i < left.length) {
            result[k++] = left[i++];
        }
        while (j < right.length) {
            result[k++] = right[j++];
        }
        return result;
    }

    public int[] heapSort(int[] srcArr) {
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        for (int i = (int) Math.floor(arr.length / 2); i >= 0; i--) {
            heapify(arr, i, arr.length);
        }
        int tmp;
        int length = arr.length;
        while (length > 0) {
            tmp = arr[0];
            arr[0] = arr[length - 1];
            arr[length - 1] = tmp;
            length --;
            heapify(arr, 0, length);
        }
        return arr;
    }

    private void heapify(int[] arr, int index, int length) {
        int leftChild = index * 2 + 1;
        int rightChild = index * 2 + 2;
        int present = index;
        int tmp;
        if (leftChild < length && arr[leftChild] > arr[present]) {
            present = leftChild;
        }
        if (rightChild < length && arr[rightChild] > arr[present]) {
            present = rightChild;
        }
        if (present != index) {
            tmp = arr[present];
            arr[present] = arr[index];
            arr[index] = tmp;
            heapify(arr, present, length);
        }
    }

    public int[] countSort(int[] srcArr) {
        if (srcArr.length == 0) {
            return srcArr;
        }
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int max = arr[0], min = arr[0];
        for (int value : arr) {
            if (value > max) {
                max = value;
            }
            if (value < min) {
                min = value;
            }
        }
        int[] bucket = new int[max - min + 1];
        for (int value : arr) {
            bucket[value - min]++;
        }
        int sortedIndex = 0;
        for (int i = 0; i < bucket.length; i++) {
            while (bucket[i] > 0) {
                arr[sortedIndex++] = i + min;
                bucket[i]--;
            }
        }
        return arr;
    }

    public int[] bucketSort(int[] srcArr) {
        if (srcArr.length == 0) {
            return srcArr;
        }
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int max = arr[0], min = arr[0], bucketSize = 5;
        for (int value : arr) {
            if (value > max) {
                max = value;
            }
            if (value < min) {
                min = value;
            }
        }
        int bucketCount = (int) Math.floor((max - min) / bucketSize) + 1;
        int[][] buckets = new int[bucketCount][0];
        for (int value : arr) {
            int index = (int) Math.floor((value - min) / bucketSize);
            buckets[index] = Arrays.copyOf(buckets[index], buckets[index].length + 1);
            int i = buckets[index].length - 1;
            while (i > 0 && value < buckets[index][i - 1]) {
                buckets[index][i] = buckets[index][i - 1];
                i--;
            }
            buckets[index][i] = value;
        }
        int sortedIndex = 0;
        for (int[] bucket : buckets) {
            if (bucket.length == 0) {
                continue;
            }
            for (int value : bucket) {
                arr[sortedIndex++] = value;
            }
        }
        return arr;
    }

    public int[] radixSort(int[] srcArr) {
        if (srcArr.length == 0) {
            return srcArr;
        }
        int[] arr = Arrays.copyOf(srcArr, srcArr.length);
        int max = arr[0], min = arr[0];
        for (int value : arr) {
            if (value > max) {
                max = value;
            }
            if (value < min) {
                min = value;
            }
        }
        if (min < 0) {
            min = - min;
            if (min > max) {
                max = min;
            }
        }
        int maxNumLen = 0;
        do {
            max = max / 10;
            maxNumLen ++;
        } while (max > 0);
        int mod = 10, dev = 1;
        for (int i = 0; i < maxNumLen; i++, dev *= 10) {
            int[][] buckets = new int[20][0];
            for (int value : arr) {
                int index = value / dev % mod + 10;
                buckets[index] = Arrays.copyOf(buckets[index], buckets[index].length + 1);
                buckets[index][buckets[index].length - 1] = value;
            }
            int pos = 0;
            for (int[] bucket : buckets) {
                for (int value : bucket) {
                    arr[pos++] = value;
                }
            }
        }
        return arr;
    }
}
```




























