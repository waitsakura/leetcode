# LeetCode 图论专题复习笔记（第 2 章）

**目录结构**

```text
一览众山小                  ← 总览表：点题号直达对应题目
│
├─ 遗留问题                  ← 各题尚未尝试的解法清单
│
├─ 一、LC 785 Is Graph Bipartite?
│    每节内部结构一致：
│      原题链接（中文站）→ 题目本质 → 解法 1/2… → [↑ 返回总览]
│      每个解法：思路 → 完整代码 → 易错点 → 时空复杂度
```

---

<a id="overview"></a>

## 一览众山小

| 题号 | 题目 | 图类型 | 本质问题 | 主流解法 | 时间 / 空间 |
|---|---|---|---|---|---|
| [785](#lc785) | Is Graph Bipartite? | 无向图（邻接表） | 二分图判定（二染色 / 无奇环） | BFS·DFS 二染色 / 奇偶并查集 | O(V+E) / O(V) |

---

## 遗留问题

> 只记录**尚未尝试的解法**；已尝试/掌握的解法与文件层面琐事不在此列。尝试并掌握一项就勾掉一项。

- [ ] **LC 785**：奇偶并查集（未尝试；BFS 二染色、DFS 二染色均已自写并归档）

---

<a id="lc785"></a>

## 一、LC 785 Is Graph Bipartite? —— 判断二分图

**原题链接**：[785. Is Graph Bipartite?](https://leetcode.cn/problems/is-graph-bipartite/)

### 题目本质

给定无向图的**邻接表**（`graph[u]` = u 的所有邻居），判断能否把所有节点分成两个集合，使每条边的两个端点分属不同集合。三个等价说法：

> **二分图 ⟺ 可用两种颜色染色使相邻节点异色 ⟺ 图中不存在奇环（长度为奇数的环）**。

直觉：沿边走颜色必须交替；若绕一个奇环走一圈回到起点，会要求"自己与自己异色"，矛盾。

### 解法 1：BFS 二染色（自写实现）

**思路**

- `color[i]` 记颜色：`-1` 未染色、`0` / `1` 两种已染色状态；
- 外层遍历所有节点，**只对未染色的节点启动 `bfs`**——每个连通分量恰好从一个起点向外扩散（图不保证连通，靠外层遍历覆盖全部连通分量）；
- `bfs` 内：起点若未染色则染 `0`；出队节点 `u` 遍历其邻居 `v`：
  - `v` 未染色 → 染成 `1 - color[u]` 并**入队**继续扩散；
  - `v` 已染色且与 `u` 同色 → 冲突 → 返回 `False`；
- 全部处理完无冲突 → 是二分图。

**完整代码**

```python
from typing import List
from collections import deque


class Solution:
    def isBipartite(self, graph: List[List[int]]) -> bool:
        n = len(graph)
        color = [-1] * n

        def bfs(start: int) -> bool:
            queue = deque([start])
            while queue:
                u = queue.popleft()
                if color[u] == -1:
                    color[u] = 0
                for v in graph[u]:
                    if color[v] == -1:
                        color[v] = 1 - color[u]
                        queue.append(v)
                    elif color[u] == color[v]:
                        return False

            return True

        for i in range(n):
            if color[i] == -1 and not bfs(i):
                return False

        return True
```

**易错点**

1. **染色后必须入队**：漏掉 `queue.append(v)` 就不会继续扩散，算法退化成"只看一层邻居"，在部分二分图上会误判为 `False`；
2. `color` 的"未染色"标记不能复用 `0`（`0` 是合法颜色）→ 用 `-1`；
3. **自环**（`u` 是自己的邻居）是长度 1 的奇环 → 必须返回 `False`；
4. **二分图允许偶环**：判据是"无奇环"，不是"无环"；
5. 图可以不连通（甚至空图、孤立点）→ 外层遍历所有节点才完整。

**时空复杂度**：时间 O(V+E)，空间 O(V)（`color` + 队列）。

### 解法 2：DFS 二染色（自写实现）

**思路**

- 与解法 1 同源，只是把 BFS 队列换成递归：`dfs(u, c)` 进入时先把 `u` 染成 `c`；
- 遍历邻居 `v`：若 `v` 已经是**同色** `c` → 冲突返回 `False`；若 `v` 未染色 → 递归 `dfs(v, 1 - c)`；
- 外层仍然遍历所有节点、只从**未染色**节点启动（覆盖不连通图的所有分量）。

**完整代码**

```python
from typing import List


class Solution:
    def isBipartite(self, graph: List[List[int]]) -> bool:
        n = len(graph)
        color = [-1] * n

        def dfs(u: int, c: int) -> bool:
            color[u] = c
            for v in graph[u]:
                if color[v] == c:
                    return False
                elif color[v] == -1 and not dfs(v, 1 - c):
                    return False

            return True

        for i in range(n):
            if color[i] == -1 and not dfs(i, 0):
                return False

        return True
```

**易错点**

1. **进入即染色**：`color[u] = c` 要放在遍历邻居之前，否则回边会把未染色的自己再走一遍；
2. 递归写法在**深图/长链**下有栈溢出风险（Python 默认递归上限约 1000）——顶点数很大时优先用解法 1 的 BFS；
3. 与 BFS 版同样的三条：`-1` 表示未染色、自环必为 `False`、图可以不连通（外层必须遍历所有节点）。

**时空复杂度**：时间 O(V+E)，空间 O(V)（`color` + 递归栈，最坏深度可达 V）。

[↑ 返回总览](#overview)
