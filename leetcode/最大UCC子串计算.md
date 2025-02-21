### 问题描述
小S有一个由字符 'U' 和 'C' 组成的字符串 S，并希望在编辑距离不超过给定值 m 的条件下，尽可能多地在字符串中找到 "UCC" 子串。编辑距离定义为将字符串 S转化为其他字符串时所需的最少编辑操作次数。允许的每次编辑操作是插入、删除或替换单个字符。你需要计算在给定的编辑距离限制 m 下，能够包含最多 "UCC" 子串的字符串可能包含多少个这样的子串。
例如，对于字符串"UCUUCCCCC"和编辑距离限制m = 3，可以通过编辑字符串生成最多包含3个"UCC"子串的序列。
约束条件：
字符串长度不超过1000
### 测试样例

样例1：
输入：m = 3,s = "UCUUCCCCC"
输出：3

样例2：
输入：m = 6,s = "U"
输出：2

样例3：
输入：m = 2,s = "UCCUUU"
输出：2

### 解释
样例1：可以将字符串修改为 "UCCUCCUCC"（2 次替换操作，不超过给定值 m = 3），包含 3 个 "UCC" 子串。

样例2：后面插入 5 个字符 "CCUCC"（5 次插入操作，不超过给定值 m = 6），可以将字符串修改为 "UCCUCC"，包含 2 个 "UCC" 子串。

样例3：替换最后 2 个字符，可以将字符串修改为 "UCCUCC"，包含 2 个 "UCC" 子串。
### code
```cpp
#include <iostream>
#include <vector>
#include <string>

using namespace std;

// dp[i][j]: 长度为i的字符串有j编辑距离时的最大ucc子串数
// 这里dp[0][3]、dp[0][6]等没有给值，会在最后ret进行处理
// 从编辑距离的角度看，为0则要求UCC，为1则要求UC或CC，为2则都可以插入

int solution(int m, std::string s) {
    int n = s.size();
    vector<vector<int>> dp(n+1,vector<int>(m+1,0));
    for(int i = 1; i <= n; i++) {
        for(int j = 0; j <= m; j++) {
            if(j >= 2) {
                dp[i][j] = max(dp[i][j],dp[i-1][j-2] + 1);
            }
            if(i >= 2 && j >= 1) {
                if(s[i-2] == 'U' && s[i-1] == 'C') {
                    dp[i][j] = max(dp[i][j],dp[i-2][j-1] + 1);
                }
                else if(s[i-2] == 'C' && s[i-1] == 'C') {
                    dp[i][j] = max(dp[i][j],dp[i-2][j-1] + 1);
                }
            }
            if(i >= 3 && s[i-3] == 'U' && s[i-2] == 'C' && s[i-1] == 'C') {
                dp[i][j] = max(dp[i][j],dp[i-3][j] + 1);
            }
            dp[i][j] = max(dp[i][j],dp[i-1][j]);
        }
    }
    int ret = 0;
    for(int j = 0; j <= m; j++) {
        ret = max(ret, dp[n][j] + (m - j)/3);
    }
    return ret; // Placeholder
}

int main() {
    std::cout << (solution(3, "UCUUCCCCC") == 3) << std::endl;
    std::cout << (solution(6, "U") == 2) << std::endl;
    std::cout << (solution(2, "UCCUUU") == 2) << std::endl;
    return 0;
}
```
### 分析

1. **定义动态规划数组**：
   - `dp[i][j]` 表示长度为 `i` 的字符串在有 `j` 个编辑距离时的最大 "UCC" 子串数。

2. **初始化**：
   - 使用大小为 `(n+1) x (m+1)` 的二维数组 `dp`，并初始化为 0。

3. **状态转移方程**：
   - 若当前编辑距离 `j` 大于或等于 2，可以考虑把当前字符变成 'U' 或 'C'。
   - 若前两个字符形成 "UC" 或 "CC"，且编辑距离 `j` 大于或等于 1，可以增加一个 "UCC" 子串。
   - 若前三个字符是 "UCC"，可以直接增加一个 "UCC" 子串，不需要额外的编辑操作。
   - 在每一步，将当前 dp 值与之前的值进行比较，取较大者。

4. **计算结果**：
   - 遍历所有编辑距离 `j` 的可能值，找到最大值，并考虑剩余的编辑距离能否再构成更多的 "UCC" 子串。

### 详细解释

#### 初始化

```cpp
vector<vector<int>> dp(n+1, vector<int>(m+1, 0));
```

- 创建一个二维数组 `dp`，其中 `dp[i][j]` 存储的是长度为 `i` 且编辑距离为 `j` 时的最大 "UCC" 子串数。

#### 填充动态规划表

```cpp
for (int i = 1; i <= n; i++) {
    for (int j = 0; j <= m; j++) {
        // 使用编辑距离为2处理当前字符
        if (j >= 2) {
            dp[i][j] = max(dp[i][j], dp[i-1][j-2] + 1);
        }

        // 使用编辑距离为1处理前两个字符
        if (i >= 2 && j >= 1) {
            if (s[i-2] == 'U' && s[i-1] == 'C') {
                dp[i][j] = max(dp[i][j], dp[i-2][j-1] + 1);
            } else if (s[i-2] == 'C' && s[i-1] == 'C') {
                dp[i][j] = max(dp[i][j], dp[i-2][j-1] + 1);
            }
        }

        // 使用零编辑距离处理前三个字符
        if (i >= 3 && s[i-3] == 'U' && s[i-2] == 'C' && s[i-1] == 'C') {
            dp[i][j] = max(dp[i][j], dp[i-3][j] + 1);
        }

        // 保留原来的值
        dp[i][j] = max(dp[i][j], dp[i-1][j]);
    }
}
```

- 这个循环更新 `dp` 表，确保对于每个可能的长度和编辑距离组合都进行了正确的处理。

#### 计算最大值

```cpp
int ret = 0;
for (int j = 0; j <= m; j++) {
    ret = max(ret, dp[n][j] + (m - j) / 3);
}
return ret;
```

- 遍历所有可能的编辑距离，计算最终能够生成的最大 "UCC" 子串数，并考虑剩余的编辑距离能否再构成更多的 "UCC" 子串。

### 测试部分

```cpp
int main() {
    std::cout << (solution(3, "UCUUCCCCC") == 3) << std::endl;
    std::cout << (solution(6, "U") == 2) << std::endl;
    std::cout << (solution(2, "UCCUUU") == 2) << std::endl;
    return 0;
}
```

这部分代码测试了几个示例来验证函数的正确性。

### 总结

- 动态规划通过保存中间结果来高效地解决问题。
- 需要特别注意状态转移方程以及边界条件的处理。
- 本算法的时间复杂度和空间复杂度都是 O(n * m)，其中 `n` 是字符串长度，`m` 是允许的编辑距离。