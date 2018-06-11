
2. 单例模式  

```java

public class No2 {

    /**
     * 设计一个类，我们只能生成该类的一个实例。
     */
    public static void main(String[] args) {

    }

}

//饿汉式  线程安全
class A {
    private static final A a = new A();
    private A() {
    }

    public static A getInstance() {
        return a;
    }
}

//懒汉式 线程安全写法
class B {
    private static B b = null;
    private B() {
    }

    public static B getInstance() {
        if (b == null) {
            synchronized (B.class) {
                if (b == null)
                    b = new B();
            }
        }
        return b;
    }
}

//真正的线程安全是加上volatile，防止初始化对象和instance指向内存两个步骤重排序
class C {
    private volatile static C instance;
    public static C getInstance() {
        if (instance == null) {
            synchronized (C.class) {
                if (instance == null) instance = new C(); // instance为volatile，现在没问题了
            }
        }
        return instance;
    }
}

//静态内部类的方法
class InstanceFactory {
    private static class InstanceHolder {
        public static C instance = new C();
    }

    public static C getInstance() {
        return InstanceHolder.instance;//这里将导致InstanceHolder类被初始化
    }
}
```

3. 数组中重复的数字  

```java

/**
 * Created by ryder on 2017/6/11. * 一个长度为n的数组，值的范围在0~n-1内，有一个或多个数字重复，求其中任意一个
 */

public class P39_DuplicationInArray {

	//方法一：暴力求解，不会修改原始数据，时间复杂度o(n^2)，空间复杂度o(1)
    public static int getDuplication(int[] data) {
        if (data == null || data.length < 2) return -1;
        for (int i = 0; i < data.length - 1; i++) {
            for (int j = i + 1; j < data.length; j++) {
                if (data[i] == data[j]) return data[i];
            }
        }
        return -1;
    }

    //方法二：排序，会修改原始数据，时间复杂度o(nlogn)，空间复杂度o(1)
    public static int getDuplication2(int[] data) {
        if (data == null || data.length < 2)
            return -1; //Arrays.sort(data); //或者使用内置函数进行排序

        quickSort(data, 0, data.length - 1);
        if (data.length < 2) return -1;
        int prev = data[0];
        for (int i = 1; i < data.length; i++) {
            if (data[i] == prev) return prev;
            else prev = data[i];
        }
        return -1;
    }

    public static void quickSort(int[] data, int start, int end) {
        if (start >= end) return;
        int bound = partion(data, start, end);
        quickSort(data, start, bound - 1);
        quickSort(data, bound + 1, end);
    }

    public static int partion(int[] data, int start, int end) {
        if (start >= end) return end;
        int pivot = data[start];
        int left = start, right = end;
        while (left < right) {
            while (left < right && data[right] >= pivot) right--;
            if (left < right) data[left++] = data[right];
            while (left < right && data[left] < pivot) left++;
            if (left < right) data[right--] = data[left];
        }
        data[left] = pivot;
        return left;
    } 

    //方法三：借助哈希表，不会修改原始数据，时间复杂度o(n),空间复杂度o(n)
    public static int getDuplication3(int[] data) {
        if (data == null || data.length < 2) return -1;
        int[] hashTable = new int[data.length];
        for (int item : data) {
            if (hashTable[item] >= 1) return item;
            else {
                hashTable[item] = 1;
            }
        }
        return -1;
    }


	//方法四：根据数字特点排序，会修改原始数据，时间复杂度o(n),空间复杂度o(1)
    public static int getDuplication4(int[] data) {
        if (data == null || data.length < 2) return -1;
        for (int i = 0; i < data.length; i++) {
            while (data[i] != i) {
                if (data[i] == data[data[i]]) return data[i];
                else {
                    int temp = data[i];
                    data[i] = data[temp];
                    data[temp] = temp;
                }
            }
        }
        return -1;
    } 

    //方法五：类似于二路归并，这个思路应该说是二路计数，不修改原始数据，时间复杂度o(nlogn),空间复杂度o(1)

    public static int getDuplication5(int[] data) {
        if (data == null || data.length < 2)
            return -1; //数组值在[start,end]间
        int start = 0;
        int end = data.length - 2;
        while (start <= end) {
            int middle = (end - start) / 2 + start;
            int count = countRange(data, start, middle);
            if (start == end) {
                if (count > 1) return start;
                else return -1;
            }
            if (count > middle - start + 1) end = middle;
            else start = middle + 1;
        }
        return -1;
    }

    public static int countRange(int[] data, int start, int end) {
        int count = 0;
        for (int i = 0; i < data.length; i++) {
            if (start <= data[i] && end >= data[i]) count++;
        }
        return count;
    }

    public static void main(String[] args) {
        int[] data = {2, 3, 1, 0, 2, 5, 3};
        System.out.println(getDuplication(data));
        System.out.println(getDuplication2(data));
        System.out.println(getDuplication3(data));
        System.out.println(getDuplication4(data));
        System.out.println(getDuplication5(data));
        int[] data1 = {2, 3, 1, 0, 4, 5, 5};
        System.out.println(getDuplication(data1));
        System.out.println(getDuplication2(data1));
        System.out.println(getDuplication3(data1));
        System.out.println(getDuplication4(data1));
        System.out.println(getDuplication5(data1));
    }
}



```

4. 二维数组中的查找  

```java
/**
 * Created by ryder on 2017/6/12. * 二维数组，从左到右递增，从上到下递增，输入一个整数，判断数组中是否含有
 */
public class P44_FindInPartiallySortedMatrix {
    public static boolean findInPartiallySortedMatrix(int[][] data, int target) {
        if (data == null || data.length == 0 || data[0].length == 0) return false;
        int rowMax = data.length - 1, colMax = data[0].length - 1;
        int rowCur = data.length - 1, colCur = 0;
        while (true) {
            if (rowCur < 0 | rowCur > rowMax | colCur < 0 | colCur > colMax) return false;
            if (data[rowCur][colCur] == target) return true;
            else if (data[rowCur][colCur] > target) rowCur--;
            else colCur++;
        }
    }

    public static void main(String[] args) {
        int[][] data = {{1, 2, 8, 9}, {2, 4, 9, 12}, {4, 7, 10, 13}, {6, 8, 11, 15}};
        System.out.println(findInPartiallySortedMatrix(data, 10));
        System.out.println(findInPartiallySortedMatrix(data, 5));
    }
}

```

5. 替换空格  

```java
public class P51_ReplaceSpaces {
    //由于java的字符数组没有结束符，所以需要多传入个原始长度 
	// 先计算好替换后的位置，从后向前替换，时间复杂度o(n) 
    public static void replaceBlank(char[] data, int length) {
        int newLength = length;
        for (int i = 0; i < length; i++) {
            if (data[i] == ' ') newLength += 2;	
        }
        for (int indexOfOld = length - 1, indexOfNew = newLength - 1; indexOfOld >= 0 && indexOfOld != indexOfNew; indexOfOld--, indexOfNew--) {
            if (data[indexOfOld] == ' ') {
                data[indexOfNew--] = '0';
                data[indexOfNew--] = '2';
                data[indexOfNew] = '%';
            } else {
                data[indexOfNew] = data[indexOfOld];
            }
        }
    }

    public static void main(String[] args) {
        char[] predata = "We are happy.".toCharArray();
        char[] data = new char[20];
        for (int i = 0; i < predata.length; i++) data[i] = predata[i];
        System.out.println(data);
        replaceBlank(data, 13);
        System.out.println(data);
    }
}

```

6. 从尾到头打印链表  

```java
public class ListNode<T> {
    public T val;
    public ListNode<T> next;

    public ListNode(T val) {
        this.val = val;
        this.next = null;
    }

    @Override
    public String toString() {
        StringBuilder ret = new StringBuilder();
        ret.append("[");
        for (ListNode cur = this; ; cur = cur.next) {
            if (cur == null) {
                ret.deleteCharAt(ret.lastIndexOf(" "));
                ret.deleteCharAt(ret.lastIndexOf(","));
                break;
            }
            ret.append(cur.val);
            ret.append(", ");
        }
        ret.append("]");
        return ret.toString();
    }
}



/**
 * Created by ryder on 2017/6/13. * 从尾到头打印链表
 */
public class P58_PrintListInReversedOrder { 
	//递归版 
    public static void printReversinglyRecursively(ListNode<Integer> node) {
        if (node == null) return;
        else {
            printReversinglyRecursively(node.next);
            System.out.println(node.val);
        }
    } 

    //非递归版 
    public static void printReversinglyIteratively(ListNode<Integer> node) {
        Stack<Integer> stack = new Stack<>();
        for (ListNode<Integer> temp = node; temp != null; temp = temp.next) 
        	stack.add(temp.val);
        while (!stack.isEmpty()) 
        	System.out.println(stack.pop());
    }

    public static void main(String[] args) {
        ListNode<Integer> head = new ListNode<Integer>(1);
        head.next = new ListNode<Integer>(2);
        head.next.next = new ListNode<Integer>(3);
        printReversinglyRecursively(head);
        System.out.println();
        printReversinglyIteratively(head);
    }
}



```

二叉树的遍历  

```java

/**
 * 二叉树的遍历：先序（递归，非递归），中序（递归，非递归），后序（递归，非递归），层序
 */
public class P60_TraversalOfBinaryTree {

	//先序遍历递归自己写
	public static void preorderRecursively(TreeNode<Integer> node) {
        System.out.println(node.val);
        if (node.left != null)
            preorderRecursively(node.left);
        if (node.right != null)
            preorderRecursively(node.right);
    }
    
    //先序遍历递归版
    public static List<Integer> preorderRecursively(TreeNode<Integer> node) {
        List<Integer> list = new ArrayList<>();
        if (node == null) return list;
        list.add(node.val);
        list.addAll(preorderRecursively(node.left));
        list.addAll(preorderRecursively(node.right));
        return list;
    }

    //中序遍历递归版
    public static List<Integer> inorderRecursively(TreeNode<Integer> node) {
        List<Integer> list = new ArrayList<>();
        if (node == null) return list;
        list.addAll(inorderRecursively(node.left));
        list.add(node.val);
        list.addAll(inorderRecursively(node.right));
        return list;
    } 

    //后序遍历递归版
    public static List<Integer> postorderRecursively(TreeNode<Integer> node) {
        List<Integer> list = new ArrayList<>();
        if (node == null) return list;
        list.addAll(postorderRecursively(node.left));
        list.addAll(postorderRecursively(node.right));
        list.add(node.val);
        return list;
    }

    //先序是当前节点入栈前，把结果放到list里面
    //中序是出栈之前，把结果放到list里面
    //先序遍历非递归版
    public static List<Integer> preorderIteratively(TreeNode<Integer> node) {
        //stack栈顶元素永远为cur的父节点
        Stack<TreeNode<Integer>> stack = new Stack<>();
        TreeNode<Integer> cur = node;
        List<Integer> list = new LinkedList<>();
        if (node == null) return list;
        while (cur != null || !stack.isEmpty()) {
            if (cur != null) {
                list.add(cur.val);
                stack.push(cur);
                cur = cur.left;
            } else {
                cur = stack.pop().right;
            }
        }
        return list;
    }

    //中序遍历非递归版
    public static List<Integer> inorderIteratively(TreeNode<Integer> node) {
        //stack栈顶元素永远为cur的父节点
        Stack<TreeNode<Integer>> stack = new Stack<>();
        TreeNode<Integer> cur = node;
        List<Integer> list = new LinkedList<>();
        while (cur != null || !stack.isEmpty()) {
            if (cur != null) {
                stack.push(cur);
                cur = cur.left;
            } else {
                list.add(stack.peek().val);
                cur = stack.pop().right;
            }
        }
        return list;
    }

    //后序遍历非递归版
    public static List<Integer> postorderIteratively(TreeNode<Integer> node) {
        //stack栈顶元素永远为cur的父节点
        // prevVisted用于区分是从左子树还是右子树返回的
        Stack<TreeNode<Integer>> stack = new Stack<>();
        TreeNode<Integer> cur = node;
        TreeNode<Integer> prevVisted = null;
        List<Integer> list = new LinkedList<>();
        while (cur != null || !stack.isEmpty()) {
            if (cur != null) {
                stack.push(cur);
                cur = cur.left;
            } else {                                     //当前为空了，把当前节点变成栈顶的右子结点
                cur = stack.peek().right;

                if (cur != null && cur != prevVisted) { //不为空，且不是刚出栈
                                                        //需要判断当前节点是否刚被弹出栈，否则会多次进栈
                    stack.push(cur);
                    cur = cur.left;
                } else {                                //当前节点还是为空或者刚出过栈，栈顶出栈并放到结果数组
                    prevVisted = stack.pop();
                    list.add(prevVisted.val);
                    cur = null;
                }
            }
        }
        return list;
    }

    //层序遍历
    public static List<Integer> levelorder(TreeNode<Integer> node) {
        Queue<TreeNode<Integer>> queue = new LinkedList<>();
        List<Integer> list = new LinkedList<>();
        TreeNode<Integer> temp = null;
        if (node == null) return list;
        queue.add(node);
        while (!queue.isEmpty()) {
            temp = queue.poll();
            list.add(temp.val);
            if (temp.left != null) queue.offer(temp.left);
            if (temp.right != null) queue.offer(temp.right);
        }
        return list;
    }

    public static void main(String[] args) {
        //            1
        //              \
        //               2
        //              /
        //            3
        //pre->123  in->132   post->321  level->123
        TreeNode<Integer> root = new TreeNode<Integer>(1);
        root.right = new TreeNode<Integer>(2);
        root.right.left = new TreeNode<Integer>(3);
        List<Integer> list_preorderRecursively = preorderRecursively(root);
        System.out.print("preorderRecursively: " + '\t');
        System.out.println(list_preorderRecursively.toString());
        List<Integer> list_inorderRecursively = inorderRecursively(root);
        System.out.print("inorderRecursively: " + '\t');
        System.out.println(list_inorderRecursively.toString());
        List<Integer> list_postorderRecursively = postorderRecursively(root);
        System.out.print("postorderRecursively: " + '\t');
        System.out.println(list_postorderRecursively.toString());
        System.out.println();
        List<Integer> list_preorderIteratively = preorderIteratively(root);
        System.out.print("preorderIteratively: " + '\t');
        System.out.println(list_preorderIteratively.toString());
        List<Integer> list_inorderIteratively = inorderIteratively(root);
        System.out.print("inorderIteratively: " + '\t');
        System.out.println(list_inorderIteratively.toString());
        List<Integer> list_postorderIteratively = postorderIteratively(root);
        System.out.print("postorderIteratively: " + '\t');
        System.out.println(list_postorderIteratively.toString());
        System.out.println();
        List<Integer> list_levelorder = levelorder(root);
        System.out.print("levelorder: " + '\t');
        System.out.println(list_levelorder.toString());
    }
}
```

7. 重建二叉树  

```java

/**
 * 重建二叉树:  先序+中序，后续+中序可以完成重建，而先序+后序无法完成
 */
public class P62_ConstructBinaryTree {
    public static TreeNode construct(int[] preorder, int[] inorder) {
        if (preorder == null || inorder == null || preorder.length == 0 || preorder.length != inorder.length)
            return null;
        return constructCore(preorder, 0, inorder, 0, inorder.length); //递归时需要传递的，先序和中序的开始位置初始为0，length是中序的长度
    }

    public static TreeNode constructCore(int[] preorder, int preorder_start, int[] inorder, int inorder_start, int length) {
        if (length == 0) return null;                                   //length是先序的长度，为零是递归出口
        int inorder_index = -1;
        for (int i = inorder_start; i < inorder_start + length; i++) {  //遍历中序的序列
            if (inorder[i] == preorder[preorder_start]) {               //找到当前的根节点位置，记录到inorder_index
                inorder_index = i;
                break;
            }
        }
        int left_length = inorder_index - inorder_start;                //左边中序序列的长度，右边序列长length-left_length-1
        TreeNode node = new TreeNode(preorder[preorder_start]);         //根节点变成TreeNode，node的左右孩子是向下递归的结果
        node.left = constructCore(preorder, preorder_start + 1, inorder, inorder_start, left_length);
        node.right = constructCore(preorder, preorder_start + left_length + 1, inorder, inorder_index + 1, length - left_length - 1);
        return node;
    }

    public static void main(String[] args) { 
        //    1 
        //   / \ 
        //  2   3 
        // / \ 
        // 4  5 
        // pre->12453 in->42513 post->45231 

        int[] pre = {1, 2, 4, 5, 3};
        int[] in = {4, 2, 5, 1, 3};
        TreeNode root = construct(pre, in); 
        //对重建后的树,进行前中后序遍历，验证是否重建正确 
        // 调用的重建函数见：http://www.jianshu.com/p/362d4ff42ab2 
        
        List<Integer> preorder = P60_TraversalOfBinaryTree.preorderIteratively(root);
        List<Integer> inorder = P60_TraversalOfBinaryTree.inorderIteratively(root);
        List<Integer> postorder = P60_TraversalOfBinaryTree.postorderIteratively(root);
        System.out.println(preorder);
        System.out.println(inorder);
        System.out.println(postorder);
    }
}

```

8. 二叉树的下一个节点

```java

/*
题目要求：
给定二叉树和其中一个节点，找到中序遍历序列的下一个节点。
树中的节点除了有左右孩子指针，还有一个指向父节点的指针。
*/

/*
思路：
（1）如果输入的当前节点有右孩子，则它的下一个节点即为该右孩子为根节点的子树的最左边的节点，比如2->5,1->3
（2）如果输入的当前节点没有右孩子，就需要判断其与自身父节点的关系：
（2.1）如果当前节点没有父节点，那所求的下一个节点不存在，返回null.
（2.2）如果输入节点是他父节点的左孩子，那他的父节点就是所求的下一个节点,比如4->2
（2.3）如果输入节点是他父节点的右孩子，那就需要将输入节点的父节点作为新的当前节点，
 返回到（2）,判断新的当前节点与他自身父节点的关系,比如5->1
 */


//带有父指针的二叉树节点
class TreeNode {
    public int val;
    public TreeNode left;
    public TreeNode right;
    public TreeNode father;
    public TreeNode(int val){
        this.val = val;
        this.left = null;
        this.right = null;
        this.father = null;
    }
}

public class P65_NextNodeInBinaryTrees {
    public static TreeNode getNext(TreeNode pNode){
        if(pNode==null)
            return null;
        else if(pNode.right!=null){
            pNode = pNode.right;
            while(pNode.left!=null)
                pNode = pNode.left;
            return pNode;
        }
        while(pNode.father!=null){
            if(pNode.father.left==pNode)
                return pNode.father;
            pNode = pNode.father;
        }
        return null;
    }
    public static void main(String[] args){
        //            1
        //          // \\
        //         2     3
        //       // \\
        //      4     5
        //    inorder->42513
        TreeNode root = new TreeNode(1);
        root.left = new TreeNode(2);
        root.left.father = root;
        root.right = new TreeNode(3);
        root.right.father = root;
        root.left.left = new TreeNode(4);
        root.left.left.father = root.left;
        root.left.right = new TreeNode(5);
        root.left.right.father = root.left;

        System.out.println(getNext(root.left.left).val);
        System.out.println(getNext(root.left).val);
        System.out.println(getNext(root.left.right).val);
        System.out.println(getNext(root).val);
        System.out.println(getNext(root.right));
    }
}

```

9. 用两个栈实现队列  


```java

/*
 思路：
（1）对于插入操作，栈与队列都是从队尾进行，因此一行代码就可以完成offer()
（2）对于弹出操作，队列先进先出从队头开始，而栈后进先出从队尾开始，要想取到队头元素，
 就得需要第二个栈stack2的协助：弹出时将stack1的元素依次取出放到stack2中，此时stack2
 进行弹出的顺序就是整个队列的弹出顺序。而如果需要插入，放到stack1中即可。
 总结下，stack1负责插入，stack2负责弹出，如果stack2为空了，将stack1的元素依次弹出并
 存放到stack2中，之后对stack2进行弹出操作。
*/

class MyQueue<T>{
    private Stack<T> stack1 = new Stack<>();
    private Stack<T> stack2 = new Stack<>();
    
    public void offer(T data){
        stack1.push(data);   
    }
    public T poll(){
        if(!stack2.isEmpty()){
            return stack2.pop();
        }
        else if(!stack1.isEmpty()){
            while(!stack1.isEmpty())
                stack2.push(stack1.pop());
            return stack2.pop();
        }
        else
            return null;
    }
}

public class P68_QueueWithTwoStacks {
    public static void main(String[] args){
        MyQueue<Integer> myQueue = new MyQueue<>();
        System.out.println(myQueue.poll());
        myQueue.offer(1);
        myQueue.offer(2);
        myQueue.offer(3);
        System.out.println(myQueue.poll());
        System.out.println(myQueue.poll());
        myQueue.offer(4);
        System.out.println(myQueue.poll());
        System.out.println(myQueue.poll());
        System.out.println(myQueue.poll());
    }
}

```

10. 斐波那契数列  

```java
/**
 * Created by ryder on 2017/6/21.
 * 斐波那契数列
 * f(0)=0,f(1)=1,f(n)=f(n-1)+f(n-2) n>1
 */

解法  解法介绍        时间复杂度   空间复杂度
解法1 依定义递归求解     o(n^2)     o(1)
解法2 从0开始迭代求解    o(n)       o(1)
解法3 借助等比数列公式   o(logn)    o(1)
解法4 借助通项公式       o(1)       o(1)

public class P74_Fibonacci {

    // 依据原始概念的递归解法，时间复杂度o(n^2)
    public static int fibonacci1(int n){
        if(n<=0)
            return 0;
        if(n==1)
            return 1;
        return fibonacci1(n-1)+fibonacci1(n-2);
    }

    // 当前状态只与前两个状态有关。存储前两个值，计算后一个，迭代进行。时间复杂度o(n)
    public static int fibonacci2(int n){
        if(n<=0)
            return 0;
        if(n==1)
            return 1;
        int temp1 =0,temp2=1;
        int result = temp1 + temp2,i=3;
        while(i<=n){
            //也可用一个队列来完成下面三行的操作
            temp1 = temp2;
            temp2 = result;
            result = temp1+temp2;
            i++;
        }
        return result;
    }

    // 借助如下数学公式解决问题。矩阵乘法部分，可用递归解决，时间复杂度o(logn)
    // [ f(n)  f(n-1) ] = [ 1  1 ] ^ n-1   (当n>2)
    // [f(n-1) f(n-2) ]   [ 1  0 ]
    // 证明:
    // [ f(n)  f(n-1) ] = [ f(n-1)+f(n-2)  f(n-1)] = [ f(n-1)  f(n-2)] * [1 1]
    // [f(n-1) f(n-2) ]   [ f(n-2)+f(n-3)  f(n-2)]   [ f(n-2)  f(n-3)]   [1 0]
    // 得到如上递推式，所以
    // [ f(n)  f(n-1) ] = [ f(2)  f(1)] * [1 1]^n-2 = [1 1]^n-1
    // [f(n-1) f(n-2) ]   [ f(1)  f(0)]   [1 0]       [1 0]
    public static int fibonacci3(int n){
        int[][] start = {{1,1},{1,0}};
        return matrixPow(start,n-1)[0][0];
    }
    public static int[][] matrixPow(int[][] start,int n){
         if((n&1)==0){
             int[][] temp = matrixPow(start,n>>1);
             return matrixMultiply(temp,temp);
         }
         else if(n==1){
             return start;
         }
         else{
             return matrixMultiply(start,matrixPow(start,n-1));
         }
    }
    public static int[][] matrixMultiply(int[][] x,int[][] y){
        int[][] result = new int[x.length][y[0].length];
        for(int i=0;i<x.length;i++){
            for(int j=0;j<y[0].length;j++){
                int sum = 0;
                for(int k=0;k<x[0].length;k++){
                    sum += x[i][k]*y[k][j];
                }
                result[i][j] = sum;
            }
        }
        return result;
    }

    // 使用通项公式完成，时间复杂度o(1)
    // f(n) = (1/√5)*{[(1+√5)/2]^n - [(1-√5)/2]^n}
    // 推导过程可参考https://wenku.baidu.com/view/57333fe36bd97f192379e936.html
    public static int fibonacci4(int n){
        double gen5 = Math.sqrt(5);
        return (int)((1/gen5)*(Math.pow((1+gen5)/2,n)- Math.pow((1-gen5)/2,n)));
    }

    public static void main(String[] args){
        System.out.println(fibonacci1(13));
        System.out.println(fibonacci2(13));
        System.out.println(fibonacci3(13));
        System.out.println(fibonacci4(13));
    }
}

```


排序算法比较表格  
https://www.jianshu.com/p/6ae77d17170c  

```java
快排：
package chapter2;
public class P79_Sort {
    //数组快排，时间o(nlogn)(最差n^2)，空间o(logn)(最差n)，递归造成的栈空间的使用，不稳定
    public static void quickSort(int[] data){
        if(data==null || data.length<=1) return;
        quickSortCore(data,0,data.length-1);
    }
    public static void quickSortCore(int[] data,int start,int end){
        if(end-start<=0)
            return;
        int index = quickSortPartition(data,start,end);
        quickSortCore(data,start,index-1);
        quickSortCore(data,index+1,end);
    }
    public static int quickSortPartition(int[] data,int start,int end){
        //选择第一个值作为基准
        int pivot = data[start];
        int left = start,right = end;
        while(left<right){
            while(left<right && data[right]>=pivot)
                right--;
            if(left<right)
                data[left] = data[right];
            while(left<right && data[left]<pivot)
                left++;
            if(left<right)
                data[right] = data[left];
        }
        data[left] = pivot;
        return left;
    }
    public static void testQuickSort(){
        int[] data = {5,4,3,1,2};
        quickSort(data);
        System.out.print("数组快速排序：\t");
        for(int item: data){
            System.out.print(item);
            System.out.print('\t');
        }
        System.out.println();
    }
}
归并排序：
package chapter2;

/**
 * Created by ryder on 2017/6/25.
 * 数组排序算法
 */
public class P79_Sort {
    //数组二路归并，时间o(nlogn)，空间o(n)，稳定
    public static int[] mergeSort(int[] data){
        if(data==null || data.length<=1)
            return data;
        mergeSortCore(data,0,data.length-1);
        return data;
    }
    //对data[start~mid]，data[mid+1~end]归并
    //典型的分治结构：结束条件+分治+和
    public static void mergeSortCore(int[] data,int start,int end){
        if(start>=end)
            return;
        int mid = start + (end - start)/2;
        mergeSortCore(data,start,mid);
        mergeSortCore(data,mid+1,end);
        mergeSortMerge(data,start,mid,end);
    }
    public static void mergeSortMerge(int[] data,int start,int mid,int end){
        if(end==start)
            return;
        int[] temp = new int[end-start+1];
        int left = start,right = mid+1,tempIndex = 0;
        while(left<=mid && right<=end){
            if(data[left]<data[right])
                temp[tempIndex++] = data[left++];
            else
                temp[tempIndex++] = data[right++];
        }
        while(left<=mid)
            temp[tempIndex++] = data[left++];
        while(right<=end)
            temp[tempIndex++] = data[right++];
        for(int i=0;i<temp.length;i++)
            data[start+i] = temp[i];
    }
    public static void testMergeSort(){
        int[] data = {5,4,3,1,2};
        mergeSort(data);
        System.out.print("数组归并排序：\t");
        for(int item: data){
            System.out.print(item);
            System.out.print('\t');
        }
        System.out.println();
    }
}
堆排序：
package chapter2;

/**
 * Created by ryder on 2017/6/25.
 * 数组排序算法
 */
public class P79_Sort {
    //数组堆排序，时间o(nlogn)，空间o(1),不稳定
    //建立最大堆，交换堆的第一个与最后一个元素，调整堆
    //注意，堆排序的0号元素不能使用，为了与其他排序统一接口，先把最小的元素放到0号元素上，再用堆排序
    public static void heapSort(int[] data){
        if(data==null || data.length<=1)
            return;
        //先把最小的元素放到0号元素上
        int minIndex = 0;
        for(int i=1;i<data.length;i++){
            if(data[i]<data[minIndex])
                minIndex = i;
        }
        if(minIndex!=0){
            int temp = data[0];
            data[0] = data[minIndex];
            data[minIndex] = temp;
        }
        //正式开始堆排序（如果0号元素未存值，省略上述代码）
        buildMaxHeap(data);
        for(int indexBound = data.length-1;indexBound>1;){
            int temp = data[indexBound];
            data[indexBound] = data[1];
            data[1] = temp;
            indexBound--;
            adjustMaxHeap(data,1,indexBound);
        }
    }
    public static void buildMaxHeap(int[] data){
        for(int i = data.length/2;i>0;i--){
            adjustMaxHeap(data,i,data.length-1);
        }
    }
    //i表示待调整元素下标，end表示最大堆的最后一个元素的下标，end值会随着排序的进行而减小到1
    public static void adjustMaxHeap(int[] data,int i,int end){
        int left = 2*i;
        int right = 2*i+1;
        int max = i;
        if(left<=end && data[left]>data[max])
            max = left;
        if(right<=end && data[right]>data[max])
            max = right;
        if(max!=i){
            int temp = data[max];
            data[max] = data[i];
            data[i] = temp;
            adjustMaxHeap(data,max,end);
        }
    }

    public static void testHeapSort(){
        int[] data = {5,4,3,1,2};
        heapSort(data);
        System.out.print("数组堆排序：\t");
        for(int item: data){
            System.out.print(item);
            System.out.print('\t');
        }
        System.out.println();
    }
}
冒泡排序：
package chapter2;

/**
 * Created by ryder on 2017/6/25.
 * 数组排序算法
 */
public class P79_Sort {
    //数组冒泡，时间o(n^2)，空间o(1)，稳定
    public static void bubbleSort(int[] data){
        if(data==null || data.length<=1)
            return;
        for(int i=0;i<data.length-1;i++){
            for(int j=1;j<data.length-i;j++){
                if(data[j-1]>data[j]){
                    int temp = data[j-1];
                    data[j-1] = data[j];
                    data[j] = temp;
                }
            }
        }
    }
    public static void testBubbleSort(){
        int[] data = {5,4,3,1,2};
        bubbleSort(data);
        System.out.print("数组冒泡排序：\t");
        for(int item: data){
            System.out.print(item);
            System.out.print('\t');
        }
        System.out.println();
    }
}
选择排序：
package chapter2;

/**
 * Created by ryder on 2017/6/25.
 * 数组排序算法
 */
public class P79_Sort {
    //数组选择排序，时间o(n^2)，空间o(1)，不稳定
    public static void selectionSort(int[] data){
        if(data==null || data.length<=1)
            return;
        for(int i=0;i<data.length-1;i++){
            int minIndex = i;
            for(int j=i+1;j<data.length;j++){
                if(data[j]<data[minIndex])
                    minIndex = j;
            }
            if(i!=minIndex) {
                int temp = data[i];
                data[i] = data[minIndex];
                data[minIndex] = temp;
            }
        }
    }
    public static void testSelectionSort(){
        int[] data = {5,4,3,1,2};
        selectionSort(data);
        System.out.print("数组选择排序：\t");
        for(int item: data){
            System.out.print(item);
            System.out.print('\t');
        }
        System.out.println();
    }
}
插入排序：
package chapter2;

/**
 * Created by ryder on 2017/6/25.
 * 数组排序算法
 */
public class P79_Sort {
    //数组插入排序，时间o(n^2)，空间o(1)，稳定
    public static void insertionSort(int[] data){
        if(data==null || data.length<=1)
            return;
        for(int i=1;i<data.length;i++){
            int j=i;
            int temp = data[i];
            while(j>0 && data[j-1]>temp) {
                data[j] = data[j-1];
                j--;
            }
            data[j] = temp;
        }
    }
    public static void testInsertionSort(){
        int[] data = {5,4,3,1,2};
        insertionSort(data);
        System.out.print("数组插入排序：\t");
        for(int item: data){
            System.out.print(item);
            System.out.print('\t');
        }
        System.out.println();
    }
}
希尔排序：
package chapter2;

/**
 * Created by ryder on 2017/6/25.
 * 数组排序算法
 */
public class P79_Sort {
    //数组希尔排序(插入排序缩小增量)，时间o(n^1.3)，空间o(1)，不稳定
    //时间复杂度是模糊的，有人在大量的实验后得出结论：
    //当n在某个特定的范围后希尔排序的比较和移动次数减少至n^1.3。次数取值在1到2之间。
    public static void shellSort(int[] data){
        if(data==null || data.length<=1)
            return;
        for(int d=data.length/2; d>0; d=d/2){
            for(int i=d;i<data.length;i++){
                int cur = i;
                int temp = data[i];
                while(cur>=d && data[cur-d]>temp){
                    data[cur] = data[cur-d];
                    cur = cur - d;
                }
                data[cur] = temp;
            }
        }
    }
    public static void testShellSort(){
        int[] data = {5,4,3,1,2};
        shellSort(data);
        System.out.print("数组希尔排序：\t");
        for(int item: data){
            System.out.print(item);
            System.out.print('\t');
        }
        System.out.println();
    }
}

```

11. 旋转数组的最小数字  

```java
/**
 * Created by ryder on 2017/6/28.
 * 旋转数组的最小数字
 */
public class P82_MinNumberInRotatedArray {
    public static int min(int[] data){
        if(data==null || data.length==0)
            return -1;
        int left = 0;
        int right = data.length-1;
        int mid;
        while(left<right){
            mid = left+(right-left)/2;
            //left < right
            if(data[left]<data[right])
                return data[left];
            //left > right
            else if(data[left]>data[right]){
                if(data[mid]>=data[left])
                    left = mid + 1;
                else
                    right = mid;
            }
            //left = right
            else{
                if(data[left]<data[mid])
                    left = mid + 1;
                else if(data[left]>data[mid])
                    right = mid;
                else{
                    left = left+1;
                    right = right-1;
                }
            }
        }
        return data[right];
    }
    public static void main(String[] args){
        int[] data1 = {3,4,5,1,2};
        int[] data2 = {1,0,1,1,1};
        int[] data3 = {1,1,1,0,1};
        System.out.println(min(data1));
        System.out.println(min(data2));
        System.out.println(min(data3));
    }
}

```

12：矩阵中的路径

题目要求：设计一个函数，用来判断一个矩阵中是否存在一条包含某字符串的路径。  
（1）起点随意；（2）路径的移动只能是上下左右；（3）访问过的位置不能再访问。  
以下图矩阵为例，包含“bfce”，但是不包含“abfb”。  

a      b      t      g
c      f      c      s
j      d      e      h

```java
/**
 * Created by ryder on 2017/7/2.
 * 矩阵中的路径
 */
public class P89_StringPathInMatrix {
    //回溯法解决
    public static boolean hasPath(char[][] data,String str){
        if(data==null || data.length==0 || str==null || str.length()==0)
            return false;
        int rowLen = data.length;
        int colLen = data[0].length;
        boolean[][] visitFlag = new boolean[rowLen][colLen];
        for(int row=0;row<rowLen;row++){
            for(int col=0;col<colLen;col++){
                visitFlag[row][col] = false;
            }
        }
        for(int row=0;row<rowLen;row++){
            for(int col=0;col<colLen;col++){
                if(hasPathCore(data,row,col,visitFlag,str,0))
                    return true;
            }
        }
        return false;
    }
    public static boolean hasPathCore(char[][] data,int rowIndex,int colIndex,
                                    boolean[][] visitFlag,String str,int strIndex){
        //结束条件
        if(strIndex>=str.length()) return true;
        if(rowIndex<0 || colIndex<0 || rowIndex>=data.length || colIndex>=data[0].length) 
            return false;

        //递归
        if(!visitFlag[rowIndex][colIndex]&&data[rowIndex][colIndex]==str.charAt(strIndex)){
            //如果未被访问，且匹配字符串，标记当前位置为已访问，分上下左右四个位置递归求解
            visitFlag[rowIndex][colIndex] = true;
            boolean result =  
                    hasPathCore(data,rowIndex+1,colIndex,visitFlag,str,strIndex+1) ||
                    hasPathCore(data,rowIndex-1,colIndex,visitFlag,str,strIndex+1) ||
                    hasPathCore(data,rowIndex,colIndex+1,visitFlag,str,strIndex+1) ||
                    hasPathCore(data,rowIndex,colIndex-1,visitFlag,str,strIndex+1);
            //已经求的结果，无需修改标记了
            if(result)
                return true;
            //当前递归的路线求解失败，要把这条线路上的标记清除掉
            //因为其他起点的路径依旧可以访问本路径上的节点。
            else{
                visitFlag[rowIndex][colIndex] = false;
                return false;
            }
        }
            else
            return false;
    }
    public static void main(String[] args){
        char[][] data = {
                {'a','b','t','g'},
                {'c','f','c','s'},
                {'j','d','e','h'}};
        System.out.println(hasPath(data,"bfce")); //true
        System.out.println(hasPath(data,"abfb")); //false,访问过的位置不能再访问
    }
}

```

13. 机器人的运动范围  

题目要求：
地上有一个m行n列的方格，一个机器人从坐标(0,0)的各自开始移动，它每次  
可以向上下左右移动一格，但不能进入横纵坐标数位之和大于k的格子。  
例如，当k等于18时，机器人能进入(35,37)，因为3+5+3+7=18；但却不能进入  
(35,38)，因为3+5+3+8=19>18。  
请问该机器人能够到达多少个格子。  

解题思路：  
本题依旧考察回溯法。  
每前进一步后，可选移动项为上下左右四个；为了判断某一个格子是否可以进入  
从而进行计数，不仅需要考虑边界值，计算各位数字之和，更要判断该格子是否  
已经被访问过，。所以需要一个布尔矩阵，用来记录各格子是否已被访问。整体思  
路与12题类似，具体请参考本系列的导航帖。  

```java
package chapter2;
/**
 * Created by ryder on 2017/7/4.
 * 机器人的运动范围
 */
public class P92_RobotMove {
    //依旧回溯
    public static int movingCount(int threshold,int rowLen,int colLen){
        if(rowLen<=0 || colLen<=0 || threshold<0)
            return 0;
        boolean[][] visitFlag = new boolean[rowLen][colLen];
        for(int row=0;row<rowLen;row++){
            for(int col=0;col<colLen;col++)
                visitFlag[row][col] = false;
        }
        return movingCountCore(threshold,rowLen,colLen,0,0,visitFlag);
    }
    public static int movingCountCore(int threshold,int rowLen,int colLen,int row,int col,boolean[][] visitFlag){
        int count = 0;
        if(canGetIn(threshold,rowLen,colLen,row,col,visitFlag)){
            visitFlag[row][col] = true;
            count = 1+movingCountCore(threshold,rowLen,colLen,row-1,col,visitFlag)+
                    movingCountCore(threshold,rowLen,colLen,row+1,col,visitFlag)+
                    movingCountCore(threshold,rowLen,colLen,row,col-1,visitFlag)+
                    movingCountCore(threshold,rowLen,colLen,row,col+1,visitFlag);
        }
        return count;
    }
    public static boolean canGetIn(int threshold,int rowLen,int colLen,int row,int col,boolean[][] visitFlag){
        return row>=0 && col>=0 && row<rowLen
                && col<colLen && !visitFlag[row][col]
                && getDigitSum(row)+getDigitSum(col)<=threshold;
    }
    public static int getDigitSum(int number){
        int sum=0;
        while (number>0){
            sum += number%10;
            number/=10;
        }
        return sum;
    }

    public static void main(String[] args){
        System.out.println(movingCount(0,3,4)); //1
        System.out.println(movingCount(1,3,4)); //3
        System.out.println(movingCount(9,2,20)); //36
    }
}
```