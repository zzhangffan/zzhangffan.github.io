---
title: 排序算法——简单选择排序
date: 2018-03-07 13:44:10
categoreies:
tags: [排序算法,Java]
---

## 实现 ##
```
public class SelectionSort {

    public static void main(String[] args) {

        /**
         * 思想：每趟选择出最小的数放到已有序列的后面
         */
        int[] needSortArr = {1, 456, 25, 352, 1, 45, 2, 5, 6, 9, 899, 52, 65, 3};

        for (int i = 0; i < needSortArr.length; i++) {
            int pos = i;
            int j = i + 1;
            for (;j < needSortArr.length; j++) {
                if (needSortArr[j] < needSortArr[pos])
                    pos = j;
            }
            //交换
            int temp = needSortArr[i];
            needSortArr[i] = needSortArr[pos];
            needSortArr[pos] = temp;
        }

        for (int i : needSortArr) {
            System.out.print(i + "\t");
        }
    }
}
```

（完）
=====