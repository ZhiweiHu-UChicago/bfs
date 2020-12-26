### BFS的另一个应用——Bucket Fill

实际上这道题是要让我们对方块进行填充，对于字母相同，位置相邻（不包括对角线相邻）的方块可以同时用一种颜色填充，最后返回总共需要的颜色。想到我们上次曾经介绍过BFS解迷宫类的题目，这道题同样可以运用BFS的方法，我们将在上次BFS的方法上稍作修改。

```python
from queue import Queue

direction = [(-1, 0), (0, 1), (1, 0), (0, -1)]
visited = set()  # 记录已经走过的坐标


def bfs(x, y, picture):
    n = len(picture)
    m = len(picture[0])
    curr_simbl = picture[x][y]  # get 当前位置的字母
    q = Queue()
    q.put((x, y))  # 首先将起始点坐标放入队列中
    while q.qsize():
        curr_x, curr_y = q.get()  # get 当前坐标
        visited.add((curr_x, curr_y))  # 目前已经所在的坐标已经visited，添加至列表中
        for dx, dy in direction:  # 尝试当前坐标的上下左右四个方向各进一步
            nx = curr_x + dx
            ny = curr_y + dy
            if 0 <= nx < n and 0 <= ny < m 
            and picture[nx][ny] == curr_simbl and (nx, ny) not in visited:
                q.put((nx, ny))


def strokesRequired(picture):
    count = 0
    for i in range(len(picture)):
        for j in range(len(picture[0])):
            if (i, j) not in visited:
                bfs(i, j, picture)
                count += 1
    return count
```

