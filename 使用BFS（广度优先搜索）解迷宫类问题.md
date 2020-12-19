### 使用BFS（广度优先搜索）解迷宫类问题

在这学期的Computing for Finance in Python上有这么一道算法题：

![Screen Shot 2020-12-19 at 12.48.09 PM](/Users/huzhiwei/Library/Application Support/typora-user-images/Screen Shot 2020-12-19 at 12.48.09 PM.png)

简单来说，输入一个list of lists，对于每一个位置，用0表示可经过的方块，用2表示金币，用1表示没有办法经过的blocks。要在获取到所有金币的情况下，找出从B(0,0) $\rightarrow$ A的最短路径。如果这个路径不存在，则返回-1.

##### **Sample Case 1**

![Screen Shot 2020-12-19 at 12.49.13 PM](/Users/huzhiwei/Library/Application Support/typora-user-images/Screen Shot 2020-12-19 at 12.49.13 PM.png)

##### **Sample Case 2**

![Screen Shot 2020-12-19 at 12.49.43 PM](/Users/huzhiwei/Library/Application Support/typora-user-images/Screen Shot 2020-12-19 at 12.49.43 PM.png)

### ==Solutions 1：==

（感谢同专业的**许卓**大佬提供的代码，我的代码只能通过一半的testcase，匿了）

同时解释一下queue这个模块，queue.Queue( ) 是一个先进先出的队列，常用的方法有：

| Methods  |                         Explanations                         |
| :------: | :----------------------------------------------------------: |
| Queue( ) |        创建队列，创建时可指定maxsize，队列的最大容量         |
|  put( )  | 在队列中插入元素，元素容量达到上限时，继续往队列中放入数据会引发 queue.Full 异常 |
|  get( )  | 在队列中取出元素，队列中没有数据元素时，取出队列中的数据会引发 queue.Empty 异常 |
| qsize()  |                   返回队列中数据元素的个数                   |

```python
from queue import Queue

# 首先构造一个BFS方法
# Bob 可以走的四个方向，分别是向左、向上、向右、向下
direction = [(-1,0),(0,1),(1,0),(0,-1)]

def bfs(x,y,target_x,target_y,maze): # 起点(x,y) 终点(target_x, target_y)，以及需要走的maze迷宫矩阵
  	n = len(maze) # 迷宫的行数
    m = len(maze[0]) # 迷宫的列数
    distance = [[-1]*m for _ in range(n)] # 一个相同的 n*m 的distance矩阵，初始各元素定为-1
    distance[x][y] = 0
    if x==target_x and y==target_y:
        return 0
    q = Queue()
    q.put((x,y)) # 首先将起始点坐标放入队列中
    while q.qsize():
        curr_x, curr_y = q.get() # get 当前坐标
        for dx, dy in direction: # 尝试当前坐标的上下左右四个方向各进一步
            nx = curr_x + dx
            ny = curr_y + dy
            if 0<=nx<n and 0<=ny<m and maze[nx][ny] != 1 and distance[nx][ny] == -1:
              # 确保当前坐标(curr_x, curr_y)不会走出maze的范围，不会碰到bloc
              # 以及这个点之前没有走过 (该点坐标在distance矩阵中对应的值是-1）
                distance[nx][ny] = distance[curr_x][curr_y] + 1  # 满足if条件即可通行，distance +1
                if nx == target_x and ny == target_y:
                    return distance[nx][ny]
                q.put((nx,ny)) 
                # 如果当前点还不是终点，那么将该点坐标放入队列作为备选点之一,下一轮for循环会以这个点为基础上下左右各走一步继续尝试，如果依然没有出界&没有撞墙，放到队列中作为备选点，继续上下左右尝试
                # 直到尝试完左右的备选点，q.qsize() 为 0
    return -1

  
# 构造完BFS方法后，再来考虑“拿到所有金币”这样一个constraint
def minMoves(maze, x, y):
    # Write your code here
    coins = []
    for i in range(len(maze)):
        for j in range(len(maze[0])):
            if maze[i][j] == 2:
                coins.append((i,j)) # 找出maze中所有的金币的坐标
    startX = 0
    startY = 0
    endX = x
    endY = y
    num_coins = len(coins)
    if num_coins == 0:
        return bfs(startX,startY,endX,endY,maze)
    
    dist = [[-1]*(num_coins+2) for _ in range(num_coins)] # 构造一个(num_coins+2)*(num_coins)的矩阵
    
    for i in range(num_coins):
        for j in range(num_coins):
            dist[i][j] = bfs(coins[i][0],coins[i][1],coins[j][0],coins[j][1],maze)
        		# 给定i，填充dist[i][j]为第i+1号金币分别到第j（0，1，2，3...）+1号金币的最短路径距离
        dist[i][num_coins] = bfs(startX,startY,coins[i][0],coins[i][1],maze) 
        # 对于[i][num_coins]，填充起点到第i+1号金币的最短路径距离
        dist[i][num_coins+1] = bfs(coins[i][0],coins[i][1],endX,endY,maze)
        # 对于[i][num_coins+1]，也就是第i行的最后一列，填充第i+1号金币到终点的最短路径距离
    
    # 如果从起点出发，永远无法碰到某一个金币；或者从某一个金币出发，永远无法到达终点，返回-1
    for i in range(num_coins):
        if dist[i][num_coins] == -1 or dist[i][num_coins+1] == -1:
            return -1 
          
    # dynamic programming
    # << : bit-wise operator, 1<<n = 2^n
    # & : bit-wise and
    # | : bit-wise or
    # 用二进制串，0 表示金币没有被捡到 1 表示金币被捡到
    # 例如 0010001101 表示总共有10个金币，其中第1，3，4，8个金币被捡到
    dp = [[-1]*num_coins for _ in range(1<<num_coins)] # 10000...000 行，num_coins 列的矩阵
    for i in range(num_coins):
        dp[1<<i][i] = dist[i][num_coins] 
        # dp[1<<i][i] 代表从起点出发，刚刚好捡到了第（i+1）号金币的最短路径
        # 实际上就是起点到第i个金币的最短路径，也即是 dist[i][num_coins] 
        
    for mask in range(1, (1<<num_coins)): 
    # mask 将从 0000...00001 开始循环到1111...11111，模拟从捡到第一个金币到最终所有金币都捡到的情况
        for i in range(num_coins):
            if mask & (1<<i) != 0: # 第（i+1）个金币捡到了
                for j in range(num_coins):
                    if mask & (1<<j) == 0: # 第（j+1）个金币没有捡到
                        next_mask = mask | (1<<j) # 【原先的mask】+【第j个金币捡到了】的二进制串
                        if dp[next_mask][j] == -1 or dp[next_mask][j] > dp[mask][i] + dist[i][j]:
                            dp[next_mask][j] = dp[mask][i] + dist[i][j]
                            # 原先的mask是第（i+1）号金币捡到了的，现在基于这个基础上，去找捡到了next_mask也即又多捡了一个（j+1）号金币的最短路径，那么一定是遍历i，找dp[mask][i]+dist[i][j]的最小值
    print(dist)
    res = -1
    last_mask = (1<<num_coins) - 1 #最终的mask是1111111，全部金币都捡到的情况
    for i in range(num_coins):
        if res == -1 or res > dp[last_mask][i] + dist[i][num_coins+1]:
            res = dp[last_mask][i] + dist[i][num_coins+1]
    
    return res
```

### ==**Solution 2**==

上面代码的第二部分，利用二进制串进行动态规划可能有些复杂，这里提供另一种方法，同样感谢同专业的**Vera Chim**

```python
from itertools import permutations

def bfs(maze, coin, i, dd): # coin is the list of coordinate tuples of coins, i is its index
    queue = [] # 用来记录备选坐标，便于下一次基于这个坐标再向四个方向进行遍历
    queue.append((coin[i], 0)) 
    records = set() # 用来记录所有访问过的坐标
    golds = 0 # 累计得到的金币
    r = len(maze)
    c = len(maze[0])
    while len(queue) > 0 and golds < len(coin): # 要么走完所有可能路径，要么已搜集到所有金币，停止循环
        (x, y), step = queue[0]
        queue.remove(((x, y), step))
        if (x, y) in coin and (x, y) not in records: # 如果当前坐标上有金币，且之前没走过
            dd[i][coin.index((x, y))] = step # 对应dd的位置赋值为 step
            golds += 1
        records.add((x, y))
        if maze[x][y] != 1: # 不会撞墙
            if x+1 < r and (x+1, y) not in records and ((x+1, y), step+1) not in queue:
                queue.append(((x+ 1, y), step + 1)) # 向右走一步，不越界，之前没走过，相同step不走重
                
            if x-1 >= 0 and (x - 1, y) not in records and ((x - 1, y), step + 1) not in queue:
                queue.append(((x- 1, y), step + 1)) # 向左走一步，不越界，之前没走过，相同step不走重
                
            if y + 1 < c and (x, y + 1) not in records and ((x, y + 1), step + 1) not in queue:
                queue.append(((x, y + 1), step+1)) # 向上走一步，不越界，之前没走过，相同step不走重
                
            if y - 1 >= 0 and (x, y-1) not in records and ((x, y - 1), step + 1) not in queue:
                queue.append(((x, y-1), step + 1)) # 向下走一步，不越界，之前没走过，相同step不走重

def minMoves(maze, x, y):
    # 找出所有的金币坐标，外加原点（0，0）
    r = len(maze)
    c = len(maze[0])
    coin = [(0, 0)]
    for i in range(r):
        for j in range(c):
            if maze[i][j] == 2:
                coin.append((i, j))
    coin.append((x, y))
    
    dd = [[-1] * len(coin) for _ in range(len(coin))] # 以金币列表的（长度+1）边长的矩阵，元素为-1
    points = []
    for i in range(len(coin)):
        bfs(maze, coin, i, dd) 
        # 用bfs方法来修改dd这个矩阵，每一个i的循环结束时，dd矩阵第j行第i个元素代表，从第j个金币出发，得到其余第i个金币对应的步数
        # 注意我们的金币列表是加上了起点和终点的
        if i > 0 and i < len(coin) - 1: # 除去起点和终点，中间金币点的数量
            points.append(i)
    all_path = list(permutations(points)) # 生成获取中间金币，不同先后顺序的排列组合
    res = -1
    for path in all_path:
        sum = 0
        current = 0 # 先从原点开始，依次访问排列组合中的金币点p
        for p in path:
            if dd[current][p] == -1: # 说明存在从一个金币出发无法获取另一个金币的情况，这样金币拿不满，-1
                return -1
            sum += dd[current][p] # 累计步数直到最后一个金币点
            current = p # 继续迭代,move on to the next point with coin
        if dd[current][len(coin) - 1] == -1: # 如果最后一个金币点到终点走不通，返回-1
            return -1
        sum += dd[current][len(coin) - 1] # 再在总路径中加上最后一个金币点到终点的步数
        if res == -1:
            res = sum # 一开始res会是-1，第一次赋值，让res等于我们得到的第一个步骤和sum
        elif res > sum: # 之后每次走完一个不同金币的排列组合，都会产生一个新的sum，最终取最小的步骤和sum
            res = sum
    return res
```

---

