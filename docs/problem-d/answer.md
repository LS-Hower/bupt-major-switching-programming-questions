使用结构体将名称与分数打包在一起，是不错的做法。在 C++ 中还可以使用类，虽然在 C++ 中 `class` 与 `struct` 其实几乎没有区别。

我们创建结构体数组，即可储存所有数据。

接下来定义比较函数，然后利用库函数将数组排序即可。

### 完整代码

C：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char name[32];
    int score;
} team_data_t;

int compare_team_data(const void* a, const void* b)
{
    const team_data_t * const lhs = (const team_data_t*) a;
    const team_data_t * const rhs = (const team_data_t*) b;

    // 先比较分数，若分数不同则直接给出结果。
    if (lhs->score > rhs->score) {
        return -1;
    }
    if (lhs->score < rhs->score) {
        return 1;
    }
    // 分数相同时，strcmp 的返回值刚好满足要求。
    return strcmp(lhs->name, rhs->name);
}

void read_team_data(team_data_t* data)
{
    scanf("%s%d", data->name, &data->score);
}

void write_team_data(const team_data_t* data)
{
    printf("%s %d", data->name, data->score);
}

int main(void)
{
    int n;
    scanf("%d", &n);
    team_data_t data[30];

    for (int i = 0; i < n; ++i) {
        read_team_data(data + i);
    }

    qsort(data, n, sizeof data[0], compare_team_data);

    for (int i = 0; i < n; ++i) {
        write_team_data(data + i);
        putchar('\n');
    }
}
```

C++：

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <vector>

struct TeamData {
    std::string name;
    int score;

    friend std::ostream& operator<<(std::ostream& os, const TeamData& data)
    {
        return os << data.name << ' ' << data.score;
    }

    friend std::istream& operator>>(std::istream& is, TeamData& data)
    {
        return is >> data.name >> data.score;
    }
};

bool operator<(const TeamData& lhs, const TeamData& rhs)
{
    if (lhs.score > rhs.score) {
        // lhs 分数大，所以 lhs 靠前。
        return true;
    }
    if (lhs.score == rhs.score && lhs.name < rhs.name) {
        // 分数相同，而 lhs 名字字典序更小，所以 lhs 靠前。
        return true;
    }
    // lhs 并不比 rhs 靠前。
    return false;
}

int main()
{
    int n;
    std::cin >> n;
    std::vector<TeamData> data(n);
    for (auto& datum : data) {
        std::cin >> datum;
    }
    std::sort(data.begin(), data.end());
    for (const auto& datum : data) {
        std::cout << datum << '\n';
    }
}
```
