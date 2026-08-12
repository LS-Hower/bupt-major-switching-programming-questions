这道题有两种解法，第一种是想出一个简洁的分治思想，第二种是把“将任意一个循环转化成递归”的步骤套用一下。至于两种做法哪一种更优雅，就见仁见智了。

## 解法一：简洁的分治

我们想要写出一个函数 `print_numbers` ，它满足：以 `n` 为参数调用它时，它能顺序打印 `n` 到 `1` 之间的所有整数。我们先写出这样一个框架：

```c
// 这个函数会按顺序打印从 n 到 1 的所有整数。
void print_numbers(int n)
{
    // ... 待编写
}
```

既然它要做的任务是“打印 `n` 到 `1` 之间的所有整数”，那我们就可以把这个分成两步做到：

1. 打印 `n` ；
2. 打印 `n - 1` 到 `1` 之间的所有整数。

我们发现，第 2 步其实和我们一开始要做的事情拥有着几乎一样的结构。既然 `print_numbers(n)` 可以从 `n` 打印到 `1` ，那么 `print_numbers(n - 1)` 自然就可以从 `n - 1` 打印到 `1` 了。前者的第 2 个步骤其实就是后者。

所以我们可以将这个思路直接翻译成代码：

```c
// 怎样从 n 打印到 1 呢？
void print_numbers(int n)
{
    // 第 1 步：打印 n
    printf("%d\n", n);
    // 第 2 步：从 n - 1 打印到 1
    print_numbers(n - 1);
}
```

但这样其实会造成递归永远无法终止。我们的函数打印了 `3` `2` `1` 之后还会继续打印 `0` `-1` `-2` 等等。究其原因，我们没有设定一个终止条件。

我们不妨把 `n` 等于 `1` 的情况设为终止条件，因为这个情况处理起来真的很简单。我们要添加一个 `if (n == 1)` ：

```c
void print_numbers(int n)
{
    if (n == 1) {
        // n == 1 时，只需要打印一个 1。
        // 然后我们立刻结束程序，也就是 return。这样递归就能终止了。
        printf("%d\n", 1);
        return;
    }
    printf("%d\n", n);
    print_numbers(n - 1);
}
```

这样，我们就设计好了 `print_numbers` 函数。

### 完整代码

C：

```c
#include <stdio.h>

void print_numbers(int n)
{
    if (n == 1) {
        printf("%d\n", 1);
        return;
    }
    printf("%d\n", n);
    print_numbers(n - 1);
}

int main(void)
{
    int n;
    scanf("%d", &n);
    print_numbers(n);
}
```

C++：

```cpp
#include <iostream>

void print_numbers(int n)
{
    if (n == 1) {
        std::cout << 1 << '\n';
        return;
    }
    std::cout << n << '\n';
    print_numbers(n - 1);
}

int main(void)
{
    int n;
    std::cin >> n;
    print_numbers(n);
}
```

## 解法二：把“将任意一个循环转化成递归”的步骤套用一下

所有循环都可以转化为递归，而且有一套通用的方法做到这一点。

我们在这里将先写出使用循环的代码，之后把它逐步转化成递归版本，从而展示具体是如何转化的。

首先，写出一个使用了 `for` 循环的版本：

```c
// 这个函数会按顺序打印从 n 到 1 的所有整数。
void print_numbers(int n)
{
    for (int i = n; i > 0; --i) {
        printf("%d\n", i);
    }
}
```

我们先将 `for` 改回等价的 `while` ：

```c
void print_numbers(int n)
{
    int i = n;
    while (i > 0) {
        printf("%d\n", i);
        --i;
    }
}
```

我们可以将所有 `while (...)` 改写成 `while (1)` ，只需要用一下 `break` ：

```c
void print_numbers(int n)
{
    int i = n;
    while (1) {
        if (i <= 0) {
            break;
        }
        printf("%d\n", i);
        --i;
    }
}
```

我们知道 `break` 可以用 `goto` 实现：

```c
void print_numbers(int n)
{
    int i = n;
    while (1) {
        if (i <= 0) {
            goto out_loop;
        }
        printf("%d\n", i);
        --i;
    }
out_loop:
    ;
    // 上面这个分号是一条空语句。
    // 空语句什么都不干。
    // 要写它是因为 C23 前的标准要求标签后面至少要有一条语句。
}
```

而一个 `while (1)` 是一个无条件的循环，我们现在就可以轻易地用 `goto` 表示它了，方法是将循环头 `while (1) {` 改成一个标签 `loop:` ，再把循环尾的 `}` 改成 `goto loop;` ：

```c
void print_numbers(int n)
{
    int i = n;

loop:
    if (i <= 0) {
        goto out_loop;
    }
    printf("%d\n", i);
    --i;
    goto loop;

out_loop:
    ;
}
```

为了更清晰，我们将循环的主体，也就是 `int i = n;` 这一行以下的部分放入另一个函数里：

```c
void print_numbers(int n)
{
    int i = n;
    return print_numbers_internal(i);
}

void print_numbers_internal(int i)
{
loop:
    if (i <= 0) {
        goto out_loop;
    }
    printf("%d\n", i);
    --i;
    goto loop;

out_loop:
    ;
}
```

好了。现在可以发现，所谓的 `goto out_loop;` 其实就是 `return` ：

```c
void print_numbers(int n)
{
    int i = n;
    return print_numbers_internal(i);
}

void print_numbers_internal(int i)
{
loop:
    if (i <= 0) {
        return;
    }
    printf("%d\n", i);
    --i;
    goto loop;
}
```

而 `goto loop:` 其实就是重新从本函数的开头开始执行罢了，只不过只有一处不同，那就是 `i` 的值是原本的 `i` 的值减去 `1` 。所以我们可以用一句 `print_numbers_internal(i - 1)` 取代掉这两行 `--i;` 和 `goto loop;` ：

```c
void print_numbers(int n)
{
    int i = n;
    return print_numbers_internal(i);
}

void print_numbers_internal(int i)
{
    if (i <= 0) {
        return;
    }
    printf("%d\n", i);
    print_numbers_internal(i - 1);
}
```

我们注意到在 `print_numbers_internal` 多包了一层 `print_numbers` ，是纯属多余了，所以我们直接将 `print_numbers` 去掉，再把 `print_numbers_internal` 改名成 `print_numbers` ：

```c
void print_numbers(int i)
{
    if (i <= 0) {
        return;
    }
    printf("%d\n", i);
    print_numbers(i - 1);
}
```

给参数改一个名字不会影响程序的任何行为，我们把 `i` 改名成 `n` 吧：

```c
void print_numbers(int n)
{
    if (n <= 0) {
        return;
    }
    printf("%d\n", n);
    print_numbers(n - 1);
}
```

至此，我们成功将循环转化成了递归。

### 完整代码

C：

```c
#include <stdio.h>

void print_numbers(int n)
{
    if (n <= 0) {
        return;
    }
    printf("%d\n", n);
    print_numbers(n - 1);
}

int main(void)
{
    int n;
    scanf("%d", &n);
    print_numbers(n);
}
```

C++：

```cpp
#include <iostream>

void print_numbers(int n)
{
    if (n <= 0) {
        return;
    }
    std::cout << n << '\n';
    print_numbers(n - 1);
}

int main(void)
{
    int n;
    std::cin >> n;
    print_numbers(n);
}
```
