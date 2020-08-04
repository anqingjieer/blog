#  排序

> 1. 定义：排序就是将原本无序的序列重新排列成有序的序列。
> 2. 排序的稳定性：如果待排序表中有两个元素Ri、Rj，其对应的关键字keyi=keyj，且在排序前Ri在Rj前面，如果使用某一排序算法排序后，Ri仍然在Rj的前面，则称这个排序算法是稳定的，否则称排序算法是不稳定的。
> 3. 自己的理解就是相等的两个数进行，没排序之前位置没变，这是稳定。排序之后位置发生改变，不稳定
> 4. ![image-20200612094030939](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200612094049.png)

## 一、冒泡排序

### 1.什么是冒泡排序？

​		 冒泡排序：从第一个数开始，和后面的数字进行比较，如果比后面的数字大，交换位置，一直到最后一个数		 字。取的最大值，然后第二次也是从第一个数字开始，到n-i

​        时间复杂度：O(n*n)
​         空间复杂度：开辟了一个temp空间。o(1)

​			

```java
    public int[] bubbleSort(int[] nums) {
        int temp;
        for (int i = nums.length - 1; i > 0 ; i--) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[j + 1]) {
                    temp = nums[j];
                    nums[j] = nums[j + 1];
                    nums[j + 1] = temp;
                }
            }
        }
        return nums;
    }
```

## 二、插入排序

### 1.什么是插入排序？



​			每次将一个待排序的记录，按其关键字大小插入到前面已经排好序的字文件中的恰当位置，直到全部记录插入完成为止。

### 2.图解插入排序。

![插入排序图解](https://gitee.com/anqingjieer/pengbo/raw/master/20200509094538.png)

### 3.时间复杂度和空间复杂度

​			时间复杂度：o（n^2)
​			空间复杂度为：o( 1)

### 4.代码实现

```java
    private int[] insertSort(int[] nums) {
        for (int i = 1; i < nums.length; i++) {
            int temp = nums[i];//保存每次需要插入的那个数
            int j;
            for (j = i; j > 0 && nums[j - 1] > temp; j--) {//这个较上面有一定的优化
			    nums[j] = nums[j - 1];//把大于需要插入的数往后移动。最后不大于temp的数就空出来j 
            }
            nums[j] = temp;//将需要插入的数放入这个位置
        }
        return nums;
    }
```

## 三.堆排序

### 1.什么是堆？

堆就是完全二叉树，其特点是：所有的父节点大于子节点称之为大顶堆，所有的父节点小于子节点称之为小顶堆。

### 2.什么是堆排序？

堆排序可以分为以下几步组成。

- 首先将数组转化为大顶堆的完全二叉树。
- 其次将二叉树的第一个节点取出，将最后一个节点放在二叉树的首部，重新转化为完全二叉树，此时二叉树的长度为n-1。
- 依次进行类推，直到数组的长度为1.则排序完成

### 3.代码实现

#### 3.1.首先将二叉树的一个节点进行转化，保证父节点永远大于子节点。

> 当前节点为 i, 则其左子节点为 int c1 = 2 * i + 1; 右子节点为 int c2=2*i + 2

```java
/**
* 其中i代表的是当前节点，n代表的是一共有这些节点。
* 之所以使用递归，如果num[i]不是最大值，将以子节点为当前节点进行递归，保证这个子节点的下面所有字节都
* 符合大顶堆的原则
*/
private void swapSort(int[] nums, int i, int n) {
        if (i >= n) {
            return;
        }
        int c1 = 2 * i + 1;
        int c2 = 2 * i + 2;
        int max = i;
        if (c1 < n && (nums[c1] > nums[max])) {
            max = c1;
        }
        if (c2 < n && (nums[c2] > nums[max])) {
            max = c2;
        }
        if (max != i) {
            swap(nums, i, max);
            swapSort(nums, max, n);
        }
    }
}
// 交换节点
private void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

#### 3.2  构建大顶堆

构建大顶堆，需要从最后一个元素开始构建。

​    ![图片](https://gitee.com/anqingjieer/pengbo/raw/master/20200509094659.png)

构建完全二叉树需要从最后一个节点的父节点依次往上递减，这样就可以将所有的父节点和子节点构建完毕。

```java
private void BuildHeapSort(int[] nums) {
    int last = nums.length - 1;//最后一个子节点
    int parent = (last - 1) / 2;//通过子节点找到父节点
    // 初始化大定堆
    for (int i = parent; i >= 0; i--) {
        swapSort(nums, i, last);
    }
}
```

#### 3.3 进行排序

```java
private int[] heapSort(int[] nums) {
    BuildHeapSort(nums);//构建大顶堆
    for (int i = nums.length - 1; i >= 0; i--) {
        swap(nums, i, 0);//交换位置
        //// 重新构建大顶堆，这个方法不用去除最后一个元素，因为i递减，最大的数不会发生改变。
        swapSort(nums, 0, i);
    }
    return nums;
}
```

### 4.时间复杂度和空间复杂度

堆排序的时间复杂度为建堆    O(n)+n-1    次    O(nlog2n)=O(nlog2n)

## 四、归并排序

### 1 什么是归并排序？

​		归并排序利用的是分治的思量，将数组划分成单个的数字，然后两两进行比较排序，最后进行合并处理。

### 2.代码实现？

想起来很容易但是实现起来还有有点难度？

首先进行一一划分，如果是8位数字，先排序 num[0]和num[1,然后开始排序num[]2]和num[3]，最后将这四个数字进行两两排序

* ①首先将整个序列的每个关键字看成一个单独的有序的子序列
* ②两两归并，49和38归并成{38 49} ，65和97归并成{65 97}，76和13归并成{13 76}，27没有归并对象
* ③两两归并，{38 49}和{65 97}归并成{38 49 65 97}，{13,76}和27归并成{13 27 76}
* ④两两归并，{38 49 65 97}和{13 27 76}归并成{13 27 38 49 65 76 97}

```java
public static void mergeSort(int[] arrays, int L, int R) {
    //如果只有一个元素，那就不用排序了
    if (L == R) {
        return;
    } else {
        //取中间的数，进行拆分
        int M = (L + R) / 2;
        //左边的数不断进行拆分
        mergeSort(arrays, L, M);
        //右边的数不断进行拆分
        mergeSort(arrays, M + 1, R);
        //合并
        merge(arrays, L, M + 1, R);
    }
}
    public static void merge(int[] arrays, int L, int M, int R) {
        //左边的数组的大小
        int[] leftArray = new int[M - L];
        //右边的数组大小
        int[] rightArray = new int[R - M + 1];
        //往这两个数组填充数据
        for (int i = L; i < M; i++) {
            leftArray[i - L] = arrays[i];
        }
        for (int i = M; i <= R; i++) {
            rightArray[i - M] = arrays[i];
        }
        int i = 0, j = 0;
        // arrays数组的第一个元素
        int k = L;
        //比较这两个数组的值，哪个小，就往数组上放
        while (i < leftArray.length && j < rightArray.length) {

            //谁比较小，谁将元素放入大数组中,移动指针，继续比较下一个
            if (leftArray[i] < rightArray[j]) {
                arrays[k] = leftArray[i];

                i++;
                k++;
            } else {
                arrays[k] = rightArray[j];
                j++;
                k++;
            }
        }
        // 排序完成之后肯定会出现i或者j先拍完，这时候只需将没有拍完的数字复制过这个大数组
        //到这边肯定如果左边的数组还没比较完，右边的数都已经完了，那么将左边的数抄到大数组中(剩下的都是大数字)
        while (i < leftArray.length) {
            arrays[k] = leftArray[i];

            i++;
            k++;
        }
        //如果右边的数组还没比较完，左边的数都已经完了，那么将右边的数抄到大数组中(剩下的都是大数字)
        while (j < rightArray.length) {
            arrays[k] = rightArray[j];

            k++;
            j++;
        }
    }
```

### 3.时间复杂度、空间复杂度

O（nLog n）空间复杂度：因为开辟了一个新的空间所以是  O（n）

稳定性：因为每次都会进行比较，所以很稳定。

## 五、快速排序

### 1.什么是快速排序？

​        通过分治进行快速排序。每一趟快速排序用任意一个数字作为枢纽（通常是第一个元素），将序列中比枢纽小的放在左边，大的放在右边

### 2.代码实现？

​		很难受，凡是涉及到递归的都不太好理解。总的思路可以分为以下几步。

1. 找一个基准数，一般用第一个数开始作为基准数，最左边和在最右边同时开始，  右边开始找一个比基准数小的，记录位置，左边开始找一个比基准数字大的。然后这两个数字交换位置。
2. 然后在找到小数和大数之间开始继续递归，直直到  i=j，将基准数插入到两个之间
3. 然后递归，上一个基准数位置，左边的进行排序。最后执行右边的进行排序。

```java
private void partition(int[] a, int left, int right) {
    int i, j, t, temp;
    if(left > right)
        return;
    temp = a[left]; //temp中存的就是基准数
    i = left;
    j = right;
    while(i != j) { //顺序很重要，要先从右边开始找
        while(a[j] >= temp && i < j)
            j--;
        while(a[i] <= temp && i < j)//再找右边的
            i++;
        if(i < j)//交换两个数在数组中的位置
        {
            t = a[i];
            a[i] = a[j];
            a[j] = t;
        }
    }
    //最终将基准数归位
    a[left] = a[i];
    a[i] = temp;
    partition(a,left, i-1);//继续处理左边的，这里是一个递归的过程
    partition(a,i+1, right);//继续处理右边的 ，这里是一个递归的过程
}
```

> ![](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200509152243.png)

> 图片来源：https://blog.csdn.net/pengzonglu7292/article/details/84938910

