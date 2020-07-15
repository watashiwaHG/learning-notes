两次异或操作变回原值 m^n^n = m
# 链表
#### 类型（有无头结点）
1. 单向链表
2. 双向链表
3. 环形链表
#### 特点 
增删快，查询慢
#### 存储方式
1. 头插法（逆序）
2. 尾插法（正序）
****
# 栈
#### 特点
先进后出
#### 应用场景
1. 逆序
2. 存储一些有关联和顺序的数据
    * 计算器
    * 虚拟机栈
****
# 递归
#### 特点 
方法自己调用自己
解决一些重复的，有规律的问题
一般都是结合栈（虚拟机栈）来使用，可以使操作回溯
#### 注意停止的条件和调用的位置
1. 调用位置在判断条件中，返回值为boolean类型，经常用来实现回溯，当递归到某一步如果返回false，那么就会返回到上一步并进入到下一个判断条件或者之后的语句中。直到第一次调用的此方法执行完毕为止。
    * 迷宫问题
    * 八皇后问题 
2. 调用位置在return语句后，返回值为数，经常用来解决一些有规律的数学问题，当到某一条件时，return后返回一个常数，递归停止。
    * 求阶乘
    * 斐波那契数列
#### 调用递归之前的语句顺序执行，递归之后的语句逆序执行 
****
# 排序
#### 时间复杂度大小
![](./复习/时间复杂度大小排列.png)
#### log2n
```java
i = 1;
while(i < n){
    i = i * 2;
}
```
#### nlog2n
```java
for(int i = 0; i < n; i++){
    int j = 1;
    while(j < n){
        j = j * 2;
    }
}
```
#### 排序算法的时间复杂度
![](./复习/排序的时间复杂度.png)
## 冒泡排序
#### 排序方式
每次从0开始，到length - 1 - i
相邻两两比较，前者更大（小）则交换两数，直到把当前序列最大（小）值放到最后
```java
boolean flag = true;

for (int i = 0; i < array.length - 1; i++){
    if(!flag){
        break;
    }
    flag = false;
    for (int j = 0; j < array.length - 1 - i; j++){
        if(array[j + 1] < array[j]){
            int temp = array[j + 1];
            array[j + 1] = array[j];
            array[j] = temp;
            flag = true;
        }
    }
}
```
#### 优化
可设定一个标记来标记此次遍历是否有元素交换，如果没有则说明当前序列已有序，可以跳出循环
#### 特点
因为可能频繁交换，所以速度相对来说较慢
## 选择排序
#### 排序方式
每次从i开始，到length
遍历当前序列找到最小（大）值，将他与索引为i的元素交换
```java
int index;
int min;

for (int i = 0; i < array.length - 1; i++) {
    index = i;
    min = array[i];

    for (int j = i + 1; j < array.length; j++) {
        if(array[j] < min){
            min = array[j];
            index = j;
        }
    }

    if(index != i){
        array[index] = array[i];
        array[i] = min;
    }
}
```
#### 优化
如果最小（大）值的索引与当前i相同，则不需要交换
#### 特点
每次最多交换一次，速度相对来说比冒泡快一些，但不稳定
## 插入排序
每次从i - 1开始，到-1
将序列分为有序列和无序列，每次将无序列中的第一个数从后向前与有序列中的数进行比较，如果待插入的数小（大）于当前有序列所比较的数，则将当前有序列所比较的数向后移一位，否则就插入有序列中
```java
int temp;

for (int i = 1; i < array.length; i++) {
    temp = array[i];
    int j;
    for (j = i - 1; j >= 0; j--) {
        if(array[j] > temp){
            array[j + 1] = array[j];
        }else {
            break;
        }
    }
    array[j + 1] = temp;
}
```
#### 特点
速度比选择快，并且稳定
## 希尔排序
#### 排序方式
每次分数量/2的组数分别进行插入排序
```java
int i = array.length / 2;
int temp;

while (i != 0) {
    //对每组进行遍历
    for (int j = i; j < array.length; j++) {
        temp = array[j];
        int k;
        //对组中元素进行插入排序
        for (k = j - i; k >= 0; k -= i) {
            if (temp < array[k]) {
                array[k + i] = array[k];
            } else {
                break;
            }
        }
        array[k + i] = temp;
    }
    i /= 2;
}
```
#### 特点
插入排序的进化版，比插入排序还快，但是不稳定了
## 快速排序
#### 排序方法
以数列中一个数为基准，将小于他的数放在左边，大于他的数放在右边，再分别对左边和右边的数列进行此步骤，利用了递归
```java
public static void quikSort(int[] array,int left,int right){
    //当前数列中的数小于1就不用再排序了
    if(right - left < 1){
        return;
    }else {
        //以最左边的数为基准
        int mid = array[left];
        int temp = 0;
        int i = left;
        int j = right;

        //当i和j指向同一个数时只有两种情况，j找到比基准数小的数，或者j指向了基准数
        while (i != j){
            //寻找比基准数小的数
            while (array[j] >= mid && i < j){
                j--;
            }
            //寻找比基准数大的数
            while (array[i] <= mid && i < j){
                i++;
            }
            //如果i和j指向同一个数则不用交换
            if(i != j){
                temp = array[i];
                array[i] = array[j];
                array[j] = temp;
            }
        }

        array[left] = array[i];
        array[i] = mid;
        
        quikSort(array,left,i - 1);
        quikSort(array,i + 1,right);
    }
}
```
#### 特点
使用了递归算法，牺牲了空间，在数量小的情况下速度和希尔排序差不多，数量大时快速排序更快，但也不稳定
## 归并排序
#### 排序方法
先将数组从中间分开，再将分开后的两个数组继续此操作，直到数组中只含有一个元素，再将分开后数组排序形成一个数组，直到数组的长度变回原始数组的长度
```java
//先进行分操作
public static void mergeSort(int[] array,int left,int right,int[] temp){
    if(left == right){
        return;
    }else {
        int mid = (right + left) / 2;
        //对分好的左边数组继续分
        mergeSort(array,left,mid,temp);
        //对分好的右边数组继续分
        mergeSort(array,mid + 1,right,temp);
        //对分好的两个数组进行合并
        merge(array,left,mid,right,temp);
    }
}

public static void merge(int[] array,int left,int mid,int right,int[] temp){
    int i = left;
    int j = mid + 1;
    int t = 0;
    //直到有一个数组的元素全部放进temp数组时退出循环
    while (i <= mid && j <= right){
        //当左边数组的数字小于右边数组时，将左边数组的数字放入temp数组中
        if(array[i] < array[j]){
            temp[t] = array[i];
            t++;
            i++;
        }else {//否则将右边数组的数字放入temp数组中
            temp[t] = array[j];
            t++;
            j++;
        }
    }
    //如果左边数组还有剩余，全部放入temp数组
    while (i <= mid){
        temp[t] = array[i];
        t++;
        i++;
    }
    //如果右边数组还有剩余，全部放入temp数组
    while (j <= right){
        temp[t] = array[j];
        t++;
        j++;
    }
    //将temp数组中已经排好序的数字放回array数组中
    int index = left;
    t = 0;
    while (index <= right){
        array[index] = temp[t];
        t++;
        index++;
    }
}
```
#### 特点
使用了两次递归，分别是递归分和递归合，将数组分开后进行合，因为也是用空间换时间，速度非常快，并且都是对相邻数操作，也很稳定。
## 基数排序
#### 排序方法
对每个数从低位开始依次排序，将每次的排序结果放回数组
```java
public static void radixSort(int[] arr){
    int[][] bucket = new int[10][arr.length];//用来放0-9不同位数的桶
    int[] bucketCount = new int[10];//记录每个桶中的元素个数
    //求数组中最大数的位数
    int max = arr[0];
    for (int i = 1; i < arr.length; i++){
        if(arr[i] > max){
            max = arr[i];
        }
    }
    int maxLength = (max + "").length();
    //根据位数循环
    for (int i = 0, n = 1; i < maxLength; i++, n *= 10){
        for (int j = 0; j < arr.length; j++){
            int digitOfNum = arr[j] / n % 10;//根据循环次数取不同位上的数
            bucket[digitOfNum][bucketCount[digitOfNum]] = arr[j];//存入桶中
            bucketCount[digitOfNum]++;//计数加一
        }
        //重新放回原数组
        int index = 0;
        for (int k = 0; k < bucket.length; k++){
            if(bucketCount[k] != 0){
                for (int l = 0; l <bucketCount[k]; l++){
                    arr[index] = bucket[k][l];
                    index++;
                }
                bucketCount[k] = 0;//计数归零
            }
        }
    }
}
```
#### 特点
思路简单，不需要递归，速度非常快，但因为创建桶的原因，非常占内存，完全以空间换时间，也稳定。
## 堆排序
#### 排序方法
完全二叉树的应用，从后数第一个非叶子节点开始排序，使完全二叉树满足每个父节点都大于两个子节点，再将堆顶最大的数放在最后，在对剩下的数进行排列。
```java
public static void heapSort(int[] array){
    //对数组第一次排序
    for (int i = (array.length - 1) / 2; i > 0; i--){
        adjustHeap(array,i,array.length);
    }

    int temp = 0;
    //将堆顶元素与堆的最后元素交换，因为整体已经排好，只有堆顶元素改变，所以再对堆顶进行排序即可
    for (int j = array.length - 1; j > 0; j--){
        temp = array[j];
        array[j] = array[0];
        array[0] = temp;
        adjustHeap(array,0,j);
    }
}

public static void adjustHeap(int[] array,int i,int length){
    //记录当前完全二叉树的堆顶
    int temp = array[i];
    //对子树遍历
    for (int k = i * 2 + 1; k < length; k = k * 2 + 1){
        //判断是否有右子树，找出子树中最大的
        if (k + 1 < length && array[k] < array[k + 1]){
                k++;
        }
        //如果父节点小于子树中最大的就将子树的值赋值给父节点
        if (temp < array[k]){
            //让父节点的索引为此子节点
            array[i] = array[k];
            i = k;
        }else {
            break;
        }
    }
    //最后将堆顶的值赋值给最后的节点
    array[i] = temp;
}
```
***
# 查找
## 二分查找算法
#### 查找方法
找到数组的中间值与查找的数比较，如果小于查找的数，就在右边的范围找，如果大于则在左边的范围找
```java
public static int binarySearch(int[] array,int left,int right,int findVal){
    if(left > right){
        return -1;
    }

    int mid = (left + right) / 2;
    if(findVal == array[mid]){
        return mid;
    }else if (findVal > array[mid]){
        return binarySearch(array,mid + 1,right,findVal);
    }else {
        return binarySearch(array,left,mid - 1,findVal);
    }
}
//如果数组中有重复的数并且需要全部查出来
public static List<Integer> binarySearch2(int[] array, int left, int right, int findVal){
    if(left > right){
        return null;
    }

    int mid = (left + right) / 2;
    if(findVal == array[mid]){
        List<Integer> indexArr = new ArrayList<>();
        indexArr.add(mid);
        int i = -1;
        if(mid + i >= left){
            while (array[mid + i] == findVal){
                indexArr.add(mid + i);
                i--;
            }
        }

        i = 1;
        if(mid + i <= right){
            while (array[mid + i] == findVal){
                indexArr.add(mid + i);
                i++;
            }
        }

        return indexArr;
    }else if (findVal > array[mid]){
        return binarySearch2(array,mid + 1,right,findVal);
    }else {
        return binarySearch2(array,left,mid - 1,findVal);
    }
}
```
#### 特点
所查找的数组必须是排序好的，查找中间值快，边缘值慢
## 插值查找算法
#### 查找方法
计算所查找数在数组中的大概分布位置（有点归一法的感觉），来替换二分查找的1/2
```java
public static int insertValueSearch(int[] array,int left,int right,int findValue){
    if(left > right || findValue < array[0] || findValue > array[array.length - 1]){
        return -1;
    }

    int mid = left + (findValue - array[left]) * (right - left) / (array[right] - array[left]) ;

    if(array[mid] == findValue){
        return mid;
    }else if (array[mid] > findValue){
        return insertValueSearch(array,left,mid - 1,findValue);
    }else {
        return insertValueSearch(array,mid + 1,right,findValue);
    }
}
```
#### 特点
所查找的数组必须是排序好的，数据量较大，关键字分布比较均匀的查找表，使用插值查找算法比较快，但是关键字分布不均匀的情况下，效果不一定比二分查找好。
## 斐波那契查找算法
#### 查找方法
用斐波那契数列（黄金分割比）代替查找的位置
```java
//求斐波那契数列
public static int[] fib(){
    int[] f = new int[20];
    f[0] = 1;
    f[1] = 1;
    for (int i = 2; i < f.length; i++){
        f[i] = f[i - 1] + f[i - 2];
    }

    return f;
}
//查找算法
public static int fibSearch(int[] array,int findVal){
    int left = 0;
    int right = array.length - 1;
    int mid = 0;
    int[] f = fib();

    int k = 0;
    while (array.length > f[k]){
        k++;
    }

    int[] temp = Arrays.copyOf(array,f[k]);
    for (int i = right; i < temp.length; i++){
        temp[i] = array[right];
    }

    while (left <= right){
        mid = left + f[k - 1] - 1;

        if(temp[mid] == findVal){
            if(mid > right){
                return right;
            }else {
                return mid;
            }
        }else if(temp[mid] > findVal){
            right = mid - 1;
            k--;
        }else {
            left = mid + 1;
            k -= 2;
        }
    }

    return  -1;
}
```
#### 特点
依然是针对有序数组的查找，每次查找的位置不和查找值有关，而是和斐波那契数列（黄金分割比）有关，并且查找的数组长度也需满足斐波那契数列，如果不够则需要扩容，新扩出来的位置用数组中最大数填充，返回索引时也需要判断是否是新扩出来的索引。
# 树
## 二叉树
![](./复习/二叉树示意图.png)
### 二叉树的遍历方式
* 前序遍历（先遍历父节点，再遍历左子节点，最后遍历右子节点）
```java
public void preOrder(){
    System.out.println(this);
    if(this.left != null){
        this.left.preOrder();
    }
    if(this.right != null){
        this.right.preOrder();
    }
}
```
* 中序遍历（先遍历左子节点，再遍历父节点，最后遍历右子节点）
```java
public void midOrder(){
    if (this.left != null){
        this.left.midOrder();
    }
    System.out.println(this);
    if (this.right != null){
        this.right.midOrder();
    }
}
```
* 后序遍历（先遍历左子节点，再遍历右子节点，最后遍历父节点）
```java
public void postOrder(){
    if (this.left != null){
        this.left.postOrder();
    }

    if (this.right != null){
        this.right.postOrder();
    }

    System.out.println(this);
}
```
### 二叉树查找
* 前序查找
```java
public HandSome preSelect(int no){
    HandSome hs = null;

    System.out.println("preSelect");
    if(this.getNo() == no){
        return this;
    }

    if (this.getLeft() != null && (hs = this.left.preSelect(no)) != null){
        return hs;
    }

    if (this.getRight() != null && (hs = this.right.preSelect(no)) != null){
        return hs;
    }

    return null;
}
```
* 中序查找
```java
public HandSome midSelect(int no){
    HandSome hs = null;

    if (this.getLeft() != null && (hs = this.left.midSelect(no)) != null){
        return hs;
    }

    System.out.println("midSelect");
    if(this.getNo() == no){
        return this;
    }

    if (this.getRight() != null && (hs = this.right.midSelect(no)) != null){
        return hs;
    }

    return null;
}
```
* 后序查找
```java
public HandSome postSelect(int no){
    HandSome hs = null;

    if (this.getLeft() != null && (hs = this.left.postSelect(no)) != null){
        return hs;
    }

    if (this.getRight() != null && (hs = this.right.postSelect(no)) != null){
        return hs;
    }

    System.out.println("postSelect");
    if(this.getNo() == no){
        return this;
    }
```
### 顺序存储二叉树
用数组存储完全二叉树
![](./复习/顺序存储二叉树.png)
* 前序遍历
```java
public static void preOrder(int[] array,int father){
    System.out.println(array[father]);
    if (father * 2 + 1 < array.length){
        preOrder(array,father * 2 + 1);
    }

    if (father * 2 + 2 < array.length){
        preOrder(array,father * 2 + 2);
    }
}
```
### 线索二叉树
* n个节点的二叉链表中含有n+1【公式2n-(n-1)=n+1】个空指针域。利用二叉链表中的空指针域，存放指向该节点在某种遍历次序下的前驱和后继节点的指针（这种附加的指针成为“线索”）
* 这种加上了线索的二叉链表称为线索链表，相应的二叉树称为线索二叉树（Threaded BinaryTree）。根据线索性质的不同，线索二叉树可分为前序线索二叉树、中序线索二叉树和后序线索二叉树三种。
* 一个节点的前一个节点，称为前驱节点
* 一个节点的后一个节点，称为后继节点
#### 中序线索化
```java
public void threadedNodes(HandSome hs){
    if (hs == null){
        return;
    }

    threadedNodes(hs.getLeft());
    if (hs.getLeft() == null){
        hs.setLeft(pre);
        hs.setLeftType(1);
    }

    if (pre != null && pre.getRight() == null){
        pre.setRight(hs);
        pre.setRightType(1);
    }

    pre = hs;

    threadedNodes(hs.getRight());
}
```
#### 中序线索遍历
```java
public void threadedMidOrder(){
    HandSome temp = root;

    while (temp != null){
        while (temp.getLeftType() == 0){
            temp = temp.getLeft();
        }

        System.out.println(temp);

        while (temp.getRightType() == 1){
            temp = temp.getRight();
            System.out.println(temp);
        }

        temp = temp.getRight();
    }
}
```
## 哈夫曼树
给定n个权值作为n个 ***叶子节点*** ，构造一颗二叉树，若该树的带权路径长度（wpl）达到最小，称这样的二叉树为最优二叉树，也成为哈夫曼树。
哈夫曼树是带权路径长度最短的树，权值较大的节点离根较近。
```java
public static Node huffmanTree(int[] array){
    //将数组的元素封装成Node放到ArrayList中
    List<Node> nodeList = new ArrayList<>();
    for (int value : array){
        nodeList.add(new Node(value));
    }
    //对ArrayList进行从小到大排序
    Collections.sort(nodeList);
    //当ArrayList中只含有一个元素代表哈夫曼树生成完毕
    while (nodeList.size() != 1){
        //取出最小的两个节点
        Node leftNode = nodeList.get(0);
        Node rightNode = nodeList.get(1);
        //生成一颗二叉树，父节点的值为这两个节点值的和，左右子节点分别是这两个节点
        Node parentNode = new Node(leftNode.getValue() + rightNode.getValue());
        parentNode.setLeft(leftNode);
        parentNode.setRight(rightNode);
        //从ArrayList中删除这两个节点，并把新生成的二叉树的根节点放入ArrayList
        nodeList.remove(leftNode);
        nodeList.remove(rightNode);
        nodeList.add(parentNode);
        //对ArrayList进行排序
        Collections.sort(nodeList);
    }
    return nodeList.get(0);
}
```
### 哈夫曼编码
哈夫曼树的应用，把有效的元素放在叶子节点保证编码不会出现前缀问题，将权值大的放在离根节点近的地方保证压缩的字节更少。
```java
//将字符串的内容进行哈夫曼编码
public static byte[] toHuffmanCode(String str){
    //将字符串中的所有字符转换为哈夫曼编码
    byte[] bytes = str.getBytes();
    StringBuilder sb = new StringBuilder();
    for (byte by : bytes){
        sb.append(huffmanCode.get(by));
    }
    //根据哈夫曼编码的长度创建字节数组
    int len = 0;
    if (sb.length() % 8 == 0){
        len = sb.length() / 8;
    }else {
        len = sb.length() / 8 + 1;
    }
    //将哈夫曼编码每八位转换为一个字节放入字节数组
    byte[] codeBytes = new byte[len];
    int index = 0;
    for (int i = 0; i < sb.length(); i += 8){
        if (i + 8 > sb.length()){
            codeBytes[index] = (byte)Integer.parseInt(sb.substring(i),2);
        }else {
            codeBytes[index] = (byte)Integer.parseInt(sb.substring(i,i + 8),2);
        }
        index++;
    }

    return codeBytes;
}
//对哈夫曼树中的叶子节点进行编码
public static void getCode(Node node,StringBuilder code,StringBuilder stringBuilder){
    if (node == null){
        return;
    }
    //生成该节点的编码
    StringBuilder newCode = new StringBuilder(stringBuilder);
    newCode.append(code);
    //如果该节点为叶子节点，则存入该节点的编码
    if (node.getData() != null){
        huffmanCode.put(node.getData(),newCode.toString());
    }else {
        //否则继续对子节点编码
        getCode(node.getLeft(),new StringBuilder("0"),newCode);
        getCode(node.getRight(),new StringBuilder("1"),newCode);
    }
}
//将字符串中的字符转成哈夫曼树
public static Node createHuffmanTree(String str){
    byte[] bytes = str.getBytes();
    //计算出每个共有多少个字符，每个字符出现多少次，也就是他的权值
    Map<Byte,Integer> counts = new HashMap<>();

    for (Byte bt : bytes){
        Integer count = null;
        if ((count = counts.get(bt)) == null){
            counts.put(bt,1);
        }else {
            counts.put(bt,count + 1);
        }
    }
    //将每个字符封装为节点，data为字符的字节数，weight为出现的次数
    List<Node> nodeList = new ArrayList<>();
    for (Map.Entry<Byte,Integer> entry : counts.entrySet()){
        nodeList.add(new Node(entry.getKey(),entry.getValue()));
    }
    //将节点转换为哈夫曼树
    while (nodeList.size() > 1){
        Collections.sort(nodeList);
        Node leftNode = nodeList.get(0);
        Node rightNode = nodeList.get(1);
        Node parentNode = new Node(null,leftNode.getWeight() + rightNode.getWeight());
        parentNode.setLeft(leftNode);
        parentNode.setRight(rightNode);
        nodeList.remove(leftNode);
        nodeList.remove(rightNode);
        nodeList.add(parentNode);
    }

    return nodeList.get(0);
}
```
### 哈夫曼解码
```java
//将哈夫曼编码的字节数组转为真正的字符串
public static String decode(byte[] huffmanCodeBytes){
    StringBuilder sb = new StringBuilder();
    //将字节转为二进制字符串
    for (int i = 0; i < huffmanCodeBytes.length; i++){
        if (i == huffmanCodeBytes.length - 1){
            sb.append(byteToString(false,huffmanCodeBytes[i]));
        }else {
            sb.append(byteToString(true,huffmanCodeBytes[i]));
        }
    }
    //将字符串中的编码与哈弗曼编码表进行匹配，存入ArrayList中
    int index = 0;
    List<Byte> byteList = new ArrayList<>();
    //当索引大于字符串的长度时退出
    while (index < sb.length()){
        int offSet = 1;
        //标记是否找到匹配的编码
        boolean flag = false;
        while (true){
            //从索引开始截取，长度为offset的字符串
            String str = sb.substring(index,index + offSet);
            for (Map.Entry<Byte,String> entry : huffmanCode.entrySet()){
                //找到匹配的编码就放入ArrayList中
                if (entry.getValue().equals(str)){
                    byteList.add(entry.getKey());
                    flag = true;
                    break;
                }
            }
            //找到匹配的编码就跳出循环
            if (flag){
                break;
            }
            offSet++;
        }
        //索引后移
        index += offSet;
    }
    System.out.println(byteList);
    //再将ArrayList中byte存入byte数组中
    byte[] bytes = new byte[byteList.size()];
    for (int i = 0; i < byteList.size(); i++){
        bytes[i] = byteList.get(i);
    }
    return new String(bytes);
}
//将字节转为二进制字符串
public static String byteToString(boolean flag,byte b){
    int temp = (int)b;
    //将不满八位的字节补零，其实就是转成int型，把第九位变为1，flag判断是否为最后一个字节
    if (flag){
        //按位或运算
        temp |= 256;
    }
//        temp += 256;
    String str = Integer.toBinaryString(temp);
    if (flag){
        //转为int型后只要后八位
        return str.substring(str.length() - 8);
    }else {
        return str;
    }
}
```
## 排序二叉树（BST）
在二叉树的基础上，规定左子节点的值小于父节点，右子节点的值大于父节点
### 向排序二叉树添加节点
```java
public void add(Node node) {
    if (node == null) {
        return;
    }

    if (this.root == null) {
        this.root = node;
        return;
    }

    Node temp = root;

    while (true) {
        if (temp.getValue() > node.getValue()) {
            if (temp.getLeft() == null) {
                temp.setLeft(node);
                break;
            } else {
                temp = temp.getLeft();
            }
        } else {
            if (temp.getRight() == null) {
                temp.setRight(node);
                break;
            } else {
                temp = temp.getRight();
            }
        }
    }
}
```
### 删除排序二叉树中的节点
```java
//分三种情况 1.删除的节点为叶子节点 2.删除的节点只有一个子树 3.删除的节点有两个子树
//要知道被删除节点的父节点，还要知道被删除的节点是父节点的哪个子节点
public void delete(Node node) {
    if (this.root == null) {
        throw new RuntimeException("当前排序二叉树为空");
    }

    if (node == null) {
        throw new RuntimeException("不能传null");
    }

    Node parent = this.root;
    Node target = this.root;

    if (node.getValue() == target.getValue()) {
        if (target.getRight() != null ) {
            moveNode(target, true);
        } else if (target.getLeft() != null) {
            moveNode(target, false);
        } else {
            this.root = null;
        }
    } else {
        boolean who = true;
        while (target != null) {
            if (node.getValue() > target.getValue()) {
                parent = target;
                target = target.getRight();
                who = true;
            } else if (node.getValue() < target.getValue()) {
                parent = target;
                target = target.getLeft();
                who = false;
            } else {
                if (target.getLeft() == null && target.getRight() == null) {
                    if (who) {
                        parent.setRight(null);
                    } else {
                        parent.setLeft(null);
                    }
                } else if (target.getLeft() != null && target.getRight() != null) {
                    moveNode(target, true);
                } else if (target.getLeft() == null) {
//                        moveNode(target, true);
                    if (who) {
                        parent.setRight(target.getRight());
                    } else {
                        parent.setLeft(target.getRight());
                    }
                } else {
//                        moveNode(target, false);
                    if (who) {
                        parent.setRight(target.getLeft());
                    } else {
                        parent.setLeft(target.getLeft());
                    }
                }
                break;
            }
        }
    }
}

private void moveNode(Node node, boolean which) {
    Node temp = null;
    if (which) {
        temp = node.getRight();
        Node parent = node;
        while (temp.getLeft() != null) {
            parent = temp;
            temp = temp.getLeft();
        }
        node.setValue(temp.getValue());
        if (temp.getRight() != null) {
            moveNode(temp, true);
        } else if (temp.getLeft() != null) {
            moveNode(temp, false);
        } else {
            if (parent == node){
                parent.setRight(null);
            }else {
                parent.setLeft(null);
            }
        }
    } else {
        temp = node.getLeft();
        Node parent = node;
        while (temp.getRight() != null) {
            parent = temp;
            temp = temp.getRight();
        }
        node.setValue(temp.getValue());
        if (temp.getRight() != null) {
            moveNode(temp, true);
        } else if (temp.getLeft() != null) {
            moveNode(temp, false);
        } else {
            if (parent == node){
                parent.setLeft(null);
            }else {
                parent.setRight(null);
            }

        }
    }
}
```
## 多叉树
一个节点有多个子节点的树

节点的度：节点拥有的子节点

树的度：树中子节点最多的个数
### B树
排序的，每个节点的子节点不是为空就是为满，所有叶子节点在同一层

数据既放在叶子节点，也放在非叶子节点
![](./复习/B树.png)
### B+树
B树的变体

叶子节点为链表，数据只放在叶子节点中,并且是有序的
![](./复习/B+树.png)
### B*树
B+树的变体

非叶子节点和非根节点也为链表，指向自己的兄弟节点

![](./复习/B星树.png)
## 图
表示多对多的关系
### 图的概念
1. 顶点
2. 边
3. 路径 （从A->D的路径）
4. 无向图
5. 有向图
6. 带权图
### 图的两种表示方式
二维数组表示（邻接矩阵）
![](./复习/图邻接矩阵.png)
链表表示（邻接表）

解决邻接矩阵空间浪费问题，只关心存在的边
![](./复习/图邻接表.png)
### 深度优先遍历
从一个点开始找与他相连的点，递归向下遍历，直到所有的点都被找到。
### 广度优先遍历
层次遍历，从一个点开始找到所有与相连的点，递归向下一个点遍历，直到所有的点都被找到。
## 常用算法
### 二分查找非递归
```java
public static int binarySearch(int[] arr,int target){
    int left = 0;
    int right = arr.length - 1;
    int mid = 0;
    int index = -1;

    while (left <= right){
        mid = (left + right) / 2;
        if (arr[mid] == target){
            index = mid;
            break;
        } else if (arr[mid] < target){
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return index;
}
```
### 汉诺塔
将汉诺塔看成两个部分，一个部分为最下面的圆盘，另一个部分为上面剩余的圆盘，对塔的移动整体分为三步

1) 将上面的圆盘放在第二根柱子
2) 将最下面的圆盘放在第三根柱子
3) 将上面的圆盘放在第三根柱子
```java
public static void hanoitower(int num,char a,char b,char c){
    //当只有一个圆盘的时候，直接移动
    if (num == 1){
        System.out.println(a + " => " + c);
        return;
    }
    //将上面的圆盘放在第二根柱子
    hanoitower(num - 1,a,c,b);
    //将最下面的圆盘放在第三根柱子
    hanoitower(1,a,b,c);
    //将上面的圆盘放在第三根柱子
    hanoitower(num - 1,b,a,c);
}
```
### 动态规划背包问题

                    重量        价值
    第一件商品        1          1500
    第二件商品        4          3000
    第三件商品        3          2000
背包容量为4

```java
int[] w = {0,1,4,3};
        int[] v = {0,1500,3000,2000};
        int[][] n = new int[4][5];
        int[][] path = new int[4][5];

        for (int i = 1; i < n.length; i++){
            for (int j = 1; j < n[i].length; j++){
                if (j < w[i]){//如果当前背包容量小于商品重量，就使用上一次的结果
                    n[i][j] = n[i - 1][j];
                }else {
                    if (n[i - 1][j] < v[i] + n[i - 1][j - w[i]]){
                        n[i][j] = v[i] + n[i - 1][j - w[i]];
                        path[i][j] = 1;
                    }else {
                        n[i][j] = n[i - 1][j];
                    }
                }
            }
        }

        for (int i = 0; i < n.length; i++){
            System.out.println(Arrays.toString(n[i]));
        }

        int i = path.length - 1;
        int j = path[0].length - 1;

        while (i > 0 && j > 0){
            if (path[i][j] == 1){
                System.out.println("放入第" + i + "件商品");
                j -= w[i];
            }
            i--;
        }
    }
```
### KMP算法
运用部分匹配表优化匹配字符串的效率（当与前面的字符出现重复时，记录前面字符的索引值）

A B C D A B D

0 0 0 0 1 2 0
```java
//kmp匹配算法
public static int kmpSearch(String str1,String str2,int[] next){
    int i = 0;
    int j = 0;
    for (; i < str1.length() && j < str2.length(); i++){
        //当不匹配时，根据部分匹配表将j移动到前面字符相同的下一个索引位置
        while (j > 0 && str1.charAt(i) != str2.charAt(j)){
            j = next[j - 1];
        }
        if (str1.charAt(i) == str2.charAt(j)){
            j++;
        }//else {
//                if (j != 0){
//                    i -= next[j - 1] + 1;
//                }
//                j = 0;
//            }

    }

    if (j == str2.length()){
        return i - j;
    }else {
        return -1;
    }
}
//计算字符的部分匹配表
public static int[] kmpNext(String str){
    int[] next = new int[str.length()];
    for (int i = 1,j = 0; i < str.length(); i++){
        while (j > 0 && str.charAt(i) != str.charAt(j)){
            j = next[j - 1];
        }
        if (str.charAt(i) == str.charAt(j)){
            j++;
        }
        next[i] = j;
    }
    return next;
}
```
### 贪心算法
每次的选择都为当前选择的最优解，结果一定满足需求，但是不一定是最优的。
```java
HashMap<String, HashSet<String>> broadcasts = new HashMap<>();

HashSet<String> hashSet1 = new HashSet<>();
hashSet1.add("北京");
hashSet1.add("上海");
hashSet1.add("天津");

HashSet<String> hashSet2 = new HashSet<>();
hashSet2.add("广州");
hashSet2.add("北京");
hashSet2.add("深圳");

HashSet<String> hashSet3 = new HashSet<>();
hashSet3.add("成都");
hashSet3.add("上海");
hashSet3.add("杭州");

HashSet<String> hashSet4 = new HashSet<>();
hashSet4.add("上海");
hashSet4.add("天津");

HashSet<String> hashSet5 = new HashSet<>();
hashSet5.add("杭州");
hashSet5.add("大连");

broadcasts.put("K1",hashSet1);
broadcasts.put("K2",hashSet2);
broadcasts.put("K3",hashSet3);
broadcasts.put("K4",hashSet4);
broadcasts.put("K5",hashSet5);

HashSet<String> allAreas = new HashSet<>();
allAreas.add("北京");
allAreas.add("上海");
allAreas.add("天津");
allAreas.add("广州");
allAreas.add("深圳");
allAreas.add("成都");
allAreas.add("杭州");
allAreas.add("大连");

ArrayList<String> selects = new ArrayList<>();

String maxKey = null;
while(allAreas.size() != 0){
    int max = 0;
    for (String key : broadcasts.keySet()){
        int count = 0;
        HashSet<String> set = broadcasts.get(key);
        Iterator<String> iterator = set.iterator();
        while (iterator.hasNext()){
            if(allAreas.contains(iterator.next())){
                count++;
            }
        }
        if (count > max){
            max = count;
            maxKey = key;
        }
    }
    selects.add(maxKey);
//            Iterator<String> iterator = broadcasts.get(maxKey).iterator();
//            while (iterator.hasNext()){
//                allAreas.remove(iterator.next());
//            }
    allAreas.removeAll(broadcasts.get(maxKey));
}
System.out.println(selects);
```
### Prim算法
求无向图的最小生成树

包含所有顶点，顶点式 - 1的边，边上权值和最小

每次都是选择局部最优解，结果为最优选择
```java
class Graph{
    public int verxs;//顶点数
    public char[] data;//顶点名称
    public int[][] weights;//邻接矩阵记录权值
    public Graph(int verxs,char[] data){
        //初始化定点数，顶点名称，邻接矩阵
        this.verxs = verxs;
        this.data = data;
        weights = new int[verxs][verxs];
        for (int i = 0; i < weights.length; i++){
            for (int j = 0; j < weights[i].length; j++){
                weights[i][j] = 10000;
            }
        }
    }
    //添加边
    public void setWeight(char a,char b,int weight){
        int index1 = 0;
        int index2 = 0;
        boolean flag1 = false;
        boolean flag2 = false;
        for (int i = 0; i < this.data.length; i++){
            if (a == this.data[i]){
                index1 = i;
                flag1 = true;
            }else if (b == this.data[i]){
                index2 = i;
                flag2 = true;
            }
            if (flag1 && flag2){
                break;
            }
        }
        this.weights[index1][index2] = weight;
        this.weights[index2][index1] = weight;
    }
    //获得最小生成树
    public void getMinTree(char a){
        //记录顶点是否已包含
        int[] isVisited = new int[this.verxs];
        int index = 0;
        int weight = 0;
        HashSet<Integer> indexSet = new HashSet<>();
        for (int i = 0; i < this.data.length; i++){
            if (this.data[i] == a){
                index = i;
                break;
            }
        }
        //将已包含的顶点索引存入indexSet中
        indexSet.add(index);
        isVisited[index] = 1;
        //如果包含的顶点数小于总顶点数，继续循环
        while (indexSet.size() < this.verxs){
            int minWeight = 100;
            int verxA = 0;
            int verxB = 0;
            //遍历以包含的顶点所连的边
            for (Integer integer : indexSet){
                for (int i = 0; i < this.weights.length; i++){
                    //找出权值最小并且不是回路的边
                    if (this.weights[integer][i] != 0 && isVisited[i] == 0){
                        if (this.weights[integer][i] < minWeight){
                            minWeight = this.weights[integer][i];
                            verxA = integer;
                            verxB = i;
                        }
                    }
                }
            }
            //将新找到的顶点存入indexSet中
            indexSet.add(verxB);
            System.out.println(data[verxA] + "->" + data[verxB]);
            //记录权值
            weight += this.weights[verxA][verxB];
            //将是否包含赋为1
            isVisited[verxB] = 1;
        }
        System.out.println(weight);
    }
    //展示邻接矩阵
    public void showWeights(){
        for (int[] weight : this.weights){
            System.out.println(Arrays.toString(weight));
        }
    }
}
```