# getMedian 算法记录

## 代码

```cpp
ListNode* getMedian(ListNode* left, ListNode* right) {
    ListNode* fast = left;
    ListNode* slow = left;
    while (fast != right && fast->next != right) {
        fast = fast->next;
        fast = fast->next;
        slow = slow->next;
    }
    return slow;
}
```

## 简要说明

这个函数用于在 `left` 到 `right` 这一段链表中寻找中间节点，采用的是快慢指针写法。

- `slow` 每次走一步，`fast` 每次走两步。
- 当 `fast` 走到 `right`，或者 `fast->next` 走到 `right` 时停止。
- 此时 `slow` 所在的位置就是这段区间的中点。

这里的 `right` 更适合理解为边界控制节点，也就是循环停止的判断依据。

- 如果把它看成半开区间 `[left, right)`，那么 `right` 表示区间外的边界。
- 如果把 `right` 看成最后一个有效节点，也可以理解成在 `[left, right]` 上找中点。
- 在这种理解下，若区间长度为偶数，这段代码通常返回左中点。

## 适用场景

- 链表二分查找
- 有序链表转平衡二叉搜索树
- 归并/分治时寻找链表中点

## 注意点

- `right` 的核心作用是控制停止条件，而不一定非要死板理解成“是否属于区间”。
- 这个写法既常见于半开区间 `[left, right)`，也可以用于理解为包含右边界的写法。
- 如果按“`right` 是有效右端点”来理解，那么偶数长度时通常返回左中点。

## 补充：有序链表转平衡 BST

下面这段代码也是链表相关题里的经典做法，它的目标是把一个**升序链表**转换成一棵**高度平衡二叉搜索树**。

```cpp
class Solution {
public:
    TreeNode* sortedListToBST(ListNode* head) {
        auto phead = head;
        int len = getListLen(head);

        return buildTreeByMidTravel(phead, 0, len - 1);
    }

    int getListLen(ListNode* head){
        ListNode* p = head;
        int len = 0;
        while (p) {
            ++len;
            p = p->next;
        }
        return len;
    }

    TreeNode* buildTreeByMidTravel(ListNode*& head, int left, int right) {
        if (left > right) {
            return nullptr;
        }

        int mid = (left + right) >> 1;
        TreeNode* root = new TreeNode();
        root->left = buildTreeByMidTravel(head, left, mid - 1);
        root->val = head->val;
        head = head->next;
        root->right = buildTreeByMidTravel(head, mid + 1, right);

        return root;
    }
};
```

## 作用

这套写法的作用是：

- 把有序链表按二叉搜索树的中序顺序“映射”成一棵树。
- 构造出来的树满足 BST 性质：`左子树 < 根节点 < 右子树`。
- 由于每次都按中间位置划分区间，所以整棵树尽量平衡。

## 核心思想

它不是先在链表里反复找中点，而是借助 **中序遍历的构造顺序** 来完成建树。

中序遍历顺序是：

- 先构造左子树
- 再处理根节点
- 最后构造右子树

而升序链表本身的节点顺序，刚好就是 BST 的中序遍历结果。因此可以这样做：

1. 先根据链表长度确定当前子树对应的下标区间 `[left, right]`。
2. 递归构造左半部分，让它成为左子树。
3. 左子树建完时，`head` 正好走到当前根节点应该对应的位置。
4. 用 `head->val` 给根节点赋值，然后把 `head` 后移一位。
5. 继续递归构造右子树。

这样就相当于一边做“中序建树”，一边顺序消费链表节点。

## 为什么 `ListNode*& head` 很关键

这里的 `head` 不是普通传值，而是**引用传递**，目的是让递归过程中对 `head` 的移动能够同步影响到外层。

- 左子树构造完成后，`head` 已经移动到了当前根节点位置。
- 根节点赋值后，`head = head->next`，继续指向右子树的起始节点。
- 如果这里不用引用，而是值传递，那么递归里移动的只是局部副本，外层指针位置不会更新，整棵树就没法按正确顺序赋值。

可以把它理解成一个“全局游标”，始终指向当前还没使用的链表节点。

## 执行过程理解

假设链表为：

```text
-10 -> -3 -> 0 -> 5 -> 9
```

长度为 `5`，初始调用：

```cpp
buildTreeByMidTravel(head, 0, 4)
```

递归的大致过程是：

- 先建 `[0, 1]` 对应的左子树
- 再建 `[0, -1]`，返回空
- 此时 `head` 指向 `-10`，把它填到当前节点
- 再建 `[1, 1]`
- 左子树整体完成后，`head` 会移动到 `0`
- 用 `0` 作为整棵树的根
- 再去构造 `[3, 4]` 的右子树

最终生成的树就是一棵平衡 BST。

## 复杂度

- 时间复杂度：`O(n)`
  每个链表节点只会被访问一次。
- 空间复杂度：`O(log n)`
  主要是递归调用栈，平衡情况下树高约为 `log n`。

## 和“快慢指针找中点”方案的区别

有序链表转 BST 还有一种常见写法：每次用快慢指针找到中点，然后中点作为根节点，再递归处理左右两段链表。

两者区别：

- 快慢指针找中点：思路直观，但每层递归都要重新找中点，时间复杂度通常是 `O(n log n)`。
- 当前这种中序构造法：先求长度，再按下标区间递归，链表只顺序扫描一次，时间复杂度是 `O(n)`。

所以这段代码的优势就在于：**不需要频繁找中点，效率更高。**
