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

```

* 重点

> 

* 时间：
  $$
  O()
  $$

* 空间：O()

* 稳定性： 

  

## Merge sort

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

  

## Quick sort

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






























