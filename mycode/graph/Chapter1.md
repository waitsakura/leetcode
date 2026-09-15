# LeetCode 图论专题复习笔记（第 1 章）

**目录结构**

```text
一览众山小                  ← 总览表：点题号直达对应题目
│
├─ 遗留问题                  ← 各题尚未尝试的解法清单
│
├─ 一、LC 133 Clone Graph
├─ 二、LC 207 Course Schedule
├─ 三、LC 210 Course Schedule II
├─ 四、LC 310 Minimum Height Trees
├─ 五、LC 399 Evaluate Division
├─ 六、LC 547 Number of Provinces
├─ 七、LC 684 Redundant Connection
├─ 八、LC 743 Network Delay Time
│    每节内部结构一致：
│      原题链接（中文站）→ 题目本质 → 解法 1/2… → [↑ 返回总览]
│      每个解法：思路 → 完整代码 → 易错点 → 时空复杂度
│
└─ 九、跨题通用方法论
```

---

<a id="overview"></a>

## 一览众山小

| 题号 | 题目 | 图类型 | 本质问题 | 主流解法 | 时间 / 空间 |
|---|---|---|---|---|---|
| [133](#lc133) | Clone Graph | 无向图 | 深拷贝（结构复制） | DFS 记忆化 / BFS | O(V+E) / O(V) |
| [207](#lc207) | Course Schedule | 有向图 | 判环（是否 DAG） | Kahn（BFS+入度）/ DFS 三色 | O(V+E) / O(V+E) |
| [210](#lc210) | Course Schedule II | 有向图 | 输出一个拓扑排序 | Kahn 输出序 / DFS 后序逆序 | O(V+E) / O(V+E) |
| [310](#lc310) | Minimum Height Trees | 无向树 | 求树中心（1~2 个） | 拓扑剥离（逐层去叶）/ 两遍 BFS 直径中点 | O(n) / O(n) |
| [399](#lc399) | Evaluate Division | 带权无向 | 路径权连乘查询 | DFS/BFS 逐查询 / 带权并查集 / Floyd 全源 | 见各节 |
| [547](#lc547) | Number of Provinces | 无向图（邻接矩阵） | 连通分量计数 | DFS / BFS / 并查集 | O(n²) / O(n) |
| [684](#lc684) | Redundant Connection | 无向图（树 + 多余 1 边） | 找出多余环边并删除 | 并查集扫描（推荐）/ DFS·BFS 找环 | O(n·α) / O(n) |
| [743](#lc743) | Network Delay Time | 有向带权图 | 单源最短路（求最远到达时间） | Dijkstra（矩阵版 / 堆版） | O(n²) / O(E log V) |

---

## 遗留问题

> 只记录**尚未尝试的解法**；已尝试/掌握的解法与文件层面琐事不在此列。尝试并掌握一项就勾掉一项。

- [ ] **LC 399**：带权并查集、Floyd 全源预处理（均未尝试）
- [ ] **LC 684**：DFS 找环、BFS 逐边查连通、BFS 生成树还原环（均未尝试）
- [ ] **LC 743**：Bellman-Ford / SPFA（未尝试；堆版 Dijkstra 已自写并归档，见"八、LC 743"解法 1）

---

<a id="lc133"></a>

## 一、LC 133 Clone Graph —— 图深拷贝

**原题链接**：[133. Clone Graph](https://leetcode.cn/problems/clone-graph/)

### 题目本质

给定**连通无向图**某节点的引用，返回深拷贝：每个节点都要新建 `Node`，克隆节点间的邻接关系与原图一致。难点：**有环 / 双向边**，直接递归会死循环。

### 解法 1：DFS（递归 + 记忆化映射）

**思路**

- `lookup: 原节点 → 克隆节点` 的字典，兼具**备忘录**与**防环**作用；
- **先登记、再递归**：`lookup[node] = clone` 必须先于遍历邻居执行；
- 遍历原邻居：已克隆 → `return lookup[neighbor]` 复用；未克隆 → 递归新建；
- 按原顺序 `append`，保持邻居顺序。

**完整代码**

```python
from typing import Optional

class Node:
    def __init__(self, val=0, neighbors=None):
        self.val = val
        self.neighbors = neighbors if neighbors is not None else []

class Solution:
    def cloneGraph(self, node: Optional["Node"]) -> Optional["Node"]:
        lookup = {}

        def dfs(node):
            if not node:
                return None
            if node in lookup:
                return lookup[node]
            clone = Node(node.val, [])
            lookup[node] = clone          # 先登记（防环的关键）
            for n in node.neighbors:
                clone.neighbors.append(dfs(n))
            return clone

        return dfs(node)
```

**易错点**

- 登记顺序反了，自环 / 回边会无限递归；
- 空图输入 `None` → 返回 `None`；
- 递归 DFS 深图有栈溢出风险（5000 节点链会 RecursionError）。

**时空复杂度**：时间 O(V+E)；空间 O(V)（lookup + 递归栈）。

### 解法 2：BFS（显式队列）

**思路**

- 队列里放**原图节点**，出队时遍历其原邻居；
- 未克隆的邻居：登记克隆并入队；已克隆的：只挂边；
- 每条边在两个端点出队时各挂一次 → 无向关系完整复制。

**完整代码**

```python
from typing import Optional
from collections import deque

class Node:
    def __init__(self, val=0, neighbors=None):
        self.val = val
        self.neighbors = neighbors if neighbors is not None else []

class Solution:
    def cloneGraph(self, node: Optional["Node"]) -> Optional["Node"]:
        if not node:
            return None

        lookup = {node: Node(node.val)}   # 原节点 -> 克隆节点
        queue = deque([node])

        while queue:
            cur = queue.popleft()
            for n in cur.neighbors:
                if n not in lookup:
                    lookup[n] = Node(n.val)
                    queue.append(n)
                lookup[cur].neighbors.append(lookup[n])

        return lookup[node]
```

**易错点（本会话真实踩坑）**

1. **挂边对象搞错 → MLE**：必须挂到克隆 `lookup[cur].neighbors`；若挂到**原节点** `cur.neighbors`，会"边遍历边改同一列表" → 无限造克隆、原图被污染、内存爆炸（实验：0.5s 内邻居列表被撑到 58 万元素）；
2. deque 用 `appendleft` + `pop` 其实是 **FIFO 队列**（不是栈），易误读，建议 `append` + `popleft`；
3. BFS / 显式栈无递归风险，适合大图。

**时空复杂度**：时间 O(V+E)；空间 O(V)（lookup + 队列）。

[↑ 返回总览](#overview)

---

<a id="lc207"></a>

## 二、LC 207 Course Schedule —— 有向图判环

**原题链接**：[207. Course Schedule](https://leetcode.cn/problems/course-schedule/)

### 题目本质

给定 `numCourses` 门课程与先修依赖 `prerequisites`，判断能否修完全部课程：把每门课当作节点、先修关系画成有向边后，问题等价于**判定有向图是否有环（是否为 DAG）**。

- 课程 = 节点；`[a, b]` 表示先修 b 才能修 a → 有向边 **b → a**；
- "能否修完" ⟺ **图是 DAG（无环）** ⟺ **可拓扑排序**。三者等价；
- 边方向反过来画不影响判环结论。

### 概念补充：什么是拓扑排序

> 给所有节点排出一个线性序，满足**每条边 u→v 中 u 在 v 前**。本质是给 DAG（偏序）找一个"线性扩展"（全序）；答案通常**不唯一**；有环必排不出。能拓扑排序 ⟺ 是 DAG。

### 解法 A：Kahn（BFS + 入度表）

**思路**

- `indeg[i]` = 课程 i 还有几门先修未修；`graph[i]` = i 修完后解锁的后继；
- 反复取**入度为 0** 的点"修掉"，消去它的出边（后继入度 -1）；
- 判环：`done == numCourses`——修不完的节点构成环。

**完整代码**

```python
from typing import List
from collections import deque

class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        indeg = [0] * numCourses
        graph = [[] for _ in range(numCourses)]
        for a, b in prerequisites:
            graph[b].append(a)
            indeg[a] += 1

        q = deque(i for i in range(numCourses) if indeg[i] == 0)
        done = 0
        while q:
            u = q.popleft()
            done += 1
            for v in graph[u]:
                indeg[v] -= 1
                if indeg[v] == 0:
                    q.append(v)

        return done == numCourses
```

### 解法 B：DFS 三色标记

**思路**

- 颜色：`0` 白（未访问）/ `1` 灰（**在递归栈中**）/ `2` 黑（已结束、子树安全）；
- **遇灰 = 有环**（有向图回边可指向任意祖先，与无向图只需 visited 不同）；
- 图不保证连通 → 必须**遍历每个节点**启动 dfs。

**完整代码**

```python
from typing import List

class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        graph = [[] for _ in range(numCourses)]
        for a, b in prerequisites:
            graph[b].append(a)

        color = [0] * numCourses

        def dfs(u: int) -> bool:
            color[u] = 1
            for v in graph[u]:
                if color[v] == 1:
                    return False
                if color[v] == 0 and not dfs(v):
                    return False
            color[u] = 2
            return True

        for i in range(numCourses):
            if color[i] == 0 and not dfs(i):
                return False
        return True
```

**易错点**

1. **三色 DFS 本质上就是"记忆化 DFS"**：黑 = 缓存"该子树无环"，跨根复用 → O(V+E)；若每个根都重建全新 visited，共享子树会被反复重跑，退化成指数/平方级；
2. 只记 visited（不区分灰 / 黑）在**有向图会漏判环**——错误示范：

```python
# 错误示范（片段）：只判"访问过"会漏掉环 1→2→1
def dfs(u):
    seen[u] = True
    for v in graph[u]:
        if seen[v]:
            continue          # 2 看到 1 已访问就跳过 -> 漏判
        if not dfs(v):
            return False
    return True
```

3. 自环 `[i, i]` 是长度为 1 的环 → False；207/210 官方保证先修对无重复，自造数据需去重。

**时空复杂度**：两种解法均时间 O(V+E)、空间 O(V+E)（邻接表）；Kahn 无递归更稳，DFS 深图可能栈溢出。

[↑ 返回总览](#overview)

---

<a id="lc210"></a>

## 三、LC 210 Course Schedule II —— 输出拓扑序

**原题链接**：[210. Course Schedule II](https://leetcode.cn/problems/course-schedule-ii/)

### 题目本质

给定 `numCourses` 门课程与先修依赖，返回一个合法的修课顺序（即任一拓扑排序）；若因存在环而无法修完，返回空数组 `[]`。

与 207 的关系：207 只判“能不能”（bool）；210 还要把合法顺序本身输出，有环返回 `[]`。

### 解法 A：Kahn 输出版

**思路**

- 与 207 几乎一样，只是把出队的课记进 `order`；
- 结束时 `len(order) < numCourses` → 有环 → `[]`；
- **出队顺序天然合法**（每次取的课其先修都已修完）。

**完整代码**

```python
from typing import List
from collections import deque

class Solution:
    def findOrder(self, numCourses: int, prerequisites: List[List[int]]) -> List[int]:
        indeg = [0] * numCourses
        graph = [[] for _ in range(numCourses)]
        for a, b in prerequisites:
            graph[b].append(a)
            indeg[a] += 1

        q = deque(i for i in range(numCourses) if indeg[i] == 0)
        order = []
        while q:
            u = q.popleft()
            order.append(u)
            for v in graph[u]:
                indeg[v] -= 1
                if indeg[v] == 0:
                    q.append(v)

        return order if len(order) == numCourses else []
```

### 解法 B：DFS 后序逆序

**思路**

- 节点染黑（处理完毕）时 `order.append(u)`；
- 因边 u→v 保证 v 先于 u 被记录 → 最终 **`order[::-1]` 才是拓扑序**；
- 遇灰（环）直接 `return []`。

**完整代码**

```python
from typing import List

class Solution:
    def findOrder(self, numCourses: int, prerequisites: List[List[int]]) -> List[int]:
        graph = [[] for _ in range(numCourses)]
        for a, b in prerequisites:
            graph[b].append(a)

        color = [0] * numCourses
        order = []

        def dfs(u: int) -> bool:
            color[u] = 1
            for v in graph[u]:
                if color[v] == 1:
                    return False
                if color[v] == 0 and not dfs(v):
                    return False
            color[u] = 2
            order.append(u)
            return True

        for i in range(numCourses):
            if color[i] == 0 and not dfs(i):
                return []
        return order[::-1]
```

**易错点与注意**

- **后序必须逆序**，顺序反了答案就错（常见笔误）；
- 合法答案通常不唯一，判题接受任意一个；
- 时空复杂度同 207：O(V+E) / O(V+E)。

[↑ 返回总览](#overview)

---

<a id="lc310"></a>

## 四、LC 310 Minimum Height Trees —— 求树的中心

**原题链接**：[310. Minimum Height Trees](https://leetcode.cn/problems/minimum-height-trees/)

### 题目本质

n 个节点的**树**（无向、无环、n-1 条边），以哪个点为根树高最小？树高 = 根到最远叶子的边数。答案 = **树的中心**，**只有 1 个或 2 个**。

### 解法 A：拓扑剥离（逐层摘叶子，"剥洋葱"）

**思路**

- 每轮把**当前所有叶子**（度 = 1）一起摘掉，直到剩余节点 ≤ 2，剩下的就是中心；
- 为什么可行：叶子显然不是最优根（往里挪一层更矮）；层层剥到最后剩下的正是中心；
- 两个关键：**终止看 `remaining > 2`**（剩余节点数），**每轮只剥本层**（`cnt = len(queue)` 快照）。

**完整代码**

```python
from typing import List
from collections import deque

class Solution:
    def findMinHeightTrees(self, n: int, edges: List[List[int]]) -> List[int]:
        if n == 1:
            return [0]

        graph = [[] for _ in range(n)]
        degree = [0] * n
        for u, v in edges:
            graph[u].append(v)
            graph[v].append(u)
            degree[u] += 1
            degree[v] += 1

        leaves = deque(i for i in range(n) if degree[i] == 1)
        remaining = n
        while remaining > 2:
            cnt = len(leaves)          # 本层叶子数快照
            remaining -= cnt
            for _ in range(cnt):       # 只剥本层，新叶子等下一轮
                leaf = leaves.popleft()
                for nb in graph[leaf]:
                    degree[nb] -= 1
                    if degree[nb] == 1:
                        leaves.append(nb)

        return list(leaves)
```

**易错点（本会话真实踩坑）**

1. **终止条件必须是 `remaining > 2`，不是 `len(queue) > 2`**：队列装的是"当前层叶子"，不是剩余节点。错误写法 `while len(queue) > 2` 会直接翻车：任意**链**初始叶子恰好 2 个 → 循环一次不进 → 直接返回两个端点（链 n=6 应返回 `[2,3]` 却返回 `[0,5]`）；
2. **必须按层批处理**：不分层（单队列边出边进）时，内层节点会被"提前"剥掉（n=5 用例应得 `[1,2]` 却得 `[2]`）；
3. `n == 1` 特判返回 `[0]`（没有叶子可入队）；
4. **度数不变量**：被剥的节点剥时度恒为 1；幸存**双中心**终度 1、**单中心**终度 0（正常收官）；全程**不会出现负度数**（每节点最多被邻居剥减"原度数"次）。

**时空复杂度**：时间 O(n)；空间 O(n)。

### 解法 B：两遍 BFS 求直径取中点

**思路**

- **直径定理**：树中任意节点到最远节点的距离 = 它到某条直径两个端点的较远者 → 高度最小的根 = **直径中点**；
- 第一遍：从任意点 BFS 找最远点 a（必为某条直径端点）；第二遍：从 a BFS 找最远点 b 并记录 `parent`；
- 从 b 沿 parent 向上走 `d // 2` 步（d = dist[b]）即到中点；d 偶 → 1 个中心，d 奇 → `[x, parent[x]]`。

**完整代码**

```python
from typing import List
from collections import deque

class Solution:
    def findMinHeightTrees(self, n: int, edges: List[List[int]]) -> List[int]:
        if n == 1:
            return [0]

        graph = [[] for _ in range(n)]
        for u, v in edges:
            graph[u].append(v)
            graph[v].append(u)

        def bfs(start):
            dist = [-1] * n
            parent = [-1] * n
            dist[start] = 0
            q = deque([start])
            while q:
                u = q.popleft()
                for v in graph[u]:
                    if dist[v] == -1:
                        dist[v] = dist[u] + 1
                        parent[v] = u
                        q.append(v)
            return dist, parent

        dist, _ = bfs(0)
        a = max(range(n), key=lambda i: dist[i])

        dist, parent = bfs(a)
        b = max(range(n), key=lambda i: dist[i])

        d = dist[b]
        x = b
        for _ in range(d // 2):
            x = parent[x]          # 关键：是 parent[x]，不是 parent[b]

        if d % 2 == 0:
            return [x]
        return [x, parent[x]]
```

**易错点（本会话真实踩坑）**

- **走中点循环里写错变量**：`x = parent[b]`（而不是 `parent[x]`）→ x 永远停在 b 的父亲，答案整体偏移到端点旁（曾实际犯过的笔误）。直径短的树（星形 d=2）会**碰巧正确**，很隐蔽，须用对拍抓；
- BFS 复用：同一函数返回 `(dist, parent)`，第一遍忽略 parent 即可，不必写两遍 BFS。

**注意**：两遍 BFS 的优势是**顺带拿到直径路径 / 长度**（衍生题如 LC 1245 树直径要用）。

**时空复杂度**：时间 O(n)；空间 O(n)。

[↑ 返回总览](#overview)

---

<a id="lc399"></a>

## 五、LC 399 Evaluate Division —— 除法求值

**原题链接**：[399. Evaluate Division](https://leetcode.cn/problems/evaluate-division/)

### 题目本质

给定若干形如 `a / b = k` 的等式与若干查询，求每个查询表达式的值。把变量当作节点、等式当作带权边后，问题化为**带权图上的可达性 + 路径权值连乘**查询。

- 变量 = 节点，`a/b = k` 即边；
- **无向带权图**（a = k×b）与**有向带权图**（只存 a→b 权 k，b→a 权 1/k 可推导）**等价**；
- **路径连乘 = 起点 ÷ 终点**；题目保证输入自洽 → 任意路径乘积相同，找一条路乘出来即可；
- `-1.0` 情形：变量未出现 / 不连通；`x==x` 恒为 1.0（但 x 未出现时仍为 -1.0）。

### 解法 A：DFS（逐查询，递归带累积值）

**思路**

- 递归参数携带**累积值 acc**（起点到当前点），每走一步 × 边权；
- 到达目标返回 acc；走遍可达点仍不到 → -1.0。

**完整代码**

```python
from typing import List
from collections import defaultdict

class Solution:
    def calcEquation(self, equations: List[List[str]], values: List[float], queries: List[List[str]]) -> List[float]:
        graph = defaultdict(dict)
        for (a, b), k in zip(equations, values):
            graph[a][b] = k
            graph[b][a] = 1 / k

        def dfs(u: str, target: str, acc: float, seen: set) -> float:
            if u == target:
                return acc
            seen.add(u)
            for v, w in graph[u].items():
                if v not in seen:
                    res = dfs(v, target, acc * w, seen)
                    if res != -1.0:
                        return res
            return -1.0

        ans = []
        for x, y in queries:
            if x not in graph or y not in graph:
                ans.append(-1.0)
            elif x == y:
                ans.append(1.0)
            else:
                ans.append(dfs(x, y, 1.0, set()))
        return ans
```

### 解法 B：BFS（队列携带"（结点，累权值）"，namedtuple 练习版）

**思路**

- 队列元素即"（结点，累权值）"，出队到目标即返回 acc；
- 用 `collections.namedtuple` 定义 `Step(node, acc)`：见名知义、不可变、仍是 tuple 子类（解包 / 下标兼容）；需要类型标注可用 `typing.NamedTuple`。

**完整代码**

```python
from typing import List
from collections import defaultdict, deque, namedtuple

Step = namedtuple("Step", ["node", "acc"])

class Solution:
    def calcEquation(self, equations: List[List[str]], values: List[float], queries: List[List[str]]) -> List[float]:
        graph = defaultdict(dict)
        for (a, b), k in zip(equations, values):
            graph[a][b] = k
            graph[b][a] = 1 / k

        def bfs(start: str, end: str) -> float:
            q = deque([Step(start, 1.0)])
            seen = {start}
            while q:
                step = q.popleft()
                if step.node == end:
                    return step.acc
                for v, w in graph[step.node].items():
                    if v not in seen:
                        seen.add(v)
                        q.append(Step(v, step.acc * w))
            return -1.0

        ans = []
        for x, y in queries:
            if x not in graph or y not in graph:
                ans.append(-1.0)
            elif x == y:
                ans.append(1.0)
            else:
                ans.append(bfs(x, y))
        return ans
```

### 解法 C：带权并查集（weighted union-find）

**思路**

- 每个变量维护 `ratio[x] = x ÷ root(x)`，能互推的并到同组；
- 查询 x/y：同组则 `答案 = ratio[x] / ratio[y]`（同一个根约掉）——类比汇率换算；
- `find`：路径压缩时**先递归、再乘旧父比值**；
- `union(x, y, k)`（x/y = k，把 ry 挂到 rx 下）：由 x = k·y 展开得 `ratio[ry] = ratio[x] / (ratio[y] * k)`；
- 判不可达：`find(x) != find(y)` → -1.0；**天然支持动态加边**。

**完整代码**

```python
from typing import List

class Solution:
    def calcEquation(self, equations: List[List[str]], values: List[float], queries: List[List[str]]) -> List[float]:
        parent = {}
        ratio = {}                          # ratio[x] = x / root(x)

        def find(x):
            if parent[x] != x:
                orig = parent[x]            # 先记旧父节点（关键）
                parent[x] = find(parent[x]) # 路径压缩
                ratio[x] *= ratio[orig]     # 先递归后乘
            return parent[x]

        def union(x, y, k):                 # 已知 x / y = k
            rx, ry = find(x), find(y)
            if rx == ry:
                return
            parent[ry] = rx
            ratio[ry] = ratio[x] / (ratio[y] * k)

        for (x, y), k in zip(equations, values):
            for v in (x, y):
                if v not in parent:
                    parent[v] = v
                    ratio[v] = 1.0
            union(x, y, k)

        ans = []
        for x, y in queries:
            if x not in parent or y not in parent:
                ans.append(-1.0)
            elif find(x) != find(y):        # 不同组 = 不连通
                ans.append(-1.0)
            else:
                ans.append(ratio[x] / ratio[y])
        return ans
```

### 解法 D：Floyd 全源预处理（路径积传递闭包）

**思路**

- 变量 ≤ 40，枚举中间点 k：`i→k→j` 已知则填 `g[i][j] = g[i][k] × g[k][j]`（把 Floyd 的 `+` 换成 `×`）；
- 因自洽，**第一个算出的值就是答案**，无需取最值；
- 预处理 O(V³)，之后每个查询 O(1)。

**完整代码**

```python
from typing import List
from collections import defaultdict

class Solution:
    def calcEquation(self, equations: List[List[str]], values: List[float], queries: List[List[str]]) -> List[float]:
        g = defaultdict(dict)               # g[i][j] = i / j
        for (a, b), k in zip(equations, values):
            g[a][b] = k
            g[b][a] = 1 / k

        for k in list(g):                   # k 作为中间点
            for i in list(g):
                if k not in g[i]:
                    continue
                for j, w in list(g[k].items()):
                    if j not in g[i]:       # i→k→j 推出 i→j
                        g[i][j] = g[i][k] * w

        ans = []
        for x, y in queries:
            if x not in g or y not in g:
                ans.append(-1.0)
            elif x == y:
                ans.append(1.0)
            elif y in g[x]:
                ans.append(g[x][y])
            else:
                ans.append(-1.0)
        return ans
```

**399 各解法易错点**

1. 递归 / BFS 需 `seen` 防环；`seen` 在**进入 / 入队时**标记，避免重复扩展；
2. 用 `-1.0` 作"不可达"哨兵安全（题面保证所有权值为正，路径积恒正）；
3. 并查集两个高危点：`find` 的"先递归后乘"顺序；`union` 的 `ratio[ry]` 公式（建议拿具体数字推一遍再写）；
4. **测试参照本身也会错**（会话踩坑）：随机"给变量赋真实值 → 由真实比值造方程"时，不连通的两个变量**应期望 -1.0**，不能拿全局真实值直接比——参照须独立实现（如 BFS）且语义正确。

**399 复杂度对比**

| 解法 | 预处理 | 单查询 | 特点 |
|---|---|---|---|
| DFS / BFS 逐查询 | 无 | O(V+E) | 最直观 |
| 带权并查集 | O(E·α) | 近 O(1) | 支持动态加边，公式易错 |
| Floyd 全源 | O(V³) | O(1) | V 小才用，查询多时最划算 |

[↑ 返回总览](#overview)

---

<a id="lc547"></a>

## 六、LC 547 Number of Provinces —— 连通分量计数

**原题链接**：[547. Number of Provinces](https://leetcode.cn/problems/number-of-provinces/)

### 题目本质

城市 = 节点，`isConnected[i][j] = 1` 表示 i、j 之间有边（无向、对称、对角线恒 1 = 自环不是边）。问省份数 = **连通分量数**。矩阵本身就是邻接矩阵 → **免建图，直接扫矩阵**；n ≤ 200，三种解法都轻松过。

### 解法 1：DFS（数"启动次数"）

**思路**

维护 `visited`，主循环遍历每个城市，每遇到一个没访问过的城市就从它启动一次 DFS 把整个分量标完，计数器 +1。启动次数 = 分量数。

**完整代码**

```python
from typing import List

class Solution:
    def findCircleNum(self, isConnected: List[List[int]]) -> int:
        n = len(isConnected)
        visited = [False] * n

        def dfs(u: int) -> None:
            visited[u] = True
            for v in range(n):
                if isConnected[u][v] == 1 and not visited[v]:
                    dfs(v)

        count = 0
        for i in range(n):
            if not visited[i]:
                dfs(i)
                count += 1
        return count
```

### 解法 2：BFS

**思路**

外壳与 DFS 完全相同，只是扩散用显式队列；**入队时标记 visited**（不是出队时），防重复入队；无递归栈风险。

**完整代码**

```python
from typing import List
from collections import deque

class Solution:
    def findCircleNum(self, isConnected: List[List[int]]) -> int:
        n = len(isConnected)
        visited = [False] * n

        def bfs(start: int) -> None:
            q = deque([start])
            visited[start] = True
            while q:
                u = q.popleft()
                for v in range(n):
                    if isConnected[u][v] == 1 and not visited[v]:
                        visited[v] = True
                        q.append(v)

        count = 0
        for i in range(n):
            if not visited[i]:
                bfs(i)
                count += 1
        return count
```

### 解法 3：并查集（数"成功合并次数"）

**思路**

初始每个城市自成一省（count = n）；扫矩阵，遇到 1 就合并两点，**只有根不同（真合并）时 count -= 1**；扫完 count 即答案。只扫上三角即可（矩阵对称，跳过自环）。

**完整代码**

```python
from typing import List

class Solution:
    def findCircleNum(self, isConnected: List[List[int]]) -> int:
        n = len(isConnected)
        parent = list(range(n))

        def find(x: int) -> int:
            while parent[x] != x:
                parent[x] = parent[parent[x]]
                x = parent[x]
            return x

        count = n
        for i in range(n):
            for j in range(i + 1, n):
                if isConnected[i][j] == 1:
                    ri, rj = find(i), find(j)
                    if ri != rj:
                        parent[ri] = rj
                        count -= 1
        return count
```

**易错点**

1. 自环（对角线）不是边：并查集扫上三角天然跳过；若全矩阵扫描则 `find(i)==find(i)` 不会误减，但属浪费；
2. **`len(set(parent))` 不能数分量**：parent 里存的不全是根（可能有指向中间节点的值），必须 `len({find(i) for i in range(n)})`（先让每个节点真正 find 一次）；这是 547 特有的并查集陷阱；
3. `find` 两种写法等价：迭代路径减半（上面）与递归完整路径压缩（`find2` 参考写法），后者更直观但深链有递归上限风险；
4. `n = 1` 返回 1：三种解法都自然正确。

**复杂度对比**

| 解法 | 时间 | 空间 | 特点 |
|---|---|---|---|
| DFS | O(n²) | O(n) | 代码最短 |
| BFS | O(n²) | O(n) | 无递归 |
| 并查集 | O(n²·α) | O(n) | 合并计数，支持动态加边 |

**验证**：官方示例（2 / 3 / 1）+ 500 个随机邻接矩阵与独立 BFS 参照一致。

[↑ 返回总览](#overview)

---

<a id="lc684"></a>

## 七、LC 684 Redundant Connection —— 删去多余环边

**原题链接**：[684. Redundant Connection](https://leetcode.cn/problems/redundant-connection/)

### 题目本质

图 = 一棵树（n-1 条边）+ **多加 1 条边** → n 条边、连通、**恰好一个环**（节点 1-indexed）。删掉环上一条边恢复成树；环上有多个候选时，**返回输入里出现最晚的那条**。

### 解法 A：并查集扫描（推荐，已完成）

**思路**

按输入顺序处理边 (u, v)：`find(u) != find(v)` → 正常树边，合并；`find(u) == find(v)` → 两端早已连通，这条边**闭合了环**，直接返回。

为什么"第一个 find 同根的边"就是题意答案：图中只有一个环，环上按输入序最后出现的边处理到时，其余环节点早已连通，因此它**唯一触发**"两端已连通"，天然满足"输入最晚的环边"。

**完整代码**

```python
from typing import List

class Solution:
    def findRedundantConnection(self, edges: List[List[int]]) -> List[int]:
        n = len(edges)
        parent = list(range(n + 1))

        def find(x: int) -> int:
            while parent[x] != x:
                parent[x] = parent[parent[x]]
                x = parent[x]
            return x

        for u, v in edges:
            ru, rv = find(u), find(v)
            if ru == rv:
                return [u, v]
            parent[ru] = rv
```

**易错点**

1. **1-indexed**：`parent` 开 `n + 1`（下标 0 闲置），别开成 n；
2. 返回的是**边** `[u, v]`（保持输入原序），不是节点；
3. 图只有一个环，第一次触发即答案，不要继续扫；
4. 删任意环边都合法（环边不是桥，删后仍连通；唯一环被拆 → 无环 → 树）。

**时空复杂度**：O(n·α) 时间 / O(n) 空间。

### 其余解法（待学，已列入章首"遗留问题"）

- **DFS 找环**：三色 DFS 遇"灰祖先"（跳过父边）得 back edge → 沿 parent 还原环节点 → **倒扫输入**取两端都在环上的第一条边（tie-break 要手动做）；
- **BFS 逐边查连通**（O(n²)）：按输入序维护已接受边，加边前 BFS 查两端是否已连通，连通即答案——语义同并查集，无 tie-break 烦恼；
- **BFS 生成树 + 还原环**（O(n)）：BFS 建生成树，唯一非树边 = 多余边，沿 parent 还原两端树上路径得环节点，再倒扫输入挑边。

**验证**：官方示例（`[2,3]` / `[1,4]`）+ 500 棵随机"树 + 多余边"（n ≥ 3）与独立参照一致。

[↑ 返回总览](#overview)

---

<a id="lc743"></a>

## 八、LC 743 Network Delay Time —— 网络延迟时间

**原题链接**：[743. Network Delay Time](https://leetcode.cn/problems/network-delay-time/)

### 题目本质

有向带权图（边 = `[u, v, w]` 单向，w 为传播延迟，节点 1..n）。信号从 k 沿所有边**同时**扩散，到达节点 x 的时刻 = k 到 x 的**最短路径长度**。答案 = `max(k 到所有点的最短路)`；有节点不可达 → 返回 -1。

> 无权 BFS 不适用（按"跳数"扩散 ≠ 按"时间"扩散，带权会算错）；正确姿势是 **Dijkstra**（可理解为"带权 BFS"：每轮取最小代替普通队列）。DFS 不参与最短路。

### 解法 1：堆版 Dijkstra（自写实现）

**思路**

- `graph = defaultdict(dict)`：`graph[u][v] = w` 记录有向边 u→v 的延迟（只存实际存在的边，稀疏省空间）；
- 优先队列 `queue` 存 `(当前最短时间, 节点)`，`heappop` 每次弹出"当前已知最早到达"的节点——这一步等价于矩阵版里"线性扫描找最小"，只是从 O(n) 变成 O(log n)；
- **弹出即固定**：`dist[u] = d` 后不再改变；同一节点可能被多条路径重复压入，出堆时若已固定就 `continue`（懒删除，对应矩阵版的 `done`）；
- 松弛：遍历 u 的每条出边，若 v 尚未确定，压入 `(d + w, v)`。

**完整代码**

```python
import heapq
from typing import List
from collections import defaultdict

class Solution:
    def networkDelayTime(self, times: List[List[int]], n: int, k: int) -> int:
        graph = defaultdict(dict)
        for u, v, w in times:
            graph[u][v] = w

        dist = {}
        queue = [(0, k)]
        while queue:
            d, u = heapq.heappop(queue)
            if u in dist:
                continue

            dist[u] = d
            for v, w in graph[u].items():
                if v not in dist:
                    heapq.heappush(queue, (d + w, v))

        return max(dist.values()) if len(dist) == n else -1
```

**易错点**

1. **图是有向的**：只填 `graph[u][v]`，不存反向（区别于前面 133/547 的无向图习惯）；
2. `if u in dist: continue` 不能省——同一节点会被多条路径重复压入堆，出堆时若已固定必须跳过；
3. `if v not in dist` 剪枝：已确定的 v 不可能被更晚的 u 改进，不必再压堆；
4. 答案语义：`len(dist) == n` 才代表全部 n 个点都收到信号，否则返回 -1；
5. 松弛逻辑的本质：`candidate = d + w`（先到 u 再走 u→v），比已知的更早才更新——即"借助中转 u 能否更早到 v"。

**时空复杂度**：时间 O(E log V)（每条边至多入堆一次），空间 O(V + E)。

### 解法 2：矩阵版 Dijkstra（对照写法）

**思路**

同一贪心骨架（取最小 + 松弛），只是"取最小"用**每轮线性扫描**实现：维护 `dist` 与 `done` 数组，每轮找未固定中 `dist` 最小的节点并固定，再用矩阵行松弛。n 小或图稠密时 O(n²) 足够，且没有"过期记录"要处理。

**完整代码**

```python
from typing import List
from math import inf

class Solution:
    def networkDelayTime(self, times: List[List[int]], n: int, k: int) -> int:
        graph = [[inf] * (n + 1) for _ in range(n + 1)]
        for u, v, w in times:
            graph[u][v] = w

        dist = [inf] * (n + 1)
        dist[k] = 0
        done = [False] * (n + 1)

        for _ in range(n):
            u = -1
            for i in range(1, n + 1):
                if not done[i] and (u == -1 or dist[i] < dist[u]):
                    u = i

            if u == -1 or dist[u] == inf:
                break

            done[u] = True
            for v in range(1, n + 1):
                if not done[v] and dist[u] + graph[u][v] < dist[v]:
                    dist[v] = dist[u] + graph[u][v]

        ans = max(dist[1:])
        return -1 if ans == inf else ans
```

**时空复杂度**：时间 O(n²)，空间 O(n²)（邻接矩阵）。两个解法已在官方示例与随机图上验证结果一致。

[↑ 返回总览](#overview)

---

## 九、跨题通用方法论

### 1. visited / memo 该放"参数"还是"函数外闭包"？

判断标准是**状态是否需要跨调用存活**，不是调用频繁度：

| 状态性质 | 放哪 | 例子 |
|---|---|---|
| 每次遍历相互独立（需全新） | 参数传新容器（或闭包 + `clear()`） | 399 每个查询 |
| 一次遍历贯穿全局 | 外置闭包即可，无需清理 | 133 克隆整图 |
| 结果要**跨调用缓存复用**（真·记忆化） | 外置持久化，**绝不能清** | 207 的黑节点（清了会退化） |

### 2. "按层批处理"技巧

BFS 逐层处理：每轮先快照本层数量 `cnt = len(queue)`，只处理 cnt 个，新入队元素留到下一轮——310 剥洋葱等"按层收缩"问题都用得上。

### 3. 递归风险与替代

- Python 递归默认上限约 1000：深图 / 长链优先 BFS、显式栈或 Kahn；
- 会话验证：5000 节点链递归 DFS 会 RecursionError，BFS 无压力。
