我们还是以题目中的示例数据为例。

- 后序: `8 3 0 9 6 1 4 7 5 2`
- 中序: `8 3 4 0 1 9 6 2 7 5`

我们观察后序遍历的最后一个值，它是 `2` ，这直接说明了整棵树的根结点就是 `2` 。我们现在再在中序遍历中找到 `2` ，并发现它的左边是 `8 3 4 0 1 9 6` ，一共是 7 个数，而右边是 `7 5` ，一共是 2 个数。而且这两小段正是左右两个子树的中序遍历。这就说明，左子树一定有 7 个结点，右子树一定有 2 个结点。我们回过头来看后序遍历序列，它的前 7 个数是 `8 3 0 9 6 1 4` ，接下来 2 个数是 `7 5` 。而且这两小段正是左右两个子树的后序遍历。

我们把刚刚得出的知识列出来：

- 根结点：就是 `2`
- 左子树：
    - 后序遍历序列是 `8 3 0 9 6 1 4`
    - 中序遍历序列是 `8 3 4 0 1 9 6`
- 右子树：
    - 后序遍历序列是 `7 5`
    - 中序遍历序列是 `7 5`

我们发现，大问题分解成了两个小问题，而这两个小问题的样子，其实刚好就和一开始的大问题是一样的。我们可以重新从头开始，用刚才的方法先后对左、右两个子树做分析。

这样不断拆分下去，最终所有的问题便都能分解成只有 `n == 1` 或者 `n == 0` 的情况了。而解决这么小的问题自然相当简单，前者表示树只有一个根结点，也就是遍历序列里唯一那一个值；后者表示树里没有结点，是个空树。

我们据此便能够写出分治算法构造出这棵树。如下是伪代码：

```text
构造二叉树(后序序列, 中序序列)
{
    if (length(后序序列) == 0) {
        return 空树;
    }
    // 其实这个分支也可以不写
    if (length(后序序列) == 1) {
        return 二叉树结点(空树, 后序序列[0], 空树);
    }

    根 = 后序序列[n - 1];
    根的索引 = 根在中序序列中的索引值;

    // arr[a .. b] 表示从 arr[a] 到 arr[b] 这一段，包含 arr[b]。
    左子树中序 = 中序序列[0 .. 根的索引 - 1];
    右子树中序 = 中序序列[根的索引 + 1 .. n - 1];

    左子树后序 = 后序序列[0 .. 根的索引 - 1];
    右子树后序 = 后序序列[根的索引 .. n - 2];

    左子树 = 构造二叉树(左子树后序, 左子树中序);
    右子树 = 构造二叉树(右子树后序, 右子树中序);
    return 二叉树结点(左子树, 根, 右子树);
}
```

除了用一个索引对来表示子数组而避免真的几乎复制整个数组外，这里还有一个优化点。我们有一个“查找根在中序序列中的索引值”操作，如果每次都线性查找，那么最坏情况下代码时间会退化到平方时间 $\mathcal{O}(n^{2})$ 。解决起来也不难，提前用散列表记录每一个值的位置即可，这样最坏情况也只是线性时间 $\mathcal{O}(n)$ 。

C 语言标准库没有散列表。不过有一个好消息，这道题的数据范围足够小，结点值都在 $[-1000, 1000]$ 之间。因此可以直接用数组充当散列表，数组大小 `2001` 即可。

接下来的问题是如何对一个二叉树做层序遍历。这等同于做一个广度优先搜索算法，需要使用队列这一数据结构。C 语言标准库当然也没有队列。不过，因为 $n \le 1000$ ，所以也能用数组模拟。甚至不需要搞循环队列。

### 完整代码

C：

```c
#include <assert.h>
#include <stddef.h>
#include <stdio.h>

typedef struct tree_node_s tree_node_t;

struct tree_node_s {
    int value;
    tree_node_t* left;
    tree_node_t* right;
};

tree_node_t nodes[1000];
size_t node_count = 0;

tree_node_t* make_node(int value, tree_node_t* left, tree_node_t* right)
{
    tree_node_t* const node = &nodes[node_count];
    ++node_count;
    node->value = value;
    node->left = left;
    node->right = right;
    return node;
}

// 从值到中序下标的映射表。用的时候记得加 1000。
size_t in_positions[2001];

// post 从下标 post_begin 开始、长为 n 的子数组是当前要处理的。
// in 从下标 in_begin 开始、长为 n 的子数组是当前要处理的。
tree_node_t* tree_from_post_in_internal(
    const int* post, size_t post_begin,
    const int* in, size_t in_begin,
    size_t n)
{
    if (n == 0) {
        // 空树。
        return NULL;
    }
    const size_t post_end = post_begin + n;
    const size_t in_end = in_begin + n;
    const int root = post[post_end - 1];
    const size_t root_i_abs = in_positions[root + 1000];
    assert(in_begin <= root_i_abs);
    assert(root_i_abs < in_end);
    const size_t root_i = root_i_abs - in_begin;
    return make_node(
        root,
        tree_from_post_in_internal(
            post, post_begin,
            in, in_begin,
            root_i
        ),
        tree_from_post_in_internal(
            post, post_begin + root_i,
            in, in_begin + root_i + 1,
            n - 1 - root_i
        )
    );
}

tree_node_t* tree_from_post_in(const int* post, const int* in, size_t n)
{
    // 事先把每个值在中序中的位置记下来，避免每次递归都线性查找
    for (size_t i = 0; i < n; ++i) {
        in_positions[in[i] + 1000] = i;
    }
    return tree_from_post_in_internal(post, 0, in, 0, n);
}

void print_pre_order(const tree_node_t* node)
{
    if (node == NULL) {
        return;
    }
    printf("%d ", node->value);
    print_pre_order(node->left);
    print_pre_order(node->right);
}

void print_level_order(const tree_node_t* root)
{
    // 用数组模拟队列。n <= 1000，所以长度 1000 的数组就够用了。
    // 甚至不用实现循环队列了。
    const tree_node_t* queue[1000];
    size_t head = 0;
    size_t tail = 0;
    queue[tail++] = root;
    while (head < tail) {
        const tree_node_t* const node = queue[head++];
        printf("%d ", node->value);
        if (node->left) {
            queue[tail++] = node->left;
        }
        if (node->right) {
            queue[tail++] = node->right;
        }
    }
}

int main(void)
{
    size_t n;
    scanf("%zu", &n);
    int post[1000];
    int in[1000];
    for (size_t i = 0; i < n; ++i) {
        scanf("%d", &post[i]);
    }
    for (size_t i = 0; i < n; ++i) {
        scanf("%d", &in[i]);
    }
    tree_node_t* const root = tree_from_post_in(post, in, n);
    print_pre_order(root);
    putchar('\n');
    print_level_order(root);
    putchar('\n');
}
```

C++：

```cpp
#include <cassert>
#include <iostream>
#include <memory>
#include <queue>
#include <unordered_map>
#include <utility>
#include <vector>

// T 是二叉树结点所存储的元素类型。
template <typename T>
struct TreeNode {
private:
    using vector_length_t = typename std::vector<T>::size_type;

public:
    T value;
    std::unique_ptr<TreeNode> left;
    std::unique_ptr<TreeNode> right;

    void print_pre_order(std::ostream& os) const
    {
        os << value << ' ';
        if (left) {
            left->print_pre_order(os);
        }
        if (right) {
            right->print_pre_order(os);
        }
    }

    void print_level_order(std::ostream& os) const
    {
        std::queue<const TreeNode*> q;
        q.emplace(this);

        while (!(q.empty())) {
            const TreeNode* const node = q.front();
            q.pop();
            os << node->value << ' ';
            if (node->left) {
                q.emplace(node->left.get());
            }
            if (node->right) {
                q.emplace(node->right.get());
            }
        }
    }

    static std::unique_ptr<TreeNode> from_post_in(
        const std::vector<T>& post, const std::vector<T>& in)
    {
        assert(post.size() == in.size());
        // 事先把每个值在中序中的位置记下来，避免每次递归都线性查找
        std::unordered_map<T, vector_length_t> in_positions{};
        for (vector_length_t i = 0; i < post.size(); ++i) {
            in_positions[in[i]] = i;
        }
        return from_post_in_internal(post, 0, in, 0, in.size(), in_positions);
    }

private:
    // post 从下标 post_begin 开始、长为 n 的子数组是当前要处理的。
    // in 从下标 in_begin 开始、长为 n 的子数组是当前要处理的。
    // C++20 起，有 std::span 可以用。
    static std::unique_ptr<TreeNode> from_post_in_internal(
        const std::vector<T>& post, vector_length_t post_begin,
        const std::vector<T>& in, vector_length_t in_begin,
        vector_length_t n,
        const std::unordered_map<T, vector_length_t>& in_positions)
    {
        if (n == 0) {
            // 空树。
            return nullptr;
        }
        const vector_length_t post_end = post_begin + n;
        const vector_length_t in_end = in_begin + n;
        const T root = post[post_end - 1];
        const auto root_i_abs_it = in_positions.find(root);
        assert(root_i_abs_it != in_positions.end());
        const vector_length_t root_i_abs = root_i_abs_it->second;
        assert(in_begin <= root_i_abs);
        assert(root_i_abs < in_end);
        const vector_length_t root_i = root_i_abs - in_begin;
        auto left = from_post_in_internal(
            post, post_begin,
            in, in_begin,
            root_i,
            in_positions
        );
        auto right = from_post_in_internal(
            post, post_begin + root_i,
            in, in_begin + root_i + 1,
            n - 1 - root_i,
            in_positions
        );
        // C++11 还没有 std::make_unique 。这里用 new 构造。
        return std::unique_ptr<TreeNode>(
            new TreeNode{
                root,
                std::move(left),
                std::move(right),
            }
        );
    }
};

int main()
{
    int n;
    std::cin >> n;
    std::vector<int> post(n);
    std::vector<int> in(n);
    for (int i = 0; i < n; ++i) {
        std::cin >> post[i];
    }
    for (int i = 0; i < n; ++i) {
        std::cin >> in[i];
    }
    const auto root = TreeNode<int>::from_post_in(post, in);
    root->print_pre_order(std::cout);
    std::cout << '\n';
    root->print_level_order(std::cout);
    std::cout << '\n';
}
```
