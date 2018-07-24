
36. 二叉搜索树与双向链表  

题目要求：  
输入一颗二叉搜索树，将该二叉搜索树转换成一个排序的双向链表，不能创建任何新  
的节点，只能调整树中节点指针的指向。  

解题思路：  
二叉树的left、right可以看做双向链表的prev、next，因此可以完成二叉搜索树到  
双向链表的转换，关键是如何保证转换后为排序好的链表。  
最终生成的双向链表有序，而二叉搜索树的中序遍历就是有序的，可按照中序遍历的  
思路完成此题。关键思路如下：从根节点开始，让左子树部分的最后一节点，中，右  
子树的第一个节点，这三个节点完成双向链表的重组，然后对于左子树，右子树继续  
上述重组过程，递归完成。  

```java
package chapter4;
import structure.TreeNode;

/**
 * Created by ryder on 2017/7/18.
 * 二叉搜索树与双向链表
 * 将二叉搜索树转换为双向链表，树的left指向prev节点，树的right指向post节点
 * 左右支转换完之后要与根节点组合，所以左右支要返回自己的最小点与最大点两个节点，返回值使用数组
 */
public class P191_ConvertBinarySearchTree {
    public static TreeNode<Integer> convert(TreeNode<Integer> root){
        if(root==null)
            return null;
        TreeNode<Integer>[] result = convertCore(root);
        return result[0];
    }
    public static TreeNode<Integer>[] convertCore(TreeNode<Integer> node){
        //java不支持泛型数组，所以声明为这种形式
        TreeNode[] result = new TreeNode[2];
        if(node.left==null&&node.right==null){
            result[0] = node;
            result[1] = node;
        }
        else if(node.right==null){
            result = convertCore(node.left);
            node.left = result[1];
            result[1].right = node;
            result[1] = node;
        }
        else if(node.left==null){
            result = convertCore(node.right);
            node.right = result[0];
            result[0].left = node;
            result[0] = node;
        }
        else{
            TreeNode[] resultLeft = convertCore(node.left);
            TreeNode[] resultRight = convertCore(node.right);
            resultLeft[1].right = node;
            node.left = resultLeft[1];
            resultRight[0].left = node;
            node.right = resultRight[0];
            result[0] = resultLeft[0];
            result[1] = resultRight[1];
        }
        return result;

    }
    public static void main(String[] args){
        //            10
        //          /   \
        //         6     14
        //       /  \   / \
        //      4    8 12  16
        TreeNode<Integer> root = new TreeNode<Integer>(10);
        root.left = new TreeNode<Integer>(6);
        root.right = new TreeNode<Integer>(14);
        root.left.left = new TreeNode<Integer>(4);
        root.left.right = new TreeNode<Integer>(8);
        root.right.left = new TreeNode<Integer>(12);
        root.right.right = new TreeNode<Integer>(16);
        TreeNode<Integer> result = convert(root);
        //转化后不可再使用二叉树的层序遍历显示结果，层序遍历会进入无限循环。
        System.out.println(result.leftToRight());

    }
}
```

37：序列化二叉树  

题目要求：  
实现两个函数，分别用来序列化和反序列化二叉树。  

解题思路：
此题能让人想到重建二叉树。但二叉树序列化为前序遍历序列和中序遍历序列，  
然后反序列化为二叉树的思路在本题有两个关键缺点：1.全部数据都读取完才  
能进行反序列化。2.该方法需要保证树中节点的值各不相同（本题无法保证）。  
其实，在遍历结果中，记录null指针后（比如用一个特殊字符$），那么任何  
一种遍历方式都能回推出原二叉树。但是如果期望边读取序列化数据，边反序  
列化二叉树，那么仅可以使用前序或层序遍历。但层序记录的null个数要远多  
于前序，因此选择使用记录null指针的前序遍历进行序列化。  

```java
package chapter4;
import structure.TreeNode;

/**
 * Created by ryder on 2017/7/19.
 * 序列化二叉树
 */
public class P194_SerializeBinaryTrees {
    public static String serialize(TreeNode<Integer> root){
        if(root==null)
            return "$,";
        StringBuilder result = new StringBuilder();
        result.append(root.val);
        result.append(",");
        result.append(serialize(root.left));
        result.append(serialize(root.right));
        return result.toString();
    }
    public static TreeNode<Integer> deserialize(String str){
        StringBuilder stringBuilder = new StringBuilder(str);
        return deserializeCore(stringBuilder);
    }
    public static TreeNode<Integer> deserializeCore(StringBuilder stringBuilder){
        if(stringBuilder.length()==0)
            return null;
        String num = stringBuilder.substring(0,stringBuilder.indexOf(","));
        stringBuilder.delete(0,stringBuilder.indexOf(","));
        stringBuilder.deleteCharAt(0);
        if(num.equals("$"))
            return null;
        TreeNode<Integer> node = new TreeNode<>(Integer.parseInt(num));
        node.left = deserializeCore(stringBuilder);
        node.right = deserializeCore(stringBuilder);
        return node;
    }
    public static void main(String[] args){
        //            1
        //          /   \
        //         2     3
        //       /      / \
        //      4      5   6
        //    1,2,4,$,$,$,3,5,$,$,6,$,$
        TreeNode<Integer> root = new TreeNode<Integer>(1);
        root.left = new TreeNode<Integer>(2);
        root.right = new TreeNode<Integer>(3);
        root.left.left = new TreeNode<Integer>(4);
        root.right.left = new TreeNode<Integer>(5);
        root.right.right = new TreeNode<Integer>(6);
        System.out.println("原始树："+root);
        String result = serialize(root);
        System.out.println("序列化结果："+result);
        TreeNode<Integer> deserializeRoot = deserialize(result);
        System.out.println("反序列后的树："+deserializeRoot);
    }
}
```


38. 字符串的排列  

题目要求：  
输入一个字符串，打印出该字符串中字符的所有排列。如输入abc，则打印  
abc，acb，bac，bca，cab，cba。  

解题思路：  
排列与组合是数学上的常见问题。解题思路与数学上求排列总数类似：首先确定  
第一个位置的元素，然后一次确定每一个位置，每个位置确实时把所有情况罗列  
完全即可。以abc为例，我之前更习惯于设置三个空，然后选择abc中的元素放入  
上述的空中。而书中给的思路是通过交换得到各种可能的排列，具体思路如下：  

```
对于a,b,c（下标为0,1,2）  
0与0交换,得a,b,c => 1与1交换,得a,b,c =>2与2交换,得a,b,c(存入)
                => 1与2交换，得a,c,b =>2与2交换,得a,c.b(存入)
0与1交换,得b,a,c => 1与1交换,得b,a,c =>2与2交换,得b,a,c(存入)
                => 1与2交换，得b,c,a =>2与2交换,得b,c,a(存入)
0与2交换,得c,b,a => 1与1交换,得c,b,a =>2与2交换,得c,b,a(存入)
                => 1与2交换，得c,a,b =>2与2交换,得c,a.b(存入)  
```

书中解法并未考虑有字符重复的问题。对于有重复字符的情况如a,a,b,交换  
0,1号元素前后是没有变化的，即从生成的序列结果上看，是同一种排列，  
因此对于重复字符，不进行交换即可，思路如下：  

```
对于a,a,b（下标为0,1,2）
0与0交换,得a,a,b => 1与1交换,得a,a,b =>2与2交换,得a,a,b(存入)
                => 1与2交换，得a,b,a =>2与2交换,得a,b,a(存入)
0与1相同,跳过
0与2交换,得b,a,a =>1与1交换,得b,a,a =>2与2交换,得b,a,a(存入)
                =>1与2相同，跳过
```

考虑了字符重复的解法的实现如下  

```java
package chapter4;

import java.util.*;

/**
 * Created by ryder on 2017/7/22.
 * 字符串的排列
 */
public class P197_StringPermutation {
    public static List<char[]> permutation(char[] strs) {
        if (strs == null || strs.length == 0)
            return null;
        List<char[]> ret = new LinkedList<>();
        permutationCore(strs, ret, 0);
        return ret;
    }
    //下标为bound的字符依次与[bound,length)的字符交换，如果相同不交换，直到最后一个元素为止。
    //如a,b,c
    //0与0交换,得a,b,c => 1与1交换,得a,b,c =>2与2交换,得a,b,c(存入)
    //                => 1与2交换，得a,c,b =>2与2交换,得a,c.b(存入)
    //0与1交换,得b,a,c => 1与1交换,得b,a,c =>2与2交换,得b,a,c(存入)
    //                => 1与2交换，得b,c,a =>2与2交换,得b,c,a(存入)
    //0与2交换,得c,b,a => 1与1交换,得c,b,a =>2与2交换,得c,b,a(存入)
    //                => 1与2交换，得c,a,b =>2与2交换,得c,a.b(存入)

    //如a,a,b
    //0与0交换,得a,a,b => 1与1交换,得a,a,b =>2与2交换,得a,a,b(存入)
    //                => 1与2交换，得a,b,a =>2与2交换,得a,b,a(存入)
    //0与1相同,跳过
    //0与2交换,得b,a,a =>2与2交换，得b,a,a(存入)
    public static void permutationCore(char[] strs, List<char[]> ret, int bound) {
        if (bound == strs.length)
            ret.add(Arrays.copyOf(strs, strs.length));
        Set<Character> set = new HashSet<>();
        for (int i = bound; i < strs.length; i++) {
            if (set.add(strs[i])) {
                swap(strs, bound, i);
                permutationCore(strs, ret, bound + 1);
                swap(strs, bound, i);
            }
        }
    }

    public static void swap(char[] strs, int x, int y) {
        char temp = strs[x];
        strs[x] = strs[y];
        strs[y] = temp;
    }

    public static void main(String[] args) {
        char[] strs = {'a', 'b', 'c'};
        List<char[]> ret = permutation(strs);
        for (char[] item : ret) {
            for (int i = 0; i < item.length; i++)
                System.out.print(item[i]);
            System.out.println();
        }
        System.out.println();
        char[] strs2 = {'a', 'a', 'b','b'};
        List<char[]> ret2 = permutation(strs2);
        for (char[] item : ret2) {
            for (int i = 0; i < item.length; i++)
                System.out.print(item[i]);
            System.out.println();
        }
    }
}
```


38.2 字符串的组合  

题目要求：
输入一个字符串，打印出该字符串中字符的所有组合。如输入abc，  
则打印a，b，c，ab，ac，bc，abc。  

解题思路：  
这道题目是在38题.字符串的排列的扩展部分出现的。排列的关键在于次序，而组合  
的关键在于状态，即该字符是否被选中进入组合中。  
对于无重复字符的情况，以a,b,c为例，每一个字符都有两种状态：被选中、不被选中；  
2*2*2=8，再排除为空的情况，一共有7种组合：  

```
a(被选中)b(被选中)c(被选中)            =>    abc
a(被选中)b(被选中)c(不被选中)          =>    ab
a(被选中)b(不被选中)c(被选中)          =>    ac
a(被选中)b(不被选中)c(不被选中)        =>    a
a(不被选中)b(被选中)c(被选中)          =>    bc
a(不被选中)b(被选中)c(不被选中)        =>    b
a(不被选中)b(不被选中)c(被选中)        =>    c
a(不被选中)b(不被选中)c(不被选中)      =>    空(不作为一种组合情况)
```

对于有重复字符的情况，不重复的字符各有两种状态；而重复的字符状态个数与  
重复次数有关。以a,a,b为例，b有两种状态：被选中、不被选中，a,a有三种状态：  
被选中2个，被选中1个，不被选中。2*3=6，排除为空的情况，一共有5种组合：  

```
a(被选中2个)b(被选中)                 =>    aab
a(被选中2个)b(不被选中)               =>    aa
a(被选中1个)b(被选中)                 =>    ab
a(被选中1个)b(不被选中)               =>    a
a(不被选中)b(被选中)                  =>    b
a(不被选中)b(不被选中)                =>    空(不作为一种组合情况)
```

上述无重复字符的情况可以看作是有重复字符的情况的特例，因此，仅实现  
有重复字符的字符串组合处理思路即可。  

```java
package chapter4;
import java.util.*;
/**
 * Created by ryder on 2017/7/22.
 * 字符串的组合
 */
public class P199_StringCombination {
    //无重复字符：对于每一个字符，都由两种选择：被选中、不被选中；
    //有重复字符：整体需要先排序，对于重复n遍的某种字符，有如下选择：不被选中，选1个，选2个...选n个。
    public static List<char[]> combination(char[] strs) {
        if (strs == null || strs.length == 0)
            return null;
        Arrays.sort(strs);
        List<char[]> ret = new LinkedList<>();
        combinationCore(strs,ret,new StringBuilder(),0);
        return ret;
    }
    public static void combinationCore(char[] strs,List<char[]> ret,StringBuilder stringBuilder,int cur){
        if(cur==strs.length ) {
            if(stringBuilder.length()>0)
                ret.add(stringBuilder.toString().toCharArray());
        }
        else if(cur+1==strs.length||strs[cur]!=strs[cur+1]){
            combinationCore(strs,ret,stringBuilder.append(strs[cur]),cur+1);
            stringBuilder.deleteCharAt(stringBuilder.length()-1);
            combinationCore(strs,ret,stringBuilder,cur+1);

        }
        else{
            //先计算出重复次数
            int dumplicateStart = cur;
            while(cur!=strs.length&&strs[dumplicateStart]==strs[cur]){
                stringBuilder.append(strs[cur]);
                cur++;
            }
            int newStart = cur;
            while (cur>=dumplicateStart) {
                combinationCore(strs, ret, stringBuilder, newStart);
                if(cur!=dumplicateStart)
                    stringBuilder.deleteCharAt(stringBuilder.length() - 1);
                cur--;
            }

        }
    }
    public static void main(String[] args) {
        char[] strs2 = {'a', 'a', 'b'};
        List<char[]> ret2 = combination(strs2);
        for (char[] item : ret2) {
            for (int i = 0; i < item.length; i++)
                System.out.print(item[i]);
            System.out.println();
        }
    }
}
```

39. 数组中出现次数超过一半的数字  

题目要求：  
找出数组中出现次数超过数组长度一半的数字。如输入{1,2,3,2,2,2,5,4,2}，则输出2。  

解题思路：
因为该数字的出现次数超过了数组长度的一半，因此可以将问题转化为求数组的中位数。  
如果按照这个思路，有以下两种方式解决：排序后求中位数、用快排的分区函数求中位数  
（topK问题），这两种思路都比较简单，此处不再赘述。  
书中还提到一种思路，相当巧妙，可以看作一种特殊的缓存机制。该思路需要一个整型的  
value变量和一个整型的count变量，记录缓存值与该缓存值被命中的次数。缓存规则以及  
执行步骤如下：  

```
步骤1： 缓存值value，命中次数count均初始化为0。  
步骤2： 从头到尾依次读取数组中的元素，判断该元素是否等于缓存值：  
     步骤2.1：如果该元素等于缓存值，则命中次数加一。  
     步骤2.2：如果该元素不等于缓存值，判断命中次数是否大于1：  
              步骤2.2.1：如果命中次数大于1，将命中次数减去1。  
              步骤2.2.2：如果命中次数小于等于1，则令缓存值等于元素值，命中次数设为1  
步骤3： 最终的缓存值value即为数组中出现次数超过一半的数字。  
此方法时间复杂度o(n)，空间复杂度o(1)，实现简单。  
```

```java
package chapter5;

/**
 * Created with IntelliJ IDEA.
 * Author: ryder
 * Date  : 2017/7/28
 * Time  : 17:51
 * Description: 数组中出现次数超过一半的数字
 **/
public class P205_MoreThanHalfNumber {
    //转化为topK问题（此处求第k小的值），使用快排的分区函数解决，求第targetIndex+1小的数字（下标为targetIndex）
    //书中说这种方法的时间复杂度为o(n)，但没懂为什么。网上也有人说为o(nlogk)
    public static int moreThanHalfNum1(int[] data){
        if(data==null || data.length==0)
            return 0;
        int left = 0,right=data.length-1;
        //获取执行分区后下标为k的数据值（即第k+1小的数字）
        int k = data.length>>>1;
        int index = partition(data,left,right);
        while(index!=k){
            if(index>k)
                index = partition(data,left,index-1);
            else
                index = partition(data,index+1,right);
        }
        return data[k];
    }
    //分区，[小，povot，大]
    public static int partition(int[] data,int left,int right){
        int pivot = data[left];
        while(left<right){
            while (left<right && data[right]>=pivot)
                right--;
            if(left<right)
                data[left] = data[right];
            while (left<right && data[left]<pivot)
                left++;
            if(left<right)
                data[right] = data[left];
        }
        data[left] = pivot;
        return left;
    }

    //根据数组特点进行记录、查找，时间复杂度o(n)
    public static int moreThanHalfNum2(int[] data){
        if(data==null || data.length==0)
            return 0;
        int count = 1;
        int value = data[0];
        for(int i=1;i<data.length;i++){
            if(data[i]==value)
                count++;
            else if(data[i]!=value && count==1)
                value = data[i];
            else
                count--;
        }
        return value;
    }
    public static void main(String[] args){
        int[] data = {1,2,3,2,2,2,5,4,2};
        System.out.println(moreThanHalfNum2(data));
        System.out.println(moreThanHalfNum1(data));
    }
}
```


40：最小的k个数  

题目要求：  
找出n个整数中最小的k个数。例如输入4,5,1,6,2,7,3,8，则最小的4个数字是1,2,3,4。  

解题思路：  
经典的topK问题，网上有很多种思路，在此仅介绍我能想到的几种：  

```
解法          介绍                            时间       空间   是否修改原数组
1   排序后，前k个即为所求                      o(nlogn)   o(1)      是
2   执行k次直接选择排序                        o(n*k)     o(1)      是
3   使用快排的分区函数求出第k小的元素           o(n)       o(1)      是
4   维护一个长度为k的升序数组，用二分法更新元素  o(nlogk)   o(k)      否
5   创建并维护一个长度为k的最大堆               o(nlogk)   o(k)      否
```

```java
package chapter5;

public class P209_KLeastNumbers {
    //选择排序,时间复杂度o(N*k),适合k较小的情况
    public static int getLeastNumbers1(int[] data,int k){
        if(data==null||data.length==0||k>data.length)
            return 0;
        for(int i=0;i<k;i++){
            int minIndex = i;
            for(int j=i+1;j<data.length;j++){
                if(data[j]<data[minIndex])
                    minIndex = j;
            }
            if(minIndex!=i){
                int temp = data[minIndex];
                data[minIndex] = data[i];
                data[i] = temp;
            }
        }
        //第k小，也就是排序后下标为k-1的元素。
        return data[k-1];
    }

    //使用分区函数解决，时间复杂度o(n)(不确定),会修改原数组
    public static int getLeastNumbers2(int[] data,int k){
        if(data==null || data.length==0 || k>data.length)
            return 0;
        int left=0,right=data.length-1;
        int index = partition(data,left,right);
        while(index!=k-1){
            if(index<k-1)
                index = partition(data,index+1,right);
            else
                index = partition(data,left,index-1);
        }
        return data[k-1];
    }
    public static int partition(int[] data,int left,int right){
        int pivot = data[left];
        while(left<right){
            while (left<right && data[right]>=pivot)
                right--;
            if(left<right)
                data[left] = data[right];
            while (left<right && data[left]<pivot)
                left++;
            if(left<right)
                data[right] = data[left];
        }
        data[left] = pivot;
        return left;
    }

    //使用最大堆解决，不会修改原数组，适合处理海量数据
    //k个元素的最大堆调整时间复杂度为o(logk),所以总的时间复杂度为o(nlogk)
    public static int getLeastNumbers3(int[] data,int k){
        if(data==null || data.length==0 || k>data.length)
            return 0;
        //最大堆，0号元素不用，因此长度需k+1
        int[] heap = new int[k+1];
        int i = 0;
        while (i<k){
            heap[i+1] = data[i];
            i++;
        }
        //初始化最大堆
        buildMaxHeap(heap);
        //调整最大堆
        while (i<data.length){
            if(data[i]<heap[k]) {
                heap[1] = data[i];
                adjustMaxHeap(heap, 1);
            }
            i++;
        }
        //长度为k的最大堆中下标为1的元素就是data数组中第k小的数据值
        return heap[1];
    }
    //0号元素不用，创建一个长度为k+1的堆
    public static void buildMaxHeap(int[] heap){
        for(int i = heap.length>>>1;i>0;i--)
            adjustMaxHeap(heap,i);
    }
    //调整最大堆,i为待调整的下标
    public static void adjustMaxHeap(int[] heap,int i){
        int left = 2*i,right = left+1;
        int max  = i;
        if(left<heap.length && heap[left]>heap[max])
            max = left;
        if(right<heap.length && heap[right]>heap[max])
            max = right;
        if(max!=i){
            int temp = heap[i];
            heap[i] = heap[max];
            heap[max] = temp;
            adjustMaxHeap(heap,max);
        }
    }
    public static void main(String[] args){
        int[] data1 = {6,1,3,5,4,2};
        System.out.println(getLeastNumbers1(data1,5));
        int[] data2 = {6,1,3,5,4,2};
        System.out.println(getLeastNumbers2(data2,5));
        int[] data3 = {6,1,3,5,4,2};
        System.out.println(getLeastNumbers3(data3,5));
    }
}
```

41. 数据流中的中位数  

题目要求：  
得到一个数据流中的中位数。  

解题思路：  
此题的关键在于“数据流”，即数字不是一次性给出，解题的关键在于重新写一个结构  
记录历史数据。下面给出三种思路，分别借助于链表、堆、二叉搜索树完成。  

思路1：使用未排序的链表存储数据，使用快排的分区函数求中位数；也可以在插入时  
进行排序，这样中位数的获取会很容易，但插入费时。  

思路2：使用堆存储，两个堆能够完成最大堆-最小堆这样的简单分区，两个堆的数字个  
数相同或最大堆数字个数比最小堆数字个数大1，中位数为两个堆堆顶的均值或者最大堆的堆顶。

思路3：使用二叉搜索树存储，每个树节点添加一个表征子树节点数目的字段用于找到中位数。  

```java
package chapter5;

public class P214_StreamMedian {
    public static interface MedianFinder{
        //模拟读取数据流的过程
        void addNum(double num);
        double findMedian();
    }
    //使用未排序的链表存储数据，使用快排的分区函数求中位数；
    //也可以在插入时进行排序，这样中位数的获取会很容易，但插入费时。
    public static class MedianFinder1 implements MedianFinder{
        List<Double> list = null;
        public MedianFinder1() {
            list = new LinkedList<>();
        }
        @Override
        public void addNum(double num) {
            list.add(num);
        }
        @Override
        public double findMedian() {
            if(list.size()==0)
                return 0;
            //如果长度为奇数，求中间的那个数;如果为偶数，求中间两个数的均值
            if((list.size()&1)==1)
                return findKth(list.size()>>>1);
            else
                return (findKth(list.size()>>>1)+findKth((list.size()>>>1)-1))/2;
        }
        //得到从小到大排序后下标为k的数据值
        private double findKth(int k){
            int start=0,end=list.size()-1;
            int index = partition(start,end);
            while (index!=k){
                if(index<k)
                    index = partition(index+1,end);
                else
                    index = partition(start,index-1);
            }
            return list.get(index);
        }
        //分区，[小，pivot，大]
        private int partition(int start,int end){
            if(start>=end)
                return end;
            double pivot = list.get(start);
            //[start,bound)小于等于pivot,bound值为pivot，(bound，end]大于pivot
            int bound = start;
            for(int i=start+1;i<=end;i++){
                if(list.get(i)<=pivot){
                    list.set(bound,list.get(i));
                    bound++;
                    list.set(i,list.get(bound));
                }
            }
            list.set(bound,pivot);
            return bound;
        }
    }

    //思路二：使用堆存储
    //两个堆能够完成最大堆-最小堆这样的简单分区，两个堆的数字个数相同或heapMax比heapMin大1
    //中位数为最大堆的堆顶或者两个堆堆顶的均值
    public static class MedianFinder2 implements MedianFinder{
        List<Double> maxHeap = null;
        List<Double> minHeap = null;
        public MedianFinder2() {
            maxHeap = new ArrayList<>();
            minHeap = new ArrayList<>();
            maxHeap.add(0.0);
            minHeap.add(0.0);
        }
        @Override
        public void addNum(double num) {
            if(maxHeap.size()==minHeap.size()){
                if(minHeap.size()<=1||num<=minHeap.get(1))
                    addItemToMaxHeap(num);
                else{
                    addItemToMaxHeap(minHeap.get(1));
                    minHeap.set(1,num);
                    adjustMinHeap(1);
                }
            }
            else{
                if(num>=maxHeap.get(1))
                    addItemToMinHeap(num);
                else{
                    addItemToMinHeap(maxHeap.get(1));
                    maxHeap.set(1,num);
                    adjustMaxHeap(1);
                }

            }
        }
        private void addItemToMaxHeap(double value){
            maxHeap.add(value);
            int curIndex = maxHeap.size()-1;
            while (curIndex>1 && maxHeap.get(curIndex)>maxHeap.get(curIndex>>>1)){
                double temp = maxHeap.get(curIndex);
                maxHeap.set(curIndex,maxHeap.get(curIndex>>>1));
                maxHeap.set(curIndex>>>1,temp);
                curIndex = curIndex>>>1;
            }
        }
        private void adjustMaxHeap(int index){
            int left = index*2,right=left+1;
            int max = index;
            if(left<maxHeap.size()&&maxHeap.get(left)>maxHeap.get(max))
                max = left;
            if(right<maxHeap.size()&&maxHeap.get(right)>maxHeap.get(max))
                max = right;
            if(max!=index){
                double temp = maxHeap.get(index);
                maxHeap.set(index,maxHeap.get(max));
                maxHeap.set(max,temp);
                adjustMaxHeap(max);
            }
        }
        private void addItemToMinHeap(double value){
            minHeap.add(value);
            int curIndex = maxHeap.size()-1;
            while (curIndex>1 && minHeap.get(curIndex)<minHeap.get(curIndex>>>1)){
                double temp = minHeap.get(curIndex);
                minHeap.set(curIndex,minHeap.get(curIndex>>>1));
                minHeap.set(curIndex>>>1,temp);
                curIndex = curIndex>>>1;
            }
        }
        private void adjustMinHeap(int index){
            int left = index*2,right=left+1;
            int min = index;
            if(left<minHeap.size()&&minHeap.get(left)<minHeap.get(min))
                min = left;
            if(right<minHeap.size()&&minHeap.get(right)<minHeap.get(min))
                min = right;
            if(min!=index){
                double temp = minHeap.get(index);
                minHeap.set(index,minHeap.get(min));
                minHeap.set(min,temp);
                adjustMinHeap(min);
            }
        }
        @Override
        public double findMedian() {
            if(maxHeap.size()>minHeap.size())
                return maxHeap.get(1);
            else
                return (maxHeap.get(1)+minHeap.get(1))/2;
        }
    }

    //思路三：使用二叉搜索树存储，每个树节点添加一个表征子树节点数目的字段用于计算中位数。
    public static class TreeNodeWithNums<T> {
        public T val;
        public int nodes;
        public TreeNodeWithNums<T> left;
        public TreeNodeWithNums<T> right;
        public TreeNodeWithNums(T val){
            this.val = val;
            this.nodes = 1;
            this.left = null;
            this.right = null;
        }
    }
    public static class MedianFinder3 implements MedianFinder {
        //左孩子小于根节点，右孩子大于等于根节点
        TreeNodeWithNums<Double> root = null;
        public MedianFinder3() {
        }
        @Override
        public void addNum(double num) {
            if(root==null)
                root = new TreeNodeWithNums<>(num);
            else
                addNode(root,num);
        }
        public void addNode(TreeNodeWithNums<Double> node, double num){
            if(num<node.val){
                if(node.left==null){
                    node.left = new TreeNodeWithNums<>(num);
                    node.nodes++;
                }
                else
                    addNode(node.left,num);
            }
            else{
                if(node.right==null){
                    node.right = new TreeNodeWithNums<>(num);
                    node.nodes++;
                }
                else
                    addNode(node.right,num);
            }
        }
        @Override
        public double findMedian() {
            if(root==null)
                return 0;
            if((root.nodes&1)==1)
                return findMedian(root,root.nodes/2+1);
            else
                return (findMedian(root,root.nodes/2)+findMedian(root,root.nodes/2+1))/2;
        }
        private double findMedian(TreeNodeWithNums<Double> node,int index){
            if(node.left!=null){
                if(index==node.left.nodes+1)
                    return node.val;
                else if(index<=node.left.nodes)
                    return findMedian(node.left,index);
                else
                    return findMedian(node.right,index-node.left.nodes-1);
            }
            else if(node.right!=null){
                return findMedian(node.right,index-1);
            }
            else
                return node.val;
        }
    }
    public static void main(String[] args){
        MedianFinder medianFinder1 = new MedianFinder1();
        medianFinder1.addNum(2);
        medianFinder1.addNum(1);
        System.out.println(medianFinder1.findMedian());
        MedianFinder medianFinder2 = new MedianFinder2();
        medianFinder2.addNum(2);
        medianFinder2.addNum(1);
        System.out.println(medianFinder2.findMedian());
        MedianFinder medianFinder3 = new MedianFinder3();
        medianFinder3.addNum(2);
        medianFinder3.addNum(1);
        System.out.println(medianFinder3.findMedian());

        medianFinder1.addNum(4);
        medianFinder2.addNum(4);
        medianFinder3.addNum(4);
        System.out.println(medianFinder1.findMedian());
        System.out.println(medianFinder2.findMedian());
        System.out.println(medianFinder3.findMedian());

    }
}
```

42. 连续子数组的最大和  

题目要求：  
输入一个整形数组，数组里有正数也有负数。数组中一个或连续多个整数组成一个子数组。  
求所有子数组的和的最大值，要求时间复杂度为o(n)。  

解题思路：  
暴力求解，简单直接，但时间复杂度o(n^2)。  
其实这种最值问题，很容易让人想到动态规划。对于数据data[],申请一个数组dp[]，  
定义dp[i]表示以data[i]为末尾元素的子数组和的最大值。dp的初始化及递推公式可表示为  

```
dp[i] =  data[i]            i=0或dp[i-1]<=0
         dp[i-1]+data[i]    i!=0且dp[i-1]>0
```

由于dp[i]仅与dp的前一个状态有关，即在计算dp[i]时dp[i-2],dp[i-3]......,dp[0]  
对于dp[i]没有影响，因此可以省去dp数组，用一个变量记录当前dp值，用另一个变量maxdp  
记录出现的最大的dp值。如此处理后，时间复杂度为o(n)，空间复杂度为o(1)。  

```java
package chapter5;

/**
 * Created with IntelliJ IDEA.
 * Author: ryder
 * Date  : 2017/8/1
 * Time  : 20:58
 * Description:连续子数组的最大和
 **/
public class P218_GreatestSumOfSubarrays {
    //动态规划，递归公式：dp[i] =  data[i]          i=0或dp[i-1]<=0
    //                           dp[i-1]+data[i]  i!=0且dp[i-1]>0
    //由于只需知道前一个情况的dp值，因此可省去dp数组，申请个变量记录即可
    public static int findGreatestSumOfSumArrays(int[] data){
        if(data==null || data.length==0)
            return 0;
        //dp[i]用于计算以data[i]为结尾元素的连续数组的最大和
        //maxdp用于记录在上述过程中的最大的dp值
        int dp = data[0],maxdp = dp;
        for(int i=1;i<data.length;i++){
            if(dp>0)
                dp += data[i];
            else
                dp = data[i];
            if(dp>maxdp)
                maxdp = dp;
        }
        return maxdp;
    }
    public static void main(String[] args){
        int[] data = {1,-2,3,10,-4,7,2,-5};
        System.out.println(findGreatestSumOfSumArrays(data));
    }
}
```

43. 1~n整数中1出现的次数  

题目要求：  
输入一个整数，求1~n这n个整数中1出现的次数。如输入12，则包含1的数字有1,10,11,12，  
一共出现了5次1，因此输入5。  

解题思路：  
思路1：通过遍历1~n，然后依次判断每个数字包含1的个数。  
思路2：通过规律递归计算出结果：  

以21345为例。  
步骤1：把1~21345的所有数字分成两段：第1段是1~1345，第2段是1346~21345。  
步骤2：计算第2段中1出现的次数。  
      步骤2.1：计算最高位万位中1出现的次数，要分最高位是否为1考虑。  
              此处最高位大于1，countFirst1 =10^4。  
      步骤2.2：计算其他位中1出现的次数countOhters1=2*10^3*4。  
             （1346~21345与1~20000的countOhters1是相等的，所以可以转化为分析1~20000）  
步骤3：依据步骤1,2，递归计算1~1345。  
代码实现如下：  

```java
package chapter5;
public class P221_NumberOf1 {
    public static int numberOf1Between1AndN(int n){
        int count = 0;
        if(n<=0)
            return count;
        for(int i=1;i<=n;i++)
            count+=numberOf1(i);
        return count;
    }
    private static int numberOf1(int i){
        int count = 0;
        while (i!=0){
            if(i%10==1)
                count++;
            i/=10;
        }
        return count;
    }
    public static int numberOf1Between1AndN2(int n){
        if(n<=0)
            return 0;
        if(n<10)
            return 1;
        String nString = Integer.toString(n);
        char firstChar = nString.charAt(0);
        String apartFirstString = nString.substring(1);
        //计算other~n中1出现的次数，递归计算apartFirstString
        int countFirst1 = 0;
        if(firstChar>'1')
            countFirst1 = power10(nString.length()-1);
        else
            countFirst1 = Integer.parseInt(apartFirstString)+1;
        int countOhters1 = (firstChar-'0')*power10(nString.length()-2)*(nString.length()-1);
        return countFirst1+countOhters1+numberOf1Between1AndN(Integer.parseInt(apartFirstString));
    }
    public static int power10(int n){
        int result = 1;
        for(int i=0;i<n;i++)
            result*=10;
        return result;
    }
    public static void main(String[] args){
        System.out.println(numberOf1Between1AndN(121));
        System.out.println(numberOf1Between1AndN2(121));
        System.out.println(numberOf1Between1AndN(789));
        System.out.println(numberOf1Between1AndN2(789));

    }
}
```

44：数字序列中某一位的数字  

题目要求：
数字以01234567891011121314...的格式排列。在这个序列中，第5位（从0开始计）是5，  
第13位是1，第19位是4。求任意第n为对应的数字。  

解题思路：
与43题类似,都是数学规律题。如果用遍历的方式，思路代码都很简单，但速度较慢。更好的  
方式是借助于数字序列的规律，感觉更像是数学题。步骤大致可以分为如下三部分：  

```
以第15位数字2为例（2隶属与12，两位数，位于12从左侧以0号开始下标为1的位置）  
步骤1：首先确定该数字是属于几位数的;
      如果是一位数，n<9;如果是两位数，n<9+90*2=189;
      说明是两位数。
步骤2：确定该数字属于哪个数。10+(15-10)/2= 12。
步骤3：确定是该数中哪一位。15-10-(12-10)*2 = 1, 所以位于“12”的下标为1的位置，即数字2。

以第1001位数字7为例
步骤1：首先确定该数字是属于几位数的;
      如果是一位数，n<9;如果是两位数，n<9+90*2=189;如果是三位数，n<189+900*3=2889;
      说明是三位数。
步骤2：确定该数字属于哪个数。100+(1001-190)/3= 370。
步骤3：确定是该数中哪一位。1001-190-(370-100)*3 = 1,所以位于“370”的下标为1的位置，即数字1。
```

```java
package chapter5;

public class P225_DigitsInSequence {
    public static int digitAtIndex(int index){
        if(index<0)
            return -1;
        if(index<10)
            return index;
        int curIndex = 10,length = 2;
        int boundNum = 10;
        while (curIndex+lengthSum(length)<index){
            curIndex+=lengthSum(length);
            boundNum*=10;
            length++;
        }
        int addNum = (index-curIndex)/length;
        int curNum = boundNum + addNum;
        return Integer.toString(curNum).charAt(index-curIndex-addNum*length)-'0';
    }
    public static int lengthSum(int length){
        int count = 9;
        for(int i=1;i<length;i++)
            count*=10;
        return count*length;
    }
    public static void main(String[] args){
        for(int i=9;i<16;i++)
            System.out.println(digitAtIndex(i));
        System.out.println(digitAtIndex(1001));

    }
}
```

45. 把数组排列成最小的数  

题目要求：  
输入一个正整数数组，把数组里所有数字拼接起来排成一个数，使其为所有可能的拼接  
结果中最小的一个。例如输入{3,32,321}，则输入321323。  

解题思路：  
此题需要对数组进行排序，关键点在于排序的规则需要重新定义。我们重新定义“大于”，  
“小于”，“等于”。如果a，b组成的数字ab的值大于ba，则称a"大于"b，小于与等于类似。  
比如3与32，因为332大于323，因此我们称3“大于”32。我们按照上述的“大于”，“小于”  
规则进行升序排列，即可得到题目要求的答案。  

```java
package chapter5;

import java.util.Arrays;
import java.util.Comparator;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/3
 * Time  : 10:13
 * Description:把数组排成最小的数
 **/
public class P227_SortArrayForMinNumber {
    public static void printMinNumber(int[] data){
        if(data==null||data.length==0)
            return;
        for(int i=0;i<data.length-1;i++){
            for(int j=0;j<data.length-1-i;j++){
                if(bigger(data[j],data[j+1])){
                    int temp = data[j];
                    data[j] = data[j+1];
                    data[j+1] = temp;
                }
            }
        }
        for(int item:data){
            System.out.print(item);
            System.out.print(" ");
        }
        System.out.println();
    }
    //if a>=b return true
    public static boolean bigger(int a,int b){
        String temp1 = a+""+b;
        String temp2 = b+""+a;
        if(temp1.compareTo(temp2)>0)
            return true;
        else
            return false;
    }
    public static void main(String[] args){
        printMinNumber(new int[]{3,32,321});
    }
}
```

46：把数字翻译成字符串  

题目要求：  
给定一个数字，按照如下规则翻译成字符串：0翻译成“a”，1翻译成“b”...25翻译成“z”。  
一个数字有多种翻译可能，例如12258一共有5种，分别是bccfi，bwfi，bczi，mcfi，mzi。  
实现一个函数，用来计算一个数字有多少种不同的翻译方法。  

解题思路：  
下面我们从自上而下和自下而上两种角度分析这道题目，以12258为例：  

```
自上而下，从最大的问题开始，递归 ：
                     12258
                   /       \
              b+2258       m+258
              /   \         /   \
          bc+258 bw+58  mc+58  mz+8
          /  \      \        \     \
      bcc+58 bcz+8   bwf+8   mcf+8  mzi
        /        \       \     \
   bccf+8        bczi    bwfi   mcfi
     /
 bccfi
有很多子问题被多次计算，比如258被翻译成几种这个子问题就被计算了两次。

自下而上，动态规划，从最小的问题开始 ：
f(r)表示以r为开始（r最小取0）到最右端所组成的数字能够翻译成字符串的种数。  
对于长度为n的数字，f(n)=0,f(n-1)=1,求f(0)。
递推公式为 f(r-2) = f(r-1)+g(r-2,r-1)*f(r)；
其中，如果r-2，r-1能够翻译成字符，则g(r-2,r-1)=1，否则为0。
因此，对于12258：
f(5) = 0
f(4) = 1
f(3) = f(4)+0 = 1
f(2) = f(3)+f(4) = 2
f(1) = f(2)+f(3) = 3 
f(0) = f(1)+f(2) = 5
```

```java
package chapter5;
public class P231_TranslateNumbersToStrings {
    public static int getTranslationCount(int number){
        if(number<0)
            return 0;
        if(number==1)
            return 1;
        return getTranslationCount(Integer.toString(number));
    }
    //动态规划，从右到左计算。
    //f(r-2) = f(r-1)+g(r-2,r-1)*f(r);
    //如果r-2，r-1能够翻译成字符，则g(r-2,r-1)=1，否则为0
    public static int getTranslationCount(String number) {
        int f1 = 0,f2 = 1,g = 0;
        int temp;
        for(int i=number.length()-2;i>=0;i--){
            if(Integer.parseInt(number.charAt(i)+""+number.charAt(i+1))<26)
                g = 1;
            else
                g = 0;
            temp = f2;
            f2 = f2+g*f1;
            f1 = temp;
        }
        return f2;
    }
    public static void main(String[] args){
        System.out.println(getTranslationCount(-10));  //0
        System.out.println(getTranslationCount(1234));  //3
        System.out.println(getTranslationCount(12258)); //5
    }
}
```

47：礼物的最大值

题目要求：
在一个m*n的棋盘的每一个格都放有一个礼物，每个礼物都有一定价值（大于0）。  
从左上角开始拿礼物，每次向右或向下移动一格，直到右下角结束。给定一个棋盘，  
求拿到礼物的最大价值。例如，对于如下棋盘  

1    10   3    8
12   2    9    6
5    7    4    11
3    7    16   5
礼物的最大价值为1+12+5+7+7+16+5=53。  

解题思路：  
思路1：动态规划  
申请一个与原矩阵行列数一样的二维数组dp[][]，初始化dp[0][i] = data[0][i]  
；然后依次更新dp的每一行即可。由于第m行的值与第m-1行和第m行有关，因此可以  
对dp进行简化，仅用两行的dp，即dp[2][]即可完成状态的记录与更新。  

思路2：广度优先遍历  
这个棋盘其实可以看成一个有向图，起点为左上角，终点为右下角，每一点仅仅指向  
右侧和下侧。因此我们可以从左上角开始进行广度优先遍历。此外，遍历过程中可以  
进行剪枝，最终移动到右下角时会仅剩下一个枝，该路径所经的点的数值之和即为所求。  

```java
package chapter5;

import java.util.LinkedList;
import java.util.Queue;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/11
 * Time  : 13:03
 * Description:礼物的最大价值
 **/
public class P233_MaxValueOfGifts {
    //方法一：动态规划
    public static int getMaxVaule(int[][] data){
        if(data.length==0||data[0].length==0)
            return 0;
        int[][] dp = new int[2][data[0].length];
        int dpCurRowIndex = 0,dpPreRowIndex = 0;
        for(int row=0;row<data.length;row++){
            dpCurRowIndex = row&1;
            dpPreRowIndex = 1 - dpCurRowIndex;
            for(int col=0;col<data[0].length;col++){
                if(col==0)
                    dp[dpCurRowIndex][col] = dp[dpPreRowIndex][col]+data[row][col];
                else{
                    if(dp[dpPreRowIndex][col]>=dp[dpCurRowIndex][col-1])
                        dp[dpCurRowIndex][col] = dp[dpPreRowIndex][col]+data[row][col];
                    else
                        dp[dpCurRowIndex][col] = dp[dpCurRowIndex][col-1]+data[row][col];
                }
            }
        }
        return dp[(data.length-1)&1][data[0].length-1];
    }

    //方法二：有向图的遍历（广度优先，可再剪枝进行优化）
    public static int getMaxVaule2(int[][] data){
        if(data.length==0||data[0].length==0)
            return 0;
        int maxRowIndex = data.length-1;
        int maxColIndex = data[0].length-1;
        Queue<Node> queue = new LinkedList<>();
        queue.offer(new Node(0,0,data[0][0]));
        Node nodeCur = null;
        while (!(queue.peek().row==maxRowIndex && queue.peek().col==maxColIndex)){
            nodeCur = queue.poll();
            if(nodeCur.row!=maxRowIndex)
                queue.offer(new Node(nodeCur.row+1,nodeCur.col,nodeCur.sum+data[nodeCur.row+1][nodeCur.col]));
            if(nodeCur.col!=maxColIndex)
                queue.offer(new Node(nodeCur.row,nodeCur.col+1,nodeCur.sum+data[nodeCur.row][nodeCur.col+1]));
        }
        int maxSum = 0,temp = 0;
        while (!queue.isEmpty()){
            temp = queue.poll().sum;
            if(temp>maxSum)
                maxSum = temp;
        }
        return maxSum;
    }
    public static class Node{
        int row;
        int col;
        int sum;
        public Node(int r,int c,int s){
            row = r;col = c;sum = s;
        }
    }
    public static void main(String[] args){
        int[][] data = {
                {1,10,3,8},
                {12,2,9,6},
                {5,7,4,11},
                {3,7,16,5}};
        System.out.println(getMaxVaule(data));
        System.out.println(getMaxVaule2(data));
    }
}
```

48. 最长不含重复字符的子字符串  

题目要求：  
输入一个字符串（只包含a~z的字符），求其最长不含重复字符的子字符串的长度。  
例如对于arabcacfr，最长不含重复字符的子字符串为acfr，长度为4。  

解题思路：  
动态规划。用dp[]记录状态，dp[i]表示以下标为i的字符结尾不包含重复字符的  
最长子字符串长度。初始化dp[0] = 1，求maxdp。每次可以根据dp的前一个状态  
推导出后一个状态，因此可以省略dp数组，使用一个变量记录dp值，使用maxdp  
记录最大的dp值。  

```java
package chapter5;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/12
 * Time  : 19:37
 * Description:最长不含重复字符的子字符串
 **/
public class P236_LongestSubstringWithoutDup {
    //动态规划
    //dp[i]表示以下标为i的字符结尾不包含重复字符的最长子字符串长度
    public static int longestSubstringWithoutDup(String str){
        if(str==null || str.length()==0)
            return 0;
        //dp数组可以省略，因为只需记录前一位置的dp值即可
        int[] dp = new int[str.length()];
        dp[0] = 1;
        int maxdp = 1;
        for(int dpIndex = 1;dpIndex<dp.length;dpIndex++){
            //i最终会停在重复字符或者-1的位置,要注意i的结束条件
            int i = dpIndex-1;
            for(;i>=dpIndex-dp[dpIndex-1];i--){
                if(str.charAt(dpIndex)==str.charAt(i))
                    break;
            }
            dp[dpIndex] =  dpIndex - i;
            if(dp[dpIndex]>maxdp)
                maxdp = dp[dpIndex];
        }
        return maxdp;
    }
    public static void main(String[] args){
        System.out.println(longestSubstringWithoutDup("arabcacfr"));
        System.out.println(longestSubstringWithoutDup("abcdefaaa"));

    }
}
```

49. 丑数  

题目要求：  
我们把只包含因子2,3,5的数成为丑数。求按照从小到大的顺序第1500个丑数。1作为第一个丑数。  

解题思路：
思路1：从1开始递增，依次判断每个数是否是丑数，不够高效；  
思路2：思路1之所以效率低，比较关键的一点是遍历的每一个数字都进行丑数判断。思路2  
不是去判断丑数，而是计算出丑数：因为每个丑数都可以看成是由1去乘以2、3、5，再乘以  
2、3、5而衍生出来的。可以用三个指针指向第一个丑数1，三个指针分别表示乘2，乘3，乘5，  
将三个指针计算出来的最小的丑数放在数组中，并将该指针向后移动一个位置。为了得到第  
1500个丑数，需要一个长度1500的数组来记录已经计算出来的丑数。因此这个思路也可以说  
是用空间换时间。  

```java
package chapter5;
public class P240_GetUglyNumber {
    public static int getUglyNumber(int num){
         if(num<=0)
             return 0;
         int number = 0,uglyFound = 0;
         while (uglyFound<num){
             number++;
             if(isUgly(number))
                 uglyFound++;
         }
         return number;
    }
    public static boolean isUgly(int number){
        while (number%2==0)
            number/=2;
        while (number%3==0)
            number/=3;
        while (number%5==0)
            number/=5;
        return number==1;
    }
    //获取从1开始的第num个丑数
    public static int getUglyNumber2(int num){
        int[] uglyNumber = new int[num];
        uglyNumber[0] = 1;
        int uglyIndex=0, multiply2=0, multiply3=0, multiply5=0;
        while (uglyIndex+1<num){
            uglyNumber[++uglyIndex] = min(uglyNumber[multiply2]*2,uglyNumber[multiply3]*3,uglyNumber[multiply5]*5);
            //2*3=6，3*2=6，会有重复值，因此下面需要使用if，而不能用if-else
            if(uglyNumber[multiply2]*2==uglyNumber[uglyIndex])
                multiply2++;
            if(uglyNumber[multiply3]*3==uglyNumber[uglyIndex])
                multiply3++;
            if(uglyNumber[multiply5]*5==uglyNumber[uglyIndex])
                multiply5++;
        }
        return uglyNumber[num-1];
    }
    public static int min(int x,int y,int z){
        int temp = x<y?x:y;
        return temp<z?temp:z;
    }
    public static void main(String[] args){
        System.out.println(getUglyNumber(10));
        System.out.println(getUglyNumber2(10));
    }
}
```

50. 第一个只出现一次的字符  

题目要求：  
字符串中第一个只出现一次的字符。在字符串中找出第一个只出现一次的字符。  
如输入abaccdeff，则输出b。  

解题思路：  
思路1：暴力求解，从前到后依次判断每个字符是否只出现一次，时间复杂度  
o(n^2)，空间复杂度o(1)；  
思路2：用空间换时间。这个思路可行的前提是题目中所说的“字符”指的是ascii编码的字符。  
0-127是7位ASCII码的范围，是国际标准。128-255称为扩展ASCII码，不是国际标准。在C++  
中，char是1字节（8bit），能表示256个不同的字符。而java中，char是unicode编码，2字  
节（16bit）。但本题中，假设所有字符都可用ascii表示（0-255）。  
在上述假设下，可以申请一个长度为256的int数组作为哈希表，占用空间1kB，用它来记录字  
符出现的次数。第一扫描字符串，修改对应字符的次数；第二遍扫描，当遇到在数组中对应值  
为1的字符，即得到所求，时间复杂度o(n)。  

```java
package chapter5;
public class P243_FirstNotRepeatingChar {
    //暴力求解，时间复杂度o(n^2),空间复杂度o(1)
    public static char firstNotRepeatingChar(String str){
        //此处，\77表示ascii为77的字符(即?),用于表征没有只出现一次的字符
        if(str==null||str.length()==0)
            return '\77';
        for(int i=0;i<str.length()-1;i++){
            char temp = str.charAt(i);
            for(int j=0;j<=str.length();j++){
                if(j==i)
                    continue;
                if(j==str.length())
                    return temp;
                if(temp==str.charAt(j))
                    break;
            }
        }
        return '\77';
    }
    //引入哈希表，用空间换时间。时间复杂度o(n),空间占用1kB
    public static char firstNotRepeatingChar2(String str){
        //使用这个数组记录字符出现次数
        int[] times = new int[256];
        for(int i=0;i<str.length();i++)
            times[str.charAt(i)]++;
        for(int i=0;i<str.length();i++){
            if(times[str.charAt(i)]==1)
                return str.charAt(i);
        }
        return '\77';
    }
    public static void main(String[] args){
        System.out.println(firstNotRepeatingChar("abaccdeff"));
        System.out.println(firstNotRepeatingChar2("abaccdeff"));

    }
}
```

ssh  