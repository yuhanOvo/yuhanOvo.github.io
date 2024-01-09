---
title: leetcode 第 121 场双周赛 补题Q3 Q4
date: 2024-01-09 13:42:51
updated: 2024-01-09 13:42:51
categories: 
  - 算法学习
tags:
  - 数位DP
  - BFS
  - 记忆化搜索
---

### Q3 [2998. 使 X 和 Y 相等的最少操作次数](https://leetcode.cn/problems/minimum-number-of-operations-to-make-x-and-y-equal/)

#### 题意

给你两个整数x和y，可以执行四种操作，要求使`x`等于`y`

- 当`x`是11的倍数，将`x`除以11
- 当`x`是5的倍数，将`x`除以5
- 将`x`减1
- 将`x`加1

求让`x`等于`y`的最少操作数

<!--more-->

#### 题解

参考[灵神题解](https://leetcode.cn/problems/minimum-number-of-operations-to-make-x-and-y-equal/solutions/2594112/liang-chong-fang-fa-bfsji-yi-hua-sou-suo-djba/)

对`x`进行操作，其中当`x`小于`y`的情况下，只能通过+1使得结果等于`y`，此时最少操作次数为`y-x`

两种方法

第1种方法：**BFS**，类似图连边，四种操作得到的目标数`v`和`x'`进行连边，从而转化成求`x`和`y`的最短路

第2种方法：**记忆化搜索**，具体见代码



#### 代码

方法1（BFS）：

```python
class Solution:
    def minimumOperationsToMakeEqual(self, x: int, y: int) -> int:
        if x <= y:
            return y-x
        vis = [False] * (x+x-y+1)
        q = []
        ans = x-y
        step = 0
        def add(v: int):
            if v < y:
                nonlocal ans
                ans = min(ans, step+1+y-v)
            elif not vis[v]:
                vis[v] = True
                q.append(v)

        add(x)        
        while True:
            tmp = q
            q = []
            for node in tmp:
                if node == y:
                    return min(ans, step)
                if node%11 == 0:
                    add(node//11)
                if node%5 == 0:
                    add(node//5)
                add(node-1)
                add(node+1)
            step += 1
```

方法2（记忆化搜索）：

```python
class Solution:
    @cache
    def minimumOperationsToMakeEqual(self, x: int, y: int) -> int: # x到y的最少操作次数
        if x <= y:
            return y-x
        return min(x-y,
                    self.minimumOperationsToMakeEqual(x//11, y)+x%11+1,
                    self.minimumOperationsToMakeEqual(x//11 +1, y)+11-x%11+1,
                    self.minimumOperationsToMakeEqual(x//5, y)+x%5+1,
                    self.minimumOperationsToMakeEqual(x//5 +1, y)+5-x%5+1)
```

其他语言记忆化对x处理

### Q4 [2999. 统计强大整数的数目](https://leetcode.cn/problems/count-the-number-of-powerful-integers/) 

#### 题意

给你三个整数 `start` ，`finish` 和 `limit`，以及一个字符串`s`，求`start`和`finishi`之间的，各个位数的数字值小于等于`limit`，后缀为`s`的所以数字的个数

#### 题解

[灵神题解](https://leetcode.cn/problems/count-the-number-of-powerful-integers/solutions/2595149/shu-wei-dp-shang-xia-jie-mo-ban-fu-ti-da-h6ci/)

数位DP

#### 代码

```python
class Solution:
    def numberOfPowerfulInt(self, start: int, finish: int, limit: int, s: str) -> int:

        @cache
        def f(i:int, is_limit:bool, is_num:bool, up_num:int) -> int:
            _s = str(up_num)
            if len(_s) < len(s):
                return 0
            if i == len(_s):
                return 1
            res = 0
            if i < len(_s)-len(s):
                if not is_num:
                    res = f(i+1, False, False, up_num)
                low = 0 if is_num else 1
                up = int(_s[i]) if is_limit else 9
                for d in range(low, up+1):
                    if d <= limit:
                        res += f(i+1, is_limit and d == up, True, up_num)
                return res
            else:
                if not is_num:
                    if len(_s) == len(s):
                        return int(s) <= int(_s)
                    return 1
                idx = i - (len(_s) - len(s))
                if is_limit:
                    if int(s[idx]) <= int(_s[i]):
                        res += f(i+1, is_limit and s[idx] == _s[i], True, up_num)
                else:
                    res += f(i+1, False, True, up_num)
                return res


        return  f(0, True, False, finish) - f(0, True, False, start-1)
```



### 总结

Q3没思路，当时以为是数学，能巧做，结果看题解其实也是一种模拟，BFS模拟所有可能的操作，然后取最小值，记忆化也类似

Q4没看，其实是一个数位DP的题，这种题直接套用模板改改就行，但是之前并不会数位DP，新技能get！

另外数位DP的板子题目如下

#### [2376. 统计特殊整数](https://leetcode.cn/problems/count-special-integers/)

如果一个正整数每一个数位都是 **互不相同** 的，我们称它是 **特殊整数** 。

给你一个 **正** 整数 `n` ，请你返回区间`[1, n]` 之间特殊整数的数目。

#### 数位DP板子/题解

如下

```python
class Solution:
    def countSpecialNumbers(self, n: int) -> int:
        s = str(n)
        @cache
        def f(i:int, mask:int, is_limit:bool, is_num:bool) -> int:
            if i == len(s):
                return int(is_num)
            res = 0
            if not is_num:
                res = f(i+1, mask, False, False)
            low = 0 if is_num else 1
            up = int(s[i]) if is_limit else 9
            for d in range(low, up+1):
                if (mask >> d & 1) == 0:
                    res += f(i+1, mask | (1 << d), is_limit and d == up, True)
            return res
        
        return f(0, 0, True, False)
```

其他语言记忆化部分只需要记忆(i, mask)对应的值即可，具体参考[灵神题解](https://leetcode.cn/problems/count-special-integers/solutions/1746956/shu-wei-dp-mo-ban-by-endlesscheng-xtgx/)

数位DP题单，[这里](https://leetcode.cn/problems/count-the-number-of-powerful-integers/solutions/2595149/shu-wei-dp-shang-xia-jie-mo-ban-fu-ti-da-h6ci/)和[这里](https://leetcode.cn/problems/count-special-integers/solutions/1746956/shu-wei-dp-mo-ban-by-endlesscheng-xtgx/)结尾处