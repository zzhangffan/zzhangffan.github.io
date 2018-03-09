---
title: 排序算法——快速排序
date: 2018-03-07 14:57:58
categoreies:
tags: [排序算法,Java]
---

## 实现 ##

```
public class QuickSort {

    public static void main(String[] args) {

        /**
         * 思想：选定一个基准，从待排序序列两侧开始探测
         * 哨兵a：从右往左找一个小于基准的数
         * 哨兵b：从左往右找一个大于基准的数
         * 交换它们，直到两个哨兵相遇，交换基准
         * 现在基准左边的数都小于基准，右边的数都大于基准
         * 递归如此
         */
        int[] needSortArr = {1, 456, 25, 352, 45, 2, 5, 6, 9, 899, 52, 65, 3};
        quickSort(needSortArr, 0, needSortArr.length - 1);

        for (int i : needSortArr) {
            System.out.print(i + "\t");
        }
    }

    public static void quickSort(int[] needSortArr, int left, int right) {
        if (left > right)
            return;
        int i = left;
        int j = right;
        //基准
        int datum = needSortArr[left];

        while (i != j) {

            //从右往左探测
            while (needSortArr[j] > datum && i < j) {
                j--;
            }
            needSortArr[i] = needSortArr[j];
            //反方向
            while (needSortArr[i] < datum && i < j) {
                i++;
            }
            needSortArr[j] = needSortArr[i];
        }
        //基准
        needSortArr[j] = datum;
        //分左右块递归
        quickSort(needSortArr, left, i - 1);
        quickSort(needSortArr, i + 1, right);
    }
}
```

（完）
=====