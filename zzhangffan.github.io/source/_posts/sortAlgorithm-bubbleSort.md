---
title: 排序算法——冒泡排序
date: 2018-03-07 13:58:29
categoreies:
tags: [排序算法,Java]
---

## 实现 ##
```
public class BubbleSort {

    public static void main(String[] args) {

        /**
         * 思想：依次比较两个相邻的数，让大的往下落，小的往上冒
         */
        int[] needSortArr = {1, 456, 25, 352, 1, 45, 2, 5, 6, 9, 899, 52, 65, 3};

        for (int i = 0; i < needSortArr.length - 1; i++) {
            for (int j = 0; j < needSortArr.length - 1 - i; j++) {
                if (needSortArr[j] > needSortArr[j + 1]) {
                    int temp = needSortArr[j];
                    needSortArr[j] = needSortArr[j + 1];
                    needSortArr[j + 1] = temp;
                }
            }
        }

        for (int i : needSortArr) {
            System.out.print(i + "\t");
        }
    }
}
```

（完）
=====