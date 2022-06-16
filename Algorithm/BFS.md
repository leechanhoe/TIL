## 너비 우선 탐색(BFS, Breadth-First Search)이란
루트 노드(혹은 다른 임의의 노드)에서 시작해서 인접한 노드를 먼저 탐색하는 방법이다.

시작 정점으로부터 가까운 정점을 먼저 방문하고 멀리 떨어져 있는 정점을 나중에 방문하는 순회 방법이다.

즉, 깊게(deep) 탐색하기 전에 **넓게(wide)** 탐색하는 것이다.
![](https://velog.velcdn.com/images/dodo4723/post/dbc48c10-3a12-4e5f-ae0a-aa95e80d6192/image.png)
<br/>

### 사용하는 경우
- 두 노드 사이의 최단 경로 구하기
- 임의의 경로 찾기
<br/>

### 특징
- 어떤 노드를 방문했었는지 여부를 반드시 검사 해야 한다.
- 방문한 노드들을 차례로 저장한 후 꺼낼 수 있는 자료 구조인 큐(Queue)를 사용한다.
- ‘Prim’, ‘Dijkstra’ 알고리즘과 유사하다.
- 시간 복잡도는 정점 개수를 V, 간선 개수를 E 라고 했을 때 인접 행렬을 사용시 $O(V^2)$, 인접 리스트 사용시 $O(V+E)$이다
<br/>

### 예시 : 백준 6593번 문제
![](https://velog.velcdn.com/images/dodo4723/post/5030f0f1-2dc9-4690-88bd-a7116ac1fe3d/image.png)
전형적인 길찾기 문제이며 최단거리 탐색을 해야 하므로 BFS를 사용할 수 있다.
```python
from collections import deque
import sys

input = sys.stdin.readline
move = [(1, 0, 0), (-1, 0, 0), (0, 1, 0), (0, -1, 0), (0, 0, 1), (0, 0, -1)]

def is_valid_loc(z, y, x): # 유효 위치 검사
    return 0 <= z < L and 0 <= y < R and 0 <= x < C 

while 1:
    L, R, C = map(int, input().strip().split())
    if not L:
        break

    b = [[] for _ in range(L)] # 빌딩을 3차원 배열로 입력받음
    visited = [[[False] * C for _ in range(R)] for __ in range(L)]
    start = end = (-1, -1, -1, 0)
    for i in range(L):
        for j in range(R):
            b[i].append(input().strip())
            if 'S' in b[i][j]:
                start = (i, j, b[i][j].find('S'), 0)
            elif 'E' in b[i][j]:
                end = (i, j, b[i][j].find('E'), 0)
        input().strip()

    dq = deque()
    dq.append(start)
    visited[start[0]][start[1]][start[2]] = True
    escaped = False
    while dq:
        z, y, x, t = dq.popleft()
        if z == end[0] and y == end[1] and x == end[2]:
            print(f"Escaped in {t} minute(s).")
            escaped = True
            break

        for dir in move:
            nz = z + dir[0]
            ny = y + dir[1]
            nx = x + dir[2]
            if is_valid_loc(nz, ny, nx) and (b[nz][ny][nx] == '.' or b[nz][ny][nx] == 'E') and not visited[nz][ny][nx]:
                visited[nz][ny][nx] = True
                dq.append((nz, ny, nx, t + 1))
                
    if not escaped:
        print("Trapped!")
```