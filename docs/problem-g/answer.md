很不幸，这个问题是 NP-完全的。

对于 NP-完全问题，我们不指望找到它们的多项式时间算法（即用时为 $\mathcal{O}(n^{a})$ 的算法，其中 $a$ 可以是任意大的正数。）

如果要查阅更多资料，可以搜索关键词“最大割”和“加权最大割”。本题就是加权最大割问题。

由于数据量比较小，我们直接采用暴力算法，枚举每一种可能的划分即可。所有可能的划分数有 $2^{n}$ 种，而计算一个划分的权重之和最多需要 $\mathcal{O}(n^{2})$ 时间，因此总用时不会超过 $\mathcal{O}(n^{2} 2^{n})$ 。

### 完整代码

C：

```c
#include <stdio.h>
#include <stdint.h>

int w[20][20];

int main(void)
{
    int n;
    scanf("%d", &n);
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < n; ++j) {
            scanf("%d", &w[i][j]);
        }
    }

    // 因为下方做了将计算量减半的优化，所以 n == 0 要特判。
    if (n == 0) {
        printf("%d\n", 0);
    }

    int best_weight_sum = 0;
    // 其实只需要枚举前 2^{n-1} 种情况，
    // 因为后 2^{n-1} 种情况和前面的一一对应地重复了。
    const uint32_t all_bits = (1 << n) - 1;
    for (uint32_t a_bits = 0; a_bits < (1 << (n - 1)); ++a_bits) {
        const uint32_t b_bits = all_bits & ~a_bits;

        int weight_sum = 0;
        // 计算这一划分下的权重之和
        for (int va = 0; va < n; ++va) {
            for (int vb = 0; vb < n; ++vb) {
                if (!(
                    ((a_bits >> va) & 1)
                    && ((b_bits >> vb) & 1)))
                {
                    // 未满足“va 在集合 A 中，vb 在集合 B 中”这一条件
                    continue;
                }
                weight_sum += w[va][vb];
            }
        }
        // 这一划分下的权重之和是一个新的最优结果，所以更新一下最优权重之和
        if (weight_sum > best_weight_sum) {
            best_weight_sum = weight_sum;
        }
    }

    printf("%d\n", best_weight_sum);
}
```

C++：

```cpp
#include <algorithm>
#include <array>
#include <cstdint>
#include <iostream>

std::array<std::array<int, 20>, 20> w{};

int main()
{
    int n;
    std::cin >> n;
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < n; ++j) {
            std::cin >> w[i][j];
        }
    }

    // 因为下方做了将计算量减半的优化，所以 n == 0 要特判。
    if (n == 0) {
        std::cout << 0 << '\n';
    }

    int best_weight_sum = 0;
    // 其实只需要枚举前 2^{n-1} 种情况，
    // 因为后 2^{n-1} 种情况和前面的一一对应地重复了。
    const std::uint32_t all_bits = (1 << n) - 1;
    for (std::uint32_t a_bits = 0; a_bits < (1 << (n - 1)); ++a_bits) {
        const std::uint32_t b_bits = all_bits & ~a_bits;

        int weight_sum = 0;
        // 计算这一划分下的权重之和
        for (int va = 0; va < n; ++va) {
            for (int vb = 0; vb < n; ++vb) {
                if (!(
                    ((a_bits >> va) & 1)
                    && ((b_bits >> vb) & 1)))
                {
                    // 未满足“va 在集合 A 中，vb 在集合 B 中”这一条件
                    continue;
                }
                weight_sum += w[va][vb];
            }
        }
        // 更新最优权重和
        best_weight_sum = std::max(best_weight_sum, weight_sum);
    }

    std::cout << best_weight_sum << '\n';
}
```

