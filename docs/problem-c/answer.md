我们先写出一个函数 `is_prime` 判断一个数是否为素数。

在这里，我们使用试除法。我们基本只需要把题目中“素数的定义”复述一遍即可。

例如要判断 $n$ 是否为素数，我们只需：

- 先判断 $n \ge 2$ 是否成立，若不成立则直接返回假；
- 对所有大于等于 $2$ 、小于 $n$ 的数 $i$ ，测试 $i$ 是否整除 $n$ 。一旦有能整除的，就说明它是一个因数，说明 $n$ 不是素数，返回假。
- 如果上述测试都通过了，就返回真。

我们可以将上述想法直接翻译成 C 代码：

```c
int is_prime(int n)
{
    if (n < 2) {
        // 小于 2，一定不是素数
        return 0;
    }

    for (int i = 2; i < n; ++i) {
        if (n % i == 0) {
            // 发现了一个非 1 非 n 的因数。
            // 所以不是素数。
            return 0;
        }
    }

    // 通过了测试，是素数。
    return 1;
}
```

但这里可以做一个优化。首先，对于 $n$ ，我们将 $1$ 和 $n$ 称为它的“平凡因数”，因为它们没有什么研究的必要；其他因数称为“非平凡因数”。所以 $n$ 是素数当且仅当 $n$ 有非平凡因数。

现在我们可以证明， $n$ 如果有非平凡因数，那么其中最小的一个不会大于 $\sqrt{n}$ 。我们将最小的非平凡因数记为 $f$ ，那么 $\dfrac{n}{f}$ 也是 $n$ 的非平凡因数了。如果 $f > \sqrt{n}$ ，那么 $\dfrac{n}{f} < \sqrt{n}$ ，从而 $\dfrac{n}{f} < f$ ，和假设“ $f$ 是最小的非平凡因数”矛盾。所以刚刚的假设 $f > \sqrt{n}$ 错误。所以一定有 $f \le \sqrt{n}$ 。

直观地，一个数 $n$ 的因数总是成对出现的，并且一个大于 $\sqrt{n}$ ，一个小于 $\sqrt{n}$ （例外情况是平方数，此时 $\sqrt{n}$ 是一个单独的因数）。举个例子， $18$ 有一个因数是 $6$ ，那么 $\dfrac{18}{6} = 3$ 也一定是个因数，而这个 $3$ 便满足了 $3 \le \sqrt{18}$ 。

回到我们的算法这里，应用这一结论，我们便只需要检查所有满足 $i^{2} \le n$ 的整数 $i$ 就可以了：

```c
int is_prime(int n)
{
    if (n < 2) {
        return 0;
    }

    for (int i = 2; i * i <= n; ++i) {
        if (n % i == 0) {
            return 0;
        }
    }

    return 1;
}
```

在这里，我们不需要担心整数溢出的问题，因为题目限制了 $n$ 的范围。

接下来是排序的事。复习一下排序函数的声明：

C（ `stdlib.h` 头文件）：

```c
void qsort(void* ptr, size_t count, size_t size,
           int (*comp)(const void*, const void*));
```

四个参数分别是数组首项指针、数组项数、数组每一项的大小，以及一个比较器。比较器应能接收两个指向数组内的项的指针，若认为前者大于后者，则返回正值，等于则零，小于则负值。

C++（ `algorithm` 头文件）：

```cpp
template <class RandomIt>
void sort(RandomIt first, RandomIt last);

template <class RandomIt, class Compare>
void sort(RandomIt first, RandomIt last, Compare comp);
```

`first` 和 `last` 参数是要排序的范围的首尾迭代器， `comp` 参数作为比较器，在认为两个参数前者小于后者时返回 `true` ，否则是 `false` 。在这里，我们可以不用考虑比较器的事情。不传入 `comp` 参数则使用第一个重载，此时排序使用默认的小于号，我们只需要这个版本即可。（C++20 起使用 `std::less` ，但这里我们只涉及 C++11。不列出带有执行策略参数的版本以及 `std::ranges::sort` 也是这个原因。）

也可以自己写排序算法，但面对这道题没有太大必要。

### 完整代码

C：

```c
#include <stdio.h>
#include <stdlib.h>

int is_prime(int n)
{
    if (n < 2) {
        return 0;
    }

    for (int i = 2; i * i <= n; ++i) {
        if (n % i == 0) {
            return 0;
        }
    }

    return 1;
}

int compare_int(const void* a, const void* b)
{
    const int x = * (const int*) a;
    const int y = * (const int*) b;
    // 数据范围比较小，所以其实这里可以实现成返回 x - y 而不用担心溢出。
    // 但这里还是使用显式比较和分支了。
    // 至于为什么在 is_prime 的实现里又不考虑溢出了，
    // 那是因为前者如果考虑溢出好像不太好写。
    if (x < y) {
        return -1;
    }
    if (x > y) {
        return 1;
    }
    return 0;
}

int main(void)
{
    int n;
    scanf("%d", &n);

    int primes[1000];
    int prime_count = 0;

    for (int i = 0; i < n; ++i) {
        int a;
        scanf("%d", &a);
        // 读入时即过滤非素数
        if (!(is_prime(a))) {
            continue;
        }
        // prime_count 表示当前已经记下的素数的个数。
        // 数组 primes 中有效的项的个数一直是 prime_count。
        primes[prime_count] = a;
        ++prime_count;
    }

    // 排序
    qsort(primes, prime_count, sizeof primes[0], compare_int);

    // 逐个输出
    for (int i = 0; i < prime_count; ++i) {
        printf("%d\n", primes[i]);
    }
}
```

C++：

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

bool is_prime(int n)
{
    if (n < 2) {
        return false;
    }

    for (int i = 2; i * i <= n; ++i) {
        if (n % i == 0) {
            return false;
        }
    }

    return true;
}

int main()
{
    int n;
    std::cin >> n;

    std::vector<int> nums{};

    for (int i = 0; i < n; ++i) {
        int a;
        std::cin >> a;
        // 读入时即过滤非素数
        if (!(is_prime(a))) {
            continue;
        }
        nums.emplace_back(a);
    }

    // 排序
    std::sort(nums.begin(), nums.end());

    // 逐个输出
    for (const int x : nums) {
        std::cout << x << '\n';
    }
}
```


