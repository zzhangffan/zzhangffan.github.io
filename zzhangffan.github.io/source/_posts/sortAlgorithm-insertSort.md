---
title: 排序算法——直接插入排序
date: 2018-03-07 13:04:23
categoreies:
tags: [排序算法,Java]
---

## 实现 ##
```
public class InsertSort {

    public static void main(String[] args) {
        /**
         * 思想：排序一组数 a[0..n-1]
         *  1.初始a[0]为有序区
         *  2.比较，将a[i]放入有序区
         *  3.直到i = n -1
         */
        int[] needSortArr = {1, 456, 25, 352, 1, 45, 2, 5, 6, 9, 899, 52, 65, 3};

        int temp;
        for (int i = 1; i < needSortArr.length; i++) {
            int j = i - 1;
            temp = needSortArr[i];
            for (; j >= 0 && temp < needSortArr[j]; j--) {
                needSortArr[j + 1] = needSortArr[j];
            }
            needSortArr[j + 1] = temp;
        }

        for (int i : needSortArr) {
            System.out.print(i + "\t");
        }
    }
}
```

（完）
=====