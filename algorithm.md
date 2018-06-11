
# 链表  

1. 反转链表  

next记录下一个结点，new_head记录上一个结点  


```cpp

#include <stdio.h>

struct ListNode {
	int val;
	ListNode *next;
	ListNode(int x) : val(x), next(NULL) {}
};

class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode *new_head = NULL;
		while(head){
			ListNode *next = head->next;	//记录下一个结点
			head->next = new_head;			//指向上一个结点
			new_head = head;				//移动new->head
			head = next;					//遍历链表
		}
		return new_head;
    }
};

int main(){	
	ListNode a(1);
	ListNode b(2);
	ListNode c(3);
	ListNode d(4);
	ListNode e(5);
	a.next = &b;
	b.next = &c;
	c.next = &d;
	d.next = &e;
	Solution solve;	
	ListNode *head = &a;
	printf("Before reverse:\n");
	while(head){
		printf("%d\n", head->val);
		head = head->next;
	}
	head = solve.reverseList(&a);
	printf("After reverse:\n");
	while(head){
		printf("%d\n", head->val);
		head = head->next;
	}
	return 0;
}


 
```

2. 部分反转链表  

```cpp
#include <stdio.h>

struct ListNode {
	int val;
	ListNode *next;
	ListNode(int x) : val(x), next(NULL) {}
};

class Solution {
public:
    ListNode* reverseBetween(ListNode* head, int m, int n) {
        int change_len = n - m + 1;  //需要逆置的位置
        ListNode *pre_head = NULL;   //开始逆置的前驱
        ListNode *result = head;     //转换后的链表头
        while(head && --m){          //向前移动m-1个位置
        	pre_head = head;  
        	head = head->next;
        }
        ListNode *modify_list_tail = head;  //逆置后的链表尾部 
        
        ListNode *new_head = NULL;
		while(head && change_len){  //逆置change_len个节点
			ListNode *next = head->next;
			head->next = new_head;
			new_head = head;
			head = next;
			change_len--;
		}
		//head指向的就是后面序列的第一个结点，new_head是逆置后的头部
		modify_list_tail->next = head;  //逆置完成连接后面的序列
		
		if (pre_head){  //不是从第一个开始逆置
			pre_head->next = new_head;  //正常的连接
		}
		else{          //pre_head为空的情况
			result = new_head;
		}
		return result;
    }
};

int main(){	
	ListNode a(1);
	ListNode b(2);
	ListNode c(3);
	ListNode d(4);
	ListNode e(5);
	a.next = &b;
	b.next = &c;
	c.next = &d;
	d.next = &e;
	Solution solve;
	ListNode *head = solve.reverseBetween(&a, 2, 4);	
	while(head){
		printf("%d\n", head->val);
		head = head->next;
	}
	return 0;
}

```

3. 求链表交点  

```cpp

#include <stdio.h>

struct ListNode {
	int val;
	ListNode *next;
	ListNode(int x) : val(x), next(NULL) {}
};

#include <set>

class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
   		std::set<ListNode*> node_set;  //set里面放指针
		while(headA){
			node_set.insert(headA);  //遍历a把指针都放进去
			headA = headA->next;
        }
        while(headB){
        	if (node_set.find(headB) != node_set.end()){   //如果set查找到
	        	return headB;
	        }
	        headB = headB->next;
        }
        return NULL;
    }
};

int main(){
	ListNode a1(1);
	ListNode a2(2);
	ListNode b1(3);
	ListNode b2(4);
	ListNode b3(5);
	ListNode c1(6);
	ListNode c2(7);
	ListNode c3(8);
	a1.next = &a2;
	a2.next = &c1;
	c1.next = &c2;
	c2.next = &c3;
	b1.next = &b2;
	b2.next = &b3;
	b3.next = &c1;
	
	Solution solve;
	ListNode *result = solve.getIntersectionNode(&a1, &b1);
	printf("%d\n", result->val);
	return 0;
}

```

4. 求交点解法二  

```cpp

#include <stdio.h>

struct ListNode {
	int val;
	ListNode *next;
	ListNode(int x) : val(x), next(NULL) {}
};

int get_list_length(ListNode *head){
	int len = 0;
	while(head){
		len++;
		head = head->next;
	}
	return len;
}

ListNode *forward_long_list(int long_len, 
				int short_len, ListNode *head){
	int delta = long_len - short_len;
	while(head && delta){
		head = head->next;
		delta--;
	}
	return head;
}

class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) { 
   		int list_A_len = get_list_length(headA);
   		int list_B_len = get_list_length(headB);   		
   		if (list_A_len > list_B_len){  //求出哪个长，然后把指针对齐
   			headA = forward_long_list(list_A_len, list_B_len, headA);  
	   	}
	   	else{
	   		headB = forward_long_list(list_B_len, list_A_len, headB);
	   	}   		
        while(headA && headB){  //遍历只要相等就是交点
        	if (headA == headB){
	        	return headA;
	        }
	        headA = headA->next;
	        headB = headB->next;
        }
        return NULL;
    }
};

int main(){
	ListNode a1(1);
	ListNode a2(2);
	ListNode b1(3);
	ListNode b2(4);
	ListNode b3(5);
	ListNode c1(6);
	ListNode c2(7);
	ListNode c3(8);
	a1.next = &a2;
	a2.next = &c1;
	c1.next = &c2;
	c2.next = &c3;
	b1.next = &b2;
	b2.next = &b3;
	b3.next = &c1;	
	Solution solve;
	ListNode *result = solve.getIntersectionNode(&a1, &b1);
	printf("%d\n", result->val);
	return 0;
}

```

5. 链表求环起始结点

```cpp


#include <stdio.h>


struct ListNode {
	int val;
	ListNode *next;
	ListNode(int x) : val(x), next(NULL) {}
};

#include <set>
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        std::set<ListNode *> node_set;
        while(head){   //遍历head往set里面插入，如果找到就返回
        	if (node_set.find(head) != node_set.end()){ 
	        	return head;
	        }
	        node_set.insert(head);
        	head = head->next;
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
	a.next = &b;
	b.next = &c;
	c.next = &d;
	d.next = &e;
	e.next = &f;
	//f.next = &c;
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

6. 求环第二种解法，快慢指针  

慢速是一，快速是二，相遇点到起点距离除以二就是环的起点

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

7. 链表划分  

```cpp 

#include <stdio.h>
	
struct ListNode {
	int val;
	ListNode *next;
	ListNode(int x) : val(x), next(NULL) {}
};

class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
    	ListNode less_head(0);                 //需要两个头结点和两个头指针
    	ListNode more_head(0);
    	ListNode *less_ptr = &less_head;
    	ListNode *more_ptr = &more_head;
        while(head){                           
        	if (head->val < x){             //比x小，就插入到less_ptr后面
        		less_ptr->next = head;
        		less_ptr = head;
			}
			else {                          //否则插入到more_ptr后
				more_ptr->next = head;
				more_ptr = head;
			}
        	head = head->next;
        }
        less_ptr->next = more_head.next;    //结束时候拼接
        more_ptr->next = NULL;
        return less_head.next;
    }
};

int main(){
	ListNode a(1);
	ListNode b(4);
	ListNode c(3);
	ListNode d(2);
	ListNode e(5);
	ListNode f(2);
	a.next = &b;
	b.next = &c;
	c.next = &d;
	d.next = &e;
	e.next = &f;	
	Solution solve;
	ListNode *head = solve.partition(&a, 3);	
	while(head){
		printf("%d\n", head->val);
		head = head->next;
	}
	return 0;
}


```

8. 复杂链表深度拷贝  

```cpp
	
#include <stdio.h>
		
struct RandomListNode {
	int label;
	RandomListNode *next, *random;
	RandomListNode(int x) : label(x), next(NULL), random(NULL) {}  
};

#include <map>
#include <vector>

class Solution {
public:
    RandomListNode *copyRandomList(RandomListNode *head) {
    	std::map<RandomListNode *, int> node_map;      //一个map,用来存储原链表指针和位置关系
    	std::vector<RandomListNode *> node_vec;		   //一个vector存储复制链表地址（位置隐含的）
    	RandomListNode *ptr = head;
    	int i = 0;
    	while (ptr){											  //遍历第一次
	    	node_vec.push_back(new RandomListNode(ptr->label));   //需要new结点
	    	node_map[ptr] = i;  								  //地址做键，位置做值
	    	ptr = ptr->next;  
	    	i++;
	    }
	    node_vec.push_back(0);  
	    ptr = head;
	    i = 0;
	    while(ptr){								    //再次遍历
    		node_vec[i]->next = node_vec[i+1];      //next域生成
    		if (ptr->random){
    			int id = node_map[ptr->random];     //取出random对应位置
		    	node_vec[i]->random = node_vec[id]; //生成复制链表random域
		    }
    		ptr = ptr->next;
    		i++;
    	}
    	return node_vec[0];
    }
};

int main(){
	RandomListNode a(1);
	RandomListNode b(2);
	RandomListNode c(3);
	RandomListNode d(4);
	RandomListNode e(5);
	a.next = &b;
	b.next = &c;
	c.next = &d;
	d.next = &e;	
	a.random = &c;
	b.random = &d;
	c.random = &c;
	e.random = &d;	
	Solution solve;
	RandomListNode *head = solve.copyRandomList(&a);	
	while(head){
		printf("label = %d ", head->label);
		if (head->random){
			printf("rand = %d\n", head->random->label);
		}
		else{
			printf("rand = NULL\n");
		}
		head = head->next;
	}
	return 0;
}

```

9. 多个排序链表的合并  

排序后相连方法 kn*log(kn)，就是排序kn个结点


```cpp
#include <stdio.h>

struct ListNode {
 	int val;
	ListNode *next;
	ListNode(int x) : val(x), next(NULL) {}
};

#include <vector>
#include <algorithm>

bool cmp(const ListNode *a, const ListNode *b){
	return a->val < b->val;
}

class Solution {
public:
    ListNode* mergeKLists(std::vector<ListNode*> &lists) {
        std::vector<ListNode *> node_vec;        
        for (int i = 0; i < lists.size(); i++){
        	ListNode *head = lists[i];          //取出每个链表头指针
        	while(head){						//把每个链表的指针放入数组
        		node_vec.push_back(head);
	        	head = head->next;
	        }
        }
        if (node_vec.size() == 0){
        	return NULL;
        }        
        std::sort(node_vec.begin(), node_vec.end(), cmp);   //数组进行排序
        for (int i = 1; i < node_vec.size(); i++){          //重新链接
        	node_vec[i-1]->next = node_vec[i];
        }
        node_vec[node_vec.size()-1]->next = NULL;
        return node_vec[0];
    }
};

int main(){
	ListNode a(1);
	ListNode b(4);
	ListNode c(6);
	ListNode d(0);
	ListNode e(5);
	ListNode f(7);
	ListNode g(2);
	ListNode h(3);
	a.next = &b;
	b.next = &c;	
	d.next = &e;
	e.next = &f;	
	g.next = &h;	
	Solution solve;	
	std::vector<ListNode *> lists;
	lists.push_back(&a);                       //把三个链表的头指针都放进list
	lists.push_back(&d);
	lists.push_back(&g);	
	ListNode *head = solve.mergeKLists(lists);
	while(head){
		printf("%d\n", head->val);
		head = head->next;
	}
	return 0;
}
```

10. 多个排序链表的合并，归并解决  

kn*log(k)

```cpp 
#include <stdio.h>

struct ListNode {
 	int val;
	ListNode *next;
	ListNode(int x) : val(x), next(NULL) {}
};

#include <vector>

class Solution {
public:
	ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
    	ListNode temp_head(0);          //定义临时头结点和头指针  
    	ListNode *pre = &temp_head;
    	while (l1 && l2){               //都不为空的情况，哪个小哪个就插入
	    	if (l1->val < l2->val){
	    		pre->next = l1;
	    		l1 = l1->next;
	    	}
	    	else{
	    		pre->next = l2;
	    		l2 = l2->next;
	    	}
	    	pre = pre->next;
	    }
	    if (l1){                        //其中一个为空就把剩下的插入
    		pre->next = l1;
    	}
    	if (l2){
	    	pre->next = l2;
	    }
        return temp_head.next;
    }	
    ListNode* mergeKLists(std::vector<ListNode*>& lists) {
    	if (lists.size() == 0){                     //考虑传入链表个数分别为0，1，2
        	return NULL;
        }
    	if (lists.size() == 1){
	    	return lists[0];
	    }
	    if (lists.size() == 2){
    		return mergeTwoLists(lists[0], lists[1]);
    	}
    	int mid = lists.size() / 2;                //链表分成两部分，进行分治
    	std::vector<ListNode*> sub1_lists;
    	std::vector<ListNode*> sub2_lists;
    	for (int i = 0; i < mid; i++){
	    	sub1_lists.push_back(lists[i]);
	    }
	    for (int i = mid; i < lists.size(); i++){
    		sub2_lists.push_back(lists[i]);
    	}
    	ListNode *l1 = mergeKLists(sub1_lists);   //递归调用
    	ListNode *l2 = mergeKLists(sub2_lists);

    	return mergeTwoLists(l1, l2);             //递归结束了返回了l1和l2再把他们合并就行了
    }
};

int main(){
	ListNode a(1);
	ListNode b(4);
	ListNode c(6);
	ListNode d(0);
	ListNode e(5);
	ListNode f(7);
	ListNode g(2);
	ListNode h(3);
	a.next = &b;
	b.next = &c;	
	d.next = &e;
	e.next = &f;	
	g.next = &h;
	Solution solve;	
	std::vector<ListNode *> lists;
	lists.push_back(&a);
	lists.push_back(&d);
	lists.push_back(&g);	
	ListNode *head = solve.mergeKLists(lists);
	while(head){
		printf("%d\n", head->val);
		head = head->next;
	}
	return 0;
}

```


# 栈、队列、堆  

1. 队列实现栈  

```cpp 

#include <stdio.h>

#include <queue>
class MyStack {
public:
    MyStack() {        
    }
    void push(int x) {
    	std::queue<int> temp_queue;         //需要data和temp两个队列
    	temp_queue.push(x);
    	while(!_data.empty()){              //先把x放到temp，然后所有的都放到temp
	    	temp_queue.push(_data.front()); 
	    	_data.pop();
	    }
	    while(!temp_queue.empty()){         //把temp放回到data
   			_data.push(temp_queue.front());
   			temp_queue.pop();
    	}
    }
    int pop() {
        int x = _data.front();
    	_data.pop();
    	return x;
    }
    int top() {
        return _data.front();
    }
    bool empty() {
        return _data.empty();
    }
private:
	std::queue<int> _data;
};

int main(){
	MyStack S;
	S.push(1);
	S.push(2);
	S.push(3);
	S.push(4);
	printf("%d\n", S.top());
	S.pop();
	printf("%d\n", S.top());
	S.push(5);
	printf("%d\n", S.top());	
	return 0;
}

```

2. 栈实现队列

还有一种双栈法


```cpp

#include <stdio.h>

#include <stack>	
class MyQueue {
public:
    MyQueue() {
    }
    void push(int x) {                 //需要data和temp两个栈
        std::stack<int> temp_stack;
        while(!_data.empty()){         //先把data放到temp变成反向，然后把x也入栈
        	temp_stack.push(_data.top());
        	_data.pop();
        }
        temp_stack.push(x);            //把temp放回到data，都变成正向
        while(!temp_stack.empty()){
        	_data.push(temp_stack.top());
        	temp_stack.pop();
        }
    }
    int pop() {
    	int x = _data.top();
    	_data.pop();
        return x;
    }
    int peek() {
        return _data.top();
    }
    bool empty() {
        return _data.empty();
    }
private:
	std::stack<int> _data;
};

int main(){
	MyQueue Q;
	Q.push(1);
	Q.push(2);
	Q.push(3);
	Q.push(4);
	printf("%d\n", Q.peek());
	Q.pop();
	printf("%d\n", Q.peek());	
	return 0;
}

```

3. 能返回最小值的栈  

```cpp
#include <stdio.h>
#include <stack>

class MinStack {
public:
    MinStack() {
    }
    void push(int x) {
    	_data.push(x);
    	if (_min.empty()){
	    	_min.push(x);
	    }
	    else{                       //入栈时候比较最小值栈栈顶元素和输入元素
	    	if (x > _min.top()){    //如果输入元素大于栈顶则还是栈顶入栈
	    		x = _min.top();
	    	}
    		_min.push(x);			//否则将x进栈
    	}
    }
    void pop() {
    	_data.pop();
    	_min.pop();
    }
    int top() {
        return _data.top();
    }
    int getMin() {
        return _min.top();
    }
private:
	std::stack<int> _data;
	std::stack<int> _min;           //引入一个最小值栈
};

int main(){
	MinStack minStack;
	minStack.push(-2);
	printf("top = [%d]\n", minStack.top());
	printf("min = [%d]\n\n", minStack.getMin());	
	minStack.push(0);
	printf("top = [%d]\n", minStack.top());
	printf("min = [%d]\n\n", minStack.getMin());
	minStack.push(-5);
	printf("top = [%d]\n", minStack.top());
	printf("min = [%d]\n\n", minStack.getMin());
	minStack.pop();
	printf("top = [%d]\n", minStack.top());
	printf("min = [%d]\n\n", minStack.getMin());	
	return 0;
}


```

4. 合法的出栈序列  

```cpp
#include <stdio.h>

#include <stack>
#include <queue>

bool check_is_valid_order(std::queue<int> &order){
	std::stack<int> S;
	int n = order.size();
	for (int i = 1; i <= n; i++){
		S.push(i);                                       //把i分别入栈
		while(!S.empty() && order.front() == S.top()){   //当遇到队列头和栈顶相等，分别pop
			S.pop();
			order.pop();
		}
	}
	if (!S.empty()){
		return false;
	}
	return true;
}

int main(){
	int n;
	int train;
	scanf("%d", &n);
	while(n){
		scanf("%d", &train);
		while (train){
			std::queue<int> order;
			order.push(train);
			for (int i = 1; i < n; i++){
				scanf("%d", &train);
				order.push(train);
			}
			if (check_is_valid_order(order)){
				printf("Yes\n");
			}
			else{
				printf("No\n");
			}
			scanf("%d", &train);
		}
		printf("\n");
		scanf("%d", &n);
	}
	return 0;
}

```

5. 计算器实现

主要看思路  

```cpp
#include <stdio.h>

#include <string>
#include <stack>

class Solution {
public:
    int calculate(std::string s) {
    	static const int STATE_BEGIN = 0;
    	static const int NUMBER_STATE = 1;
    	static const int OPERATION_STATE = 2;
        std::stack<int> number_stack;
        std::stack<char> operation_stack;
        int number = 0;
        int STATE = STATE_BEGIN;
        int compuate_flag = 0;
        for (int i = 0; i < s.length(); i++){
        	if (s[i] == ' '){
	        	continue;
	        }
	        switch(STATE){
        	case STATE_BEGIN:
        		if (s[i] >= '0' && s[i] <= '9'){
        			STATE = NUMBER_STATE;
				}
				else{
					STATE = OPERATION_STATE;
				}
				i--;
				break;
       		case NUMBER_STATE:
			  	if (s[i] >= '0' && s[i] <= '9'){
	  				number = number * 10 + s[i] - '0';
	    		}
	    		else{
	    			number_stack.push(number);
	    			if (compuate_flag == 1){
			    		compute(number_stack, operation_stack);
			    	}
	    			number = 0;
	    			i--;
	    			STATE = OPERATION_STATE;
	       		}
      			break;
  			case OPERATION_STATE:
  				if (s[i] == '+' || s[i] == '-'){
  					operation_stack.push(s[i]);
  					compuate_flag = 1;
			  	}
			  	else if (s[i] == '('){
	  				STATE = NUMBER_STATE;
	  				compuate_flag = 0;
	  			}
	  			else if (s[i] >= '0' && s[i] <= '9'){
			  		STATE = NUMBER_STATE;			  		
			  		i--;
			  	}
			  	else if (s[i] == ')'){
			  		compute(number_stack, operation_stack);
	  			}
  				break;
        	}
        }
        if (number != 0){
        	number_stack.push(number);
       		compute(number_stack, operation_stack);
        }
        if (number == 0 && number_stack.empty()){
        	return 0;
        }
        return number_stack.top();
    }
private:
	void compute(std::stack<int> &number_stack,
				 std::stack<char> &operation_stack){
		if (number_stack.size() < 2){
			return;
		}
		int num2 = number_stack.top();
		number_stack.pop();
		int num1 = number_stack.top();
		number_stack.pop();
		if (operation_stack.top() == '+'){
			number_stack.push(num1 + num2);
		}
		else if(operation_stack.top() == '-'){
			number_stack.push(num1 - num2);
		}
		operation_stack.pop();
	}
};

int main(){	
	std::string s = "1+121 - (14+(5-6) )";
	Solution solve;
	printf("%d\n", solve.calculate(s));
	return 0;
}

```

6. 最大值堆，最小值堆构建

```cpp

#include <stdio.h>
#include <queue>
int main(){
	std::priority_queue<int> big_heap;                  //最大值堆
	std::priority_queue<int, std::vector<int>,          //最小值堆
					std::greater<int> > small_heap;
	std::priority_queue<int, std::vector<int>,          //第二种最大值堆
					std::less<int> > big_heap2;
					
	if (big_heap.empty()){
		printf("big_heap is empty!\n");
	}	
	int test[] = {6, 10, 1, 7, 99, 4, 33};
	for (int i = 0; i < 7; i++){
		big_heap.push(test[i]);
	}
	printf("big_heap.top = %d\n", big_heap.top());
	big_heap.push(1000);
	printf("big_heap.top = %d\n", big_heap.top());
	for (int i = 0; i < 3; i++){
		big_heap.pop();
	}
	printf("big_heap.top = %d\n", big_heap.top());
	printf("big_heap.size = %d\n", big_heap.size());
	return 0;
}


```

7. 第k大的元素  

```cpp
#include <stdio.h>

#include <vector>
#include <queue>
class Solution {
public:
    int findKthLargest(std::vector<int>& nums, int k) {
        std::priority_queue<int, std::vector<int>, std::greater<int> > Q;  //最小值堆
        for (int i = 0; i < nums.size(); i++){
        	if (Q.size() < k){              //堆内不够k个，直接进入
	        	Q.push(nums[i]);
	        }
	        else if (Q.top() < nums[i]){    //否则比较，比堆顶大则堆顶出去，元素进堆
        		Q.pop();
	        	Q.push(nums[i]);
	        }
        }
        return Q.top();
    }
};

int main(){
	std::vector<int> nums;
	nums.push_back(3);
	nums.push_back(2);
	nums.push_back(1);
	nums.push_back(5);
	nums.push_back(6);
	nums.push_back(4);
	Solution solve;
	printf("%d\n", solve.findKthLargest(nums, 2));	
	return 0;
}

```

8. 寻找中位数  

一个最大值堆，一个最小值堆  
最大值堆堆顶要小于最小值堆堆顶元素  

```cpp

#include <stdio.h>
#include <queue>

class MedianFinder {
public:
    MedianFinder() {
    }
    void addNum(int num) {
    	if (big_queue.empty()){
	    	big_queue.push(num);
	    	return;
	    }
        if (big_queue.size() == small_queue.size()){      //两堆数量相等
        	if (num < big_queue.top()){
	        	big_queue.push(num);
	        }
	        else{
        		small_queue.push(num);
        	}
        }
        else if(big_queue.size() > small_queue.size()){   //最小值堆数量小一
        	if (num > big_queue.top()){					  //大于最大值堆的顶，直接放到最小值堆
	        	small_queue.push(num);
	        }
	        else{										  //否则先把最大值堆顶放到最小值堆
        		small_queue.push(big_queue.top());        //再把元素放到最大值堆顶
        		big_queue.pop();
        		big_queue.push(num);
        	}
        }
        else if(big_queue.size() < small_queue.size()){
        	if (num < small_queue.top()){
	        	big_queue.push(num);
	        }
	        else{
        		big_queue.push(small_queue.top());
        		small_queue.pop();
        		small_queue.push(num);
        	}
        }
    }
    double findMedian(){
    	if (big_queue.size() == small_queue.size()){
        	return (big_queue.top() + small_queue.top()) / 2;
        }
        else if (big_queue.size() > small_queue.size()){
        	return big_queue.top();
        }
        return small_queue.top();
    }
private:
	std::priority_queue<double> big_queue;
	std::priority_queue<double, std::vector<double>,
					std::greater<double> > small_queue;
};

int main(){
	MedianFinder M;
	int test[] = {6, 10, 1, 7, 99, 4, 33};
	for (int i = 0; i < 7; i++){
		M.addNum(test[i]);
		printf("%lf\n", M.findMedian());
	}
	return 0;
}

```

# 贪心  

预备知识，钞票选择  
由于大面值总是小面值的倍数，所以可以应用贪心

```cpp
#include <stdio.h>

int main(){
	const int RMB[] = {100, 50, 20, 10, 5, 2, 1};
	const int NUM = 7;
	int X = 628;
	int count = 0;
	for (int i = 0; i < NUM; i++){
		int use = X / RMB[i];
		count += use;
		X = X - RMB[i] * use;
		printf("需要面额为%d的%d张, ", RMB[i], use);
		printf("剩余需要支付RMB %d.\n", X);
	}
	printf("总共需要%d张RMB\n", count);
	return 0;
}


```


1. 给孩子分配糖  

```cpp
#include <stdio.h>

#include <vector>
#include <algorithm>
class Solution {
public:
    int findContentChildren(std::vector<int>& g, std::vector<int>& s) {
    	std::sort(g.begin(), g.end());
    	std::sort(s.begin(), s.end());
    	int child = 0;
    	int cookie = 0;
    	while(child < g.size() && cookie < s.size()){
	    	if (g[child] <= s[cookie]){
	    		child++;
			}
			cookie++;
	    }
    	return child;
    }
};

int main(){
	Solution solve;
	std::vector<int> g;
	std::vector<int> s;
	g.push_back(5);
	g.push_back(10);
	g.push_back(2);
	g.push_back(9);
	g.push_back(15);
	g.push_back(9);
	s.push_back(6);
	s.push_back(1);
	s.push_back(20);
	s.push_back(3);
	s.push_back(8);	
	printf("%d\n", solve.findContentChildren(g, s));
	return 0;
}

```


2. 摇摆序列  

求最长摇摆子序列的长度

```cpp
#include <stdio.h>

#include <vector>
class Solution {
public:
    int wiggleMaxLength(std::vector<int>& nums) {
    	if (nums.size() < 2){
	    	return nums.size();
	    }
	    static const int BEGIN = 0;                 //三个状态
	    static const int UP = 1;
	    static const int DOWN = 2;
	    int STATE = BEGIN;
	    int max_length = 1;                         //子序列长度
    	for (int i = 1; i < nums.size(); i++){
    		switch(STATE){
	    	case BEGIN:								//变成上升或下降，len++
	    		if (nums[i-1] < nums[i]){
	    			STATE = UP;
		    		max_length++;
		    	}
		    	else if (nums[i-1] > nums[i]){
	    			STATE = DOWN;
	    			max_length++;
	    		}
	    		break;
	    	case UP:								//如果在上升状态则下降时才改变状态，len++
	    		if (nums[i-1] > nums[i]){
	    			STATE = DOWN;
		    		max_length++;
		    	}
		    	break;
	    	case DOWN:
	    		if (nums[i-1] < nums[i]){
	    			STATE = UP;
		    		max_length++;
		    	}
		    	break;
		    }
	    }
    	return max_length;
    }
};

int main(){
	std::vector<int> nums;
	nums.push_back(1);
	nums.push_back(17);
	nums.push_back(5);
	nums.push_back(10);
	nums.push_back(13);
	nums.push_back(15);
	nums.push_back(10);
	nums.push_back(5);
	nums.push_back(16);
	nums.push_back(8);
	Solution solve;
	printf("%d\n", solve.wiggleMaxLength(nums));
	return 0;
}

```

3. 去除k个数

去除k个数，得到最小的数   
前一个数比后一个数大就去除  
0之前数字被去除，0就不入栈  

```cpp

#include <stdio.h>
#include <string>
#include <vector>

class Solution { 
public:
    std::string removeKdigits(std::string num, int k) {
    	std::vector<int> S;
    	std::string result = "";
    	for (int i = 0; i < num.length(); i++){                           //遍历字符串
	    	int number = num[i] - '0';									  //当前数字
	    	while(S.size() != 0 && S[S.size()-1] > number && k > 0){      //如果栈顶大于当前数字，栈顶就被去除
	    		S.pop_back();
	    		k--;
	    	}
	    	if (number != 0 || S.size() != 0){   //不为0可以直接入栈，或者栈中有数字
	    		S.push_back(number);
	    	}
	    }
	    while(S.size() != 0 && k > 0){           //还有多余的需要除去
    		S.pop_back();
    		k--;
    	}
	    for (int i = 0; i < S.size(); i++){
    		result.append(1, '0' + S[i]);
    	}
    	if (result == ""){
	    	result = "0";
	    }
    	return result; 
    }
};

int main(){
	Solution solve;
	std::string result = solve.removeKdigits("1432219", 3);
	printf("%s\n", result.c_str());
	std::string result2 = solve.removeKdigits("100200", 1);     //结果是200
	printf("%s\n", result2.c_str());
	return 0;
}


```


4. 跳跃游戏  

判断能否跳到最后

```cpp
#include <stdio.h>

#include <vector>
class Solution {
public:
    bool canJump(std::vector<int>& nums) {
        std::vector<int> index;
        for (int i = 0; i < nums.size(); i++){			  //index保存当前位置能到达最远的地方
        	index.push_back(i + nums[i]);
        }
        int jump = 0;
        int max_index = index[0];
        while(jump < index.size() && jump <= max_index){  //不能越界，不能超过当前最远index
        	if (max_index < index[jump]){
	        	max_index = index[jump];
	        }
        	jump++;
        }
        if (jump == index.size()){
        	return true;
        }
        return false;
    }
};

int main(){
	std::vector<int> nums;
	nums.push_back(2);
	nums.push_back(3);
	nums.push_back(1);
	nums.push_back(1);
	nums.push_back(4);
	Solution solve;
	printf("%d\n", solve.canJump(nums));
	return 0;
}

```

求出最小跳跃步数  
当无法到达更远的地方时，说明前面的位置必有一次跳跃  

```cpp
#include <stdio.h>
#include <vector>
class Solution {
public:
    int jump(std::vector<int>& nums) {
    	if (nums.size() < 2){
	    	return 0;
	    }
        int current_max_index = nums[0];                //当前能跳的最远位置
        int pre_max_max_index = nums[0];				//下一个能跳最远的位置
        int jump_min = 1;
        for (int i = 1; i < nums.size(); i++){
        	if (i > current_max_index){                 //i超过了当前能跳的最远位置，就更新当前最远位置
        		jump_min++;                             //并且跳一步
	        	current_max_index = pre_max_max_index;
	        }
        	if (pre_max_max_index < nums[i] + i){       //保存i遍历过的位置能跳到的最远位置
       			pre_max_max_index = nums[i] + i;
        	}
        }
        return jump_min;
    }
};

int main(){
	std::vector<int> nums;
	nums.push_back(2);
	nums.push_back(3);
	nums.push_back(1);
	nums.push_back(1);
	nums.push_back(4);
	Solution solve;
	printf("%d\n", solve.jump(nums));
	return 0;
}

```

5. 射击气球问题  

```cpp
#include <stdio.h>

#include <algorithm>
#include <vector>

bool cmp(const std::pair<int, int> &a, const std::pair<int ,int> &b) {
    return a.first < b.first;
}

class Solution {
public:
    int findMinArrowShots(std::vector<std::pair<int, int> >& points) {
    	if (points.size() == 0){
	    	return 0;
	    }
    	std::sort(points.begin(), points.end(), cmp); //先按气球开始位置大小排序
    	int shoot_num = 1;
    	int shoot_begin = points[0].first;
    	int shoot_end = points[0].second;
    	for (int i = 1; i < points.size(); i++){      //遍历气球个数减一次
	    	if (points[i].first <= shoot_end){        //下一个气球的开始在射击区间上 
	    		shoot_begin = points[i].first;        //改变射击区间
    			if (shoot_end > points[i].second){    //if嵌套，条件并列，下一个气球的结束在射击区间上
			    	shoot_end = points[i].second;     //改变射击区间
			    }
	    	}
	    	else{									  //先一个气球的开始不在区间内
	    		shoot_num++;						  //多一次射击，新开一个射击区间
	    		shoot_begin = points[i].first;
	    		shoot_end = points[i].second;
	    	}
	    }
	    return shoot_num;
    }
};

int main(){
	std::vector<std::pair<int, int> > points;
	points.push_back(std::make_pair(10, 16));
	points.push_back(std::make_pair(2, 8));
	points.push_back(std::make_pair(1, 6));
	points.push_back(std::make_pair(7, 12));	
	Solution solve;
	printf("%d\n", solve.findMinArrowShots(points));
	return 0;
}

```

6. 最优加油方法  

```cpp
#include <stdio.h>

#include <vector>
#include <algorithm>
#include <queue>

bool cmp(const std::pair<int, int> &a, const std::pair<int ,int> &b) {
    return a.first > b.first;
}

int get_minimum_stop(int L, int P, std::vector<std::pair<int, int> > &stop){ //起点到终点距离，油量
	std::priority_queue<int> Q;												 //stop站点到终点距离和油量
	int result = 0;
	stop.push_back(std::make_pair(0, 0));           //把终点也放到stop中
	std::sort(stop.begin(), stop.end(), cmp);		//按站点位置大到小排序
	for (int i = 0; i < stop.size(); i++){			//遍历站点
		int dis = L - stop[i].first;				//已经走过的距离
		while (!Q.empty() && P < dis){              //油量不够到达当前位置，应该回去堆中的站点加油
			P += Q.top();
			Q.pop();
			result++;								//加油次数加一
		}
		if (Q.empty() && P < dis){					//没有油可以加
			return -1;
		}
		P = P - dis;								//更新油量、距终点距离、堆中的
		L = stop[i].first;
		Q.push(stop[i].second);
	}
	return result;
}

int main(){
	std::vector<std::pair<int, int> > stop;
	int N;
	int L;
	int P;
	int distance;
	int fuel;
	scanf("%d", &N);
	for (int i = 0; i < N; i++){
		scanf("%d %d", &distance, &fuel);
		stop.push_back(std::make_pair(distance, fuel));
	}
	scanf("%d %d", &L, &P);
	printf("%d\n", get_minimum_stop(L, P, stop));
	return 0;
}

```

# 树和图  

树是n(n>=0)个节点的有限集，且这些节点满足如下关系:  
(1)有且仅有一个节点没有父结点，该节点称为树的根。  
(2)除根外，其余的每个节点都有且仅有一个父结点。  
(3)树中的每一个节点都构成一个以它为根的树。  

二叉树在满足树的条件时，满足如下条件:  
每个节点最多有两个孩子(子树)，  
这两个子树有左右之分，次序不可颠倒。  

二叉树遍历  

```cpp
#include <stdio.h>

struct TreeNode {
	int val;
	TreeNode *left;
	TreeNode *right;
	TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

void traversal(TreeNode *node){
	if (!node){
		return;
	}
	
	traversal(node->left);
	
	traversal(node->right);
	
}

void traversal_print1(TreeNode *node,int layer){  //中序遍历
	if (!node){
		return;
	}
	traversal_print1(node->left, layer + 1);
	for (int i = 0; i < layer; i++){
		printf("-----");
	}
	printf("[%d]\n", node->val);
	traversal_print1(node->right, layer + 1);
}

void traversal_print2(TreeNode *node,int layer){  //后续遍历
	if (!node){
		return;
	}
	traversal_print2(node->left, layer + 1);
	traversal_print2(node->right, layer + 1);
	for (int i = 0; i < layer; i++){
		printf("-----");
	}
	printf("[%d]\n", node->val);
}

void traversal_print3(TreeNode *node,int layer){   //先序遍历
	if (!node){
		return;
	}
	for (int i = 0; i < layer; i++){
		printf("-----");
	}
	printf("[%d]\n", node->val);
	traversal_print3(node->left, layer + 1);
	traversal_print3(node->right, layer + 1);
}

int main(){
	TreeNode a(1);
	TreeNode b(2);
	TreeNode c(5);
	TreeNode d(3);
	TreeNode e(4);
	TreeNode f(6);
	a.left = &b;
	a.right = &c;
	b.left = &d;
	b.right = &e;
	c.right = &f;
	traversal_print1(&a, 0);
	printf("\n");
	traversal_print2(&a, 0);
	printf("\n");
	traversal_print3(&a, 0);
	printf("\n");
	return 0;
}

```

路径和  

重点：从右子树返回到结点时，结点出栈  

```cpp
#include <stdio.h>

#include <vector>
struct TreeNode {
	int val;
	TreeNode *left;
	TreeNode *right;
	TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

class Solution {
public:
    std::vector<std::vector<int> > pathSum(TreeNode* root, int sum) {
    	std::vector<std::vector<int> > result;                //嵌套数组存储结果
    	std::vector<int> path;								  //存储路径
    	int path_value = 0;
    	preorder(root, path_value, sum, path, result);
    	return result;
    }
private:
	void preorder(TreeNode *node, int &path_value, int sum,
				std::vector<int> &path,
				std::vector<std::vector<int> > &result){     
		if (!node){											   //空树返回，以及递归出口
			return;
		}
		path_value += node->val;                               //记录路径上的数，和总和
		path.push_back(node->val);
		if (!node->left && !node->right && path_value == sum){ //到达叶子结点并且总和为sum
			result.push_back(path);							   //保存路径
		}
		preorder(node->left, path_value, sum, path, result);   //先序周游
		preorder(node->right, path_value, sum, path, result);
		path_value -= node->val;							   //从右子树返回到结点时，结点出栈
		path.pop_back();
	}
};

int main(){
	TreeNode a(5);
	TreeNode b(4);
	TreeNode c(8);
	TreeNode d(11);
	TreeNode e(13);
	TreeNode f(4);
	TreeNode g(7);
	TreeNode h(2);
	TreeNode x(5);
	TreeNode y(1);
	a.left = &b;
	a.right = &c;
	b.left = &d;
	c.left = &e;
	c.right = &f;
	d.left = &g;
	d.right = &h;
	f.left = &x;
	f.right = &y;
	Solution solve;
	std::vector<std::vector<int> > result = solve.pathSum(&a, 22);
	for (int i = 0; i < result.size(); i++){
		for (int j = 0; j < result[i].size(); j++){
			printf("[%d]", result[i][j]);
		}
		printf("\n");
	}
	return 0;
}

```

最小公共祖先  

递归出口，或找到后标记为真所有的结点都返回  

```cpp

#include <stdio.h>

struct TreeNode {
	int val;
	TreeNode *left;
	TreeNode *right;
	TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

#include <vector>
#include <set>

void preorder(TreeNode* node,							  //求到达search的路径
			  TreeNode *search,
		   	  std::vector<TreeNode*> &path,
		   	  std::vector<TreeNode*> &result,
			  int &finish){
	if (!node || finish){			                      //递归出口，或找到后标记为真所有的结点都返回
		return;
	}
	path.push_back(node);
	if (node == search){
		finish = 1;
		result = path;
	}
	preorder(node->left, search, path, result, finish);
	preorder(node->right, search, path, result, finish);
	path.pop_back();                                      //从右子树返回到结点时，结点出栈
}

class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        std::vector<TreeNode*> path;
        std::vector<TreeNode*> node_p_path;				  //两个path是到达p,q的路径
        std::vector<TreeNode*> node_q_path;
        int finish = 0;
        preorder(root, p, path, node_p_path, finish);
        path.clear();
        finish = 0;
        preorder(root, q, path, node_q_path, finish);        
        int path_len = 0;
        if (node_p_path.size() < node_q_path.size()){     //哪个短就取哪个
        	path_len = node_p_path.size();
        }
        else{
        	path_len = node_q_path.size();
        }
        TreeNode *result = 0;
        for (int i = 0; i < path_len; i++){
        	if (node_p_path[i] == node_q_path[i]){		  //相等就更新result
	        	result = node_p_path[i];
	        }
        }
        return result;
    }
};

int main(){
	TreeNode a(3);
	TreeNode b(5);
	TreeNode c(1);
	TreeNode d(6);
	TreeNode e(2);
	TreeNode f(0);
	TreeNode x(8);
	TreeNode y(7);
	TreeNode z(4);
	a.left = &b;
	a.right = &c;
	b.left = &d;
	b.right = &e;
	c.left = &f;
	c.right = &x;
	e.left = &y;
	e.right = &z;
	
	Solution solve;
	TreeNode *result = solve.lowestCommonAncestor(&a, &b, &f);
	printf("lowestCommonAncestor = %d\n", result->val);
	result = solve.lowestCommonAncestor(&a, &d, &z);
	printf("lowestCommonAncestor = %d\n", result->val);
	result = solve.lowestCommonAncestor(&a, &b, &y);
	printf("lowestCommonAncestor = %d\n", result->val);
	
	return 0;
}
```

二叉树变链表

第一种用一个数组保存所有的指针

```cpp
#include <stdio.h>

struct TreeNode {
	int val;
	TreeNode *left;
	TreeNode *right;
	TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

#include <vector>
class Solution {
public:
    void flatten(TreeNode *root) {
    	std::vector<TreeNode *> node_vec;
    	preorder(root, node_vec);
    	for (int i = 1; i < node_vec.size(); i++){
	    	node_vec[i-1]->left = NULL;
	    	node_vec[i-1]->right = node_vec[i];
	    }
    }
private:
	void preorder(TreeNode *node, std::vector<TreeNode *> &node_vec){
		if (!node){
			return;
		}
		node_vec.push_back(node);
		preorder(node->left, node_vec);
		preorder(node->right, node_vec);
	}
};

int main(){
	TreeNode a(1);
	TreeNode b(2);
	TreeNode c(5);
	TreeNode d(3);
	TreeNode e(4);
	TreeNode f(6);
	a.left = &b;
	a.right = &c;
	b.left = &d;
	b.right = &e;
	c.right = &f;
	Solution solve;
	solve.flatten(&a);
	TreeNode *head = &a;
	while(head){
		if (head->left){
			printf("ERROR\n");
		}
		printf("[%d]", head->val);
		head = head->right;
	}
	printf("\n");
	return 0;
}

```

第二种直接变  

很好的递归练习  

```cpp
#include <stdio.h>

struct TreeNode {
	int val;
	TreeNode *left;
	TreeNode *right;
	TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

class Solution {
public:
    void flatten(TreeNode *root) {
        TreeNode *last = NULL;
        preorder(root, last);
    }
private:
	void preorder(TreeNode *node, TreeNode *&last){   //last需要在递归中传递
		if (!node){
			return;
		}
		if (!node->left && !node->right){		//到达叶节点
			last = node;
			return;
		}
		TreeNode *left = node->left;            //备份左右结点因为node->left和right都变了
		TreeNode *right = node->right;
		TreeNode *left_last = NULL;				//定义左右子树last
		TreeNode *right_last = NULL;
		if (left){
			preorder(left, left_last);			//左子树递归，之后返回
			node->left = NULL;					//node左置为空，右置为左，last是左子树的last
			node->right = left;
			last = left_last;					//last变成左子树last
		}
		if (right){								
			preorder(right, right_last);		//右子树递归，之后返回
			if (left_last){						//如果之前有左子树，就放到左子树后
				left_last->right = right;
			}
			last = right_last;					//last变成右子树last
		}
	}
};

int main(){
	TreeNode a(1);
	TreeNode b(2);
	TreeNode c(5);
	TreeNode d(3);
	TreeNode e(4);
	TreeNode f(6);
	a.left = &b;
	a.right = &c;
	b.left = &d;
	b.right = &e;
	c.right = &f;
	Solution solve;
	solve.flatten(&a);
	TreeNode *head = &a;
	while(head){
		if (head->left){
			printf("ERROR\n");
		}
		printf("[%d]", head->val);
		head = head->right;
	}
	printf("\n");
	return 0;
}

```

二叉树层次遍历  

```cpp

#include <stdio.h>
#include <vector>
#include <queue>

struct TreeNode {
	int val;
	TreeNode *left;
	TreeNode *right;
	TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

void BFS_print(TreeNode* root){
	std::queue<TreeNode *> Q;
	Q.push(root);							//根入队
  	while(!Q.empty()){						//队列不为空
  		TreeNode *node = Q.front();			//取出队头结点
   		Q.pop();
	   	printf("[%d]\n", node->val);		
	   	if (node->left){					//将左右孩子入队
	   		Q.push(node->left);
	   	}
	   	if (node->right){
	   		Q.push(node->right);
	   	}
   	}
}

int main(){
	TreeNode a(1);
	TreeNode b(2);
	TreeNode c(5);
	TreeNode d(3);
	TreeNode e(4);
	TreeNode f(6);
	a.left = &b;
	a.right = &c;
	b.left = &d;
	b.right = &e;
	c.right = &f;
	BFS_print(&a);
	return 0;
}

```

侧面看二叉树  

```cpp
#include <stdio.h>

#include <vector>
#include <queue>

struct TreeNode {
	int val;
	TreeNode *left;
	TreeNode *right;
	TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

class Solution {
public:
    std::vector<int> rightSideView(TreeNode* root) {
        std::vector<int> view;                          //保存结果
    	std::queue<std::pair<TreeNode *, int> > Q;		//保存结点和层数对应  
    	if (root){										//根入队
	    	Q.push(std::make_pair(root, 0));
	    }
    	while(!Q.empty()){
	    	TreeNode *node = Q.front().first;			//队列第一个元素出队
	    	int depth = Q.front().second;
	    	Q.pop();
	    	if (view.size() == depth){					//遇到的该层第一个结点
	    		view.push_back(node->val);
	    	}
	    	else{										//后面的结点，更新view的值
	    		view[depth] = node->val;
	    	}
	    	if (node->left){							//结点的子节点进队
	    		Q.push(std::make_pair(node->left, depth + 1));
	    	}
	    	if (node->right){
	    		Q.push(std::make_pair(node->right, depth + 1));
	    	}
	    }
    	return view;
    }
};

int main(){
	TreeNode a(1);
	TreeNode b(2);
	TreeNode c(5);
	TreeNode d(3);
	TreeNode e(4);
	TreeNode f(6);
	a.left = &b;
	a.right = &c;
	b.left = &d;
	b.right = &e;
	c.right = &f;
	Solution solve;
	std::vector<int> result = solve.rightSideView(&a);
	for (int i = 0; i < result.size(); i++){
		printf("[%d]\n", result[i]);	
	}
	return 0;
}

```

# 递归、分治、回溯  

1. 求所有子集  

看下视频的递归图很清楚  

选择放入item，item = [1]，继续递归处理后续[2,3,4,5,…]元素；  
选择不放入item，item = []，继续递归处理后续[2,3,4,5,…]元素；  


```cpp
#include <stdio.h>

#include <vector>
class Solution {
public:
    std::vector<std::vector<int> > subsets(std::vector<int>& nums) {
        std::vector<std::vector<int> > result;
    	std::vector<int> item;
    	result.push_back(item);
        generate(0, nums, item, result);
        return result;
    }
private:
	void generate(int i, std::vector<int>& nums,
				  std::vector<int> &item, 
				  std::vector<std::vector<int> > &result){
		if (i >= nums.size()){
			return;
		}
		item.push_back(nums[i]);					//每一层要做的先选择放入
		result.push_back(item);                     //记录结果
		generate(i + 1, nums, item, result);		//递归下一层
		item.pop_back();							//返回到当前结点，选择不放入，是一种回溯法
		generate(i + 1, nums, item, result);		//再递归下一层
	}
};

int main(){
	std::vector<int> nums;
	nums.push_back(1);
	nums.push_back(2);
	nums.push_back(3);
	std::vector<std::vector<int> > result;
	Solution solve;
	result = solve.subsets(nums);
	for (int i = 0; i < result.size(); i++){
		if (result[i].size() == 0){
			printf("[]");
		}
		for (int j = 0; j < result[i].size(); j++){
			printf("[%d]", result[i][j]);
		}
		printf("\n");
	}
	return 0;
}

```

求所有子集，用位运算  

```cpp
#include <stdio.h>

#include <vector>
class Solution {
public:
    std::vector<std::vector<int> > subsets(std::vector<int>& nums) {
    	std::vector<std::vector<int> > result;
    	int all_set = 1 << nums.size();					//一共的子集数，2的size次方
    	for (int i = 0; i < all_set; i++){				//遍历所有子集，一个子集是一个item
    		std::vector<int> item;
    		for (int j = 0; j < nums.size(); j++){		//遍历判断元素是否应该出现在当前子集
		    	if (i & (1 << j)){						//i & (1 << j)判断第j位置的元素是否在第i个item
	    			item.push_back(nums[j]);			//相当于把二进制表示的i转成数组
	    		}
		    }
		    result.push_back(item);
	    }
        return result;
    }
};

int main(){
	std::vector<int> nums;
	nums.push_back(1);
	nums.push_back(2);
	nums.push_back(3);
	std::vector<std::vector<int> > result;
	Solution solve;
	result = solve.subsets(nums);
	for (int i = 0; i < result.size(); i++){
		if (result[i].size() == 0){
			printf("[]");
		}
		for (int j = 0; j < result[i].size(); j++){
			printf("[%d]", result[i][j]);
		}
		printf("\n");
	}
	return 0;
}

```

序列无序且可重复，求所有子集  

```cpp
#include <stdio.h>

#include <vector>
#include <set>
#include <algorithm>
class Solution {
public:
    std::vector<std::vector<int> > subsetsWithDup(std::vector<int>& nums) {
        std::vector<std::vector<int> > result;
    	std::vector<int> item;
    	std::set<std::vector<int> > res_set;
    	std::sort(nums.begin(), nums.end());              //给序列排序
    	result.push_back(item);
        generate(0, nums, result, item, res_set);
        return result;
    }
private:
	void generate(int i, std::vector<int>& nums,
			std::vector<std::vector<int> > &result,
			std::vector<int> &item,
			std::set<std::vector<int> > &res_set){
		if (i >= nums.size()){
			return;
		}
		item.push_back(nums[i]);                          //这种情况是当前元素加进去
		if (res_set.find(item) == res_set.end()){		  //注意这个是在已有的结果中没找到
			result.push_back(item);						  //就加进去
			res_set.insert(item);
		}
		generate(i + 1, nums, result, item, res_set);     //递归下一层
		item.pop_back();								  //返回当前元素时把当前元素取出来
		generate(i + 1, nums, result, item, res_set);	  //递归下一层
	}
};

int main(){
	std::vector<int> nums;
	nums.push_back(2);
	nums.push_back(1);
	nums.push_back(2);
	nums.push_back(2);
	
	std::vector<std::vector<int> > result;
	Solution solve;
	result = solve.subsetsWithDup(nums);
	for (int i = 0; i < result.size(); i++){
		if (result[i].size() == 0){
			printf("[]");
		}
		for (int j = 0; j < result[i].size(); j++){
			printf("[%d]", result[i][j]);
		}
		printf("\n");
	}
	return 0;
}

```

序列无序且可重复，求子集和为特定整数的子集  

```cpp
#include <stdio.h>

#include <vector>
#include <set>
#include <algorithm>
class Solution {
public:
	std::vector<std::vector<int> > combinationSum2(
									std::vector<int>& candidates,
							 		int target){
        std::vector<std::vector<int> > result;
    	std::vector<int> item;
    	std::set<std::vector<int> > res_set;
    	std::sort(candidates.begin(), candidates.end());
        generate(0, candidates, result, item, res_set, 0, target);
        return result;
    }
private:
	void generate(int i, std::vector<int>& nums,
			std::vector<std::vector<int> > &result,
			std::vector<int> &item,
			std::set<std::vector<int> > &res_set,
			int sum, int target){
		if (i >= nums.size() || sum > target){                         //增加条件进行剪枝
			return;
		}
		sum += nums[i];
		item.push_back(nums[i]);
		if (target == sum && res_set.find(item) == res_set.end()){     //当sum为target时才放到result
			result.push_back(item);
			res_set.insert(item);
		}
		generate(i + 1, nums, result, item, res_set, sum, target);
		sum -= nums[i];												   //取出当前元素时，sum也要减去
		item.pop_back();
		generate(i + 1, nums, result, item, res_set, sum, target);
	}
};

int main(){
	std::vector<int> nums;
	nums.push_back(10);
	nums.push_back(1);
	nums.push_back(2);
	nums.push_back(7);
	nums.push_back(6);
	nums.push_back(1);
	nums.push_back(5);
	std::vector<std::vector<int> > result;
	Solution solve;
	result = solve.combinationSum2(nums, 8);
	for (int i = 0; i < result.size(); i++){
		if (result[i].size() == 0){
			printf("[]");
		}
		for (int j = 0; j < result[i].size(); j++){
			printf("[%d]", result[i][j]);
		}
		printf("\n");
	}
	return 0;
}

```

2. 生成括号  

只给定括号的组数，判断  
有两种选择就有两个generate  

```cpp
#include <stdio.h>

#include <string>
#include <vector>
class Solution {
public:
    std::vector<std::string> generateParenthesis(int n) {
    	std::vector<std::string> result;
    	generate("", n, n, result);
    	return result;
    }
private:
	void generate(std::string item, int left, int right,
					std::vector<std::string> &result){
		if (left == 0 && right == 0){
			result.push_back(item);
			return;
		}
		if (left > 0){											//剪枝，保证结果正确
			generate(item + '(', left - 1, right, result);      //item变化直接在参数中
		}
		if (left < right){										//剪枝，保证结果正确，递归树左边返回
			generate(item + ')', left, right - 1, result);		
		}
	}
};

int main(){
	Solution solve;
	std::vector<std::string> result = solve.generateParenthesis(3);
	for (int i = 0; i < result.size(); i++){
		printf("%s\n", result[i].c_str());
	}
	return 0;
}

```


3. N皇后问题  

每行有n种选择，就有n个generate  

```cpp
#include <stdio.h>

#include <string>
#include <vector>

class Solution {
public:
    std::vector<std::vector<std::string> > solveNQueens(int n) {
        std::vector<std::vector<std::string> > result;
        std::vector<std::vector<int> > mark;
        std::vector<std::string> location;
        for (int i = 0; i < n; i++){
        	mark.push_back((std::vector<int>()));
        	for (int j = 0; j < n; j++){
	        	mark[i].push_back(0);
	        }
	        location.push_back("");
	        location[i].append(n, '.');
        }
        generate(0, n, location, result, mark);
        return result;
    }
private:
	void put_down_the_queen(int x, int y,
							std::vector<std::vector<int> > &mark){
		static const int dx[] = {-1, 1, 0, 0, -1, -1, 1, 1};		//两个数组对应八个方向
		static const int dy[] = {0, 0, -1, 1, -1, 1, -1, 1};
		mark[x][y] = 1;												//皇后位置置为一
		for (int i = 1; i < mark.size(); i++){						//八个方向的点置为一
			for (int j = 0; j < 8; j++){
				int new_x = x + i * dx[j];
				int new_y = y + i * dy[j];
				if (new_x >= 0 && new_x < mark.size() &&			//不能出棋盘
					new_y >= 0 && new_y < mark.size()){
					mark[new_x][new_y] = 1;
				}
			}
		}
	}
	void generate(int k, int n,
				std::vector<std::string> &location,
				std::vector<std::vector<std::string> > &result,
				std::vector<std::vector<int> > &mark){
		if (k == n){												//当到达第n行结束
			result.push_back(location);
			return;
		}
		for (int i = 0; i < n; i++){								//遍历n列
			if (mark[k][i] == 0){									//位置可以放皇后
				std::vector<std::vector<int> > tmp_mark = mark;		//记录mark，回溯需要变回来
				location[k][i] = 'Q';								//放入改变location和mark
				put_down_the_queen(k, i, mark);						
				generate(k + 1, n, location, result, mark);			//递归下一行
				mark = tmp_mark;									//从下一行返回时，恢复没放入状态
				location[k][i] = '.';
			}
		}
	}
};

int main(){
	std::vector<std::vector<std::string> > result;
	Solution solve;
	result = solve.solveNQueens(4);
	for (int i = 0; i < result.size(); i++){
		printf("i = %d\n", i);
		for (int j = 0; j < result[i].size(); j++){
			printf("%s\n", result[i][j].c_str());
		}
		printf("\n");
	}	
	return 0;
}

```

4. 归并排序  

```cpp

#include <vector>

void merge_sort_two_vec(std::vector<int> &sub_vec1,         //合并两个数组，仍然有序
						std::vector<int> &sub_vec2,
						std::vector<int> &vec){
	int i = 0;
	int j = 0;
	while(i < sub_vec1.size() && j < sub_vec2.size()){
		if (sub_vec1[i] <= sub_vec2[j]){
			vec.push_back(sub_vec1[i]);
			i++;
		}
		else{
			vec.push_back(sub_vec2[j]);
			j++;
		}
	}
	for (; i < sub_vec1.size(); i++){				//后面元素没有push完
		vec.push_back(sub_vec1[i]);
	}
	for (; j < sub_vec2.size(); j++){
		vec.push_back(sub_vec2[j]);
	}
}

void merge_sort(std::vector<int> &vec){
	if (vec.size() < 2){							//只剩一个直接返回
		return;
	}
	int mid = vec.size() / 2;						//拆成两半，分别放到sub_vec1，sub_vec2
	std::vector<int> sub_vec1;
	std::vector<int> sub_vec2;
	for (int i = 0; i < mid; i++){
		sub_vec1.push_back(vec[i]);
	}
	for (int i = mid; i < vec.size(); i++){
		sub_vec2.push_back(vec[i]);
	}
	merge_sort(sub_vec1);
	merge_sort(sub_vec2);							//vec相当于每一步的返回
	vec.clear();
	merge_sort_two_vec(sub_vec1, sub_vec2, vec);
}

#include <stdio.h>
#include <stdlib.h>
#include <algorithm>
#include <assert.h>
int main(){
	std::vector<int> vec1;
	std::vector<int> vec2;
	srand(time(NULL));
	for (int i = 0; i < 10000; i++){
		int num = (rand() * rand()) % 100003;
		vec1.push_back(num);
		vec2.push_back(num);
	}
	merge_sort(vec1);
	std::sort(vec2.begin(), vec2.end());
	assert(vec1.size() == vec2.size());
	for (int i = 0; i < vec1.size(); i++){
		assert(vec1[i] == vec2[i]);
	}
	return 0;
}

```

5. 逆序数  

合并时更新count，i指向的元素逆序数就是j的值，还要加上之前的逆序数  

```cpp
#include <stdio.h>

#include <vector>
class Solution {
public:
    std::vector<int> countSmaller(std::vector<int>& nums) {
    	std::vector<std::pair<int, int> > vec;
    	std::vector<int> count;
    	for (int i = 0; i < nums.size(); i++){				//vec里存pair，存数字和原位置
	    	vec.push_back(std::make_pair(nums[i], i));		//count里存0
	    	count.push_back(0);
	    }
        merge_sort(vec, count);
        return count;
    }
private:
	void merge_sort_two_vec(
				std::vector<std::pair<int, int> > &sub_vec1,
				std::vector<std::pair<int, int> > &sub_vec2,
				std::vector<std::pair<int, int> > &vec,
				std::vector<int> &count){
		int i = 0;
		int j = 0;
		while(i < sub_vec1.size() && j < sub_vec2.size()){
			if (sub_vec1[i].first <= sub_vec2[j].first){     //左侧的值小，j就加到对应逆序数
				count[sub_vec1[i].second] += j;
				vec.push_back(sub_vec1[i]);
				i++;
			}
			else{											//右侧值小，值放到vec，j++
				vec.push_back(sub_vec2[j]);
				j++;
			}
		}
		for (; i < sub_vec1.size(); i++){					//左侧后面的序列没有push完
			count[sub_vec1[i].second] += j;
			vec.push_back(sub_vec1[i]);
		}
		for (; j < sub_vec2.size(); j++){
			vec.push_back(sub_vec2[j]);
		}
	}
	void merge_sort(std::vector<std::pair<int, int> > &vec,
					std::vector<int> &count){
		if (vec.size() < 2){								//和上面一样  
			return;
		}
		int mid = vec.size() / 2;
		std::vector<std::pair<int, int> > sub_vec1;
		std::vector<std::pair<int, int> > sub_vec2;
		for (int i = 0; i < mid; i++){
			sub_vec1.push_back(vec[i]);
		}
		for (int i = mid; i < vec.size(); i++){
			sub_vec2.push_back(vec[i]);
		}
		merge_sort(sub_vec1, count);
		merge_sort(sub_vec2, count);
		vec.clear();
		merge_sort_two_vec(sub_vec1, sub_vec2, vec, count);
	}
};

int main(){
	int test[] = {5, -7, 9, 1, 3, 5, -2, 1};
	std::vector<int> nums;
	for (int i = 0; i < 8; i++){
		nums.push_back(test[i]);
	}
	Solution solve;
	std::vector<int> result = solve.countSmaller(nums);
	for (int i = 0; i < result.size(); i++){
		printf("[%d]", result[i]);
	}
	printf("\n");
	return 0;
}

```


# 动态规划  

1. 爬楼梯  

回溯法  

```cpp
#include <stdio.h>

class Solution {
public:
    int climbStairs(int n) {
    	if (n == 1 || n == 2){							//递归出口，剩下两节返回两种方法
			return n;
		}
	    return climbStairs(n-1) + climbStairs(n-2);		//递归树分两枝
    }
};

int main(){
	Solution solve;
	printf("%d\n", solve.climbStairs(3));	
	return 0;
}

```

动态规划  

第i阶的方式数量 = 第i-1阶方式数量 + 第i-2阶方式方式数量  

```cpp
#include <stdio.h>

#include <vector>
class Solution {
public:
    int climbStairs(int n) {
    	std::vector<int> dp(n + 3, 0);			
    	dp[1] = 1;								//边界条件
    	dp[2] = 2;
    	for (int i = 3; i <= n; i++){			//写出递推公式
	    	dp[i] = dp[i-1] + dp[i-2];
	    }
	    return dp[n];
    }
};

int main(){
	Solution solve;
	printf("%d\n", solve.climbStairs(3));	
	return 0;
}
```

盗取财宝，不能盗相邻房间  

dp[i] = max(dp[i-1],dp[i-2]+nums[i])  

```cpp
#include <stdio.h>

#include <vector>
class Solution {
public:
    int rob(std::vector<int>& nums) {
    	if (nums.size() == 0){								//边界条件
	    	return 0;
	    }
	    if (nums.size() == 1){
    		return nums[0];
    	}
    	std::vector<int> dp(nums.size(), 0);				//递推公式
    	dp[0] = nums[0];
    	dp[1] = std::max(nums[0], nums[1]);
    	for (int i = 2; i < nums.size(); i++){
	    	dp[i] = std::max(dp[i-1], dp[i-2] + nums[i]);
	    }
	    return dp[nums.size() - 1];
    }
};

int main(){
	Solution solve;
	std::vector<int> nums;
	nums.push_back(5);
	nums.push_back(2);
	nums.push_back(6);
	nums.push_back(3);
	nums.push_back(1);
	nums.push_back(7);	
	printf("%d\n", solve.rob(nums));
	return 0;
}

```

最大加和序列  

若dp[i-1]>0,dp[i]=dp[i-1]+nums[i]
否则dp[i]=nums[i]  


```cpp
#include <stdio.h>

#include <vector>
class Solution {
public:
    int maxSubArray(std::vector<int>& nums) {
    	std::vector<int> dp(nums.size(), 0);
    	dp[0] = nums[0];									//边界条件
    	int max_res = dp[0];
    	for (int i = 1; i < nums.size(); i++){
	    	dp[i] = std::max(dp[i-1] + nums[i], nums[i]);	//递推公式
	    	if (max_res < dp[i]){							//找到dp[i]中最大的
	    		max_res = dp[i];
	    	}
	    }
        return max_res;
    }
};

int main(){
	Solution solve;
	std::vector<int> nums;
	nums.push_back(-2);
	nums.push_back(1);
	nums.push_back(-3);
	nums.push_back(4);
	nums.push_back(-1);
	nums.push_back(2);
	nums.push_back(1);
	nums.push_back(-5);
	nums.push_back(4);
	printf("%d\n", solve.maxSubArray(nums));
	return 0;
}

```

最小数值钞票  

当i-coins[j] >= 0且dp[i-coins[j]!=-1]时：  表示可以减去相应的钞票值，并且减去后对应的数组元素有值
j = 0,1,2,3,4; coins[j] = 1,2,5,7,10      
dp[i] = getmin(dp[i-coins[j]]) + 1        dp[i]就是减去一张钞票结果的最小值加一  

```cpp
#include <stdio.h>

#include <vector>

class Solution {
public:
    int coinChange(std::vector<int>& coins, int amount) {
		std::vector<int> dp;
		for (int i = 0; i <= amount; i++){
			dp.push_back(-1);
		}
		dp[0] = 0;														//边界条件  
		for (int i = 1; i <= amount; i++){
			for (int j = 0; j < coins.size(); j++){
				if (i - coins[j] >= 0 && dp[i - coins[j]] != -1){		//可以减去相应的钞票值，并且减去后对应的数组元素有值
					if (dp[i] == -1 || dp[i] > dp[i - coins[j]] + 1){	//dp[i]为-1或者dp[i]大于当前计算的结果
						dp[i] = dp[i - coins[j]] + 1;					//更新dp[i]
					}
				}
			}
		}
		return dp[amount];
    }
};

int main(){	
	Solution solve;
	std::vector<int> coins;
	coins.push_back(1);
	coins.push_back(2);
	coins.push_back(5);
	coins.push_back(7);
	coins.push_back(10);	
	for (int i = 1; i<= 14; i++){
		printf("dp[%d] = %d\n", i, solve.coinChange(coins, i));
	}
	return 0;
}

```

三角形，最小路径  

dp[i][j]=min(dp[i+1][j],dp[i+1][j+1])+triangle[i][j]  

```cpp

#include <stdio.h>
#include <vector>

class Solution {
public:
    int minimumTotal(std::vector<std::vector<int> >& triangle){
    	if (triangle.size() == 0){
	    	return 0;
	    }
    	std::vector<std::vector<int> > dp;						//构造dp和三角形相同
    	for (int i = 0; i < triangle.size(); i++){			
	    	dp.push_back(std::vector<int>());
	    	for (int j = 0; j < triangle.size(); j++){
	    		dp[i].push_back(0);
	    	}
	    }
	    for (int i = 0; i < dp.size(); i++){					//边界条件，最后一行就是triangle的值
    		dp[dp.size()-1][i] = triangle[dp.size()-1][i];
    	}
	    for (int i = dp.size() - 2; i >= 0; i--){				//遍历列和行
	    	for (int j = 0; j < dp[i].size(); j++)				
	    		dp[i][j] = std::min(dp[i+1][j], dp[i+1][j+1])	//递推公式
							 + triangle[i][j];
    	}
	    return dp[0][0];
    }
};

int main(){
	std::vector<std::vector<int> > triangle;
	int test[][10] = {{2}, {3, 4}, {6, 5, 7}, {4, 1, 8, 3}};
	for (int i = 0; i < 4; i++){
		triangle.push_back(std::vector<int>());
		for (int j = 0; j < i + 1; j++){
			triangle[i].push_back(test[i][j]);
		}
	}
	Solution solve;
	printf("%d\n", solve.minimumTotal(triangle));
	return 0;
}

```

最长上升子序列  

若nums[i]>nums[j]，说明nums[i]可放置在nums[j]的后面，组成最长上升子序列  
若dp[i]<dp[j]+1: dp[i]=dp[j]+1  

```cpp
#include <stdio.h>

#include <vector>

class Solution {
public:
    int lengthOfLIS(std::vector<int>& nums) {
    	if (nums.size() == 0){
	    	return 0;
	    }
        std::vector<int> dp(nums.size(), 0);
        dp[0] = 1;
        int LIS = 1;
        for (int i = 1; i < dp.size(); i++){					//遍历所有
        	dp[i] = 1;
        	for (int j = 0; j < i; j++){						//遍历之前的
	        	if (nums[i] > nums[j] && dp[i] < dp[j] + 1){	//当前数字能排到某序列后，并且比该序列长度加一小
	        		dp[i] = dp[j] + 1;
	        	}
	        }
	        if (LIS < dp[i]){
        		LIS = dp[i];
        	}
        }
        return LIS;
    }
};

int main(){
	int test[] = {10, 9, 2, 5, 3, 7, 101, 18};
	std::vector<int> nums;
	for (int i = 0; i < 8; i++){
		nums.push_back(test[i]);
	}
	Solution solve;
	printf("%d\n", solve.lengthOfLIS(nums));
	return 0;
}

```

第二种，用栈解决  

```cpp
#include <stdio.h>

#include <vector>

class Solution {
public:
    int lengthOfLIS(std::vector<int>& nums) {
    	if (nums.size() == 0){
	    	return 0;
	    }
	    std::vector<int> stack;
	    stack.push_back(nums[0]);
	    for (int i = 1; i < nums.size(); i++){
	    	if (nums[i] > stack.back()){					//比栈顶大的就入栈
	    		stack.push_back(nums[i]);
	    	}
	    	else{
	    		for (int j = 0; j < stack.size(); j++){		//否则替换栈中第一个比他大的元素  
		    		if (stack[j] >= nums[i]){
		    			stack[j] = nums[i];
		    			break;
		    		}
		    	}
	    	}
    	}
        return stack.size();
    }
};

int main(){
	int test[] = {1, 3, 2, 3, 1, 4};
	std::vector<int> nums;
	for (int i = 0; i < 6; i++){
		nums.push_back(test[i]);
	}
	Solution solve;
	printf("%d\n", solve.lengthOfLIS(nums));
	return 0;
}

```

最小路径之和  

```cpp
#include <stdio.h>

#include <vector>

class Solution {
public:
    int minPathSum(std::vector<std::vector<int> >& grid) {
    	if (grid.size() == 0){
	    	return 0;
	    }
	    int row = grid.size();
    	int column = grid[0].size();
    	std::vector<std::vector<int> > 
						dp(row, std::vector<int>(column, 0));
    	
	    dp[0][0] = grid[0][0];
	    for (int i = 1; i < column; i++){					//先初始化第一行
    		dp[0][i] = dp[0][i-1] + grid[0][i];
    	}
	    for (int i = 1; i < row; i++){						//初始化其他  
	    	dp[i][0] = dp[i-1][0] + grid[i][0];				//第一列直接加
    		for (int j = 1; j < column; j++){
		    	dp[i][j] = std::min(dp[i-1][j], dp[i][j-1]) + grid[i][j];	//其他的要对比两次加的结果  
		    }
    	}
	    return dp[row-1][column-1];
    }
};

int main(){
	int test[][3] = {{1,3,1}, {1,5,1}, {4,2,1}};
	std::vector<std::vector<int> > grid;
	for (int i = 0; i < 3; i++){
		grid.push_back(std::vector<int>());
		for (int j = 0; j < 3; j++){
			grid[i].push_back(test[i][j]);
		}
	}
	Solution solve;
	printf("%d\n", solve.minPathSum(grid));	
	return 0;
}

```

# 二分查找树和二叉排序树  

二分查找递归  

```cpp
#include <stdio.h>

#include <vector>

bool binary_search(std::vector<int> &sort_array,
				   int begin, int end, int target){
	if (begin > end){
		return false;
	}
	int mid = (begin + end) / 2;								//中点
	if (target == sort_array[mid]){
		return true;
	}
	else if (target < sort_array[mid]){							//左边查找
		return binary_search(sort_array, begin, mid-1, target);	
	}
	else if (target > sort_array[mid]){							//右边查找
		return binary_search(sort_array, mid + 1, end, target);
	}
}

std::vector<int> search_array(std::vector<int> &sort_array,
							  std::vector<int> &random_array){
	std::vector<int> result;
	for (int i = 0; i < random_array.size(); i++){
		int res = binary_search(sort_array,
								0,
								sort_array.size() - 1,
								random_array[i]);
		result.push_back(res);
	}
	return result;
}

int main(){
	int A[] = {-1, 2, 5, 20, 90, 100, 207, 800};
	int B[] = {50, 90, 3, -1, 207, 80};
	std::vector<int> sort_array;
	std::vector<int> random_array;
	std::vector<int> C;
	for (int i = 0; i < 8; i++){
		sort_array.push_back(A[i]);
	}
	for (int i = 0; i < 6; i++){
		random_array.push_back(B[i]);
	}
	C = search_array(sort_array, random_array);
	for (int i = 0; i < C.size(); i++){
		printf("%d\n", C[i]);
	}
	return 0;
}

```

循环版本  

```cpp
#include <stdio.h>
#include <vector>

bool binary_search(std::vector<int> &sort_array, int target){
	int begin = 0;
	int end = sort_array.size() - 1;
	while(begin <= end){
		int mid = (begin + end) / 2;			//计算中点
		if (target == sort_array[mid]){
			return true;
		}
		else if (target < sort_array[mid]){		//改变区间
			end = mid - 1;
		}
		else if (target > sort_array[mid]){
			begin = mid + 1;
		}
	}
	return false;
}

std::vector<int> search_array(std::vector<int> &sort_array,
							  std::vector<int> &random_array){
	std::vector<int> result;
	for (int i = 0; i < random_array.size(); i++){
		int res = binary_search(sort_array,	random_array[i]);
		result.push_back(res);
	}
	return result;
}

int main(){
	int A[] = {-1, 2, 5, 20, 90, 100, 207, 800};
	int B[] = {50, 90, 3, -1, 207, 80};
	std::vector<int> sort_array;
	std::vector<int> random_array;
	std::vector<int> C;
	for (int i = 0; i < 8; i++){
		sort_array.push_back(A[i]);
	}
	for (int i = 0; i < 6; i++){
		random_array.push_back(B[i]);
	}
	C = search_array(sort_array, random_array);
	for (int i = 0; i < C.size(); i++){
		printf("%d\n", C[i]);
	}
	return 0;
}

```

1.插入位置  

```cpp
#include <stdio.h>

#include <vector>

class Solution {
public:
    int searchInsert(std::vector<int>& nums, int target) {
        int index = -1;
		int begin = 0;
		int end = nums.size() - 1;
		while (index == -1){
			int mid = (begin + end) / 2;
			if (target == nums[mid]){						//直接找到这个数
				index = mid;
			}
			else if (target < nums[mid]){
				if (mid == 0 || target > nums[mid - 1]){	//找到位置的情况
					index = mid;
				}
				end = mid - 1;
			}
			else if (target > nums[mid]){
				if (mid == nums.size() - 1 || target < nums[mid + 1]){//找到位置的情况
					index = mid + 1;
				}
				begin = mid + 1;
			}
		}
		return index;
    }
};

int main(){
	int test[] = {1, 3, 5, 6};
	std::vector<int> nums;
	Solution solve;
	for (int i = 0; i < 4; i++){
		nums.push_back(test[i]);
	}
	for (int i = 0; i < 8; i++){
		printf("i = %d index = %d\n", i, solve.searchInsert(nums, i));
	}
	return 0;
}


```

2. 查找区间  
转换成查找左右端点  

```cpp
#include <stdio.h>

#include <vector>

int left_bound(std::vector<int>& nums, int target){
	int begin = 0;
	int end = nums.size() - 1;
	while(begin <= end){
		int mid = (begin + end) / 2;
		if (target == nums[mid]){						//在数组里找到
			if (mid == 0 || nums[mid -1] < target){		//是开头或者左侧小于target
				return mid;								//说明就是区间左端点
			}
			end = mid - 1;
		}
		else if (target < nums[mid]){
			end = mid - 1;
		}
		else if (target > nums[mid]){
			begin = mid + 1;
		}
	}
	return -1;
}

int right_bound(std::vector<int>& nums, int target){
	int begin = 0;
	int end = nums.size() - 1;
	while(begin <= end){
		int mid = (begin + end) / 2;
		if (target == nums[mid]){
			if (mid == nums.size() - 1 || nums[mid + 1] > target){
				return mid;
			}
			begin = mid + 1;
		}
		else if (target < nums[mid]){
			end = mid - 1;
		}
		else if (target > nums[mid]){
			begin = mid + 1;
		}
	}
	return -1;
}

class Solution {
public:
    std::vector<int> searchRange(std::vector<int>& nums, int target) {
        std::vector<int> result;
        result.push_back(left_bound(nums, target));
        result.push_back(right_bound(nums, target));
        return result;
    }
};

int main(){
	int test[] = {5, 7, 7, 8, 8, 8, 8, 10};
	std::vector<int> nums;
	Solution solve;
	for (int i = 0; i < 8; i++){
		nums.push_back(test[i]);
	}
	for (int i = 0; i < 12; i++){
		std::vector<int> result = solve.searchRange(nums, i);
		printf("%d : [%d, %d]\n",i , result[0], result[1]);
	}
	return 0;
}

```

3. 在旋转数组中查找  
中点可以把数组分为旋转区间和递增区间  
开始结点一定比最后的结点大  

```cpp
#include <stdio.h>

#include <vector>

class Solution {
public:
    int search(std::vector<int>& nums, int target) {
    	int begin = 0;
		int end = nums.size() - 1;
		while(begin <= end){
			int mid = (begin + end) / 2;
			if (target == nums[mid]){
				return mid;
			}
			else if (target < nums[mid]){
				if (nums[begin] < nums[mid]){		//递增区间在前面
					if (target >= nums[begin]){		//target < nums[mid]，target >= nums[begin]这种情况应该在前面的递增区间找
						end = mid - 1;
					}
					else{							//否则应该在旋转区间找
						begin = mid + 1;
					}
				}
				else if (nums[begin] > nums[mid]){	//递增区间在后面
					end = mid -1;					//target < nums[mid]去前面找
				}
				else if (nums[begin] == nums[mid]){	//两个元素的情况
					begin = mid + 1;
				}
			}
			else if (target > nums[mid]){			
				if (nums[begin] < nums[mid]){		
					begin = mid + 1;				
				}
				else if (nums[begin] > nums[mid]){
					if (target >= nums[begin]){
						end = mid - 1;
					}
					else{
						begin = mid + 1;
					}
				}
				else if (nums[begin] == nums[mid]){
					begin = mid + 1;
				}
			}
		}
		return -1;
    }
};

int main(){
	int test[] = {9, 12, 15, 20, 1, 3, 6, 7};
	std::vector<int> nums;
	Solution solve;
	for (int i = 0; i < 8; i++){
		nums.push_back(test[i]);
	}
	for (int i = 0; i < 22; i++){
		printf("%d : %d\n", i, solve.search(nums, i));
	}
	return 0;
}

```

二叉查找树插入  

```cpp
#include <stdio.h>
#include <vector>

struct TreeNode {
	int val;
	TreeNode *left;
	TreeNode *right;
	TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

void BST_insert(TreeNode *node, TreeNode *insert_node){
	if (insert_node->val < node->val){
		if (node->left){
			BST_insert(node->left, insert_node);
		}
		else{
			node->left = insert_node;
		}
	}
	else{
		if (node->right){
			BST_insert(node->right, insert_node);
		}
		else{
			node->right = insert_node;
		}
	}
}

void preorder_print(TreeNode *node,int layer){
	if (!node){
		return;
	}
	for (int i = 0; i < layer; i++){
		printf("-----");
	}
	printf("[%d]\n", node->val);
	preorder_print(node->left, layer + 1);
	preorder_print(node->right, layer + 1);
}

int main(){
	TreeNode root(8);
	std::vector<TreeNode *> node_vec;
	int test[] = {3, 10, 1, 6, 15};
	for (int i = 0; i < 5; i++){
		node_vec.push_back(new TreeNode(test[i]));
	}
	for (int i = 0; i < node_vec.size(); i++){
		BST_insert(&root, node_vec[i]);
	}
	preorder_print(&root, 0);
	return 0;
}

```

二叉查找树查找  

```cpp
#include <stdio.h>
#include <vector>

struct TreeNode {
	int val;
	TreeNode *left;
	TreeNode *right;
	TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

bool BST_search(TreeNode *node, int value){
	if (node->val == value){
		return true;
	}
	if (value < node->val){
		if (node->left){
			return BST_search(node->left, value);
		}
		else{
			return false;
		}
	}
	else{
		if (node->right){
			return BST_search(node->right, value);
		}
		else{
			return false;
		}
	}
}

int main(){
	TreeNode a(8);
	TreeNode b(3);
	TreeNode c(10);
	TreeNode d(1);
	TreeNode e(6);
	TreeNode f(15);
	a.left = &b;
	a.right = &c;
	b.left = &d;
	b.right = &e;
	c.right = &f;
	for (int i = 0; i < 20; i++){
		if (BST_search(&a, i)){
			printf("%d is in the BST.\n", i);
		}
		else{
			printf("%d is not in the BST.\n", i);
		}
	}
	return 0;
}

```

4. 二叉排序树编码和解码  
编码通过前序遍历，并在每个数字后加#  
解码过程通过数字数组还原二叉树  

```cpp

#include <stdio.h>


struct TreeNode {
	int val;
	TreeNode *left;
	TreeNode *right;
	TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

#include <string>
#include <vector>

void BST_insert(TreeNode *node, TreeNode *insert_node){	//解码过程，将结点依次插入
	if (insert_node->val < node->val){					//插入结点比当前结点小插入左边
		if (node->left){								//左子树不为空
			BST_insert(node->left, insert_node);		//递归左子树
		}
		else{											//左子树为空直接插入
			node->left = insert_node;
		}
	}
	else{
		if (node->right){
			BST_insert(node->right, insert_node);
		}
		else{
			node->right = insert_node;
		}
	}
}

void change_int_to_string(int val, std::string &str_val){//整形转字符串
	std::string tmp;
	while(val){											//顺序是和原数字相反
		tmp += val % 10 + '0';						
		val = val / 10;
	}
	for (int i = tmp.length() - 1; i >= 0; i--){		//逆序一次
		str_val += tmp[i];
	}
	str_val += '#';
}

void BST_preorder(TreeNode *node, std::string &data){	//通过前序遍历，编码二叉树
	if (!node){
		return;
	}
	std::string str_val;
	change_int_to_string(node->val, str_val);
	data += str_val;
	BST_preorder(node->left, data);
	BST_preorder(node->right, data);
}

class Codec {
public:
    std::string serialize(TreeNode* root) {
    	std::string data;
        BST_preorder(root, data);
        return data;
    }
    TreeNode *deserialize(std::string data) {
    	if (data.length() == 0){
	    	return NULL;
	    }
    	std::vector<TreeNode *> node_vec;
    	int val = 0;
    	for (int i = 0; i < data.length(); i++){		//将字符串还原成整形，转成TreeNode，放入node_vec数组
	    	if (data[i] == '#'){
	    		node_vec.push_back(new TreeNode(val));	
	    		val = 0;
	    	}
	    	else{
	    		val = val * 10 + data[i] - '0';
	    	}
	    }
	    for (int i = 1; i < node_vec.size(); i++){
    		BST_insert(node_vec[0], node_vec[i]);
    	}
    	return node_vec[0];
    }
};

void preorder_print(TreeNode *node,int layer){
	if (!node){
		return;
	}
	for (int i = 0; i < layer; i++){
		printf("-----");
	}
	printf("[%d]\n", node->val);
	preorder_print(node->left, layer + 1);
	preorder_print(node->right, layer + 1);
}

int main(){
	TreeNode a(8);
	TreeNode b(3);
	TreeNode c(10);
	TreeNode d(1);
	TreeNode e(6);
	TreeNode f(15);	
	a.left = &b;
	a.right = &c;
	b.left = &d;
	b.right = &e;
	c.left = &f;	
	Codec solve;	
	std::string data = solve.serialize(&a);
	printf("%s\n", data.c_str());
	TreeNode *root = solve.deserialize(data);
	preorder_print(root, 0);	
	return 0;
}

```

5. 求逆序数  
先把数组逆序，然后看前面有几个比当前插入值小  
二叉排序树的结点存储count，表示当前结点的左子树结点个数，即小于当前结点的个数  


```cpp
#include <stdio.h>

#include <vector>
struct BSTNode {
	int val;
	int count;
	BSTNode *left;
	BSTNode *right;
	BSTNode(int x) : val(x), left(NULL), right(NULL), count(0) {}
};

void BST_insert(BSTNode *node, BSTNode *insert_node, int &count_small){
	if (insert_node->val <= node->val){							//插入结点比当前结点小
		node->count++;											//比当前结点小的结点数加一
		if (node->left){										//左子树不为空
			BST_insert(node->left, insert_node, count_small);	//递归左子树
		}
		else{													//没有左子树直接插入
			node->left = insert_node;
		}
	}
	else{														//插入结点比当前结点大
		count_small += node->count + 1;							//插入结点的逆序数要加当前结点的count再加一
		if (node->right){										//递归右子树
			BST_insert(node->right, insert_node, count_small);
		}
		else{													//没有右子树直接插入
			node->right = insert_node;
		}
	}
}

class Solution {
public:
    std::vector<int> countSmaller(std::vector<int>& nums) {
    	std::vector<int> result;
    	std::vector<BSTNode *> node_vec;
    	std::vector<int> count;
    	for (int i = nums.size() - 1; i >= 0; i--){
    		node_vec.push_back(new BSTNode(nums[i]));
	    }
	    count.push_back(0);
	    for (int i = 1; i < node_vec.size(); i++){
	    	int count_small = 0;									//每个结点都有一个逆序数  
    		BST_insert(node_vec[0], node_vec[i], count_small);
    		count.push_back(count_small);
    	}
        for (int i = node_vec.size() - 1; i >= 0; i--){
        	delete node_vec[i];
        	result.push_back(count[i]);
        }
        return result;
    }
};

int main(){
	int test[] = {5, -7, 9, 1, 3, 5, -2, 1};
	std::vector<int> nums;
	for (int i = 0; i < 8; i++){
		nums.push_back(test[i]);
	}
	Solution solve;
	std::vector<int> result = solve.countSmaller(nums);
	for (int i = 0; i < result.size(); i++){
		printf("[%d]", result[i]);
	}
	printf("\n");
	return 0;
}

```

# 哈希表字符串  

拉链法解决冲突  

```cpp

#include <stdio.h>
#include <vector>

struct ListNode {
	int val;
	ListNode *next;
	ListNode(int x) : val(x), next(NULL) {}
};

int hash_func(int key, int table_len){
	return key % table_len;
}

void insert(ListNode *hash_table[], ListNode *node, int table_len){	//头插法插入node
	int hash_key = hash_func(node->val, table_len);					//hash_func求应该插入的位置
	node->next = hash_table[hash_key];
	hash_table[hash_key] = node;
}

bool search(ListNode *hash_table[], int value, int table_len){	//查找元素
	int hash_key = hash_func(value, table_len);					//先取到在链表数组中的位置hash_key
	ListNode *head = hash_table[hash_key];						//找到对应链表，并遍历
	while(head){
		if (head->val == value){
			return true;
		}
		head = head->next;
	}
	return false;
}

int main(){
	const int TABLE_LEN = 11;
	ListNode *hash_table[TABLE_LEN] = {0};				//ListNode指针数组保存所有头的地址
	std::vector<ListNode *> hash_node_vec;				//hash_node_vec存放所有ListNode结点的指针
	int test[8] = {1, 1, 4, 9, 20, 30, 150, 500};
	for (int i = 0; i < 8; i++){
		hash_node_vec.push_back(new ListNode(test[i]));	
	}	
	for (int i = 0; i < hash_node_vec.size(); i++){		//hash_node_vec数据插入到哈希表
		insert(hash_table, hash_node_vec[i], TABLE_LEN);
	}	
	printf("Hash table:\n");							//遍历打印
	for (int i = 0; i < TABLE_LEN; i++){
		printf("[%d]:", i);
		ListNode *head = hash_table[i];
		while(head){
			printf("->%d", head->val);
			head = head->next;
		}
		printf("\n");
	}
	printf("\n");	
	printf("Test search:\n");							//在表中查找0-9
	for (int i = 0; i < 10; i++){
		if (search(hash_table, i, TABLE_LEN)){
			printf("%d is in the hash table.\n");
		}
		else{
			printf("%d is not in the hash table.\n");
		}
	}
	return 0;
}

```

1. 最大回文长  

求一个字符串能重新组成的最大回文长度  

```cpp
#include <stdio.h>
#include <string>

class Solution {
public:
    int longestPalindrome(std::string s) {
    	int char_map[128] = {0};
    	int max_length = 0;
    	int flag = 0;
    	for (int i = 0; i < s.length(); i++){		//统计每个字符个数
	    	char_map[s[i]]++;						//s[i]char能直接转成int
	    }
	    for (int i = 0; i < 128; i++){
    		if (char_map[i] % 2 == 0){				//偶数就直接加
		    	max_length += char_map[i];
		    }
		    else{									//奇数要减去一个
    			max_length += char_map[i] - 1;
    			flag = 1;							//只要有奇数就可以放到所有数的中心一个
    		}
    	}
    	return max_length + flag;
    }
};

int main(){
	std::string s = "abccccddaa";
	Solution solve;
	printf("%d\n", solve.longestPalindrome(s));
	return 0;
}

```

2. 词语匹配  

pattern=“abba”,str=“dogcatcatdog”匹配.  
pattern=“abba”,str=“dogcatcatfish”不匹配.  

```cpp
#include <stdio.h>

#include <string>
#include <map>
class Solution {
public:
    bool wordPattern(std::string pattern, std::string str) {
    	std::map<std::string, char> word_map;				//string, char映射的map
    	char used[128] = {0};								//记录char是否用过，用过的就必须在map中查找到string
    	std::string word;
    	int pos = 0;										//遍历pattern中的char
    	str.push_back(' ');                                 //给最后也加上空格
    	for (int i = 0; i < str.length(); i++){
	    	if (str[i] == ' '){
	    		if (pos == pattern.length()){				//pos指到最后还有单词，pattern比str短，返回false
	    			return false;
		    	}
	    		if (word_map.find(word) == word_map.end()){	//map中没有word
	    			if (used[pattern[pos]]){				//但是pattern被用过，返回false
			    		return false;
			    	}
		    		word_map[word] = pattern[pos];			//没用过，就改变word对应的值
		    		used[pattern[pos]] = 1;
		    	}
		    	else{										//map中存在word，就判断两个pattern是否相等
	    			if (word_map[word] != pattern[pos]){
			    		return false;
			    	}
	    		}
	    		word = "";
	    		pos++;
	    	}
	    	else{
	    		word += str[i];
	    	}
	    }
	    if (pos != pattern.length()){						//pattern比str长
    		return false;
    	}
        return true;
    }
};

int main(){
	std::string pattern = "abba";
	std::string str = "dog cat cat dog";
	Solution solve;
	printf("%d\n", solve.wordPattern(pattern, str));
	return 0;
}

```

3. 使用相同字母的乱序词分成一组  

把字符串按字母顺序排序 eat变成aet

```cpp
#include <stdio.h>

#include <vector>
#include <string>
#include <map>
#include <algorithm>

class Solution {
public:
    std::vector<std::vector<std::string> > groupAnagrams(
			std::vector<std::string>& strs) {
		std::map<std::string, std::vector<std::string> > anagram;			//string存储aet，vector存储出现的不同顺序
		std::vector<std::vector<std::string> > result;						//返回的结果
		for (int i = 0; i < strs.size(); i++){
			std::string str = strs[i];
			std::sort(str.begin(), str.end());								//按字符顺序排序
			if (anagram.find(str) == anagram.end()){						//map中没找到键
				std::vector<std::string> item;								//创建一个新的vector，并存到anagram对应位置
				anagram[str] = item;
			}
			anagram[str].push_back(strs[i]);								//把strs[i]当前字符串加进去
		}
		std::map<std::string, std::vector<std::string> > ::iterator it;		
		for (it = anagram.begin(); it != anagram.end(); it++){
			result.push_back((*it).second);									//把所有map的second对应的vector，push到result
		}																	//result是二维数组
    	return result;				
    }
};

int main(){
	std::vector<std::string> strs;
	strs.push_back("eat");
	strs.push_back("tea");
	strs.push_back("tan");
	strs.push_back("ate");
	strs.push_back("nat");
	strs.push_back("bat");
	Solution solve;
	std::vector<std::vector<std::string> > result 
		= solve.groupAnagrams(strs);
	for (int i = 0; i < result.size(); i++){
		for (int j = 0; j < result[i].size(); j++){
			printf("[%s]", result[i][j].c_str());
		}
		printf("\n");
	}	
	return 0;
}

```

第二种，使用的字母次数统计，做为map的键  

```cpp
#include <stdio.h>
#include <vector>
#include <string>
#include <map>

void change_to_vector(std::string &str, std::vector<int> &vec){			//编码的函数
	for (int i = 0; i < 26; i++){										//二十六个初始为0
		vec.push_back(0);
	}
	for (int i = 0; i < str.length(); i++){								//统计每个字母次数
		vec[str[i]-'a']++;
	}
}

class Solution {
public:
    std::vector<std::vector<std::string> > groupAnagrams(
			std::vector<std::string>& strs) {
		std::map<std::vector<int>, std::vector<std::string> > anagram;
		std::vector<std::vector<std::string> > result;
		for (int i = 0; i < strs.size(); i++){
			std::vector<int> vec;
			change_to_vector(strs[i], vec);
			if (anagram.find(vec) == anagram.end()){					//编码没出现过
				std::vector<std::string> item;
				anagram[vec] = item;
			}
			anagram[vec].push_back(strs[i]);
		}
		std::map<std::vector<int>,
			std::vector<std::string> > ::iterator it;
		for (it = anagram.begin(); it != anagram.end(); it++){
			result.push_back((*it).second);
		}
    	return result;
    }
};

int main(){
	std::vector<std::string> strs;
	strs.push_back("eat");
	strs.push_back("tea");
	strs.push_back("tan");
	strs.push_back("ate");
	strs.push_back("nat");
	strs.push_back("bat");
	Solution solve;
	std::vector<std::vector<std::string> > result = solve.groupAnagrams(strs);
	for (int i = 0; i < result.size(); i++){
		for (int j = 0; j < result[i].size(); j++){
			printf("[%s]", result[i][j].c_str());
		}
		printf("\n");
	}	
	return 0;
}

```

4. 最长无重复子串  

i,begin两个指针向前移动  

```cpp
#include <stdio.h>
#include <string>
class Solution {
public:
    int lengthOfLongestSubstring(std::string s) {
    	int begin = 0;
    	int result = 0;
    	std::string word = "";
    	int char_map[128] = {0};							//记录字母使用次数
    	for (int i = 0; i < s.length(); i++){				
    		char_map[s[i]]++;								//次数加一
    		if (char_map[s[i]] == 1){						//字母只出现一次
		    	word += s[i];								//word加上当前字母
		    	if (result < word.length()){				//更新result
	    			result = word.length();
	    		}
		    }
		    else{											//字母出现不止一次了，移动begin到字母前一次出现位置后面
    			while(begin < i && char_map[s[i]] > 1){		//知道出现次数为一次
    				char_map[s[begin]]--;
		    		begin++;
		    	}
		    	word = "";									//word变成了新的begin到i
		    	for (int j = begin; j <= i; j++){
	    			word += s[j];
	    		}
    		}
	    }
    	return result;
    }
};

int main(){
	std::string s = "abcbadab";
	Solution solve;
	printf("%d\n", solve.lengthOfLongestSubstring(s));	
	return 0;
}

```

5. 重复DNA序列  

找长度为10，并且重复出现的序列  

```cpp
#include <stdio.h>
#include <vector>
#include <string>
#include <map>

class Solution {
public:
    std::vector<std::string> findRepeatedDnaSequences(std::string s) {
    	std::map<std::string, int> word_map;						//序列和出现次数的映射
    	std::vector<std::string> result;							
    	for (int i = 0; i < s.length(); i++){						
    		std::string word = s.substr(i, 10);						//每次截取十个字母
	    	if (word_map.find(word) != word_map.end()){				//统计序列出现次数
	    		word_map[word] += 1;
	    	}
	    	else{
	    		word_map[word] = 1;
	    	}
	    }
	    std::map<std::string, int> ::iterator it;					//遍历word_map，如果次数大于1，就添加该序列
	    for (it = word_map.begin(); it != word_map.end(); it++){
    		if (it->second > 1){
		    	result.push_back(it->first);
		    }
    	}
    	return result;
    }
};

int main(){
	std::string s = "AAAAACCCCCAAAAACCCCCCAAAAAGGGTTT";
	Solution solve;
	std::vector<std::string> result = solve.findRepeatedDnaSequences(s);
	for (int i = 0; i < result.size(); i++){
		printf("%s\n", result[i].c_str());
	}
	return 0;
}
```

第二种，使用位运算  

```cpp
#include <stdio.h>
#include <vector>
#include <string>

int g_hash_map[1048576] = {0};

std::string change_int_to_DNA(int DNA){
	static const char DNA_CHAR[] = {'A', 'C', 'G', 'T'};
	std::string str;		    	
	for (int i = 0; i < 10; i++){
		str += DNA_CHAR[DNA & 3];		//取出最后两位，转成char，int最后两位是str的最高一位
		DNA = DNA >> 2;
	}
	return str;
}

class Solution {
public:
    std::vector<std::string> findRepeatedDnaSequences(std::string s) {
    	std::vector<std::string> result;
		if (s.length() < 10){
	    	return result;
	    }
	    for (int i = 0; i < 1048576; i++){
	    	g_hash_map[i] = 0;
	    }
    	int char_map[128] = {0};
    	char_map['A'] = 0;
    	char_map['C'] = 1;
    	char_map['G'] = 2;
    	char_map['T'] = 3;    	
    	int key = 0;
    	for (int i = 9; i >= 0; i--){						//从9到0，取第一个十位，左移两位再加s[i]对应的编码char_map[s[i]]
	    	key = (key << 2) + char_map[s[i]];
	    }
    	g_hash_map[key] = 1;
    	for (int i = 10; i < s.length(); i++){				
    		key = key >> 2;									//最低的两位去掉，就是把开头的字母去掉
    		key = key | (char_map[s[i]] << 18);				//新的字母变成最高位
			g_hash_map[key]++;								//序列出现次数加一
	    }
	    for (int i = 0; i < 1048576; i++){					//遍历一遍，返回所有的重复序列
    		if (g_hash_map[i] > 1){
	    		result.push_back(change_int_to_DNA(i));
		    }
    	}
    	return result;
    }
};

int main(){
	std::string s = "AAAAACCCCCAAAAACCCCCCAAAAAGGGTTT";
	Solution solve;
	std::vector<std::string> result = solve.findRepeatedDnaSequences(s);
	for (int i = 0; i < result.size(); i++){
		printf("%s\n", result[i].c_str());
	}
	return 0;
}

```

6. 最小窗口子串  

例如:S=“ADOBECODEBANC”；T="ABC"包含T的子区间中，  
有“ADOBEC”，“CODEBA”，“BANC”等等；最小窗口区间是“BANC”  

如果当前begin指向的字符T中没出现，直接前移begin；  
如果begin指向的字符T中出现了，但是当前区间窗口中的该字符数量足够，向前移动begin，并更新map_s；  
否则不能移动begin，跳出检查。  

```cpp
#include <stdio.h>
#include <string>
#include <vector>
class Solution {
private:
	bool is_window_ok(int map_s[], int map_t[], std::vector<int> &vec_t){	//判断map_t是否是map_s的子集
    	for (int i = 0; i < vec_t.size(); i++){				//vec_t保存map_t所有的字符
    		if (map_s[vec_t[i]] < map_t[vec_t[i]]){			//map_s中对应字符的数量比map_t小，就返回false
		    	return false;
		    }
	    }
	    return true;
    }
public:
    std::string minWindow(std::string s, std::string t) {
        const int MAX_ARRAY_LEN = 128;
        int map_t[MAX_ARRAY_LEN] = {0};						//map_t存储t字符串中，所有字符对应次数
        int map_s[MAX_ARRAY_LEN] = {0};						//map_s存储s字符串中，所有字符对应次数
        std::vector<int> vec_t;								//vec_t保存map_t所有的字符，不用遍历128个了
        for (int i = 0; i < t.length(); i++){				
        	map_t[t[i]]++;
        }
        for (int i = 0; i < MAX_ARRAY_LEN; i++){
        	if (map_t[i] > 0){
	        	vec_t.push_back(i);
	        }
        }
        
        int window_begin = 0;
        std::string result;
        for (int i = 0; i < s.length(); i++){ 				//i不断向后滑动，遍历字符串s
        	map_s[s[i]]++;									//map_s存储s中，所有字符对应次数
        	while(window_begin < i){
        		char begin_ch = s[window_begin];			//当前begin指向的字符
	        	if (map_t[begin_ch] == 0){					//字符在t字符串中没出现过，begin直接向前
	        		window_begin++;
	        	}
	        	else if	(map_s[begin_ch] > map_t[begin_ch]){//当前区间窗口中的该字符数量足够，向前移动begin，并更新map_s
	        		map_s[begin_ch]--;
	        		window_begin++;
	        	}
	        	else{										//否则不能移动begin
	        		break;
	        	}
	        }
	        if (is_window_ok(map_s, map_t, vec_t)){			//满足条件，判断是否要更新结果
        		int new_window_len = i - window_begin + 1;
        		if (result == "" || result.length() > new_window_len){
		        	result = s.substr(window_begin, new_window_len);
				}
        	}
        }
        return result;
    }
};

int main(){
	
	Solution solve;
	std::string result = solve.minWindow("ADOBECODEBANC", "ABC");
	printf("%s\n", result.c_str());
	result = solve.minWindow("MADOBCCABEC", "ABCC");
	printf("%s\n", result.c_str());
	result = solve.minWindow("aa", "aa");
	printf("%s\n", result.c_str());
	
	return 0;	
}
```

# 搜索  

1. 搜索独立小岛  

DFS

```cpp
#include <stdio.h>

#include <vector>

void DFS(std::vector<std::vector<int> > &mark,
		 std::vector<std::vector<char> > &grid,
		 int x, int y){
	mark[x][y] = 1;													//标记为1
	static const int dx[] = {-1, 1, 0, 0};							//x和y的方向数组
	static const int dy[] = {0, 0, -1, 1};
	for (int i = 0; i < 4; i++){									//上下左右四个方向
		int newx = dx[i] + x;
		int newy = dy[i] + y;
		if (newx < 0 || newx >= mark.size() ||						//越界就什么也不做
			newy < 0 || newy >= mark[newx].size()){
			continue;
		}
		if (mark[newx][newy] == 0 && grid[newx][newy] == '1'){		//还未标记并且是岛屿土地
			DFS(mark, grid, newx, newy);
		}
	}
}

class Solution {
public:
    int numIslands(std::vector<std::vector<char> >& grid) {
    	int island_num = 0;
    	std::vector<std::vector<int> > mark;						//mark都初始化成0
    	for (int i = 0; i < grid.size(); i++){
    		mark.push_back(std::vector<int>());
	    	for (int j = 0; j < grid[i].size(); j++){
	    		mark[i].push_back(0);
	    	}
	    }
	    for (int i = 0; i < grid.size(); i++){						//遍历grid土地数组
	    	for (int j = 0; j < grid[i].size(); j++){
	    		if (mark[i][j] == 0 && grid[i][j] == '1'){			//两数组标记不同的时候
		    		DFS(mark, grid, i, j);							//搜索当前位置所有相连的土地并标记
		    		island_num++;									//岛屿数目加一
		    	}
	    	}
	    }
	    return island_num;
    }
};

int main(){
	std::vector<std::vector<char> > grid;
	char str[10][10] = {"11100", "11000", "00100", "00011"};
	for (int i = 0; i < 4; i++){
		grid.push_back(std::vector<char>());
		for (int j = 0; j < 5; j++){
			grid[i].push_back(str[i][j]);
		}
	}
	Solution solve;
	printf("%d\n", solve.numIslands(grid));
	return 0;
}

```

BFS  

```cpp
#include <stdio.h>

#include <vector>
#include <queue>

void BFS(std::vector<std::vector<int> > &mark,
		 std::vector<std::vector<char> > &grid,
		 int x, int y){
	static const int dx[] = {-1, 1, 0, 0};
	static const int dy[] = {0, 0, -1, 1};
	std::queue<std::pair<int, int> > Q;
	Q.push(std::make_pair(x, y));									//mark[i][j]，grid[i][j]不同，坐标就入队
	mark[x][y] = 1;
	while(!Q.empty()){												//出队
		x = Q.front().first;
		y = Q.front().second;
		Q.pop();
		for (int i = 0; i < 4; i++){								//上下左右四个方向
			int newx = dx[i] + x;
			int newy = dy[i] + y;
			if (newx < 0 || newx >= mark.size() ||					//越界了，什么都不做
				newy < 0 || newy >= mark[newx].size()){
				continue;
			}
			if (mark[newx][newy] == 0 && grid[newx][newy] == '1'){	//还未标记并且是岛屿土地
				Q.push(std::make_pair(newx, newy));
				mark[newx][newy] = 1;
			}
		}
	}
}

class Solution {
public:
    int numIslands(std::vector<std::vector<char> >& grid) {		//同上
    	int island_num = 0;
    	std::vector<std::vector<int> > mark;
    	for (int i = 0; i < grid.size(); i++){
    		mark.push_back(std::vector<int>());
	    	for (int j = 0; j < grid[i].size(); j++){
	    		mark[i].push_back(0);
	    	}
	    }
	    for (int i = 0; i < grid.size(); i++){
	    	for (int j = 0; j < grid[i].size(); j++){
	    		if (mark[i][j] == 0 && grid[i][j] == '1'){
		    		BFS(mark, grid, i, j);
		    		island_num++;
		    	}
	    	}
	    }
	    return island_num;
    }
};

int main(){
	std::vector<std::vector<char> > grid;
	char str[10][10] = {"11100", "11000", "00100", "00011"};
	for (int i = 0; i < 4; i++){
		grid.push_back(std::vector<char>());
		for (int j = 0; j < 5; j++){
			grid[i].push_back(str[i][j]);
		}
	}
	Solution solve;
	printf("%d\n", solve.numIslands(grid));
	return 0;
}

```

2. 词语阶梯a 判断能否转换    

图的宽度（广度）搜索  
例如:beginWord=“hit”；endWord=“cog”；wordList=["hot","dot","dog","lot","log","cog"]  
最短转换方式:"hit"->"hot"->"dot"->"dog"->"cog",结果为5。  

```cpp
#include <stdio.h>
#include <vector>
#include <string>
#include <map>
#include <set>
#include <queue>

bool connect(const std::string &word1, const std::string &word2){		//判断是否只有一个字母不同
	int cnt = 0;
	for (int i = 0; i < word1.length(); i++){
		if (word1[i] != word2[i]){
			cnt++;
		}
	}
	return cnt == 1;
}

void construct_graph(std::string &beginWord,
			std::vector<std::string>& wordList,
 	 		std::map<std::string, std::vector<std::string> > &graph){	//构造无向图，wordlist是邻接表的第一列，是map的键
	wordList.push_back(beginWord);
	for (int i = 0; i < wordList.size(); i++){
		graph[wordList[i]] = std::vector<std::string>();
	}
 	for (int i = 0; i < wordList.size(); i++){
 		for (int j = i + 1; j < wordList.size(); j++){
  			if (connect(wordList[i], wordList[j])){						//判断可以相连
   				graph[wordList[i]].push_back(wordList[j]);				//无向图，一条边需要添加两次
     			graph[wordList[j]].push_back(wordList[i]);
       		}
        } 
	}
}

int BFS_graph(std::string &beginWord, std::string &endWord,
			std::map<std::string, std::vector<std::string> > &graph){
	std::queue<std::pair<std::string, int> > Q;						//queue用于搜索结点
	std::set<std::string> visit;									//visit记录结点是否被访问
 	Q.push(std::make_pair(beginWord, 1));							//放入beginword
  	visit.insert(beginWord);
  	while(!Q.empty()){												//遍历队列
  		std::string node = Q.front().first;							//获取node，step
   		int step = Q.front().second;
    	Q.pop();
    	if (node == endWord){										//搜索到了返回步数
    		return step;
      	}
      	const std::vector<std::string> &neighbors = graph[node];	//node连接的所有结点
      	for (int i = 0; i < neighbors.size(); i++){
   			if (visit.find(neighbors[i]) == visit.end()){			//结点还没被访问过
   				Q.push(std::make_pair(neighbors[i], step + 1));
   				visit.insert(neighbors[i]);
    		}
        }
	}
 	return 0;
}

class Solution {
public:
    int ladderLength(std::string beginWord, std::string endWord,
				std::vector<std::string>& wordList) {
		std::map<std::string, std::vector<std::string> > graph;
		construct_graph(beginWord, wordList, graph);
  		return BFS_graph(beginWord, endWord, graph);
    }
};

int main(){	
	std::string beginWord = "hit";
	std::string endWord = "cog";
	std::vector<std::string> wordList;
	wordList.push_back("hot");
	wordList.push_back("dot");
	wordList.push_back("dog");
	wordList.push_back("lot");
	wordList.push_back("log");
	wordList.push_back("cog");
	Solution solve;
	int result = solve.ladderLength(beginWord, endWord, wordList);
	printf("result = %d\n", result);
	return 0;
}

```

词语阶梯b 搜索最短路径  

1.在宽度优先搜索时，如何保存宽度搜索时的路径？  
2.如果起始点与终点间有多条路径，如何将多条路径全部搜索出？  
3.在建立beginWord与wordList的连接时，若单词表中已包含beginWord，  
按照例2-a的方法建立图，会出现什么问题？  

1.将普通队列更换为vector实现队列，保存所有的搜索节点，即在pop节点时不会丢弃队头元素，只是移动front指针。  
2.在队列节点中增加该节点的前驱节点在队列中的下标信息，可通过该下标找到是队列中的哪个节点搜索到的当前节点。  


```cpp

#include <stdio.h>

#include <vector>
#include <string>
#include <map>

bool connect(std::string &word1, std::string &word2){		//判断是否只有一个字母不同
	int cnt = 0;
	for (int i = 0; i < word1.length(); i++){
		if (word1[i] != word2[i]){
			cnt++;
		}
	}
	return cnt == 1;
}

void construct_graph(std::string &beginWord,
			std::vector<std::string>& wordList,
 	 		std::map<std::string, std::vector<std::string> > &graph){
	int has_begin_word = 0;
	for (int i = 0; i < wordList.size(); i++){							//看wordList中是否有begin_word
		if (wordList[i] == beginWord){
 			has_begin_word = 1;
  		}
		graph[wordList[i]] = std::vector<std::string>();
	}
 	for (int i = 0; i < wordList.size(); i++){
 		for (int j = i + 1; j < wordList.size(); j++){
  			if (connect(wordList[i], wordList[j])){
   				graph[wordList[i]].push_back(wordList[j]);
     			graph[wordList[j]].push_back(wordList[i]);
       		}
       	}
       	if (has_begin_word == 0 && connect(beginWord, wordList[i])){	//如果没有，就beginWord放到wordList前面
	       	graph[beginWord].push_back(wordList[i]);
		}
   	}
}

struct Qitem{					//node、前驱、走过的步数
	std::string node;
	int parent_pos;
	int step;
	Qitem(std::string _node, int _parent_pos, int _step)
		: node(_node), parent_pos(_parent_pos), step(_step){
	}
};

void BFS_graph(std::string &beginWord, std::string &endWord,
			std::map<std::string, std::vector<std::string> > &graph,//graph邻接表，记录string可以跳到哪个单词
			std::vector<Qitem> &Q,									//将普通队列更换为vector实现队列
			std::vector<int> &end_word_pos){						//记录所有的end_word_pos传出去
	std::map<std::string, int> visit;								//单词和步数
	int min_step = 0;												//到达end_word的最小步数
 	Q.push_back(Qitem(beginWord.c_str(), -1, 1));					//beginWord入队
 	visit[beginWord] = 1;											//标记置为1
	int front = 0;													//队列指针，先指向队列头
	while(front != Q.size()){										//遍历队列
		const std::string &node = Q[front].node;					//当前的node
		int step = Q[front].step;									//当前的step
		if (min_step != 0 && step > min_step){						//当前的step大于遍历到end_word的step，跳出循环
			break;
		}
		if (node == endWord){										//到达endWord，更新min_step
			min_step = step;
			end_word_pos.push_back(front);
		}
		const std::vector<std::string> &neighbors = graph[node];	//取出所有可以跳到的单词，并遍历
		for (int i = 0; i < neighbors.size(); i++){					
			if (visit.find(neighbors[i]) == visit.end() ||			//如果没访问过，或者是访问过且步数为step+1，就是和上次访问次数相同
				visit[neighbors[i]] == step + 1){
				Q.push_back(Qitem(neighbors[i], front, step + 1));	//就要把这个neighbor放到队列中，记录了前驱结点
				visit[neighbors[i]] = step + 1;
			}
		}
 		front++;													//队列指针向前
	}

}

class Solution {
public:
    std::vector<std::vector<std::string> > findLadders(
		std::string beginWord, std::string endWord,
		std::vector<std::string>& wordList) {
		std::map<std::string, std::vector<std::string> > graph;
		construct_graph(beginWord, wordList, graph);				//构建邻接表
		std::vector<Qitem> Q;										//数组实现队列
        std::vector<int> end_word_pos;								//所有可行的end_word位置end_word_pos
		BFS_graph(beginWord, endWord, graph, Q, end_word_pos);		//搜索
        std::vector<std::vector<std::string> > result;				//获取搜索路径结果
        for (int i = 0; i < end_word_pos.size(); i++){				//遍历所有end_word_pos
        	int pos = end_word_pos[i];								
        	std::vector<std::string> path;							//路径
        	while(pos != -1){										//队列头的前驱是-1
        		path.push_back(Q[pos].node);						//放到路径数组中
				pos = Q[pos].parent_pos;							//pos指向前一个结点
        	}
        	result.push_back(std::vector<std::string>());			//result是二维数组
        	for (int j = path.size() - 1; j >= 0; j--){				//对应的path反转，存到result[i]
	        	result[i].push_back(path[j]);
	        }
        }
        return result;
    }
};

int main(){	
	std::string beginWord = "hit";
	std::string endWord = "cog";
	std::vector<std::string> wordList;
	wordList.push_back("hot");
	wordList.push_back("dot");
	wordList.push_back("dog");
	wordList.push_back("lot");
	wordList.push_back("log");
	wordList.push_back("cog");
	Solution solve;
	std::vector<std::vector<std::string> > result 
		= solve.findLadders(beginWord, endWord, wordList);	
	for (int i = 0; i < result.size(); i++){
		for (int j = 0; j < result[i].size(); j++){
			printf("[%s] ", result[i][j].c_str());
		}
		printf("\n");
	}
	return 0;
}


```

3. 火柴棍摆正方形  

已知一个数组，保存了n个(n<=15)火柴棍，问可否使用这n个火柴棍摆成1个正方形？  

优化1:n个火柴杆的总和对4取余需要为0，否则返回假。  
优化2:火柴杆按照从大到小的顺序排序，先尝试大的减少回溯可能。  
优化3:每次放置时，每条边上不可放置超过总和的1/4长度的火柴杆。  

```cpp
#include <stdio.h>
#include <vector>
#include <algorithm>

class Solution {
public:
    bool makesquare(std::vector<int>& nums) {
    	if (nums.size() < 4){						//火柴数不能小于4
	    	return false;
	    }
        int sum = 0;
        for (int i = 0; i < nums.size(); i++){
        	sum += nums[i];
        }
        if (sum % 4){								//和不是4的倍数
        	return false;
        }
        std::sort(nums.rbegin(), nums.rend());		//排序，先放长的火柴能减少回溯次数
        int bucket[4] = {0};
        return generate(0, nums, sum / 4, bucket);	
    }
private:
	bool generate(int i, std::vector<int>& nums, int target, int bucket[]){//target每条边需要的长度，bucket是实际的长度
		if (i >= nums.size()){
			return bucket[0] == target && bucket[1] == target 
				&& bucket[2] == target && bucket[3] == target;
		}
		for (int j = 0; j < 4; j++){					//每根火柴可以放到四个边
			if (bucket[j] + nums[i] > target){			//超出边长就不能放到第j边
				continue;
			}
			bucket[j] += nums[i];						//不超出，第i个棍就放到第j边 
			if (generate(i + 1, nums, target, bucket)){	//放下一根
				return true;
			}
			bucket[j] -= nums[i];						//回溯，不返回true说明i应该放到别的边
		}
		return false;									//说明i之前的放错了
	}
};

int main(){
	std::vector<int> nums;
	nums.push_back(1);
	nums.push_back(1);
	nums.push_back(2);
	nums.push_back(4);
	nums.push_back(3);
	nums.push_back(2);
	nums.push_back(3);	
	Solution solve;
	printf("%d\n", solve.makesquare(nums));	
	return 0;
}

```

位运算方法  

```cpp
#include <stdio.h>
#include <vector>
class Solution {
public:
    bool makesquare(std::vector<int>& nums) {
    	if (nums.size() < 4){
	    	return false;
	    }
        int sum = 0;
        for (int i = 0; i < nums.size(); i++){
        	sum += nums[i];
        }
        if (sum % 4){
        	return false;
        }
        int target = sum / 4;        
        std::vector<int> ok_subset;
        std::vector<int> ok_half;
        int all = 1 << nums.size();							//总共有多少种i，2的size次方个
        for (int i = 0; i < all; i++){						//i是每个火柴棍是否出现的编码
        	int sum = 0;
        	for (int j = 0; j < nums.size(); j++) {			//判断第j个火柴是否在某个编码i中出现
        		if (i & (1 << j)){							//i & (1 << j)为真说明当前i有第j个火柴，把j加到sum中
        			sum += nums[j];
	    		}
            }
            if (sum == target){								//编码中选用火柴长度和是正方形边长target
            	ok_subset.push_back(i);						//编码i就放到ok_subset
            }
        }
        for (int i = 0; i < ok_subset.size(); i++){			//这样的双重循环相当于两两比较，比较编码
        	for (int j = i + 1; j < ok_subset.size(); j++){
	        	if ((ok_subset[i] & ok_subset[j]) == 0){	//没有选择任何同一个火柴，两个编码合并放到ok_half
	        		ok_half.push_back(ok_subset[i] | ok_subset[j]);
	        	}
	        }
        }
        for (int i = 0; i < ok_half.size(); i++){			//两两比较ok_half里面的编码
        	for (int j = i + 1; j < ok_half.size(); j++){
	        	if ((ok_half[i] & ok_half[j]) == 0){		//没有选择任何同一个火柴，就返回真
					return true;
	        	}
	        }
        }
        return false;
    }
};

int main(){
	std::vector<int> nums;
	nums.push_back(1);
	nums.push_back(1);
	nums.push_back(2);
	nums.push_back(4);
	nums.push_back(3);
	nums.push_back(2);
	nums.push_back(3);	
	Solution solve;
	printf("%d\n", solve.makesquare(nums));	
	return 0;
}
```

4. 积水问题  

```cpp
#include <stdio.h>
#include <vector>
#include <queue>
struct Qitem{
	int x;
	int y;
	int h;
	Qitem(int _x, int _y, int _h) : 								//坐标和高度
		x(_x), y(_y), h(_h){
	}
};
struct cmp{															//这样比较可以构造最小值堆
	bool operator()(const Qitem &a, const Qitem &b){
		return a.h > b.h;											//比较两个qitem高度
	}
};

class Solution {
public:
    int trapRainWater(std::vector<std::vector<int> >& heightMap) {  //二维数组保存所有的高度height
    	std::priority_queue<Qitem, std::vector<Qitem>, cmp> Q;
		if (heightMap.size() < 3 || heightMap[0].size() < 3){		//行或者列小于3无法积水
			return 0;
		}
		int row = heightMap.size();									//获取行数、列数
		int column = heightMap[0].size();
		std::vector<std::vector<int> > mark;						//构造mark二维数组判断是否被搜索
		for (int i = 0; i < row; i++){
			mark.push_back(std::vector<int> ());
			for (int j = 0; j < column; j++){
				mark[i].push_back(0);
			}
		}
		for (int i = 0; i < row; i++){								//第一列和最后一列放到优先队列
			Q.push(Qitem(i, 0, heightMap[i][0]));
			mark[i][0] = 1;
			Q.push(Qitem(i, column - 1, heightMap[i][column - 1]));
			mark[i][column - 1] = 1;
		}
		for (int i = 1; i < column - 1; i++){						//第一行和最后一行放到有限队列
			Q.push(Qitem(0, i, heightMap[0][i]));
			mark[0][i] = 1;
			Q.push(Qitem(row - 1, i, heightMap[row - 1][i]));
			mark[row - 1][i] = 1;
		}
		static const int dx[] = {-1, 1, 0, 0};						//方向数组，上下左右四个方向
		static const int dy[] = {0, 0, -1, 1};
		int result = 0;												//积水的结果
		while(!Q.empty()){											//队列头高度小的元素出队
			int x = Q.top().x;
			int y = Q.top().y;
			int h = Q.top().h;
			Q.pop();
			for (int i = 0; i < 4; i++){							//四个方向
				int newx = x + dx[i];
				int newy = y + dy[i];
				if(newx < 0 || newx >=row ||						//出界了，或者已经搜索过了
				   newy < 0 || newy >= column || mark[newx][newy]){
					continue;
				}
				if (h > heightMap[newx][newy]){						//当前的比新搜索到的结点高度大
					result += h - heightMap[newx][newy];			//可以积水，更新结果和高度
					heightMap[newx][newy] = h;
				}
				Q.push(Qitem(newx, newy, heightMap[newx][newy]));	//新结点入队并且更新标记
				mark[newx][newy] = 1;
			}
		}
		return result;		//队列空了，就返回结果
    }
};

int main(){
	int test[][10] = {
		{1, 4, 3, 1, 3, 2},
		{3, 2, 1, 3, 2, 4},
		{2, 3, 3, 2, 3, 1}
	};
	std::vector<std::vector<int> > heightMap;
	for (int i = 0; i < 3; i++){
		heightMap.push_back(std::vector<int> ());
		for (int j = 0; j < 6; j++){
			heightMap[i].push_back(test[i][j]);
		}
	}
	Solution solve;
	printf("%d\n", solve.trapRainWater(heightMap));	
	return 0;
}
```