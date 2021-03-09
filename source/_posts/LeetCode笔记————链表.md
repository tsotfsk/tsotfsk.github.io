---
 title: LeetCode笔记————链表
date: 2021-01-19 22:06:09
categories: LeetCode笔记
tags:
 - 优先队列
 - 链表
 - 哈希表
---

>由于刷题时间比较充足，所以想用C++刷下leetcode，但是自己语法真的已经忘到不知道哪去了，所以就要补很多东西，做的时候也是各种头疼，可能做法都非常丑陋，还请见谅。

## 2. 两数相加

**原题:**  [2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

**难度:** 中等

**题目分析:** 题目很好理解，主要的难点就是要解决的其实就是列表长度不一致，以及最高位也可能进位的问题。

**补充知识:**

链表的头节点前加一个前驱节点会有助于做很多判断，尤其是在删除head节点的时候。

**我的代码:**

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode *curptr = new ListNode(0);
        ListNode *re_ptr = curptr;
        int cur_l1_value = 0;
        int cur_l2_value = 0;
        bool next_plus = false;
        while (l1 != nullptr || l2 != nullptr) {
            if (l1 != nullptr) {
                cur_l1_value = l1_ptr->val;
                l1 = l1->next;
            } else {
                cur_l1_value = 0;
                l1 = nullptr;
            }
            if (l2_ptr != nullptr) {
                cur_l2_value = l2_ptr->val;
                l2_ptr = l2_ptr->next;
            } else {
                cur_l2_value = 0;
                l2_ptr = nullptr;
            }

            next_plus = (cur_l1_value + cur_l2_value + curptr->val) >= 10;
            curptr->val = (cur_l1_value + cur_l2_value + curptr->val) % 10;

            // 当都还没到最高位或者 下一位有进位的时候要构造新的node
            if ((l1_ptr != nullptr || l2_ptr != nullptr) || next_plus) {
                curptr->next = new ListNode(int(next_plus));
                curptr = curptr->next;
            }
        }
    return re_ptr;
    }
};
```

## 19. 删除链表的倒数第 N 个结点

**题目:** [19. 删除链表的倒数第 N 个结点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

**难度:** 中等

**题目分析:** 这个没啥好说的，快慢指针就可以了。

**我的代码:**

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
    		 // 构造一个前置node，避免了node在起始节点的判断
        ListNode * tmp_head = new ListNode(0, head);
        ListNode *fast_ptr = tmp_head;
        ListNode *slow_ptr = tmp_head;
        for (int i = 0; i < n; i++) {
            fast_ptr = fast_ptr->next;
        }
        while(fast_ptr->next) {
            fast_ptr = fast_ptr->next;
            slow_ptr = slow_ptr->next;
        }
        ListNode *tag_ptr = slow_ptr->next;
        slow_ptr->next = tag_ptr->next;
        tag_ptr->next = nullptr;         
        return tmp_head->next;
    }
};
```

## 21. 合并两个有序链表

题目: [21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

**难度:** 中等

**题目分析:** 没啥说的，很简单。注意添加个前置节点，可以方便判断很多特殊情况。

**我的代码:**

```cpp
class Solution {
public:

    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode *cur_ptr = new ListNode(0);
        ListNode *head = cur_ptr;
        while(l1 && l2) {
            if (l1->val >= l2->val) {
                cur_ptr->next = l2;
                l2 = l2->next;
            } else {
                cur_ptr->next = l1;
                l1 = l1->next;
            }
            cur_ptr = cur_ptr->next;
        }
        if (l1) {
            cur_ptr->next = l1;
        }
        if (l2) {
            cur_ptr->next = l2;
        }
        return head->next;
    }
};
```

## 23. 合并K个升序链表

题目：[23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

**难度:** 难

**题目分析:** 虽然难度是难，但是这道题感觉暴力解还是可以的，我的初始做法比较傻逼。主要还是对各种数据结构不太熟悉，比如这道题我的大致想法还是可以的，但是我是用了一个非常丑陋的方式。原题解给出的有一种最优队列的想法，其实和我的比较接近，只是我用的是vector，就会产生很多无用的判断。本质上来说是维护一个队列，总记录各个链表尚未合并的首元素，然后在这些元素中选择一个最小值即可。

**补充知识:** 

优先队列

**我的代码:**

```cpp
class Solution {
public:

    vector<int> find_min(vector<ListNode*> &ptr_list) {
        int min = 10000;
        vector<int> result {0,  1};
        // int idx = 0;
        // int all_end = true;
        for (int i = 0; i < ptr_list.size(); i++) {
            if (ptr_list[i] != nullptr) {
                result[1] = 0;
                if (ptr_list[i]->val < min) {
                    min = ptr_list[i]->val;
                    result[0] = i;    
                }
            }
        }
        return result;
    }

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        ListNode* cur_ptr = new ListNode(0);
        ListNode* head = cur_ptr;
        vector<int> result;

        while (true) {
            result = find_min(lists);
            if (result[1]) {
                break;
            }
            cur_ptr->next = lists[result[0]];
            lists[result[0]] = lists[result[0]]->next;
            cur_ptr = cur_ptr->next;
        }
        return head->next;
    }
};
```

**更优的代码**

```cpp
class Solution {
public:
    struct Status {
        int val;
        ListNode *ptr;
        bool operator < (const Status &rhs) const {
            return val > rhs.val;
        }
    };

    priority_queue <Status> q;

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        for (auto node: lists) {
            if (node) q.push({node->val, node});
        }
        ListNode head, *tail = &head;
        while (!q.empty()) {
            auto f = q.top(); q.pop();
            tail->next = f.ptr; 
            tail = tail->next;
            if (f.ptr->next) q.push({f.ptr->next->val, f.ptr->next});
        }
        return head.next;
    }
};
```

## 148. 排序链表

**题目:** [148. 排序链表](https://leetcode-cn.com/problems/sort-list/)

**难度:** 中等

**题目分析: ** 我傻逼的写了个选择排序，然后好不容易调通了，给我来个超出时间限制，难受啊。

**补充知识:** 链表我很容易绕晕，尤其是next乱七八糟的做交换。比如两个节点之间如何做交换，我整整研究了一下午，那是真滴头疼，好不容易vscode把代码调通了，结果运行时间过不了。

**我的代码:**

```cpp
class Solution {
public:

    // void print(ListNode* head) {
    //     while(head != nullptr) {
    //         cout << head->val;
    //         head = head->next; 
    //     }
    //     cout << endl;
    // };
		 // l1 和 l2 分别是交换的目标节点的前驱节点
    void swap(ListNode* l1, ListNode* l2) {
        if(l1->next == l2) {
            // 声明两个变量是因为交换两个节点实际涉及到四个节点，所以把另外两个节点也拿出来
            // l1->l2->tmp1->tmp2
            // 那么要交换的就是 l2 以及 tmp1
            // 因为有三条边所以有三条赋值
            ListNode* tmp1 = l2->next;
            ListNode* tmp2 = tmp1->next;
            l1->next = tmp1;
            tmp1->next = l2;
            l2->next = tmp2;
        } else {
            // 声明四个变量是因为交换两个节点实际涉及到四个节点，所以把另外两个节点也拿出来
            // l1->tmp1->tmp2->...->l2->tmp3->tmp4
            // 那么要交换的就是 tmp1 以及 tmp3
            // 其中四条边需要重定向(tmp2->以及->l2不受影响，因此即使tmp2就是l2也是无所谓的)
            ListNode* tmp1 = l1->next;
            ListNode* tmp2 = l1->next->next;

            ListNode* tmp3 = l2->next;
            ListNode* tmp4 = l2->next->next;

            l1->next = tmp3;
            tmp3->next = tmp2;
            l2->next = tmp1;
            tmp1->next = tmp4;

        }

    };

    ListNode* sortList(ListNode* head) {
        ListNode * ptr = new ListNode(0, head);
        ListNode * result = ptr;
        ListNode * cptr = nullptr;
        int min = 0;
        ListNode * mptr = nullptr;
        while(ptr->next != nullptr) {
            cptr = ptr->next;
            mptr = nullptr;
            min = cptr->val;
            while (cptr->next != nullptr) {
                if (cptr->next->val < min) {
                    min = cptr->next->val;
                    mptr = cptr;
                }
                cptr = cptr->next;
            }
            if (mptr != nullptr) {
                swap(ptr, mptr);
            }
            ptr = ptr->next;
        }
        return result->next;
    }
};
```

**更优的解法: **



