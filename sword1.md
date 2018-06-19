
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


14. 剪绳子  

题目要求：  
给你一根长度为n的绳子，请把绳子剪成m段，记每段绳子长度为  k[0],k[1]...k[m-1],  
求k[0]k[1]...k[m-1]的最大值。已知绳子长度n为整数，m>1(至少要剪一刀，不能不剪)，  
k[0],k[1]...k[m-1]均要求为整数。  
例如，绳子长度为8时，把它剪成3-3-2，得到最大乘积18；绳子长度为3时，把它剪成2-1，  
得到最大乘积2。  

我们定义长度为n的绳子剪切后的最大乘积为f(n),剪了一刀后,f(n)=max(f(i)*f(n-i));  
假设n为10，第一刀之后分为了4-6，而6也可能再分成2-4（6的最大是3-3，但过程中还是  
要比较2-4这种情况的），而上一步4-6中也需要求长度为4的问题的最大值，可见，各个子  
问题之间是有重叠的，所以可以先计算小问题，存储下每个小问题的结果，逐步往上，求得  
大问题的最优解。  

上述算法的时间复杂度为o(n^2);但其实，可以使用贪婪算法在o(1)时间内得到答案：  
n<5时，和动态规划一样特殊处理；n>=5时，尽可能多地剪长度为3的绳子，当剩下的绳子长  
度为4时，剪成2-2；比如长度为13的绳子， 剪成3-3-3-2-2；贪婪算法虽然快，但一般都思  
路奇特，可遇不可求。且面试官一般都会要求证明，数学功底要好 。  


```java
/**
 * Created by ryder on 2017/7/5.
 * 剪绳子
 */
public class P96_CuttingRope {
    public static int maxCutting(int length){
        if(length<2) return 0;
        if(length==2)return 1;
        if(length==3)return 2;
        int[] dp = new int[length+1];
        dp[0]=0;
        dp[1]=1;
        dp[2]=2;
        dp[3]=3;
        int max = 0;
        int temp = 0;
        for(int i=4;i<=length;i++){
            max = 0;
            for(int j=1;j<=i/2;j++){
                temp = dp[j]*dp[i-j];
                if(temp>max)
                    max = temp;
            }
            dp[i] = max;
        }
        return dp[length];
    }
    public static void main(String[] args){
        for(int i=2;i<10;i++){
            System.out.println("长度为"+i+"的最大值->"+maxCutting(i));
        }
    }
}
```


15. 位运算  

题目要求：
实现一个函数，输入一个int型整数，输出该数字在计算机中二进制表示形式的1的个数。  
例如9->1001,输出2；-3->11111111111111111111111111111101,输出31。  

解题思路：
考查位运算，此题要注意负数的处理。首先要明确计算机中，数字是以补码的形式存储的，  
原码反码补码不清楚的话请自己谷歌百度。其次，明确位运算符，与&，或|，非~，异或^,  
<<左移位，>>带符号右移位，>>>无符号右移位(java有此符号，c++没有)

解法一：将数字无符号右移，直到为0。  

解法二：使用一个标记，初始为1，让标记值与原输入数字异或，然后标记值左移。解法一  
是原数字右移，而解法二是标记左移，从java来看思路类似但换了个角度；但这个思路在  
C++就很关键，因为C++中没有>>>运算符，只能用解法二。  

解法三：没接触过的人应该会觉得比较新颖。对于二进制数有如下结论：【把一个整数减去1  
之后再和原来的整数做位与运算，得到的结果相当于把原整数的二进制表示形式的最右边的  
1变成0】。比如1001，执行一次上述结论，1001&1000=1000，将最右边的1改为了0；再执行  
一次，1000&0111=0000，第二个1也改成了0。因此能执行几次该结论，就有几个1。  
对于解法一二，都需要循环32次，判断每一个比特位是否为1，而解法三，循环次数等于比特  
位为1的个数。时间上是有改进的。  

```java
/**
 * Created by ryder on 2017/7/6.
 * 二进制中的1的个数
 */
public class P100_NumberOf1InBinary {
    public static int numberOfOne1(int n){
        int count=0;
        while(n!=0){
            if((n&1)!=0)
                count++;
            n>>>=1;
        }
        return count;
    }
    public static int numberOfOne2(int n){
        int count=0;
        int flag=1;
        while(flag!=0){
            if((n&flag)!=0)
                count++;
            flag<<=1;
        }
        return count;
    }
    public static int numberOfOne3(int n){
        int count=0;
        while(n!=0){
            n = n&(n-1);
            count++;
        }
        return count;
    }
    public static void main(String[] args){
        System.out.println(numberOfOne1(3));
        System.out.println(numberOfOne1(-3));
        System.out.println(numberOfOne2(3));
        System.out.println(numberOfOne2(-3));
        System.out.println(numberOfOne3(3));
        System.out.println(numberOfOne3(-3));
    }
}

```

16. 数值的整数次方  

题目要求：  
实现函数double power（double base，int exponent），求base的exponent次方。  
不能使用库函数，不需要考虑大数问题。  

解题思路：本题考查考虑问题的完整性。如下几个点要注意：  
要考虑一些特殊情况，如指数为负、指数为负且底数为0、0的0次方要定义为多少。  
底数为0的定义。对于一个double类型的数，判断它与另一个数是否相等，不能用“==”，  
一般都需要一个精度，见下面的equal函数。  
对于报错的情况，比如0的负数次方，要如何处理。书中提了三种错误处理方法：用函数  
返回值来提示错误；用一个全局变量来提示错误；抛出异常；三种方法优缺点比较如下  

错误处理方法             优点                        缺点  
返回值           和相关函数的API一致          不能方便地使用计算结果  
全局变量        能够方便地使用计算结果       用户可能忘记检查全局变量  
异常        可自定义异常类型，逻辑清晰明了   抛出异常时对性能有负面影响  

```java
/**
 * Created by ryder on 2017/7/6.
 * 数值的整数次方
 */
public class P110_Power {
    static boolean invalidInput = false;
    public static double power(double base,int exponent){
        //0的0次方在数学上没有意义，为了方便也返回1，也可特殊处理
        if(exponent==0)
            return 1;
        if(exponent<0){
            if(equal(base,0)){
               //通过全局变量报错
                invalidInput = true;
                return 0;
            }
            else
                return 1.0/powerWithPositiveExponent(base,-1*exponent);
        }
        else
            return powerWithPositiveExponent(base,exponent);
    }
    public static boolean equal(double x,double y){
       return -0.00001<x-y && x-y<0.00001;
    }
    public static double powerWithPositiveExponent(double base,int exponent){
        if(exponent==0)
            return 1;
        else if((exponent&1)==0){//为偶数
            double temp = powerWithPositiveExponent(base,exponent>>1);
            return temp*temp;
        }
        else{
            double temp = powerWithPositiveExponent(base,exponent>>1);
            return base*temp*temp;
        }
    }
    public static void main(String[] args){
        System.out.println("2^3="+power(2,3)+"\t是否报错:"+invalidInput);
        System.out.println("2^-3="+power(2,-3)+"\t是否报错:"+invalidInput);
        System.out.println("0^3="+power(0,3)+"\t是否报错:"+invalidInput);
        System.out.println("0^-3="+power(0,-3)+"\t是否报错:"+invalidInput);
    }
}

```


17. 打印从1到最大的n位数  

题目要求：  
比如输入2，打印1,2......98,99；  

解题思路：  
此题需要考虑大数问题。本帖是使用字符串模拟数字的加法。  

```java
/**
 * Created by ryder on 2017/7/6.
 *
 */
public class P114_Print1ToMaxOfNDigits {
    //在字符串上模拟加法
    public static void print1ToMaxOfNDigits(int num){
        if(num<=0)
            return;
        StringBuilder number = new StringBuilder(num);
        for(int i=0;i<num;i++)
            number.append('0');
        while(increment(number)){
            printNumber(number);
        }
    }
    public static boolean increment(StringBuilder str){
        for(int i=str.length()-1;i>=0;i--){
            if(str.charAt(i)<'9' && str.charAt(i)>='0'){
                str.setCharAt(i,(char)(str.charAt(i)+1));
                return true;
            }
            else if(str.charAt(i)=='9'){
                str.setCharAt(i,'0');
            }
            else{
                return false;
            }
        }
        return false;
    }
    public static void printNumber(StringBuilder number){
        boolean flag = false;
        for(int i=0;i<number.length();i++){
            if(flag)
                System.out.print(number.charAt(i));
            else{
                if(number.charAt(i)!='0'){
                    flag = true;
                    System.out.print(number.charAt(i));
                }
            }
        }
        System.out.println();
    }
    public static void main(String[] args){
        print1ToMaxOfNDigits(2);
    }
}
```


18. 删除链表的节点  

题目要求：  
在o(1)时间内删除单链表的节点。  

解题思路：  
直接删除单链表某一节点，无法在o(1)时间得到该节点的前一个节点，  
因此无法完成题目要求。可以将欲删节点的后一个节点的值拷贝到欲删  
节点之上，删除欲删节点的后一个节点，从而可以在o(1)时间内完成  
删除。（对于尾节点，删除仍然需要o(n),其他点为o(1)，因此平均时  
间复杂度为o(1)，满足要求）  

```java
package structure;
/**
 * Created by ryder on 2017/6/13.
 */
public class ListNode<T> {
    public T val;
    public ListNode<T> next;
    public ListNode(T val){
        this.val = val;
        this.next = null;
    }
    @Override
    public String toString() {
        StringBuilder ret = new StringBuilder();
        ret.append("[");
        for(ListNode cur = this;;cur=cur.next){
            if(cur==null){
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
package chapter3;
import structure.ListNode;
/**
 * Created by ryder on 2017/7/7.
 * o(1)时间删除链表的节点
 */
public class P119_DeleteNodeInList {
    public static ListNode<Integer> deleteNode(ListNode<Integer> head,ListNode<Integer> node){
        if(node==head){
            return head.next;
        }
        else if(node.next!=null){
            node.val = node.next.val;
            node.next = node.next.next;
            return head;
        }
        else{
            ListNode<Integer> temp=head;
            while(temp.next!=node)
                temp = temp.next;
            temp.next = null;
            return head;
        }
    }
    public static void main(String[] args){
        ListNode<Integer> head = new ListNode<>(1);
        ListNode<Integer> node2 = new ListNode<>(2);
        ListNode<Integer> node3 = new ListNode<>(3);
        head.next = node2;
        node2.next = node3;
        System.out.println(head);
        head = deleteNode(head,node3);
        System.out.println(head);
        head = deleteNode(head,head);
        System.out.println(head);
    }
}
```


18. 题目二：删除排序链表中重复的节点  

题目要求：  
比如[1,2,2,3,3,3],删除之后为[1];  

解题思路：  
由于是已经排序好的链表，需要确定重复区域的长度，删除后还需要将被删去的前与后连接，  
所以需要三个节点pre,cur,post，cur-post为重复区域，删除后将pre与post.next连接即可。  
此外，要注意被删结点区域处在链表头部的情况，因为需要修改head。  

```java
package structure;
/**
 * Created by ryder on 2017/6/13.
 */
public class ListNode<T> {
    public T val;
    public ListNode<T> next;
    public ListNode(T val){
        this.val = val;
        this.next = null;
    }
    @Override
    public String toString() {
        StringBuilder ret = new StringBuilder();
        ret.append("[");
        for(ListNode cur = this;;cur=cur.next){
            if(cur==null){
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
package chapter3;

import structure.ListNode;

/**
 * Created by ryder on 2017/7/7.
 * 删除排序链表中的重复节点
 */
public class P122_deleteDuplicatedNode {
    public static ListNode<Integer> deleteDuplication(ListNode<Integer> head){
        if(head==null||head.next==null)
            return head;
        ListNode<Integer> pre = null;
        ListNode<Integer> cur = head;
        ListNode<Integer> post = head.next;
        boolean needDelete = false;
        while (post!=null){
            if(cur.val.equals(post.val)){
                needDelete = true;
                post=post.next;
            }
            else if(needDelete && !cur.val.equals(post.val)){
                if(pre==null)
                    head = post;
                else
                    pre.next=post;
                cur = post;
                post = post.next;
                needDelete = false;
            }
            else{
                pre = cur;
                cur = post;
                post = post.next;
            }
        }
        if(needDelete && pre!=null)
            pre.next = null;
        else if(needDelete && pre==null)
            head = null;
        return head;
    }
    public static void main(String[] args){
        ListNode<Integer> head = new ListNode<>(1);
        head.next= new ListNode<>(1);
        head.next.next = new ListNode<>(2);
        head.next.next.next = new ListNode<>(2);
        head.next.next.next.next = new ListNode<>(2);
        head.next.next.next.next.next = new ListNode<>(3);
        System.out.println(head);
        head = deleteDuplication(head);
        System.out.println(head);
    }
}
```

19. 正则表达式匹配  

题目要求：  
实现正则表达式中.和*的功能。.表示任意一个字符，*表示他前面的字符的任意次（含0次）。  
比如aaa与a.a和ab*ac*a匹配，但与aa.a和ab*a不匹配。  

解题思路：  
.就相当于一个万能字符，正常匹配即可；但*的匹配会涉及到前一个字符。所以要分模式串后  
一个字符不是*或没有后一个字符，模式串后一个字符是*这几个大的情况，之再考虑.的问题。  

```java
package chapter3;
/**
 * Created by ryder on 2017/7/13.
 * 正则表达式匹配
 * 完成.(任何一个字符)和*(前面字符的任意次数)
 */
public class P124_RegularExpressionsMatching {
    public static boolean match(String str,String pattern){
        if(str==null || pattern==null)
            return false;
        return matchCore(new StringBuilder(str),0,new StringBuilder(pattern),0);
    }
    public static boolean matchCore(StringBuilder str,Integer strIndex,StringBuilder pattern, Integer patternIndex){
        //如果匹配串和模式串都匹配结束
        if(strIndex==str.length() && patternIndex==pattern.length())
            return true;
        if(strIndex!=str.length() && patternIndex==pattern.length())
            return false;
        if(strIndex==str.length() && patternIndex!=pattern.length()) {
            if(patternIndex+1<pattern.length()&&pattern.charAt(patternIndex+1)=='*')
                return matchCore(str,strIndex,pattern,patternIndex+2);
            else
                return false;
        }
        //如果模式串的第二个字符不是*或者已经只剩一个字符了
        if(patternIndex==pattern.length()-1|| pattern.charAt(patternIndex+1)!='*'){
            if(pattern.charAt(patternIndex)=='.' || pattern.charAt(patternIndex)==str.charAt(strIndex))
                return matchCore(str,strIndex+1,pattern,patternIndex+1);
            else
                return false;
        }
        //如果模式串的第二个字符是*
        else{
            //模式和当前字符匹配上
            if(pattern.charAt(patternIndex)=='.'||pattern.charAt(patternIndex)==str.charAt(strIndex))
                return matchCore(str,strIndex+1,pattern,patternIndex)     //*匹配多个相同字符
                        ||matchCore(str,strIndex+1,pattern,patternIndex+2)//*只匹配当前字符
                        ||matchCore(str,strIndex,pattern,patternIndex+2); //*一个字符都不匹配
            //模式和当前字符没有匹配上
            else
                return matchCore(str,strIndex,pattern,patternIndex+2);
        }
    }
    public static void main(String[] args){
        System.out.println(match("aaa","a.a"));//true
        System.out.println(match("aaa","ab*ac*a"));//true
        System.out.println(match("aaa","aa.a"));//false
        System.out.println(match("aaa","ab*a"));//false
    }
}
```

20. 表示数值的字符串  

题目要求：  
判断一个字符串是否表示数值，如+100,5e2，-123，-1E-16都是，  
12e，1e3.14，+-5,1.2.3,12e+5.4都不是。  
提示：表示数值的字符串遵循模式A[.[B]][e|EC] 或者 .B[e|EC];  
A,B,C表示整数，|表示或。[]表示可有可无。  

解题思路：  
此题也没有没什么特殊思路，就按照A[.[B]][e|EC] 或者 .B[e|EC];  
A,B,C这两种模式匹配下即可。  

```java
package chapter3;
/**
 * Created by ryder on 2017/7/13.
 * 表示数值的字符串
 */
public class P127_NumberStrings {
    public static boolean isNumeric(String str){
        //正确的形式：A[.[B]][e|EC] 或者 .B[e|EC];
        if(str==null||str.length()==0)
            return false;
        int index;
        if(str.charAt(0)!='.'){
            index = scanInteger(str,0);
            if(index==-1)
                return false;
            if(index==str.length())
                return true;
            if(str.charAt(index)=='.'){
                if(index==str.length()-1)
                    return true;
                index = scanInteger(str,index+1);
                if(index==str.length())
                    return true;
            }
            if(str.charAt(index)=='e'||str.charAt(index)=='E'){
                index = scanInteger(str,index+1);
                if(index==str.length())
                    return true;
                else
                    return false;
            }
            return false;
        }
        else{
            index = scanInteger(str,1);
            if(index==-1)
                return false;
            if(index==str.length())
                return true;
            if(str.charAt(index)=='e'||str.charAt(index)=='E'){
                index = scanInteger(str,index+1);
                if(index==str.length())
                    return true;
            }
            return false;

        }

    }
    public static int scanInteger(String str,Integer index){
        if(index>=str.length())
            return -1;
        if(str.charAt(index)=='+'||str.charAt(index)=='-')
            return scanUnsignedInteger(str,index+1);
        else
            return scanUnsignedInteger(str,index);
    }
    public static int scanUnsignedInteger(String str,Integer index){
        int origin = index;
        while(str.charAt(index)>='0'&&str.charAt(index)<='9'){
            index++;
            if(index==str.length())
                return index;
        }
        if(origin==index)
            index = -1;
        return index;
    }
    public static void main(String[] args){
        System.out.println(isNumeric("+100"));//true
        System.out.println(isNumeric("5e2")); //true
        System.out.println(isNumeric("-123"));//true
        System.out.println(isNumeric("3.1416"));//true
        System.out.println(isNumeric("-1E-16"));//true
        System.out.println(isNumeric(".6"));//true
        System.out.println(isNumeric("6."));//true
        System.out.println(isNumeric("12e"));//false
        System.out.println(isNumeric("1a3.14"));//false
        System.out.println(isNumeric("1.2.3"));//false
        System.out.println(isNumeric("+-5"));//false
        System.out.println(isNumeric("12e+5.4"));//false
    }
}
```

21. 调整数组顺序使奇数位于偶数前面  

题目要求：  
实现一个函数来调整数组中的数字，使得所有奇数位于数组的前半部分，偶数位于后半部分。  

解题思路：  
其实我想到的第一个思路就是用双指针从两端向中间扫描，处理过程与快排很相似，时间复杂度o(n)  ，这应该是最快的解法了。思路简单，就当复习下快排吧。至于复杂度更高的解法，我就不再写了。  
书中提到了一点，是将判断部分写成函数，这样能够提升代码的可扩展性，这的确是很好的一个提醒。  
那样处理之后，这一类问题（非负数在前，负数在后；能被3整除的在前，不能的在后；）都只需修改  
下判断函数即可解决。  

```java
package chapter3;
/**
 * Created by ryder on 2017/7/14.
 * 调整数组顺序使奇数位于偶数前面
 */
public class P129_ReorderArray {
    public static void reorder(int[] data){
        if(data==null||data.length<2)
            return;
        int left=0,right=data.length-1;
        while(left<right){
            while (left<right&&!isEven(data[left]))
                left++;
            while (left<right&&isEven(data[right]))
                right--;
            if(left<right){
                int temp=data[left];
                data[left]=data[right];
                data[right]=temp;
            }
        }
    }
    public static boolean isEven(int n){
        return (n&1)==0;
    }
    public static void main(String[] args){
        int[] data = {1,2,3,4,5,7,7};
        reorder(data);
        for(int i=0;i<data.length;i++) {
            System.out.print(data[i]);
            System.out.print('\t');
        }
        System.out.println();
    }
}
```


22. 链表中倒数第k个节点  

题目要求：  
求链表中倒数第k个节点。链表的尾节点定义为倒数第1个节点。  

解题思路：  
如果先求链表的长度，计算后再从头遍历，虽然时间复杂度是o(n),但需要两遍  
扫描。更好的方式是使用两个距离为k的指针向右移动，这种方式只需扫描一遍。  
chapter3主要考察细节，这道题也不例外。需要注意链表是否为空，链表长度  
是否达到k，k是否是个正数。  

```java
package structure;
/**
 * Created by ryder on 2017/6/13.
 */
public class ListNode<T> {
    public T val;
    public ListNode<T> next;
    public ListNode(T val){
        this.val = val;
        this.next = null;
    }
    @Override
    public String toString() {
        StringBuilder ret = new StringBuilder();
        ret.append("[");
        for(ListNode cur = this;;cur=cur.next){
            if(cur==null){
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
package chapter3;
import structure.ListNode;
/**
 * Created by ryder on 2017/7/14.
 * 链表中倒数第k个节点
 */
public class P134_KthNodeFromEnd {
    public static ListNode<Integer> findKthToTail(ListNode<Integer> head,int k){
        if(head==null||k<=0)
            return null;
        ListNode<Integer> slow=head,fast=head;
        for(int i=0;i<k;i++){
            //i==k-1,第三个测试例通不过
            if(fast.next!=null || i==k-1)
                fast = fast.next;
            else
                return null;
        }
        while(fast!=null){
            slow = slow.next;
            fast = fast.next;
        }
        return slow;
    }
    public static void main(String[] args){
        ListNode<Integer> head = new ListNode<>(1);
        head.next= new ListNode<>(2);
        head.next.next = new ListNode<>(3);
        System.out.println(findKthToTail(head,1).val);
        System.out.println(findKthToTail(head,2).val);
        System.out.println(findKthToTail(head,3).val);
        System.out.println(findKthToTail(head,4));
    }
}
```

23. 链表中环的入口节点  

题目要求：  
假设一个链表中包含环，请找出入口节点。若没有环则返回null。  

解题思路：  
当然可以使用遍历。顺序访问，当第二次访问同一个节点时，  
那么那个节点就是入口节点。不过这种解法需要o(n)的空间。  
能不能把空间降为o(1)，时间依旧为o(n)。当然可以。这种  
解法的思路是：首先申请两个指针，快指针一次前进两步，  
慢指针一次前进一步，初始化都再链表头部。然后开始移动，  
如果他们指向了同一个节点，则证明有环，否则没环。当他  
们指向了同一个节点时，慢指针再次初始化，指向头结点。  
快慢指针前进步数都改为1，当他们再次指向同一个节点，  
这个节点就是环的入口节点。不明白的话请先看证明过程链  
接,然后再看我说的思路总结。  


```cpp 
#include <stdio.h>

struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};

class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode *fast = head;
        ListNode *slow = head;
        ListNode *meet = NULL;
        while(fast){
            slow = slow->next;           //先各走一步
            fast = fast->next;
            if (!fast){                  //fast到链表尾，就说明没有环，结点为奇数个
                return NULL;
            }
            fast = fast->next;           //fast再走一步
            if (fast == slow){           //记录相遇位置
                meet = fast;
                break;
            }
        }

        while(head && meet){          //head和meet相遇就是环起点
            if (head == meet){
                return head;
            }
            head = head->next;
            meet = meet->next;
        }
        return NULL;
    }
};

int main(){
    ListNode a(1);
    ListNode b(2);
    ListNode c(3);
    ListNode d(4);
    ListNode e(5);
    ListNode f(6);
    ListNode g(7);
    a.next = &b;
    b.next = &c;
    c.next = &d;
    d.next = &e;
    e.next = &f;
    f.next = &g;
    g.next = &c;
    Solution solve;
    ListNode *node = solve.detectCycle(&a);
    if (node){
        printf("%d\n", node->val);
    }
    else{
        printf("NULL\n");
    }
    return 0;
}

```


24. 反转链表  

解题思路：  
想要链表反转时不断裂，至少需要3个变量记录，pre，cur，post。与前面的题目类似，  
初始化pre为null，cur为head，post为head.next。初始化之前要注意检查链表的长度。  

```java
package structure;
/**
 * Created by ryder on 2017/6/13.
 */
public class ListNode<T> {
    public T val;
    public ListNode<T> next;
    public ListNode(T val){
        this.val = val;
        this.next = null;
    }
    @Override
    public String toString() {
        StringBuilder ret = new StringBuilder();
        ret.append("[");
        for(ListNode cur = this;;cur=cur.next){
            if(cur==null){
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
package chapter3;
import structure.ListNode;
/**
 * Created by ryder on 2017/7/14.
 * 反转链表
 */
public class P142_ReverseList {
    public static ListNode<Integer> reverseList(ListNode<Integer> head){
        if(head==null || head.next==null)
            return head;
        ListNode<Integer> pre = null;
        ListNode<Integer> cur = head;
        ListNode<Integer> post = head.next;
        while(true){
            cur.next = pre;
            pre = cur;
            cur = post;
            if(post!=null)
                post = post.next;
            else
                return pre;
        }
    }
    public static void main(String[] args){
        ListNode<Integer> head = new ListNode<>(1);
        head.next= new ListNode<>(2);
        head.next.next = new ListNode<>(3);
        System.out.println(head);
        head = reverseList(head);
        System.out.println(head);
    }
}
```


25. 合并两个排序的链表  

题目要求：  
输入两个递增排序的链表，要求合并后保持递增。  

解题思路：  
这个题目是二路链表归并排序的一部分，或者说是最关键的归并函数部分。熟悉归并排序  
的话做这个题应该很容易。思路很简单，注意链表的next指针，初始条件，结束条件即可。  

```java
package structure;
/**
 * Created by ryder on 2017/6/13.
 */
public class ListNode<T> {
    public T val;
    public ListNode<T> next;
    public ListNode(T val){
        this.val = val;
        this.next = null;
    }
    @Override
    public String toString() {
        StringBuilder ret = new StringBuilder();
        ret.append("[");
        for(ListNode cur = this;;cur=cur.next){
            if(cur==null){
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
package chapter3;
import structure.ListNode;
/**
 * Created by ryder on 2017/7/14.
 * 合并两个排序的链表
 */
public class P145_MergeSortedLists {
    public static ListNode<Integer> merge(ListNode<Integer> head1,ListNode<Integer> head2){
        if(head1==null)
            return head2;
        if(head2==null)
            return head1;
       ListNode<Integer> index1 = head1;
       ListNode<Integer> index2 = head2;
       ListNode<Integer> index = null;
       if(index1.val<index2.val) {
           index = index1;
           index1 = index1.next;
       }
       else {
           index = index2;
           index2 = index2.next;
       }
       while(index1!=null && index2!=null){
           if(index1.val<index2.val) {
               index.next = index1;
               index = index.next;
               index1 = index1.next;
           }
           else {
               index.next = index2;
               index = index.next;
               index2 = index2.next;
           }
       }
       if(index1!=null)
           index.next = index1;
       else
           index.next = index2;
       return head1.val<head2.val?head1:head2;
    }
    public static void main(String[] args){
        ListNode<Integer> head1 = new ListNode<>(1);
        head1.next= new ListNode<>(3);
        head1.next.next = new ListNode<>(5);
        head1.next.next.next = new ListNode<>(7);
        ListNode<Integer> head2 = new ListNode<>(2);
        head2.next= new ListNode<>(4);
        head2.next.next = new ListNode<>(6);
        head2.next.next.next = new ListNode<>(8);
        System.out.println(head1);
        System.out.println(head2);
        ListNode<Integer> head =merge(head1,head2);
        System.out.println(head);
    }
}
```

26. 树的子结构  

题目要求：  
输入两棵二叉树A和B，判断B是不是A的子结构。  

解题思路：  
当A有一个节点与B的根节点值相同时，则需要从A的那个节点开始严格匹配，对应于下面的  
tree1HasTree2FromRoot函数。如果匹配不成功，则返回到开始匹配的那个节点，对它的  
左右子树继续判断是否与B的根节点值相同，重复上述过程。  

```java
package structure;
/**
 * Created by ryder on 2017/6/12.
 * 树节点
 */
public class TreeNode<T> {
    public T val;
    public TreeNode<T> left;
    public TreeNode<T> right;
    public TreeNode(T val){
        this.val = val;
        this.left = null;
        this.right = null;
    }
}
package chapter3;
import structure.TreeNode;
/**
 * Created by ryder on 2017/7/14.
 * 树的子结构
 */
public class P148_SubstructureInTree {
    public static boolean hasSubtree(TreeNode<Integer> root1, TreeNode<Integer> root2){
        if(root2==null)
            return true;
        if(root1==null)
            return false;
        if(root1.val.equals(root2.val)){
            if(tree1HasTree2FromRoot(root1,root2))
                return true;
        }
        return hasSubtree(root1.left,root2) || hasSubtree(root1.right,root2);
    }
    public static boolean tree1HasTree2FromRoot(TreeNode<Integer> root1, TreeNode<Integer> root2){
        if(root2==null)
            return true;
        if(root1==null)
            return false;
        if(root1.val.equals(root2.val) && tree1HasTree2FromRoot(root1.left,root2.left) && tree1HasTree2FromRoot(root1.right,root2.right))
            return true;
        else
            return false;

    }
    public static void main(String[] args){
        TreeNode<Integer> root1 = new TreeNode<>(8);
        root1.left = new TreeNode<>(8);
        root1.right = new TreeNode<>(7);
        root1.left.left = new TreeNode<>(9);
        root1.left.right = new TreeNode<>(2);
        root1.left.right.left = new TreeNode<>(4);
        root1.left.right.right = new TreeNode<>(7);
        TreeNode<Integer> root2 = new TreeNode<>(8);
        root2.left = new TreeNode<>(9);
        root2.right = new TreeNode<>(2);
        TreeNode<Integer> root3 = new TreeNode<>(2);
        root3.left = new TreeNode<>(4);
        root3.right = new TreeNode<>(3);
        System.out.println(hasSubtree(root1,root2));
        System.out.println(hasSubtree(root1,root3));
    }
}
```

27. 二叉树的镜像  

题目要求：  
求一棵二叉树的镜像。  

解题思路：  
二叉树的镜像，即左右子树调换。从上到下，递归完成即可。  

```java
package structure;
import java.util.LinkedList;
import java.util.Queue;
/**
 * Created by ryder on 2017/6/12.
 * 树节点
 */
public class TreeNode<T> {
    public T val;
    public TreeNode<T> left;
    public TreeNode<T> right;
    public TreeNode(T val){
        this.val = val;
        this.left = null;
        this.right = null;
    }
    //层序遍历
    @Override
    public String toString() {
        StringBuilder stringBuilder = new StringBuilder("[");
        Queue<TreeNode<T>> queue = new LinkedList<>();
        queue.offer(this);
        TreeNode<T> temp;
        while(!queue.isEmpty()){
            temp = queue.poll();
            stringBuilder.append(temp.val);
            stringBuilder.append(",");
            if(temp.left!=null)
                queue.offer(temp.left);
            if(temp.right!=null)
                queue.offer(temp.right);
        }
        stringBuilder.deleteCharAt(stringBuilder.lastIndexOf(","));
        stringBuilder.append("]");
        return stringBuilder.toString();
    }
}
package chapter4;
import structure.TreeNode;
/**
 * Created by ryder on 2017/7/15.
 * 二叉树的镜像
 */
public class P151_MirrorOfBinaryTree {
    public static void mirrorRecursively(TreeNode<Integer> root){
        if(root==null)
            return;
        if(root.left==null&&root.right==null)
            return;
        TreeNode<Integer> temp = root.left;
        root.left = root.right;
        root.right = temp;
        mirrorRecursively(root.left);
        mirrorRecursively(root.right);
    }
    public static void main(String[] args){
        TreeNode<Integer> root = new TreeNode<>(8);
        root.left = new TreeNode<>(6);
        root.right = new TreeNode<>(10);
        root.left.left = new TreeNode<>(5);
        root.left.right = new TreeNode<>(7);
        root.right.left = new TreeNode<>(9);
        root.right.right = new TreeNode<>(11);
        System.out.println(root);
        mirrorRecursively(root);
        System.out.println(root);
    }
}
```

28. 对称的二叉树  

题目要求：  
判断一棵二叉树是不是对称的。如果某二叉树与它的镜像一样，称它是对称的。  

解题思路：  
比较直接的思路是比较原树与它的镜像是否一样。书中就是用的这种方式  
（比较二叉树的前序遍历和对称前序遍历）。但这种思路下，树的每个节  
点都要读两次，也就是遍历两遍。  
其实可以只遍历一次完成判断：我们可以通过判断待判断二叉树的左子树  
与右子树是不是对称的来得知该二叉树是否是对称的。   

```java
package structure;
import java.util.LinkedList;
import java.util.Queue;
/**
 * Created by ryder on 2017/6/12.
 * 树节点
 */
public class TreeNode<T> {
    public T val;
    public TreeNode<T> left;
    public TreeNode<T> right;
    public TreeNode(T val){
        this.val = val;
        this.left = null;
        this.right = null;
    }
    //层序遍历
    @Override
    public String toString() {
        StringBuilder stringBuilder = new StringBuilder("[");
        Queue<TreeNode<T>> queue = new LinkedList<>();
        queue.offer(this);
        TreeNode<T> temp;
        while(!queue.isEmpty()){
            temp = queue.poll();
            stringBuilder.append(temp.val);
            stringBuilder.append(",");
            if(temp.left!=null)
                queue.offer(temp.left);
            if(temp.right!=null)
                queue.offer(temp.right);
        }
        stringBuilder.deleteCharAt(stringBuilder.lastIndexOf(","));
        stringBuilder.append("]");
        return stringBuilder.toString();
    }
}
package chapter4;
import structure.TreeNode;
import java.util.LinkedList;
import java.util.Queue;
/**
 * Created by ryder on 2017/7/15.
 * 对称的二叉树
 */
public class P159_SymmetricalBinaryTree {
    //递归实现
    public static boolean isSymmetrical(TreeNode<Integer> root){
        if(root==null)
            return true;
        if(root.left==null && root.right==null)
            return true;
        if(root.left==null || root.right==null)
            return false;
        return isSymmetrical(root.left,root.right);
    }
    public static boolean isSymmetrical(TreeNode<Integer> root1,TreeNode<Integer> root2){
        if(root1==null && root2==null)
            return true;
        if(root1==null || root2==null)
            return false;
        if(!root1.val.equals(root2.val))
            return false;
        return isSymmetrical(root1.left,root2.right) && isSymmetrical(root1.right,root2.left);
    }
    //迭代实现
    public static boolean isSymmetrical2(TreeNode<Integer> root){
        if(root==null)
            return true;
        if(root.left==null && root.right==null)
            return true;
        if(root.left==null || root.right==null)
            return false;
        Queue<TreeNode<Integer>> queueLeft = new LinkedList<>();
        Queue<TreeNode<Integer>> queueRight = new LinkedList<>();
        queueLeft.offer(root.left);
        queueRight.offer(root.right);
        TreeNode<Integer> tempLeft,tempRight;
        while(!queueLeft.isEmpty()|| !queueRight.isEmpty()){
            tempLeft = queueLeft.poll();
            tempRight = queueRight.poll();
            if(tempLeft.val.equals(tempRight.val)){
                if(tempLeft.left!=null)
                    queueLeft.offer(tempLeft.left);
                if(tempLeft.right!=null)
                    queueLeft.offer(tempLeft.right);
                if(tempRight.right!=null)
                    queueRight.offer(tempRight.right);
                if(tempRight.left!=null)
                    queueRight.offer(tempRight.left);
            }
            else
                return false;
        }
        if(queueLeft.isEmpty() && queueRight.isEmpty())
            return true;
        else
            return false;
    }
    public static void main(String[] args){
        TreeNode<Integer> root = new TreeNode<>(8);
        root.left = new TreeNode<>(6);
        root.right = new TreeNode<>(6);
        root.left.left = new TreeNode<>(5);
        root.left.right = new TreeNode<>(7);
        root.right.left = new TreeNode<>(7);
        root.right.right = new TreeNode<>(5);
        System.out.println(isSymmetrical(root));
        System.out.println(isSymmetrical2(root));
    }
}
```

29. 顺时针打印矩阵  

题目要求：  
输入一个矩阵，按照从外向里以顺时针的顺序一次打印出每一个数字。  
如果输入如下矩阵：  

1    2    3    4
5    6    7    8
9    10   11   12
13   14   15   16
则依次打印出1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10。

解题思路：  
此题的任务清晰明了，需要小心的是要考虑清楚边界情况。  
上例中，环绕一次后，剩下的矩阵行数为2，列数为2，可以看成一个新的矩阵，  
继续环绕打印。可见，此题可以用递归解决。  
示例中行数与列数是相等的，所以能够组成完整的环（完整指能够环绕一圈）；  
其实，只要行数和列数中，比较小的那个是偶数，就能够组成完整的环。  
如果行数和列数中比较小的那个是奇数，则递归到终止前的剩余元素无法成环。  
如果较小的是行数，则剩余元素的组成的形状类似于“|”；如果较小的是列数，  
则剩余元素的组成的形状类似于“—”。  

重点：  
因此，当未访问行数和未访问列数都大于等于2时，按照完整环的逻辑递归访问  
即可。当不满足上述条件，判断剩余元素是“|”型还是“—”型，然后按照不完整环  
的逻辑访问即可。  

```java
package chapter4;

/**
 * Created by ryder on 2017/7/16.
 *
 */
public class P161_PrintMatrix {
    public static void printMatrix(int[][] data){
        if(data==null)
            return ;
        if(data.length==0||data[0].length==0)
            return ;
        int rowMax = data.length-1,rowLen = data.length;
        int colMax =data[0].length-1,colLen = data[0].length;
        int row=0,col=0,round=0;
        while(rowLen-2*row>1 && colLen-2*col>1){
            for(;col<=colMax-round;col++){
                System.out.print(data[row][col]);
                System.out.print("\t");
            }
            for(col=col-1,row=row+1;row<rowMax-round;row++){
                System.out.print(data[row][col]);
                System.out.print("\t");
            }
            for(;col>=round;col--){
                System.out.print(data[row][col]);
                System.out.print("\t");
            }
            for(col=col+1,row=row-1;row>round;row--){
                System.out.print(data[row][col]);
                System.out.print("\t");
            }
            row=row+1;
            col=col+1;
            round++;
        }
        //如果行数与列数中较小的那个是偶数，则能组成完整的环，在while中就完成了遍历
        if(rowLen-2*row==0 || colLen-2*col==0){
            System.out.println();
        }
        //如果行数与列数中较小的是行数，且是奇数，则还需补充访问一行
        if(rowLen-2*row==1){
            for(;col<=colMax-round;col++){
                System.out.print(data[row][col]);
                System.out.print("\t");
            }
            System.out.println();
        }
        //如果行数与列数中较小的是列数，且是奇数，则还需补充访问一列
        if(colLen-2*col==1){
            for(;row<=rowMax-round;row++){
                System.out.print(data[row][col]);
                System.out.print("\t");
            }
            System.out.println();
        }

    }
    public static void main(String[] args){
        int[][] data1={
                {1,2,3,4},
                {12,13,14,5},
                {11,16,15,6},
                {10,9,8,7},
        };
        int[][] data2={
                {1,2,3,4},
                {10,11,12,5},
                {9,8,7,6},
        };
        int[][] data3={
                {1,2,3},
                {10,11,4},
                {9,12,5},
                {8,7,6},
        };
        int[][] data4={
                {1,2,3},
                {8,9,4},
                {7,6,5},
        };
        printMatrix(data1);
        printMatrix(data2);
        printMatrix(data3);
        printMatrix(data4);
    }
}
```

30. 包含min函数的栈  

题目要求：  
定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的min函数。  
要求在该栈中，调用min，push及pop的时间复杂度都是o(1)。  

解题思路：  
期望获知当前栈的最小值，最直接的方法是全部弹出求最小值，然后再全部压入。  
这种方式时间消耗较大。另一种思路，可以用空间换时间：自己实现一个栈，栈中  
有记录数据的数据栈，同时也有记录最小值的min栈，本帖就是采用的此方法。记  
得曾经看过一个更巧妙地方法，用一个变量来记录最小值，压入与弹出都会因一定  
的规则修改该变量，思路比较精奇，可遇不可求，有兴趣的可以去搜搜看,比如:  
http://www.cnblogs.com/javawebsoa/archive/2013/05/21/3091727.html  

```java
package chapter4;
import java.util.Stack;
/**
 * Created by ryder on 2017/7/16.
 * 包含min函数的栈
 */
public class P165_StackWithMin {
    public static void main(String[] args){
        StackWithMin<Integer> stack = new StackWithMin<>();
        stack.push(3);
        stack.push(4);
        stack.push(2);
        stack.push(1);
        System.out.println(stack.min());
        stack.pop();
        System.out.println(stack.min());
        stack.pop();
        System.out.println(stack.min());
        stack.pop();
        System.out.println(stack.min());
        stack.pop();
        System.out.println(stack.min());
    }
}
class StackWithMin<T extends Comparable>{
    private Stack<T> stackData = new Stack<>();
    private Stack<T> stackMin = new Stack<>();
    public void push(T data){
        stackData.push(data);
        if(stackMin.isEmpty())
            stackMin.push(data);
        else{
            T temp = stackMin.peek();
            if(temp.compareTo(data)<0)
                stackMin.push(temp);
            else
                stackMin.push(data);
        }
    }
    public T pop(){
        if(stackData.isEmpty())
            return null;
        stackMin.pop();
        return stackData.pop();
    }
    public T min(){
        if(stackMin.isEmpty())
            return null;
        return stackMin.peek();
    }
}
```


31. 栈的压入弹出序列  

题目要求：  
输入两个整数序列，第一个序列表示栈的压入顺序，判断第二个序列是否为该栈的弹出序序列。  
假设压入栈的所有数字均不相等。例如，压入序列为(1,2,3,4,5)，序列(4,5,3,2,1)是它的  
弹出序列，而(4,3,5,1,2)不是。  

解题思路：  
对于一个给定的压入序列，由于弹出的时机不同，会出现多种弹出序列。如果是选择题，依照后  
进先出的原则，复现一下栈的压入弹出过程就很容易判断了。写成程序同样如此，主要步骤如下：  

步骤1：栈压入序列第一个元素，弹出序列指针指弹出序列的第一个；  
步骤2：判断栈顶元素是否等于弹出序列的第一个元素：  
步骤2.1：如果不是，压入另一个元素，进行结束判断，未结束则继续执行步骤2；  
步骤2.2：如果是，栈弹出一个元素，弹出序列指针向后移动一位，进行结束判断，  
未结束则继续执行步骤2；  

结束条件：如果弹出序列指针还没到结尾但已经无元素可压入，则被测序列不是弹出序列。  
         如果弹出序列指针以判断完最后一个元素，则被测序列是弹出序列。  
当然，进行判断前需要进行一些检查，比如传入参数是否为null，压入序列与弹出序列长度是否一致。  

```java
package chapter4;
import java.util.Stack;
/**
 * Created by ryder on 2017/7/17.
 * 栈的压入弹出序列
 */
public class P168_StackPushPopOrder {
    public static boolean isPopOrder(int[] pushSeq,int[] popSeq){
        if(pushSeq==null||popSeq==null||pushSeq.length!=popSeq.length)
            return false;
        Stack<Integer> stack = new Stack<>();
        int pushSeqIndex=0,popSeqIndex=0;
        while (popSeqIndex<popSeq.length){
            if(stack.isEmpty()||stack.peek()!=popSeq[popSeqIndex]) {
                if(pushSeqIndex<pushSeq.length)
                    stack.push(pushSeq[pushSeqIndex++]);
                else
                    return false;
            }
            else{
                stack.pop();
                popSeqIndex++;
            }
        }
        return true;
    }
    public static void main(String[] args){
        int[] push = {1,2,3,4,5};
        int[] pop1 = {4,5,3,2,1};
        int[] pop2 = {4,3,5,1,2};
        System.out.println(isPopOrder(push,pop1));
        System.out.println(isPopOrder(push,pop2));
    }
}
```

32. 32-1 从上到下打印二叉树  

```java

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
```


32. 32-2 分行从上到下打印二叉树  

题目要求：  
从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印 ，每一层打印一行。  

解题思路：  
同样是层序遍历，与上一题不同的是，此处要记录每层的节点个数，在每层打印结束后多打印一个回车符。  

```java
package structure;
import java.util.LinkedList;
import java.util.Queue;
/**
 * Created by ryder on 2017/6/12.
 * 树节点
 */
public class TreeNode<T> {
    public T val;
    public TreeNode<T> left;
    public TreeNode<T> right;
    public TreeNode(T val){
        this.val = val;
        this.left = null;
        this.right = null;
    }
}
package chapter4;
import structure.TreeNode;
import java.util.LinkedList;
import java.util.Queue;
/**
 * Created by ryder on 2017/7/18.
 * 分行从上到下打印二叉树
 */
public class P174_printTreeInLine {
    public static void printTreeInLine(TreeNode<Integer> root){
        if(root==null)
            return;
        Queue<TreeNode<Integer>> queue = new LinkedList<>();
        queue.offer(root);
        TreeNode<Integer> temp;
        while (!queue.isEmpty()){
            for(int size=queue.size();size>0;size--){
                temp = queue.poll();
                System.out.print(temp.val);
                System.out.print("\t");
                if(temp.left!=null)
                    queue.offer(temp.left);
                if(temp.right!=null)
                    queue.offer(temp.right);
            }
            System.out.println();
        }
    }
    public static void main(String[] args){
        //            1
        //          /   \
        //         2     3
        //       /  \   / \
        //      4    5 6   7
        TreeNode<Integer> root = new TreeNode<Integer>(1);
        root.left = new TreeNode<Integer>(2);
        root.right = new TreeNode<Integer>(3);
        root.left.left = new TreeNode<Integer>(4);
        root.left.right = new TreeNode<Integer>(5);
        root.right.left = new TreeNode<Integer>(6);
        root.right.right = new TreeNode<Integer>(7);
        printTreeInLine(root);
    }
}
```


32. 32-3 之字形打印二叉树  

题目要求：  
请实现一个函数按照之字形打印二叉树。即第一层从左到右打印，第二层从右到左打印，  
第三层继续从左到右，以此类推。  

解题思路：  
第k行从左到右打印，第k+1行从右到左打印，可以比较容易想到用两个栈来实现。  
另外要注意，根据是从左到右还是从右到左访问的不同，压入左右子节点的顺序也有所不同。  

```java
package structure;
import java.util.LinkedList;
import java.util.Queue;
/**
 * Created by ryder on 2017/6/12.
 * 树节点
 */
public class TreeNode<T> {
    public T val;
    public TreeNode<T> left;
    public TreeNode<T> right;
    public TreeNode(T val){
        this.val = val;
        this.left = null;
        this.right = null;
    }
}
package chapter4;
import structure.TreeNode;
import java.util.Stack;
/**
 * Created by ryder on 2017/7/18.
 * 之字形打印二叉树
 */
public class P176_printTreeInSpecial {
    public static void printTreeInSpeical(TreeNode<Integer> root){
        if(root==null)
            return;
        Stack<TreeNode<Integer>> stack1 = new Stack<>();
        Stack<TreeNode<Integer>> stack2 = new Stack<>();
        TreeNode<Integer> temp;
        stack1.push(root);
        while(!stack1.isEmpty() || !stack2.isEmpty()){
            if(!stack1.isEmpty()) {
                while (!stack1.isEmpty()) {
                    temp = stack1.pop();
                    System.out.print(temp.val);
                    System.out.print('\t');
                    if (temp.left != null)
                        stack2.push(temp.left);
                    if (temp.right != null)
                        stack2.push(temp.right);
                }
            }
            else {
                while (!stack2.isEmpty()) {
                    temp = stack2.pop();
                    System.out.print(temp.val);
                    System.out.print('\t');
                    if (temp.right != null)
                        stack1.push(temp.right);
                    if (temp.left != null)
                        stack1.push(temp.left);
                }
            }
            System.out.println();
        }
    }
    public static void main(String[] args){
        //            1
        //          /   \
        //         2     3
        //       /  \   / \
        //      4    5 6   7
        TreeNode<Integer> root = new TreeNode<Integer>(1);
        root.left = new TreeNode<Integer>(2);
        root.right = new TreeNode<Integer>(3);
        root.left.left = new TreeNode<Integer>(4);
        root.left.right = new TreeNode<Integer>(5);
        root.right.left = new TreeNode<Integer>(6);
        root.right.right = new TreeNode<Integer>(7);
        printTreeInSpeical(root);
    }
}
```

33. 二叉搜索树的后序遍历  

题目要求：  
输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历结果，假设输入数组的任意两个数都互不相同。  

解题思路：  
二叉搜索树的中序遍历是有序的，而此题是后序遍历。  
后序遍历可以很容易找到根节点，然后根据二叉搜索树的性质（左子树小于根节点，右子树大于根节点），  
可以将序列分为根节点的左子树部分与右子树部分，而后递归判断两个子树。  

```java
package chapter4;
/**
 * Created by ryder on 2017/7/18.
 * 二叉搜索树的后序遍历序列
 */
public class P179_SequenceOfBST {   
    public static boolean verifySquenceOfBST(int[] data){
        //空树
        if(data==null||data.length==0)
            return false;
        return verifySquenceOfBST(data,0,data.length-1);
    }
    public static boolean verifySquenceOfBST(int[] data,int start,int end){
        //数组长度为2，一定能够组成BST
        if(end-start<=1)
            return true;
        int root = data[end];
        int rightStart =0;
        for(int i=0;i<end;i++){
            if(data[i]>root){
                rightStart = i;
                break;
            }
        }
        for(int i=rightStart;i<end;i++){
            if(data[i]<root)
                return false;
        }
        return verifySquenceOfBST(data,start,rightStart-1)&&verifySquenceOfBST(data,rightStart,end-1);
    }
    public static void main(String[] args){
        //            8
        //          /   \
        //         6     10
        //       /  \   / \
        //      5    7 9   11
        int[] data = {5,7,6,9,4,10,8};
        int[] data1 = {5,7,6,9,11,10,8};
        System.out.println(verifySquenceOfBST(data));
        System.out.println(verifySquenceOfBST(data1));
    }
}
```

34. 二叉树中和为某一值的路径  

题目要求：  
输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。  
从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。  

解题思路：  
需要得到所有从根节点到叶节点的路径，判断和是否为给定值。自己写一个小的二叉树，  
通过模拟上述过程，发现获取所有路径的过程与前序遍历类似。  
因此，可以对给定二叉树进行前序遍历。当被访问节点时叶节点时，判断路径和是否为  
给定值。此外，为了记录路径上的节点，需要一个数组。  

```java
package chapter4;
import structure.TreeNode;
import java.util.ArrayList;
import java.util.List;

/**
 * Created by ryder on 2017/7/18.
 * 二叉树中和为某一值的路径
 */
public class P182_FindPath {
    //用类似于前序遍历的思路解决
    public static void findPath(TreeNode<Integer> root,int exceptedSum){
        if(root==null)
            return;
        List<Integer> path = new ArrayList<>();
        findPath(root,path,exceptedSum,0);
    }
    //curNode为将要被访问的节点，还未被加入到path中
    public static void findPath(TreeNode<Integer> curNode,List<Integer> path,int exceptedSum,int currentSum){
        path.add(curNode.val);
        currentSum+=curNode.val;
        if(curNode.left!=null)
            findPath(curNode.left,path,exceptedSum,currentSum);
        if(curNode.right!=null)
            findPath(curNode.right,path,exceptedSum,currentSum);
        if(curNode.left==null && curNode.right==null && currentSum==exceptedSum)
            System.out.println(path);
        path.remove(path.size()-1) ;//从右子树返回当前结点
    }
   
    public static void main(String[] args) {
        //            10
        //          /   \
        //         5     12
        //       /  \
        //      4    7
        TreeNode<Integer> root = new TreeNode<Integer>(10);
        root.left = new TreeNode<Integer>(5);
        root.right = new TreeNode<Integer>(12);
        root.left.left = new TreeNode<Integer>(4);
        root.left.right = new TreeNode<Integer>(7);
        findPath(root,22);
        findPath2(root,22);
    }
}

如果二叉树的所有节点值都是大于0的（原题中并没有这个条件），可以进行剪枝。

//如果所有节点值均大于0，可进行剪枝
    public static void findPath2(TreeNode<Integer> root,int exceptedSum){
        if(root==null)
            return;
        List<Integer> path = new ArrayList<>();
        findPath2(root,path,exceptedSum,0);
    }
     //curNode为将要被访问的节点，还未被加入到path中
    public static void findPath2(TreeNode<Integer> curNode,List<Integer> path,int exceptedSum,int currentSum){
        path.add(curNode.val);
        currentSum+=curNode.val;
        //只有当currentSum小于exceptedSum时需要继续当前节点的子节点的遍历
        if(currentSum<exceptedSum){
            if(curNode.left!=null)
                findPath2(curNode.left,path,exceptedSum,currentSum);
            if(curNode.right!=null)
                findPath2(curNode.right,path,exceptedSum,currentSum);
        }
        //currentSum大于等于exceptedSum时可以直接停止当前分支的遍历，因为当前分支下currentSum只会越来越大，不会再有符合要求的解
        else if(currentSum==exceptedSum && curNode.left==null && curNode.right==null)
            System.out.println(path);
        path.remove(path.size()-1) ;
    }
```

35. 复杂链表的复制  
题目要求：在复杂链表中，每个节点除了有一个next指针指向下一个节点，还有一个random  
指针指向链表中的任意节点或null，请完成一个能够复制复杂链表的函数。  

解题思路：  
此题定义了一种新的数据结构，复杂链表。与传统链表的区别是多了一个random指针。本题的  
关键点也就在如何高效地完成random指针的复制。  

解法    时间复杂度   空间复杂度
解法一   o(n^2)      o(1)
解法二   o(n)        o(n)
解法三   o(n)        o(1)

```java

/*解法一比较直接：
先把这个复杂链表当做传统链表对待，只复制val域与next域（时间o(n)），再从新链表的头部开始，
对random域赋值（时间o(n^2)）。*/

package chapter4;
import java.util.HashMap;

/**
 * Created by ryder on 2017/7/18.
 * 复制复杂链表
 */
public class P187_CopyComplexList {
    public static class ComplexListNode{
        int val;
        ComplexListNode next;
        ComplexListNode random;

        public ComplexListNode(int val) {
            this.val = val;
        }
        @Override
        public String toString() {
            StringBuilder ret = new StringBuilder();
            ComplexListNode cur = this;
            while(cur!=null) {
                ret.append(cur.val);
                if(cur.random!=null)
                    ret.append("("+cur.random.val+")");
                else{
                    ret.append("(_)");
                }
                ret.append('\t');
                cur = cur.next;
            }
            return ret.toString();
        }
    }
    //解法一
    //time:o(n^2) space:o(1) 新链表使用的n个长度的空间不算入
    //先复制val与next（时间o(n)），再复制random域（时间o(n^2)）
    public static ComplexListNode clone1(ComplexListNode head){
        if(head==null)
            return null;
        ComplexListNode newHead = new ComplexListNode(head.val);
        ComplexListNode cur = head.next;
        ComplexListNode newCur = null;
        ComplexListNode newCurPrev = newHead;
        while (cur!=null){
            newCur = new ComplexListNode(cur.val);
            newCurPrev.next = newCur;
            newCurPrev = newCurPrev.next;
            cur = cur.next;
        }
        cur = head;
        newCur = newHead;
        ComplexListNode temp = head;
        ComplexListNode newTemp = newHead;
        while(cur!=null){
            if(cur.random!=null){
                temp = head;
                newTemp = newHead;
                //重点在这，temp和newTemp一直往后面找，找到cur的random指向结点
                while (temp!=cur.random){
                    temp = temp.next;
                    newTemp = newTemp.next;
                }
                newCur.random = newTemp;
            }
            cur = cur.next;
            newCur = newCur.next;
        }
        return newHead;
    }
    public static void main(String[] args){
        ComplexListNode head = new ComplexListNode(1);
        ComplexListNode c2 = new ComplexListNode(2);
        ComplexListNode c3 = new ComplexListNode(3);
        ComplexListNode c4 = new ComplexListNode(4);
        ComplexListNode c5 = new ComplexListNode(5);
        head.next = c2;
        head.random = c3;
        head.next.next = c3;
        head.next.random = c5;
        head.next.next.next = c4;
        head.next.next.next.next = c5;
        head.next.next.next.random = c2;
        System.out.println("original:"+'\t'+head);
        System.out.println("clone1:  "+'\t'+clone1(head));
        System.out.println("clone2:  "+'\t'+clone2(head));
        System.out.println("clone3:  "+'\t'+clone3(head));
    }
}

/*解法二是用空间换时间:
解法一时间复杂度高的原因在于确定random域的费时，即假设原链表第m个节点指向第k个节点，
而在新链表的第m个节点处无法直接得到第k个节点，需从头遍历。很自然的想法是用一个哈希表
记录旧链表每个节点到新链表对应节点的映射，从而可以将时间复杂度降低为o(n)。
*/
    //解法二
    //time:o(n) space:o(n)
    //使用o(n)的空间，换取了时间复杂度的降低
    public static ComplexListNode clone2(ComplexListNode head) {
        if(head==null)
            return null;
        HashMap<ComplexListNode,ComplexListNode> oldToNew = new HashMap<>();
        ComplexListNode newHead = new ComplexListNode(head.val);
        oldToNew.put(head,newHead);
        ComplexListNode cur = head.next;
        ComplexListNode newCur = null;
        ComplexListNode newCurPrev = newHead;
        while (cur!=null){
            newCur = new ComplexListNode(cur.val);
            oldToNew.put(cur,newCur);
            newCurPrev.next = newCur;
            newCurPrev = newCurPrev.next;
            cur = cur.next;
        }
        cur = head;
        newCur = newHead;
        while(cur!=null){
            if(cur.random!=null){
                newCur.random = oldToNew.get(cur.random);
            }
            cur = cur.next;
            newCur = newCur.next;
        }
        return newHead;
    }

/*解法三：
思路很巧妙。将复制的任务分为如下三个部分：
1）cloneNodes完成新链表节点的创建，仅对val域赋值，且每个新节点接在原链表对应节点的后面。
如A->B->C,处理完后为A->A'->B->B'->C->C'，时间复杂度o(n)。
2）connectRandomNode完成random域的赋值。假设A.random=C,我们需要设置A'.random=C'，此处
获取C'可以在o(1)的时间复杂度完成，全部赋值完毕时间复杂度为o(n)。
3）reconnectNodes就是将上述链表重组，使A->A'->B->B'->C->C'变为A->B->C，A'->B'->C'。
此处需要注意尾部null的处理。*/

    //解法三
    //time:o(n) space:o(1)
    public static ComplexListNode clone3(ComplexListNode head) {
        if(head==null)
            return null;
        cloneNodes(head);
        connectRandomNodes(head);
        return reconnectNodes(head);
    }
    public static void cloneNodes(ComplexListNode head){
        ComplexListNode cur = head;
        ComplexListNode temp = null;
        while (cur!=null){
            temp = new ComplexListNode(cur.val);
            temp.next = cur.next;
            cur.next = temp;
            cur = cur.next.next;
        }
    }
    public static void connectRandomNodes(ComplexListNode head){
        ComplexListNode cur = head;
        ComplexListNode curNext = head.next;
        while (true){
            if(cur.random!=null)
                curNext.random = cur.random.next;
            cur = cur.next.next;
            if(cur == null)
                break;
            curNext = curNext.next.next;
        }
    }
    public static ComplexListNode reconnectNodes(ComplexListNode head){
        ComplexListNode newHead = head.next;
        ComplexListNode cur = head;
        ComplexListNode newCur = newHead;
        while (true){
            cur.next = cur.next.next;
            cur = cur.next;
            if(cur==null){
                newCur.next = null;
                break;
            }
            newCur.next = newCur.next.next;
            newCur = newCur.next;
        }
        return newHead;
    }
```

