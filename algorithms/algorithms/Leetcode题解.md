# Leetcode题解

- [Leetcode题解](#Leetcode题解)
  - [多路归并](#多路归并)
  - [分治](#分治)


## 多路归并

### 23.合并k个升序链表
[leetcode.23](https://leetcode.cn/problems/merge-k-sorted-lists/description/)  
- 思路：小根堆（优先队列法）把每个有序序列的第一个元素放进小根堆；堆中除了值，还要记录：属于哪个数组、当前下标，在c++中使用priority_queue。
每次弹出堆顶最小值，放入结果；取出这个元素所属数组的下一个元素，推入堆；直到堆为空。  
- 复杂度：堆大小最多 k，每次堆操作 \(O(\log k)\)，一共 N 次操作： \(O(N\log k)\)
- 代码
~~~cpp
class Solution {
public:
    struct cmp{
        bool operator()(ListNode* a,ListNode* b){
            return a -> val > b -> val;
        }
    };

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        priority_queue<ListNode*,vector<ListNode*>,cmp> que;

        for(auto c : lists){
            if(c) que.push(c);
        }

        ListNode dummy;
        ListNode* cur = &dummy;
        
        while(!que.empty()){
            ListNode* Min = que.top();
            que.pop();
            cur -> next = Min;
            if(Min -> next != nullptr) que.push(Min -> next);
            cur = cur -> next;
        }
        return dummy.next;
    }
};
~~~



