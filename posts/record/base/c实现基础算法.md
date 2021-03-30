### 冒泡排序

两两交换，每次“内循环”完成后，最后一位肯定是最大（小）的

```c
#include <stdio.h>

int main()
{
    int arr[16] = {1,2,34,2,1,3,23,1,2,34,6,63,4634,34,53,5};
    int i = 0;
    int j = 0;
    int len = 16;
    int tmp;
    int isMatch = 0;
    for (i = 0; i < len; i++) {
        j = 0;
        isMatch = 0;
        while (j < (len - i - 1)) {
            if (arr[j] > arr[j + 1]) {
                tmp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = tmp;
                isMatch = 1;
            }
            j++;
        }
        if (isMatch == 0) {
            break;
        }
    }
    i = 0;
    while(i < len) {
        printf("%d ", arr[i]);
        i++;
    }
}

```



### 插入排序

双循环，内循环从 i - 1 开始，当 arr[j] 的值大（小）于目标值（arr[i]），停止内循环，并进行替换。内循环过程中，将循环的值往前一个个移动

```c
int main()
{
    int len = 16;
    int arr[16] = {1,2,34,2,1,3,23,1,2,34,6,63,4634,34,53,5};
    int i = 1;
    int j = 0;
    int tmp;
    for (i = 1; i < len; i++) {
        if (arr[i] < arr[i - 1]) {
            tmp = arr[i];
            for (j = i - 1; arr[j] >= tmp; j--) {
                arr[j + 1] = arr[j];
            }
            arr[j + 1] = tmp;
        }
    }
    i = 0;
    while(i < len) {
        printf("%d ", arr[i]);
        i++;
    }
}
```



### 简单选择排序

双循环，每次从 i + 1 出发，将最大（小）值移动到 i 的位置；这样使交换次数减少

```c
#include <stdio.h>

int main()
{
    int len = 16;
    int arr[16] = {1,2,34,2,1,3,23,1,2,34,6,63,4634,34,53,5};
    int i = 0;
    int j = 0;
    int min;
    int tmp;
    for (i = 0; i < len; i++) {
        j = i + 1;
        min = i;
        while (j < len) {
            if (arr[min] > arr[j]) {
                min = j;
            }
            j++;
        }
        tmp = arr[min];
        arr[min] = arr[i];
        arr[i] = tmp;
    }
    i = 0;
    while(i < len) {
        printf("%d ", arr[i]);
        i++;
    }
}

```



### 快速排序

### 归序排序

### 二叉树

#### 前序

#### 中序

#### 后序

#### 层序