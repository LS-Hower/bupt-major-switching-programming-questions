这道题主要考查循环与分支结构。将一定区间内的整数相加需要循环结构，而过滤掉不需要的数需要分支结构。在常见语言中，前者一般使用 `for` 或 `while` ，后者一般使用 `if` 和 `else` ，可能还有 `elif` 。

### 完整代码

C：

```c
#include <stdio.h>

int main(void)
{
    int a, b;
    scanf("%d%d", &a, &b);

    int sum = 0;
    for (int i = a; i <= b; ++i) {
        if (i % 5 == 0 || i % 13 == 0) {
            sum += i;
        }
    }

    printf("%d\n", sum);
}
```

C++：

```cpp
#include <iostream>

int main()
{
    int a, b;
    std::cin >> a >> b;

    int sum = 0;
    for (int i = a; i <= b; ++i) {
        if (i % 5 == 0 || i % 13 == 0) {
            sum += i;
        }
    }

    std::cout << sum << '\n';
}
```
