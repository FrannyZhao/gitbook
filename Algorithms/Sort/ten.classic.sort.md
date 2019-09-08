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
        int[] arr = Arrays.copyof(srcArr, srcArr.length);
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
            arr[i] = tmp;
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

```

* 重点

> 

* 时间：
  $$
  O()
  $$

* 空间：O()

* 稳定性： 

  

## Counting sort

```java

```

* 重点

> 

* 时间：
  $$
  O()
  $$

* 空间：O()

* 稳定性： 

  

## Bucket sort

```java

```

* 重点

> 

* 时间：
  $$
  O()
  $$

* 空间：O()

* 稳定性： 

  

## Radix sort

```java

```

* 重点

> 

* 时间：
  $$
  O()
  $$

* 空间：O()

* 稳定性： 






























