像这样，如果一道题要求我们每一步该干什么，而我们确实让代码一步步照着它说的做就可以解决它，那么这道题就是“模拟题”。

模拟题会考验你的细致程度和耐心程度。

对于这道题，复杂的规则其实可以用比较简单的说法统一描述：

- 从第一行的中间开始填。
- 一直往右上方向走，一边走一边填数。如果碰到了边界，那就从另一边出来。就像吃豆人小游戏一样：吃豆人如果走向屏幕右侧边界，那它就会从左侧同一高度重新出现。对于其他方向的边界也是一样的。
- 如果要往右上填数时发现右上这个地方已经填过了，那么这一步就不往右上走，而是往下走。如果碰到边界，也是一样要循环。

这样的“边界循环”和模算术有着很好的关系。我们引入数论中的模运算。 $x \bmod y$ 表示 $x$ 除以 $y$ 所得的余数，要求 $y > 0$ 。结果一定落在区间 $[0, y)$ 之中。

我们以一维的格子为例。如果有 $10$ 个格子，编号为 $0, 1, 2, \ldots, 9$ ，现在有一个小人站在编号为 $7$ 的格子上。小人可以向左、向右走，在超出边界时会从另一边出现。现在想知道，如果小人往右走了 $26$ 格 ，那么他最终所在格子的编号是几呢？这可以通过计算 $(7 + 26) \bmod 10 = 3$ 得到结果。与之类似，向左走 $26$ 格后所在格子的编号将是 $(7 - 26) \bmod 10 = 1$ 。

数论的 $\bmod$ 运算和 C 或 C++ 的 `%` 运算符不太一样。（C 语言标准是 C99 起，C++ 标准是 C++11 起。）

在接下来代码里，我们定义了一个函数 `ntmod` 来表示数论中的 $\bmod$ 运算。

如果要了解更多关于模、余数的事情，可以参考我的博客文章 [带余除法策略](https://ls-hower.github.io/blog/2026-08-02-divmod.html) 。

### 完整代码

C：

```c
#include <stdio.h>

// 全局变量 int 数组会自动初始化为全 0。
int mat[39][39];

// 数论 mod。
// i = ntmod(i + 1, n); 就是向下走一格。
// i = ntmod(i - 1, n); 就是向上走一格。
// j = ntmod(j + 1, n); 就是向右走一格。
// j = ntmod(j - 1, n); 就是向左走一格。
int ntmod(int a, int n)
{
    const int res = a % n;
    return res < 0 ? res + n : res;
}

// 向全局变量 mat 填入幻方内容。
// 参数 n 表示应该填多大范围。
void fill_in(int n)
{
    // 当前坐标。这里用了 x y 而不是 i j，因为循环变量叫 i 了。
    int x = 0;
    int y = (n - 1) / 2;
    for (int i = 1; i <= n * n; ++i) {
        // 填数
        mat[x][y] = i;
        // 算出当前坐标右上方的坐标
        const int nx = ntmod(x - 1, n);
        const int ny = ntmod(y + 1, n);
        if (mat[nx][ny]) {
            // 若右上方已经填入了数，则向下走
            x = ntmod(x + 1, n);
        } else {
            // 否则就向右上走
            x = nx;
            y = ny;
        }
    }
}

// 将全局变量 mat 中的幻方内容打印出来。
// 参数 n 表示应该打印多大范围的。
void print_out(int n)
{
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < n; ++j) {
            printf("%d ", mat[i][j]);
        }
        putchar('\n');
    }
}

int main()
{
    int n;
    scanf("%d", &n);
    fill_in(n);
    print_out(n);
}
```

C++：

```cpp
#include <iostream>
#include <vector>

// 数论 mod。
// i = ntmod(i + 1, n); 就是向下走一格。
// i = ntmod(i - 1, n); 就是向上走一格。
// j = ntmod(j + 1, n); 就是向右走一格。
// j = ntmod(j - 1, n); 就是向左走一格。
int ntmod(int a, int n)
{
    const int res = a % n;
    return res < 0 ? res + n : res;
}

struct MagicSquare {
    std::vector<std::vector<int>> mat;

    // 构造函数同时承担了分配内存和填入幻方内容的职责。
    MagicSquare(int n)
        : mat(n, std::vector<int>(n, 0))
    {
        // 当前坐标。这里用了 x y 而不是 i j，因为循环变量叫 i 了。
        int x = 0;
        int y = (n - 1) / 2;
        for (int i = 1; i <= n * n; ++i) {
            // 填数
            mat[x][y] = i;
            // 算出当前坐标右上方的坐标
            const int nx = ntmod(x - 1, n);
            const int ny = ntmod(y + 1, n);
            if (mat[nx][ny]) {
                // 若右上方已经填入了数，则向下走
                x = ntmod(x + 1, n);
            } else {
                // 否则就向右上走
                x = nx;
                y = ny;
            }
        }
    }

    friend std::ostream& operator<<(std::ostream& os, const MagicSquare& sq)
    {
        for (const auto& row : sq.mat) {
            for (const int cell : row) {
                os << cell << ' ';
            }
            os << '\n';
        }
        return os;
    }
};

int main()
{
    int n;
    std::cin >> n;
    std::cout << MagicSquare{n};
}
```
