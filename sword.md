

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
 * 二叉树的遍历：前序（递归，非递归），中序（递归，非递归），后序（递归，非递归），层序
 */
public class P60_TraversalOfBinaryTree {

	//前序遍历递归自己写
	public static void preorderRecursively(TreeNode<Integer> node) {
        System.out.println(node.val);
        if (node.left != null)
            preorderRecursively(node.left);
        if (node.right != null)
            preorderRecursively(node.right);
    }
    
    //前序遍历递归版
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

    //前序遍历非递归版
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
            } else {
                cur = stack.peek().right;
                if (cur != null && cur != prevVisted) {
                    stack.push(cur);
                    cur = cur.left;
                } else {
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