## 다익스트라 알고리즘이란
음의 가중치가 없는 그래프의 한 정점(Vertex)에서 모든 정점까지의 최단거리를 각각 구하는 알고리즘이다. 동적 계획법을 활용한 대표적인 최단 경로 탐색 알고리즘이다.
흔히 인공위성 GPS 소프트웨어에 등에서 가장 많이 사용된다.

다익스트라 알고리즘이 동적 계획법 문제인 이유는 최단 거리는 여러 개의 최단 거리로 이루어져 있기 때문이다. 하나의 최단 거리를 구할 때, 그 이전까지 구했던 최단 거리 정보를 그대로 사용한다는 특징이 있다.
<br/>

### 동작 방식

1. 출발점으로부터의 최단거리를 저장할 배열 d[v]를 만들고, 출발 노드에는 0을, 출발점을 제외한 다른 노드들에는 매우 큰 값 INF를 채워 넣는다. (정말 무한이 아닌, 무한으로 간주될 수 있는 값을 의미한다.) 보통은 최단거리 저장 배열의 이론상 최대값에 맞게 INF를 정한다. 실무에서는 보통 d의 원소 타입에 대한 최대값으로 설정하는 편이다.

2. 현재 노드를 나타내는 변수 A에 출발 노드의 번호를 저장한다.

3. A로부터 갈 수 있는 임의의 노드 B에 대해, d[A] + P[A][B](A를 거쳐서 B로 가는 최단 거리)와 d[B](현재까지 알려진 최단 거리)의 값을 비교한다. INF와 비교할 경우 무조건 전자가 작다.

4. 만약 d[A] + P[A][B]의 값이 더 작다면, 즉 더 짧은 경로라면, d[B]의 값을 이 값으로 갱신시킨다.

5. A의 모든 이웃 노드 B에 대해 이 작업을 수행한다.

6. A의 상태를 "방문 완료"로 바꾼다. 그러면 이제 더 이상 A는 사용하지 않는다.

7. "미방문" 상태인 모든 노드들 중, 출발점으로부터의 거리가 제일 짧은 노드 하나를 골라서 그 노드를 A에 저장한다.

8. 도착 노드가 "방문 완료" 상태가 되거나, 혹은 더 이상 미방문 상태의 노드를 선택할 수 없을 때까지, 3~7의 과정을 반복한다.

이 작업을 마친 뒤, 도착 노드에 저장된 값이 바로 A로부터의 최단 거리이다. 만약 이 값이 INF라면, 중간에 길이 끊긴 것임을 의미한다.
<br/>

### 백준 1753번 문제
![](https://velog.velcdn.com/images/dodo4723/post/05125dfc-57e9-4622-bfee-cbe9aaa0a956/image.png)
전형적인 다익스트라 알고리즘 문제이다.
위의 동작방식을 적용한 코드는 다음과 같다.
```python
import sys
import heapq
input = lambda : sys.stdin.readline().rstrip()
INF = 987654321

V, E = map(int, input().split())
start = int(input())
graph = [[] for _ in range(V + 1)]
for _ in range(E):
    u, v, w = map(int, input().split())
    graph[u].append((v, w))
distance = [INF] * (V + 1) # 최단거리를 저장할 배열

def dijkstra(start):
    hq = []
    heapq.heappush(hq, (0, start))
    distance[start] = 0
    while hq:
        dist, now = heapq.heappop(hq) # 가장 최단 거리가 짧은 노드에 대한 정보 꺼내기
        if distance[now] < dist:
            continue # 현재 노드의 상태가 방문 완료면 무시
        for i in graph[now]: # 현재 노드와 연결된 다른 인접 노드 확인
            cost = dist + i[1] # 현재 노드를 거쳐서 다른 노드로 이동하는 거리가 더 짧은 경우
            if cost < distance[i[0]]:
                distance[i[0]] = cost
                heapq.heappush(hq, (cost, i[0]))

dijkstra(start)
for i in range(1, V + 1):
    if distance[i] == INF:
        print("INF")
    else:
        print(distance[i])
```
$V$는 노드의 개수이고, $E$는 간선의 개수를 의미한다.
가장 최단 거리가 짧은 노드에 대한 정보를 꺼낼 때, 선형 탐색을 하면 시간 복잡도가 $O(V^2)$지만, 위 코드처럼 힙을 이용하게 되면 최악의 경우에도 시간 복잡도가 $O(ElogV)$를 보장하여 해결할 수 있다.