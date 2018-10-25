
36：二叉搜索树与双向链表  

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
/*
方法一：非递归版
解题思路：
1.核心是中序遍历的非递归算法。
2.修改当前遍历节点与前一遍历节点的指针指向。
*/
    import java.util.Stack;
    public TreeNode ConvertBSTToBiList(TreeNode root) {
        if(root==null)
            return null;
        Stack<TreeNode> stack = new Stack<TreeNode>();
        TreeNode p = root;
        TreeNode pre = null;// 用于保存中序遍历序列的上一节点
        boolean isFirst = true;
        while(p!=null||!stack.isEmpty()){
            while(p!=null){
                stack.push(p);
                p = p.left;
            }
            p = stack.pop();
            if(isFirst){
                root = p;// 将中序遍历序列中的第一个节点记为root
                pre = root;
                isFirst = false;
            }else{
                pre.right = p;
                p.left = pre;
                pre = p;
            }      
            p = p.right;
        }
        return root;
    }
/*
方法二：递归版
解题思路：
1.将左子树构造成双链表，并返回链表头节点。
2.定位至左子树双链表最后一个节点。
3.如果左子树链表不为空的话，将当前root追加到左子树链表。
4.将右子树构造成双链表，并返回链表头节点。
5.如果右子树链表不为空的话，将该链表追加到root节点之后。
6.根据左子树链表是否为空确定返回的节点。
*/
    public TreeNode Convert(TreeNode root) {
        if(root==null)
            return null;
        if(root.left==null&&root.right==null)
            return root;
        // 1.将左子树构造成双链表，并返回链表头节点
        TreeNode left = Convert(root.left);
        TreeNode p = left;
        // 2.定位至左子树双链表最后一个节点
        while(p!=null&&p.right!=null){
            p = p.right;
        }
        // 3.如果左子树链表不为空的话，将当前root追加到左子树链表
        if(left!=null){
            p.right = root;
            root.left = p;
        }
        // 4.将右子树构造成双链表，并返回链表头节点
        TreeNode right = Convert(root.right);
        // 5.如果右子树链表不为空的话，将该链表追加到root节点之后
        if(right!=null){
            right.left = root;
            root.right = right;
        }
        return left!=null?left:root;       
    }
/*
方法三：改进递归版
解题思路：
思路与方法二中的递归版一致，仅对第2点中的定位作了修改，
新增一个全局变量记录左子树的最后一个节点。
*/
    // 记录子树链表的最后一个节点，终结点只可能为只含左子树的非叶节点与叶节点
    protected TreeNode leftLast = null;
    public TreeNode Convert(TreeNode root) {
        if(root==null)
            return null;
        if(root.left==null&&root.right==null){
            leftLast = root;// 最后的一个节点可能为最右侧的叶节点
            return root;
        }
        // 1.将左子树构造成双链表，并返回链表头节点
        TreeNode left = Convert(root.left);
        // 3.如果左子树链表不为空的话，将当前root追加到左子树链表
        if(left!=null){
            leftLast.right = root;
            root.left = leftLast;
        }
        leftLast = root;// 当根节点只含左子树时，则该根节点为最后一个节点
        // 4.将右子树构造成双链表，并返回链表头节点
        TreeNode right = Convert(root.right);
        // 5.如果右子树链表不为空的话，将该链表追加到root节点之后
        if(right!=null){
            right.left = root;
            root.right = right;
        }
        return left!=null?left:root;       
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


38：字符串的排列  

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

39：数组中出现次数超过一半的数字  

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

41：数据流中的中位数  

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

42：连续子数组的最大和  

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

43：1~n整数中1出现的次数  

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

45：把数组排列成最小的数  

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
对于长度为n的数字，f(n)=1,f(n-1)=1,求f(0)。
递推公式为 f(r-2) = f(r-1)+g(r-2,r-1)*f(r)；
其中，如果r-2，r-1能够翻译成字符，则g(r-2,r-1)=1，否则为0。
因此，对于12258：
f(5) = 1
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
        int f1 = 1,f2 = 1,g = 0;
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
        System.out.println(getTranslationCount(5));  //1
        System.out.println(getTranslationCount(12222));  //8
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

48：最长不含重复字符的子字符串  

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

49：丑数  

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

50：50-1 第一个只出现一次的字符  

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


50：50-2 找出字符流中第一个只出现一次的字符  

例如，当从字符流google中只读出前两个字符go时，  
第一个只出现一次的字符是g；当读完google时，第一个只出现一次的字符是l。  

解题思路：  
此题的关键在于“字符流”。因此最好能够记住在哪个位置出现了哪个字符，从而可以完成每读到  
一个字符，就能动态更新到目前为止第一个出现一次的字符。此题同样使用了长度为256的int数  
组作为哈希表，用字符的ascii码值作为表的键值，当字符仅出现了一次，就把字符的位置作为  
哈希表的值，如果没有出现则值为-1，如果出现的次数大于1则哈希表对应的值为-2。  
当想要知道到某一位置时第一个出现一次的字符，可以通过扫描该哈希表，找到大于等于0的值中  
的最小值，该值所对应的字符就是当前状态第一个出现一次的字符。  


```java
package chapter5;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/13
 * Time  : 16:41
 * Description:字符流中第一个只出现一次的字符
 **/
public class P247_FirstNotRepeatingCharInStream {
    public static class CharStatistics{
        private int[] times;
        private int index;
        public CharStatistics(){
            index = 0;
            times = new int[256];
            //-1表示未出现，>=0表示出现的位置且仅出现一次，-2表示出现两次及以上
            for(int i=0;i<256;i++)
                times[i] = -1;
        }
        public void insert(char ch){
            if(times[ch]==-1)
                times[ch] = index;
            else
                times[ch] = -2;
            index++;
        }
        public char find(){
            int minIndex = 256;
            char ret = '\77'; //若没有只出现一次的字符，显示\77，即？
            for(int i=0;i<256;i++){
                if(times[i]>=0 && times[i]<minIndex) {
                    minIndex = times[i];
                    ret = (char)i;
                }
            }
            return ret;
        }
    }
    public static void main(String[] args){
        String str = "google";
        CharStatistics charStatistics = new CharStatistics();
        for(int i=0;i<str.length();i++){
            charStatistics.insert(str.charAt(i));
            System.out.print(charStatistics.find());
        }
    }
}
```


51：数组中的逆序对

题目要求：  
如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，  
求出这个数组中的逆序对总数。例如输入{7,5,6,4}，一共有5个逆序对，分别是  
（7,6），（7,5），（7,4），（6,4），（5,4）。  

解题思路：  
思路1：暴力解决。顺序扫描数组，对于每个元素，与它后面的数字进行比较，  
因此这种思路的时间复杂度为o(n^2)。  

思路2：
上述思路在进行比较后，并没有将相关信息留下，其实在比较之后可以进行局部的排序，  
从而降低比较的次数，降低时间复杂度。
可通过如下步骤求逆序对个数：先把数组逐步分隔成长度为1的子数组，统计出子数组  
内部的逆序对个数，然后再将相邻两个子数组合并成一个有序数组并统计数组之间的逆  
序对数目，直至合并成一个大的数组。其实，这是二路归并的步骤，只不过在归并的同事  
要多进行一步统计。因此时间复杂度o(nlogn)，空间复杂度o(n)，如果使用原地归并排  
序，可以将空间复杂度降为o(1)。  
本文使用经典二路归并排序实现。以{7,5,6,4}为例，过程如下：  

```

           [7     5    6    4]                 
            /              \                  分：将长度为4的数组分成长度为2的数组
        [7    5]         [6    4]
        /      \         /      \             分：将长度为2的数组分成长度为1的数组
     [7]       [5]    [6]        [4]
      \         /        \       /            和：1->2，并记录子数组内的逆序对
        [5    7]         [4    6] 
           \                 /                和：2->4，并记录子数组内的逆序对
           [4     5    6    7]
```

```java
package chapter5;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/14
 * Time  : 19:36
 * Description:数组中的逆序对
 **/
public class P249_InversePairs {
    public static int inversePairs(int[] data){
        if(data==null || data.length<2)
            return 0;
        return mergeSortCore(data, 0, data.length - 1);
    }
    public static int mergeSortCore(int[] data,int start,int end){
        if(start>=end)
            return 0;
        int mid = start+(end-start)/2;
        int left = mergeSortCore(data,start,mid);
        int right = mergeSortCore(data,mid+1,end);
        int count = mergerSortMerge(data,start,mid,end);
        return left+right+count;
    }
    //start~mid, mid+1~end
    public static int mergerSortMerge(int[] data,int start,int mid,int end){
        int[] temp = new int[end-start+1];
        for(int i=0;i<=end-start;i++)
            temp[i] = data[i+start];
        int left = 0,right = mid+1-start,index = start,count = 0;
        while (left<=mid-start && right<=end-start){
            if(temp[left]<=temp[right])
                data[index++] = temp[left++];
            else{
                data[index++] = temp[right++];
                count += (mid-start)-left+1;
            }
        }
        while (left<=mid-start)
            data[index++] = temp[left++];
        while (right<=end-start)
            data[index++] = temp[right++];
        return count;
    }
    public static void main(String[] args){
        System.out.println(inversePairs(new int[]{7,5,6,4}));
        System.out.println(inversePairs(new int[]{5,6,7,8,1,2,3,4}));
    }
}
```


52：两个链表的第一个公共节点  

题目要求：  
输入两个单链表，找出它们的第一个公共节点。以下图为例，对一个公共节点为6所在的节点。  

```
1 -> 2 -> 3 -> 6 -> 7
     4 -> 5 ↗
```

解题思路：


|解法  |思路  |时间复杂度   |空间复杂度  
| ------ | ------ | ------ |------ |  
|1   |暴力求解    |o(mn)   |o(1)
|2   |分别存入两个栈，求栈中最深的相同节点  |o(m+n)  |o(m+n)
|3   |长链表先行n-m绝对值步，转化为等长链表 |o(m+n)  |o(1)  

解法1：比较直接，不再赘述。

解法2：从链表的尾部向前看，发现尾部是相同的，向前走会分叉，找到分叉点即可。  
按照这个思路，如果这是双向链表，即得到尾节点并能够得到每个节点的前一个节点，  
那这个题就很简单了。对于单链表，本身是前节点->后节点，想要得到后节点->前节点，  
可以借助于栈的后进先出特性。两个单链表分别入栈，得到[1,2,3,6,7],[4,5,6,7]，  
然后不断出栈即可找到分叉点。  

解法3：对于单链表，如果从头结点开始找公共节点，一个麻烦点是两个链表长度可能  
不一致，因此两个指针不同步（指两个指针无法同时指向公共点）。解决这个麻烦点，  
整个问题也就能顺利解决。那么，能否让两个链表长度一致？长链表先行几步即可，因为  
长链表头部多出的那几个节点一定不会是两链表的公共节点。以题目中的图为例，可以让  
1所在的链表先向前移动1个节点到2，这样就转化为求下面这两个链表的第一个公共节点：  

```
2 -> 3 -> 6 -> 7
4 -> 5 ↗
```

一个指针指向2，另一个指向4，然后同时遍历，这应该很容易了吧。需要对两个链表先  
进行一次遍历求出长度，然后再同时遍历求公共点，时间复杂度o(n)，空间复杂度o(1)。  

三种解法的代码实现如下。  

```java
package chapter5;
import structure.ListNode;

import java.util.Stack;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/15
 * Time  : 10:28
 * Description:两个链表的第一个公共节点
 **/
public class P253_CommonNodesInLists {
    //思路一：暴力解决，时间复杂度o(mn)，空间复杂度o(1)
    //思路二：借助于两个栈，时间复杂度o(m+n)，空间复杂度o(m+n)
    //思路三：转化为等长链表，时间复杂度o(m+n)，空间复杂度o(1)
    public static ListNode<Integer> findFirstCommonNode(ListNode<Integer> head1,ListNode<Integer> head2){
        for(ListNode<Integer> node1=head1;node1!=null;node1=node1.next){
            for(ListNode<Integer> node2=head2;node2!=null;node2=node2.next){
                if(node1==node2)
                    return node1;
            }
        }
        return null;
    }
    public static ListNode<Integer> findFirstCommonNode2(ListNode<Integer> head1,ListNode<Integer> head2){
        Stack<ListNode<Integer>> stack1 = new Stack<>();
        Stack<ListNode<Integer>> stack2 = new Stack<>();
        for(ListNode<Integer> node1=head1;node1!=null;node1=node1.next)
            stack1.push(node1);
        for(ListNode<Integer> node2=head2;node2!=null;node2=node2.next)
            stack2.push(node2);
        ListNode<Integer> commonNode = null;
        while (!stack1.isEmpty() && !stack2.isEmpty()){
            if(stack1.peek()==stack2.peek()){
                commonNode = stack1.pop();
                stack2.pop();
            }
            else
                break;
        }
        return commonNode;
    }
    public static ListNode<Integer> findFirstCommonNode3(ListNode<Integer> head1,ListNode<Integer> head2){
        ListNode<Integer> node1 = head1,node2 = head2;
        int size1 = 0,size2 = 0;
        for (;node1!=null;node1 = node1.next)
            size1++;
        for (;node2!=null;node2 = node2.next)
            size2++;
        node1 = head1;
        node2 = head2;
        while (size1>size2){
            node1 = node1.next;
            size1--;
        }
        while (size2>size1){
            node2 = node2.next;
            size2--;
        }
        while (node1!=null){
            if(node1!=node2){
                node1 = node1.next;
                node2 = node2.next;
            }
            else
                break;
        }
        return node1;
    }
    public static void main(String[] args){
        // 1->2->3->6->7
        //    4->5↗
        ListNode<Integer> node1 = new ListNode<>(1);
        ListNode<Integer> node2 = new ListNode<>(2);
        ListNode<Integer> node3 = new ListNode<>(3);
        ListNode<Integer> node4 = new ListNode<>(4);
        ListNode<Integer> node5 = new ListNode<>(5);
        ListNode<Integer> node6 = new ListNode<>(6);
        ListNode<Integer> node7 = new ListNode<>(7);
        node1.next = node2;
        node2.next = node3;
        node3.next = node6;
        node4.next = node5;
        node5.next = node6;
        node6.next = node7;
        ListNode<Integer> commonNode = findFirstCommonNode(node1,node4);
        System.out.println(commonNode.val);
        ListNode<Integer> commonNode2 = findFirstCommonNode2(node1,node4);
        System.out.println(commonNode2.val);
        ListNode<Integer> commonNode3 = findFirstCommonNode2(node1,node4);
        System.out.println(commonNode3.val);
    }
}
```


53：数字在排序数组中出现的次数  

题目要求：  
统计一个数字在排序数组中出现的次数。例如，输入{1,2,3,3,3,3,4,5}和数字3，  
由于3在这个数组中出现了4次，因此输出4。  

解题思路：  
排序数组，定位某一个数值的位置，很容易想到二分查找。分成两部分：求第一个出现  
该值的位置start，求最后一个出现该值得位置end，end-start+1即为所求。  

二分法比较关键的一个逻辑如下：  

```java
mid = left + (right - left + 1) / 2;
if (data[mid] > k)
     right = mid-1;
else if(data[mid] < k)
     left = mid+1;
else
     right = mid;
mid = left + (right - left + 1) / 2;
if (data[mid] > k)
     right = mid-1;
else if(data[mid] < k)
     left = mid+1;
else
     left = mid;
 ```

此处要注意求mid时如下两种写法的差别：  

```
（1） mid = left + (right - left) / 2;      
（2） mid = left + (right - left + 1) / 2;
```

如果left，right之前的数字个数为奇数，那两者没差异，比如left=2，right=6，  
中间有3,4,5，那么(1)(2)求出的mid都是4。如果中间数字的个数是偶数，比如left=2，  
right=5，中间有3,4,(1)求出的mid都是3，(2)求出的mid都是4，即(1)求出的mid偏左，  
(2)求出的偏右。极端特殊情况下，比如有个数组data[7,7,7,7,7]，求k=7出现的次数。  

```
使用(1)的迭代过程为
left=0，right=4，mid=2
left=0，right=2，mid=1
left=0，right=1，mid=0 (注意这里)
left=0，right=0，left==right 且 data[left]=k，返回0

使用(2)的迭代过程为
left=0，right=4，mid=2
left=2，right=4，mid=3
left=3，right=4，mid=4(注意这里)
left=4，right=4，left==right 且 data[left]=k，返回4

4-0+1 = 5，即为所求。
```

另一个在用二分查找时，要注意的点是求mid时的写法：

```
（a） mid =  (left + right) / 2;      
（b） mid = left + (right - left) / 2;      
推荐用(b)，最好不要出现(a)，当left、right都是比较大的数时，完成同样的功能，(a)可能造成上溢，但(b)不会。
```

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/15
 * Time  : 11:24
 * Description:数字在排序数组中出现的次数
 **/
public class P263_NumberOfK {
    //遍历的话时间复杂度为o(n)
    //考虑到数组是排序好的，可使用二分法，时间复杂度o(logn)
    public static int getNumberOfK(int[] data,int k){
        int start = getStartOfK(data,k);
        if(start==-1)
            return 0;
        int end = getEndOfK(data,start,k);
        return end-start+1;
    }
    public static int getStartOfK(int[] data,int k){
        if(data[0]>k || data[data.length-1]<k)
            return -1;
        int left = 0,right = data.length-1,mid;
        while (left<=right) {
            if(right==left){
                if(data[left]==k)
                    return left;
                else
                    return -1;
            }
            //当长度为2，mid取左值
            mid = left + (right - left) / 2;
            if (data[mid] > k)
                right = mid-1;
            else if(data[mid] < k)
                left = mid+1;
            else
                right = mid;
        }
        return -1;
    }
    public static int getEndOfK(int[] data,int start, int k){
        int left = start,right = data.length-1,mid;
        while (left<=right){
            if(left==right){
                if(data[left]==k)
                    return left;
                else
                    return -1;
            }
            //当长度为2，mid取右值
            mid = left + (right - left + 1) / 2;
            if (data[mid] > k)
                right = mid-1;
            else if(data[mid] < k)
                left = mid+1;
            else
                left = mid;
        }
        return -1;
    }
    public static void main(String[] args){
        int[] data1 = new int[]{1,2,3,3,3,3,5,6};
        int[] data2 = new int[]{1,2,3,3,3,3,4,5};
        int[] data3 = new int[]{3,3,3,3,3,3,3,3};
        System.out.println(getNumberOfK(data1,4));
        System.out.println(getNumberOfK(data2,3));
        System.out.println(getNumberOfK(data3,3));
    }
}
```


53.2：0~n中缺失的数字  

题目要求：  
一个长度为n的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0~n之内。  
在范围0~n内的n个数字有且只有一个数字不在该数组中，请找出。  

解题思路：  
用二分法找到数组中第一个数值不等于下标的数字。  

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/15
 * Time  : 15:03
 * Description:0~n中缺失的数字
 **/
public class P266_GetMissingNumber {
    public static int getMissingNumber(int[] data){
        int left = 0,right = data.length-1,mid;
        while (left<=right){
            //mid=left+(right-left)/2 用位运算替换除法
            //注意加减法优先级高于位运算
            mid = left+((right-left)>>1);
            if(data[mid]==mid)
                left = mid+1;
            else
                right = mid-1;
        }
        return left;
    }
    public static void main(String[] args){
        int[] data1 = new int[]{0,1,2,3,4,5}; //6
        int[] data2 = new int[]{0,1,3,4,5}; //2
        int[] data3 = new int[]{1,2}; //0
        System.out.println(getMissingNumber(data1));
        System.out.println(getMissingNumber(data2));
        System.out.println(getMissingNumber(data3));
    }
}
```

54：二叉搜索树的第k大节点  

题目要求：  
找出二叉搜索树的第k大节点。例如，在下图的树里，第3大节点的值为4，输入该树的根节点，3，则输出4。  

```
        5
       / \
      3   7
    / \   / \
   2  4  6   8
```

解题思路：  
二叉搜索树的中序遍历是有序的。可以引入一个计数器，每访问一个节点，计数器+1，  
当计数器等于k时，被访问节点就是该二叉搜索树的第k大节点。   

```java
package chapter6;
import structure.TreeNode;
import java.util.Stack;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/15
 * Time  : 19:17
 * Description:二叉搜索树的第k大节点
 **/
public class P269_KthNodeInBST {
    public static TreeNode<Integer> kthNode(TreeNode<Integer> root,int k){
        //栈顶元素保证一直是cur的父节点
        if(root==null || k<0)
            return null;
        Stack<TreeNode<Integer>> stack = new Stack<>();
        TreeNode<Integer> cur = root;
        int count = 0;
        while (!stack.isEmpty()||cur!=null){
            if(cur!=null){
                stack.push(cur);
                cur = cur.left;
            }
            else{
                cur = stack.pop();
                count++;
                if(count==k)
                    return cur;
                cur = cur.right;
            }
        }
        return null;
    }
    public static void main(String[] args){
        TreeNode<Integer> root = new TreeNode<>(5);
        root.left = new TreeNode<>(3);
        root.left.left = new TreeNode<>(2);
        root.left.right = new TreeNode<>(4);
        root.right = new TreeNode<>(7);
        root.right.left = new TreeNode<>(6);
        root.right.right = new TreeNode<>(8);
        System.out.println(kthNode(root,3).val);//4
        System.out.println(kthNode(root,6).val);//7
        System.out.println(kthNode(root,8));//null
    }
}
```

55：二叉树的深度  

题目要求：  
求二叉树的深度。仅仅包含一个根节点的二叉树深度为1。  

解题思路：  
二叉树root的深度比其子树root.left与root.right的深度的最大值大1。因此可以通过上述结论递归求解。  
如果不使用递归，可以通过层序遍历（广度优先遍历）解决。  


```java
package chapter6;
import structure.TreeNode;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.Queue;
import java.util.Stack;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/15
 * Time  : 22:25
 * Description:二叉树的深度
 **/
public class P271_TreeDepth {
    //递归
    public static int treeDepth(TreeNode<Integer> root){
        if(root==null)
            return 0;
        int left = treeDepth(root.left);
        int right = treeDepth(root.right);
        return left>right?(left+1):(right+1);
    }
    //非递归，广度优先/层序遍历
    public static int treeDepth2(TreeNode<Integer> root){
        if(root==null)
            return 0;
        Queue<TreeNode<Integer>> queue = new LinkedList<>();
        queue.offer(root);
        int depth = 0;
        while (!queue.isEmpty()){
            int size = queue.size();
            TreeNode<Integer> cur = null;
            for(int i=0;i<size;i++){
                cur = queue.poll();
                if(cur.left!=null) queue.offer(cur.left);
                if(cur.right!=null) queue.offer(cur.right);
            }
            depth++;
        }
        return depth;

    }
    public static void main(String[] args){
        TreeNode<Integer> root = new TreeNode<>(1);
        root.left = new TreeNode<>(2);
        root.left.left = new TreeNode<>(4);
        root.left.right = new TreeNode<>(5);
        root.left.right.left = new TreeNode<>(7);
        root.right = new TreeNode<>(3);
        root.right.right = new TreeNode<>(6);
        System.out.println(treeDepth(root));
        System.out.println(treeDepth2(root));
    }
}
```


55.2：平衡二叉树  

题目要求：  
输入二叉树的根节点，判断该树是否是平衡二叉树。如果某二叉树的任意节点的  
左右子树深度之差不超过1，则该树是平衡二叉树。  

解题思路：  
思路1：依据平衡二叉树的定义解决。借助于上一题二叉树的深度，从根节点开始逐点判断树  
的左右子树的深度差值是否满足要求。由于此解法是从根到叶的判断，每一次获取节点有需要  
从当前节点遍历到叶节点，因此需要多次遍历。  

思路2：如果用后序遍历，那么访问某一个节点时已经访问了它的左右子树。同时，在访问节  
点时记录下它的深度，并由左右子树的深度推出父亲节点的深度，这样我们就能通过遍历一  
遍完成整棵树的平衡性判断。  
对于某一个子树，需要给出它的平衡性的判断，还要给出它的深度，此处我将平衡性的判断  
作为返回值，深度通过长度为1的数组传递出去，  
见boolean isBalanced2Core(TreeNode<Integer> node,int[] depth)。  

```java
package chapter6;

import structure.TreeNode;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/17
 * Time  : 10:03
 * Description:平衡二叉树
 **/
public class P273_isBalanced {
    //借助于深度，判断是否是平衡二叉树,由于是从根到叶逐点判断，需要多次遍历树
    public static boolean isBalanced(TreeNode<Integer> node){
        if(node==null)
            return true;
        int left = treeDepth(node.left);
        int right = treeDepth(node.right);
        int diff = left - right;
        if(diff<-1||diff>1)
            return false;
        return isBalanced(node.left)&&isBalanced(node.right);
    }
    public static int treeDepth(TreeNode<Integer> root){
        if(root==null)
            return 0;
        int left = treeDepth(root.left);
        int right = treeDepth(root.right);
        return left>right?(left+1):(right+1);
    }
    //用后序遍历，并记录每个节点的深度，从而可以通过一次遍历完成整棵树的判断
    public static boolean isBalanced2(TreeNode<Integer> node){
        if(node==null)
            return true;
        return isBalanced2Core(node,new int[]{0});
    }

    public static boolean isBalanced2Core(TreeNode<Integer> node,int[] depth){
        if(node==null){
            depth[0] = 0;
            return true;
        }
        int[] left = new int[]{0};
        int[] right = new int[]{0};
        if(isBalanced2Core(node.left,left)&&isBalanced2Core(node.right,right)){
            int diff = left[0]-right[0];
            if(diff<=1&&diff>=-1){
                depth[0] = 1+(left[0]>right[0]?left[0]:right[0]);
                return true;
            }
            else
                return false;
        }
        return false;
    }
    public static void main(String[] args){
        TreeNode<Integer> root = new TreeNode<>(1);
        root.left = new TreeNode<>(2);
        root.left.left = new TreeNode<>(4);
        root.left.right = new TreeNode<>(5);
        root.left.right.left = new TreeNode<>(7);
        root.right = new TreeNode<>(3);
        root.right.right = new TreeNode<>(6);
        System.out.println(isBalanced(root));
        System.out.println(isBalanced2(root));
    }
}
```


56：数组中只出现一次的两个数字  

这种类型题目的总结：  
[题目总结](https://www.jianshu.com/p/896accc5bc6d)  

题目要求：  
一个整数数组里除了两个数字出现一次，其他数字都出现两次。请找出这两个数字。  
要求时间复杂度为o(n)，空间复杂度为o(1)。  

解题思路：  
这道题可以看成“数组中只出现一次的一个数字”的延伸。如果所有数字都出现两次，  
只有一个数字是出现1次，那么可以通过把所有所有进行异或运算解决。因为x^x = 0。  
但如果有两个数字出现一次，能否转化成上述问题？依旧把所有数字异或，最终的结果  
就是那两个出现一次的数字a,b异或的结果。因为a，b不想等，因此结果肯定不为0，那  
么结果的二进制表示至少有一位为1，找到那个1的位置p，然后我们就可以根据第p位是  
否为1将所有的数字分成两堆，这样我们就把所有数字分成两部分，且每部分都是只包含  
一个只出现一次的数字、其他数字出现两次，从而将问题转化为最开始我们讨论的“数组  
中只出现一次的一个数字”问题。  

```
实例分析(以2,4,3,6,3,2,5,5为例)：  
相关数字的二进制表示为：  

2 = 0010   3 = 0011    4 = 0100    5 = 0101    6 = 0110
步骤1 全体异或：2^4^3^6^3^2^5^5 = 4^6 = 0010  
步骤2 确定位置：对于0010，从右数的第二位为1，因此可以根据倒数第2位是否为1进行分组  
步骤3 进行分组：分成[2,3,6,3,2]和[4,5,5]两组  
步骤4 分组异或：2^3^6^3^2 = 6，4^5^5 = 4，因此结果为4，6。  
```

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/17
 * Time  : 10:58
 * Description:数组中数字出现的次数
 * 有两个数字分别出现一次，其他的都出现两次，找到这两个数字
 **/
public class P275_NumberAppearOnce {
    public static int[] findNumsAppearOnce(int[] data){
        int result = 0;
        for(int i=0;i<data.length;i++)
            result^=data[i];
        int indexOf1 = findFirstBit1(result);
        int ret[] = new int[]{0,0};
        if(indexOf1<0)
            return ret;
        for(int i=0;i<data.length;i++){
            if((data[i]&indexOf1)==0)
                ret[0]^=data[i];
            else
                ret[1]^=data[i];
        }
        return ret;
    }
    public static int findFirstBit1(int num){
        if(num<0)
            return -1;
        int indexOf1 = 1;
        while (num!=0){
            if((num&1)==1)
                return indexOf1;
            else{
                num = num>>1;
                indexOf1*=2;
            }
        }
        return -1;
    }
    public static void main(String[] args){
        int[] data = new int[]{2,4,3,6,3,2,5,5};
        int[] result = findNumsAppearOnce(data); // 4,6
        System.out.println(result[0]);
        System.out.println(result[1]);

    }
}
```


57：和为s的数字  

题目要求：  
输入一个递增排序的数组和一个数字s，在数组中查找两个数，使它们的和为s。  
如果有多对和为s，输入任意一对即可。  

解题思路：  
使用两个指针分别指向数组的头、尾。如果和大于s，头部指针后移，如果和小于s，尾部指针前移。  

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/17
 * Time  : 13:25
 * Description:和为s的数字
 **/
public class P280_TwoNumbersWithSum {
    public static int[] findNumbersWithSum(int[] data,int sum){
        int[] result = new int[]{0,0};
        if(data==null || data.length<2)
            return result;
        int curSum = data[0]+data[data.length-1];
        int left = 0,right = data.length-1;
        while (curSum!=sum && left<right){
            if(curSum>sum)
                right--;
            else
                left++;
            curSum = data[left]+data[right];
        }
        if(curSum==sum){
            result[0] = data[left];
            result[1] = data[right];
        }
        return result;
    }
    public static void main(String[] args){
        int[] data = new int[]{1,2,4,7,11,15};
        int[] result = findNumbersWithSum(data,15);
        System.out.println(result[0]);
        System.out.println(result[1]);
    }
}
```

57.2：和为s的连续正数序列

题目要求：
输入一个整数s，打印所有和为s的连续正数序列(至少两个)。例如，输入15，  
由于1+2+3+4+5=4+5+6=7+8=15，所以打印出三个连续序列15,46,7~8。  

解题思路：  
与上一题类似，依旧使用两个指针small，big，值分别为1，2。如果从small加到  
big的和等于s，即找到了一组解，然后让big后移，继续求解。如果和小于s，big  
后移，如果和大于s，small前移。直到small大于s/2停止。  

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/17
 * Time  : 15:47
 * Description:和为s的连续正数序列
 **/
public class P282_ContinuousSequenceWithSum {
    public static void findContinuousSequence(int sum){
        if(sum<3)
            return;
        int small = 1,big = 2,middle = sum>>1;
        int curSum = small+big;
        while (small<=middle){
            if(curSum==sum){
                printContinousSequence(small,big);
                big++;
                curSum+=big;
            }
            else if(curSum<sum){
                big++;
                curSum+=big;
            }
            else{
                curSum-=small;
                small++;
            }
        }
    }
    public static void printContinousSequence(int small,int big){
        for(int i=small;i<=big;i++){
            System.out.print(i);
            System.out.print(" ");
        }
        System.out.println();
    }
    public static void main(String[] args){
        findContinuousSequence(15);
    }
}
```


58：翻转单词顺序  

题目要求：  
输入一个英文句子，翻转单词顺序，单词内字符不翻转，标点符号和普通字母一样  
处理。例如输入输入“I am a student.”，则输出“student. a am I”。  

解题思路：  
首先字符串整体翻转，得到“.tneduts a ma I”；然后以空格作为分隔符进行切分，  
对于切分下来的每一部分再进行翻转，得到“student. a am I”。  

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/17
 * Time  : 16:01
 * Description:翻转字符串
 **/
public class P284_ReverseWordsInSentence {
    public static String reverse(String str){
        StringBuilder stringBuilder = new StringBuilder(str);
        reverseSubString(stringBuilder,0,stringBuilder.length()-1);
        int start = 0,end = stringBuilder.indexOf(" ");
        while (start<stringBuilder.length()&&end!=-1){
            reverseSubString(stringBuilder,start,end-1);
            start = end+1;
            end = stringBuilder.indexOf(" ",start);
        }
        if(start<stringBuilder.length())
            reverseSubString(stringBuilder,start,stringBuilder.length()-1);
        return stringBuilder.toString();
    }
    //翻转stringBuilder[start,end]
    public static void reverseSubString(StringBuilder stringBuilder,int start,int end){
        for(int i=start;i<=start+(end-start)/2;i++){
            char temp = stringBuilder.charAt(i);
            stringBuilder.setCharAt(i,stringBuilder.charAt(end-i+start));
            stringBuilder.setCharAt(end-i+start,temp);
        }
    }
    public static void main(String[] args){
        System.out.println(reverse("I am a student."));
    }
}
```

58.2：左旋转字符串  

题目要求：  
实现一个函数完成字符串的左旋转功能。比如，输入abcdefg和数字2，输出为cdefgab。  

解题思路：  
类似于58.翻转单词顺序。首先对于字符串“abcdefg”整体翻转，得到“gfedcba”；  
然后对于后2个字符“ba”进行翻转，对于剩下的字符“gfedc”进行翻转，得到“cdefgab”。  

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/18
 * Time  : 16:05
 * Description:左旋转字符串
 * abcdeftg 2 => cdefgab
 **/
public class P286_LeftRotateString {
    public static String leftRotateString(String str,int i){
        if(str==null||str.length()==0||i<=0||i>=str.length())
            return str;
        StringBuilder stringBuilder = new StringBuilder(str);
        reverseSubString(stringBuilder,0,stringBuilder.length()-1);
        reverseSubString(stringBuilder,0,stringBuilder.length()-i-1);
        reverseSubString(stringBuilder,stringBuilder.length()-i,stringBuilder.length()-1);
        return stringBuilder.toString();
    }
    //翻转stringBuilder[start,end]
    public static void reverseSubString(StringBuilder stringBuilder,int start,int end){
        for(int i=start;i<=start+(end-start)/2;i++){
            char temp = stringBuilder.charAt(i);
            stringBuilder.setCharAt(i,stringBuilder.charAt(end-i+start));
            stringBuilder.setCharAt(end-i+start,temp);
        }
    }
    public static void main(String[] args){
        String str = "abcdefg";
        System.out.println(leftRotateString(str,2));
    }
}
```

59：滑动窗口的最大值  

题目要求：  
给定一个数组和滑动窗口的大小，请找出所有滑动窗口的最大值。例如，输入数组  
{2,3,4,2,6,2,5,1}和数字3，那么一共存在6个滑动窗口，他们的最大值分别为  
{4,4,6,6,6,5}。  

解题思路：  
思路1：使用暴力求解。假设滑动窗口长度为k，每到一个点都向前遍历k-1个节点，  
那么总时间复杂度为o(nk)。  

思路2：长度为k的滑动窗口其实可以看成一个队列，而滑动窗口的移动可以看成  
队列的出队和入队，因此本题可以转化为求长度为k的队列的最大值。借助之前做  
过的9.用两个栈实现队列和30.包含min函数的栈，我们可以实现求队列的最大值。  
下面我们分析下时间与空间复杂度。用两个栈实现长度为k的队列的入队时间复杂  
度o(1)，出队时间复杂度o(k),空间复杂度为o(k)；包含最值函数的栈（最大深  
度为k）的时间复杂度为o(1),空间复杂度为o(k)。因此长度为k的队列的一次出队  
+入队操作（即窗口前移一位）时间复杂度为o(k)，空间复杂度为o(k)。而这个窗口  
需要完成在长度为n的数组上从头到尾的滑动，因此，本解法的时间复杂度为o(nk)，  
空间复杂度o(k)。  

思路3：把可能成为最大值数字的下标放入双端队列deque，从而减少遍历次数。  
首先，所有在没有查看后面数字的情况下，任何一个节点都有可能成为某个状态  
的滑动窗口的最大值，因此，数组中任何一个元素的下标都会入队。关键在于出队，  
以下两种情况下，该下标对应的数字不会是窗口的最大值需要出队：(1)该下标已  
经在窗口之外，比如窗口长度为3，下标5入队，那么最大值只可能在下标3,4,5中  
出现，队列中如果有下标2则需要出队；(2)后一个元素大于前面的元素，那么前面  
的元素出对，比如目前队列中有下标3、4，data[3] = 50,data[4]=40，下标5入  
队，但data[5] = 70，则队列中的3，4都需要出队。  
数组{2,3,4,2,6,2,5,1}的长度为3的滑动窗口最大值求解步骤如下:  

```
步骤    插入数字    滑动窗口    队列中的下标   最大值
1       2          2           0(2)          N/A          
2       3          2,3         1(3)          N/A   
3       4          2,3,4       2(4)          4   
4       2          3,4,2       2(4),3(2)     4   
5       6          4,2,6       4(6)          6   
6       2          2,6,2       4(6),5(2)     6   
7       5          6,2,5       4(6),6(5)     6   
8       1          2,5,1       6(5),7(1)     5   
时间复杂度在o(n)~o(nk)之间，空间复杂度o(k)。
```

思路3的代码实现如下：

```java
package chapter6;

import java.util.ArrayDeque;
import java.util.Deque;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/18
 * Time  : 17:58
 * Description:滑动窗口的最大值
 **/
public class P288_MaxInSlidingWindow {
    //把可能会成为最大值的下标存储下来，从而降低扫描次数
    public static int[] maxInWindows(int[] data,final int size){
        if(data==null ||data.length==0||data.length<size)
            return new int[0];
        int[] result = new int[data.length-size+1];
        Deque<Integer> deque = new ArrayDeque<>();

        for(int i=0;i<size-1;i++){
            while (!deque.isEmpty()&&data[i]>=data[deque.getLast()])
                deque.removeLast();
            deque.addLast(i);
        }
        for(int i=size-1;i<data.length;i++){
            while (!deque.isEmpty()&&i-deque.getFirst()+1>size)
                deque.removeFirst();
            while (!deque.isEmpty()&&data[deque.getLast()]<=data[i])
                deque.removeLast();
            deque.addLast(i);
            result[i-(size-1)] = data[deque.getFirst()];
        }
        return result;
    }
    public static void main(String[] args){
        int[] data = new int[]{2,3,4,2,6,2,5,1};
        int[] result = maxInWindows(data,3);
        for(int i=0;i<result.length;i++){
            System.out.print(result[i]);
            System.out.print("\t");
        }
    }
}
```


59.2：队列的最大值  

题目要求：  
定义一个队列并实现函数max得到队列里的最大值。要求max，pushBack，  
popFront的时间复杂度都是o(1)。  

解题思路：  
两个思路均延续自59.滑动窗口的最大值  
思路1：借助之前做过的9.用两个栈实现队列和30.包含min函数的栈，我们可以  
实现求队列的最大值。入队时间复杂度o(1)，出队时间复杂度o(n),获取最大值  
时间复杂度o(1)，空间复杂度o(n)。  

思路2：将上一题的滑动窗口看成一个队列即可。入队时间复杂度o(1)，出队时间  
复杂度o(1)，调整记录下标的双端队列的时间复杂度最差为o(n)，获取最值得时间  
复杂度为o(1)。  

思路2的代买实现如下：  

```java
package chapter6;

import java.util.*;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/19
 * Time  : 13:46
 * Description: 队列的最大值
 **/
public class P292_QueueWithMax {
    public static class QueueWithMax<T extends Comparable> {
        private Deque<InternalData<T>> queueData;
        private Deque<InternalData<T>> queueMax;
        private int currentIndex;
        public QueueWithMax() {
            this.queueData = new ArrayDeque<>();
            this.queueMax = new ArrayDeque<>();
            this.currentIndex = 0;
        }
        public T max(){
            if(queueMax.isEmpty())
                return null;
            return queueMax.getFirst().value;
        }
        public void pushBack(T value){
            while (!queueMax.isEmpty()&&value.compareTo(queueMax.getLast().value)>=0)
                queueMax.removeLast();
            InternalData<T> addData = new InternalData<>(value,currentIndex);
            queueMax.addLast(addData);
            queueData.addLast(addData);
            currentIndex++;
        }
        public T popFront(){
            if(queueData.isEmpty())
                return null;
            InternalData<T> delData = queueData.removeFirst();
            if(delData.index==queueMax.getFirst().index)
                queueMax.removeFirst();
            return delData.value;
        }
        private static class InternalData<M extends Comparable> {
            public M value;
            public int index;
            public InternalData(M value,int index){
                this.value = value;
                this.index = index;
            }
        }
    }
    public static void main(String[] args) {
        QueueWithMax<Integer> queue = new QueueWithMax<>();
        queue.pushBack(3);
        System.out.println(queue.max());
        queue.pushBack(5);
        System.out.println(queue.max());
        queue.pushBack(1);
        System.out.println(queue.max());
        System.out.println("开始出队后，调用max");
        System.out.println(queue.max());
        queue.popFront();
        System.out.println(queue.max());
        queue.popFront();
        System.out.println(queue.max());
        queue.popFront();
        System.out.println(queue.max());


    }
}
```


60：n个骰子的点数  

题目要求：  
把n个骰子仍在地上，所有骰子朝上一面的点数之和为s。输入n，  
打印出s的所有可能的值的出现概率。  

解题思路：
新加入一个骰子，它出现1-6的概率是相等的，可以看成各出现一次，那么出现和  
为s的次数等于再加入之前出现和为s-1,s-2,s-3,s-4,s-5,s-6这6种情况的次数  
之和。如此循环，直到加入n个骰子结束。  

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/20
 * Time  : 12:07
 * Description:n个骰子的点数
 **/
public class P294_DicesProbability {
    public static void printProbability(int number){
        if(number<=0)
            return;
        int result[][] = new int[2][6*number+1];
        for(int i=1;i<=6;i++)
            result[1][i] = 1;
        for (int num=2;num<=number;num++){
            for(int i=num;i<6*num+1;i++){
                for(int j=i-6;j<i;j++)
                    if(j>0)
                        result[num%2][i] += result[(num-1)%2][j];
            }
        }
        double sum = 0;
        for(int i=number;i<6*number+1;i++)
            sum += result[number%2][i];
        System.out.println("number = "+number);
        for(int i=number;i<6*number+1;i++)
            System.out.println("probability "+i+":"+result[number%2][i]/sum);
    }
    public static void main(String[] args){
        printProbability(2);
        printProbability(0);
        printProbability(11);
    }
}
```

61：扑克牌中的顺子  

题目要求：   
抽取5张牌，判断是不是一个顺子。2-10为数字本身，A为1，J为11，Q为12，  
K为13，大小王可堪称任意数字。  

解题思路：  
将1-10,J,Q,K记作1-13，大小王记作0，0可以转化为任意的1-13的数字，  
判断输入的5个数字是否能组成连续的5个数字即可。  

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/20
 * Time  : 14:59
 * Description:扑克牌中的顺子
 **/
public class P298_ContinousCards {
    public static boolean isContinous(int[] data){
        if(data==null || data.length!=5)
            return false;
        int[] table = new int[14];
        for(int i=0;i<data.length;i++){
            if(data[i]>13||data[i]<0)
                return false;
            table[data[i]]++;
        }
        int start = 1;
        while (table[start]==0)
            start++;
        int king = table[0];
        for(int i=start;i<start+5;i++){
            if(i>13)
                break;
            if(table[i]>1||table[i]<0)
                return false;
            else if(table[i]==0){
                if(king==0)
                    return false;
                else
                    king--;
            }
        }
        return true;
    }
    public static void main(String[] args){
        int[] data1 = new int[]{4,2,7,12,1}; //false
        int[] data2 = new int[]{0,5,6,12,0}; //false
        int[] data3 = new int[]{6,5,8,7,4};  //true
        int[] data4 = new int[]{0,5,6,9,8};  //true
        int[] data5 = new int[]{0,13,0,12,0}; //true
        System.out.println(isContinous(data1));
        System.out.println(isContinous(data2));
        System.out.println(isContinous(data3));
        System.out.println(isContinous(data4));
        System.out.println(isContinous(data5));
    }
}
```

62：圆圈中最后剩下的数字  

题目要求：  
0，1，2...n-1这n个数字拍成一个圆圈，从数字0开始，每次从这个圆圈里删除  
第m个数字，求剩下的最后一个数字。例如0，1，2，3，4这5个数字组成的圈，  
每次删除第3个数字，一次删除2，0，4，1，因此最后剩下的是3。  

解题思路：  
最直接的思路是用环形链表模拟圆圈，通过模拟删除过程，可以得到最后剩下的  
数字，那么这道题目就变成了删除链表中某一个节点。假设总节点数为n，删除一个  
节点需要走m步，那么这种思路的时间复杂度为o(mn)，空间复杂度o(n)。  

思路2比较高级，较难理解，可遇不可求。将圆圈表示成一个函数表达式，将删除节  
点的过程表示成函数映射的变化，时间复杂度o(n)，空间复杂度o(1)!有兴趣的话  
可以搜素”约瑟夫环“去详细了解。  

```java
package chapter6;
import structure.ListNode;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/20
 * Time  : 16:20
 * Description:圆圈中最后剩下的数字
 * n=5,m=3,从0,1,2,3,4组成的圆中删除第3个数字
 * 依次删除3,0,4,1，最终剩下的是3
 **/
public class P300_LastNumberInCircle {
    public static int lastRemaining(int n,int m){
        if(n<1||m<1)
            return -1;
        ListNode<Integer> head = new ListNode<>(0);
        ListNode<Integer> cur = head;
        for(int i=1;i<n;i++){
            ListNode<Integer> node = new ListNode<>(i);
            cur.next = node;
            cur = cur.next;
        }
        cur.next = head;
        cur = head;
        while (true){
            //长度为1结束循环
            if(cur.next==cur)
                return cur.val;
            //向后移动
            for(int i=1;i<m;i++)
                cur=cur.next;
            //删除当前节点
            cur.val = cur.next.val;
            cur.next = cur.next.next;
            //删除后，cur停在被删节点的后一节点处
        }
    }
    //另一个思路分析过程较复杂，不强求了。可搜约瑟夫环进行了解。
    public static void main(String[] args){
        System.out.println(lastRemaining(5,3)); //3
        System.out.println(lastRemaining(6,7)); //4
        System.out.println(lastRemaining(0,7)); //-1
    }
}
```


63：股票的最大利润  

题目要求：  
求买卖股票一次能获得的最大利润。例如，输入{9，11，8，5，7，12，16，14}，  
5的时候买入，16的时候卖出，则能获得最大利润11。  

解题思路：  
遍历过程中记录最小值min，然后计算当前值与min的差值diff，并更新maxDiff，  
maxDiff=max(diff)。  

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/20
 * Time  : 19:16
 * Description:股票的最大利润（仅能买卖一次）
 **/
public class P304_MaximalProfit {
    public static int maxDiff(int[] data){
        if(data==null||data.length<2)
            return 0;
        int min = data[0];
        int maxDiff = data[1] - min;
        if(data[1]<min)
            min = data[1];
        for(int i=2;i<data.length;i++){
            if(data[i]-min>maxDiff)
                maxDiff = data[i]-min;
            if(data[i]<min)
                min = data[i];
        }
        return maxDiff;
    }
    public static void main(String[] args){
        int[] data1 = new int[]{9,11,8,5,7,12,16,14};
        int[] data2 = new int[]{9,8,7,6,5,4,3,1};
        System.out.println(maxDiff(data1)); //11
        System.out.println(maxDiff(data2)); //-1
    }
}
```

64：求1+2+...+n  

题目要求：  
求1+2+...+n，要求不能使用乘除法，for，while，if，else，switch，  
case等关键词及条件判断语句？：。  

解题思路：  
不能用循环，那么可以使用递归调用求和。但又不能使用if，结束条件如何生效？  
可以使用如下形式替代if语句  

替代if的一种方式：boolean b=判断条件&&(t=递归执行语句)>0  
b并没有实际的用途，只是为了使表达式完成。当判断条件不满足，递归也就结束了。  

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/20
 * Time  : 19:59
 * Description:不使用乘除、for、while、if、switch、？:求和
 * 省略if的一种方式：boolean b=判断条件&&待执行语句>0
 **/
public class P307_Accumulate {
    public static int getSum(int num){
        int t=0;
        boolean b = (num>0)&&((t=num+getSum(num-1))>0);
        return t;
    }
    public static void main(String[] args){
        System.out.println(getSum(10));
    }
}
```


65：不用加减乘除做加法  

题目要求：  
写一个函数，求两个正数之和，要求在函数体内不能使用四则运算符号。  

解题思路：  
不能用四则运算，那只能通过位运算了。其实四则运算是针对十进制，位运算是针对  
二进制，都能用于运算。下面以0011（即3）与0101（即5）相加为例说明  

```
1.两数进行异或：  0011^0101=0110 这个数字其实是把原数中不需进位的二进制位进行了组合
2.两数进行与：    0011&0101=0001 这个数字为1的位置表示需要进位，而进位动作是需要向前一位进位
3.左移一位：      0001<<1=0010

此时我们就完成0011 + 0101 = 0110 + 0010的转换

如此转换下去，直到其中一个数字为0时，另一个数字就是原来的两个数字的和
```

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/20
 * Time  : 21:03
 * Description:不用加减乘除做加法
 **/
public class P310_AddTwoNumbers {
    public static int add(int a,int b){
        int sum = a^b;
        int carry = (a&b)<<1;
        int temp;
        while (carry!=0){
            temp = sum;
            sum = sum^carry;
            carry = (carry&temp)<<1;
        }
        return sum;
    }
    public static void main(String[] args){
        System.out.println(add(3,5)); //8
        System.out.println(add(3,-5)); //-2
        System.out.println(add(0,1));  //1
    }
}
```


66：构建乘积数组  

题目要求：  
给定数组A[0,1...n-1]，求B[0,1...n-1]，  
要求B[i] = A[0]*A[1]...A[i-1]*A[i+1]...A[n-1]，不能使用除法。  

解题思路：  
如果没有不能用除法的限制，可以通过∑A/A[i]求得B[i]，同时要注意A[i]等于0的情况。  
既然不能使用除法，那就只好用纯乘法计算。如果每一个B中的元素都用  
乘法计算，那么时间复杂度为0(n^2)。  
使用纯乘法，还有更高效的思路。定义C[i]=A[0]*A[1]...A[i-1]，  
那么C[i]=C[i-1]*A[n-1]；定义D[i]=A[i+1]*...A[n-2]*A[n-1],  
那么D[i] =D[i+1]*A[i+1]；因此C[i],D[i]都可以递推地求出来，  
且B[i]=C[i]*D[i]。步骤如下：  

1.计算C数组；  
2.依次求得D[0],D[1]...D[n-1]，同时完成B[i]=C[i]\*D[i]的计算。  
如果最终结果存放于result[]中，第一步的C的值就可以先放到result中，  
而D的元素可以存放于一个变量temp中，通过result[i] = temp\*result[i]求得B。  

时间复杂度o(n)，由于计算B必然要申请长度为n的空间，除此之外，该方法  
仅申请了几个变量，空间复杂度o(1)。

```java
package chapter6;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/21
 * Time  : 8:45
 * Description:构建乘积数组
 **/
public class P313_ConstructArray {
    public static int[] multiply(int[] data){
        if(data==null||data.length<2)
            return null;
        int[] result = new int[data.length];
        //求得数组C，存于result中
        result[0] = 1;
        for(int i=1;i<result.length;i++)
            result[i] = result[i-1]*data[i-1];
        //先求得数组D中元素，再与C中元素相乘，存于result中
        int temp = 1;
        for(int i=data.length-2;i>=0;i--){
            //数组D中的元素值
            temp = temp * data[i+1];
            //计算B[i]=C[i]*D[i]
            result[i] = result[i] * temp;
        }
        return result;
    }
    public static void main(String[] args){
        int[] data = new int[]{1,2,3,4,5};
        int[] result = multiply(data);
        for(int i=0;i<result.length;i++){
            System.out.print(result[i]);
            System.out.print("  ");
        }
    }
}
```

67：把字符串转换成整数  

解题思路：  
此题比较麻烦的点是特殊情况较多，需要考虑完全。  
1.如果字符串前后有空格，要去除；  
2.如果是空串或null，要特殊处理；  
3.如果头部有多个正负号，要特殊处理；  
4.如果数值超出int的范围，要特殊处理；比int的最大值还要大，已经上溢，这肯定  
不能通过数字的大小比较，所以需要在字符串的状态下判断是否上溢或下溢。  
5.遇到非数字的字符，则转换停止；  
刚刚发现一个新的情况未做处理，在数字之前有多个0,如果0之后的数字有溢出情况，  
而前面的0又没有去处，这种溢出就不会被捕获，还缺这个情况的判断。这种思路真心  
不好，一条主线是转换成整数，中间穿插出很多特殊情况的分支，感觉就像在打补丁。  

```java
package chapter7;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/22
 * Time  : 16:06
 * Description:把字符串转换成整数
 **/
public class P318_StringToInt {
//    atoi的需求是这样的：
//    如果前面有空格，需要剔除空格；
//    剔除空格后，第一个字符串如果是+号，认为是正数；如果是-号，认为是负数；
//    后面的字符如果不是数字，那么返回0，如果是数字，返回实际的数字。遇到不是数字的字符，转换结束。
//    此外，要考虑空串问题，数值溢出问题[2^(-31) ~ 2^31-1]。

    public static int strToInt(String str) throws Exception{
        if(str==null || str.length()==0)
            throw new Exception("待转换字符串为null或空串");
        String MAX_INT_PLUS_1 = Integer.toString(Integer.MIN_VALUE).substring(1);
        StringBuilder stringBuilder = new StringBuilder(str.trim());
        int flag = 0; //记录无符号的正（2）正（1），负（-1）,初始值（0）
        if(stringBuilder.charAt(0)=='-')
            flag = -1;
        else if(stringBuilder.charAt(0)=='+')
            flag = 1;
        else if(stringBuilder.charAt(0)>='0'&&stringBuilder.charAt(0)<='9')
            flag = 2;
        else
            return 0;
        int endIndex = 1;
        while (endIndex<stringBuilder.length()&&stringBuilder.charAt(endIndex)>='0'&&stringBuilder.charAt(endIndex)<='9')
            endIndex++;
        if(flag==2){
            if(stringBuilder.substring(0,endIndex).toString().compareTo(MAX_INT_PLUS_1)>=0)
                throw new Exception("数值上溢,待转换字符串为"+str);
            return Integer.parseInt(stringBuilder.substring(0,endIndex));
        }
        else{
            if(flag==1&&stringBuilder.substring(1,endIndex).compareTo(MAX_INT_PLUS_1)>=0)
                throw new Exception("数值上溢,待转换字符串为"+str);
            if(flag==-1&&stringBuilder.substring(1,endIndex).compareTo(MAX_INT_PLUS_1)>0)
                throw new Exception("数值下溢,待转换字符串为"+str);
            if(flag==-1&&stringBuilder.substring(1,endIndex).compareTo(MAX_INT_PLUS_1)==0)
                //此处注意，此种情况不能用绝对值*（-1），该绝对值已经超出正数的最大值
                return Integer.MIN_VALUE;
            return flag*Integer.parseInt(stringBuilder.substring(1,endIndex));
        }
    }

    public static void funcTest(){
        try {
        System.out.println(strToInt(" 100")); //100
        System.out.println(strToInt("-100")); //-100
        System.out.println(strToInt("0")); //0
        System.out.println(strToInt("-0"));//0
        System.out.println(strToInt("1.23"));  //1
        System.out.println(strToInt("-1.23")); //-1
        System.out.println(strToInt(".123"));  //0
        }
        catch (Exception e){
            e.printStackTrace();
        }
    }
    public static void edgeTest(){
        try {
            System.out.println(strToInt("2147483647"));  //2147483647
        }
        catch (Exception e){
            System.out.println(e.getMessage());
        }
        try {
            System.out.println(strToInt("-2147483647")); //-2147483647
        }
        catch (Exception e){
            System.out.println(e.getMessage());
        }
        try {
            System.out.println(strToInt("2147483647"));  //2147483647
        }
        catch (Exception e){
            System.out.println(e.getMessage());
        }
        try {
            System.out.println(strToInt("2147483648"));  //上溢
        }
        catch (Exception e){
            System.out.println(e.getMessage());
        }
        try {
            System.out.println(strToInt("-2147483648")); //-2147483648
        }
        catch (Exception e){
            System.out.println(e.getMessage());
        }
        try {
            System.out.println(strToInt("-2147483649")); //下溢
        }
        catch (Exception e){
            System.out.println(e.getMessage());
        }
        try {
            System.out.println(strToInt(null)); //待转换字符串为null或空串
        }
        catch (Exception e){
            System.out.println(e.getMessage());
        }
        try {
            System.out.println(strToInt(""));   //待转换字符串为null或空串
        }
        catch (Exception e){
            System.out.println(e.getMessage());
        }
    }
    public static void main(String[] args){
        funcTest();
        edgeTest();
    }
}
```


68：树中两个节点的最低公共祖先  

题目要求：  
输入一棵树的根节点，输入两个被观察节点，求这两个节点的最低(最近)公共祖先。  

解题思路：  
此题比较开放，主要是对于“树”没有做明确说明，所以原书中就对树的  
可能情况做了假设，然后就衍生出多种思路  

```
1）如果是二叉，搜索树：
    遍历找到比第一个节点大，比第二个节点小的节点即可
2）如果是父子间有双向指针的树：
    由下往上看，转化为找两个链表的第一个公共节点问题
3）如果只是一个包含父到子的指针的普通树：
   3.1）如果不能使用额外空间，从根节点开始判断他的子树是否包含那两个节点，找到最小的的子树即可
        时间复杂度o(n^2)(此为最差，平均不太好算。。。),空间复杂度为o(1)
   3.2) 如果能用额外空间，可以遍历两次(深度优先)获取根节点到那两个节点的路径，然后求两个路径的最后一个公共节点
        时间复杂度o(n)，空间复杂度o(logn)
(1)(2)比较简单。下面仅对(3)，以下图所示的树为例，进行思路实现与求解验证


                        A
                      /   \
                     B     C 
                  /     \
                D        E 
               / \     / | \
              F   G  H   I   J
```

```java
package chapter7;

import java.util.*;

/**
 * Created with IntelliJ IDEA
 * Author: ryder
 * Date  : 2017/8/22
 * Time  : 17:15
 * Description:树中两个节点的最低公共祖先
 *
 **/
public class P326CommonParentInTree {
    public static class CommonTreeNode{
        public char val;
        public List<CommonTreeNode> children;
        public CommonTreeNode(char val){
            this.val = val;
            children = new LinkedList<>();
        }
        public void addChildren(CommonTreeNode... children){
            for(CommonTreeNode child:children)
                this.children.add(child);
        }
    }
    // 3.1所述的解法
    public static CommonTreeNode getLastParent1(CommonTreeNode root,CommonTreeNode node1,CommonTreeNode node2){
        if(root==null || node1==null || node2==null || !isInSubTree(root,node1,node2))
            return null;
        CommonTreeNode curNode = root;
        while (true){
            for(CommonTreeNode child:curNode.children){
                if(isInSubTree(child,node1,node2)){
                    curNode = child;
                    break;
                }
                if(child==curNode.children.get(curNode.children.size()-1))
                    return curNode;
            }
        }
    }
    public static boolean isInSubTree(CommonTreeNode root,CommonTreeNode node1,CommonTreeNode node2){
        Queue<CommonTreeNode> queue = new LinkedList<>();
        CommonTreeNode temp = null;
        int count = 0;
        queue.add(root);
        while (count!=2 && !queue.isEmpty()){
            temp = queue.poll();
            if(temp==node1||temp==node2)
                count++;
            if(!temp.children.isEmpty())
                queue.addAll(temp.children);
        }
        if(count==2)
            return true;
        return false;
    }
    // 3.2所述的解法
    public static CommonTreeNode getLastParent2(CommonTreeNode root,CommonTreeNode node1,CommonTreeNode node2){
        List<CommonTreeNode> path1 = new ArrayList<>();
        List<CommonTreeNode> path2 = new ArrayList<>();
        getPath(root,node1,path1);
        getPath(root,node2,path2);
        CommonTreeNode lastParent = null;
        for(int i=0;i<path1.size()&&i<path2.size();i++){
            if(path1.get(i)==path2.get(i))
                lastParent = path1.get(i);
            else
                break;
        }
        return lastParent;
    }
    public static boolean getPath(CommonTreeNode root,CommonTreeNode node,List<CommonTreeNode> curPath){
        if(root==node)
            return true;
        curPath.add(root);
        for(CommonTreeNode child:root.children){
            if(getPath(child,node,curPath))
                return true;
        }
        curPath.remove(curPath.size()-1);
        return false;
    }

    public static void main(String[] args){
        CommonTreeNode root = new CommonTreeNode('A');
        CommonTreeNode b = new CommonTreeNode('B');
        CommonTreeNode c = new CommonTreeNode('C');
        CommonTreeNode d = new CommonTreeNode('D');
        CommonTreeNode e = new CommonTreeNode('E');
        CommonTreeNode f = new CommonTreeNode('F');
        CommonTreeNode g = new CommonTreeNode('G');
        CommonTreeNode h = new CommonTreeNode('H');
        CommonTreeNode i = new CommonTreeNode('I');
        CommonTreeNode j = new CommonTreeNode('J');
        root.addChildren(b,c);
        b.addChildren(d,e);
        d.addChildren(f,g);
        e.addChildren(h,i,j);
        System.out.println(getLastParent1(root,f,h).val);
        System.out.println(getLastParent2(root,f,h).val);
        System.out.println(getLastParent1(root,h,i).val);
        System.out.println(getLastParent2(root,h,i).val);

    }
}
```