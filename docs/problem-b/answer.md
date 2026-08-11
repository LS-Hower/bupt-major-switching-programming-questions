所有循环都可以转化为递归。以这个题目为例，我们看一看具体是如何转化的。我们将写出使用循环的代码，之后把它逐步转化成递归版本。

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
    // 上面这个分号是一条空语句。空语句什么都不干，要写它是因为 C23 前的标准要求“标签”，即“out_loop:”后面至少要有一条语句。
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

至此，我们成功将循环转化成了递归。

***

C 代码：

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

C++ 代码：

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
