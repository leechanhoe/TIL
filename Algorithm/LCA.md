# 최소 공통 조상이란?

**최소 공통 조상(LCA, Lowest Common Ancestor)** 은 트리 구조에서 임의의 두 정점이 갖는 가장 가까운 조상 정점을 말한다.

<br/>
<br/>

## 최소 공통 조상을 구하는 방법

<br/>

### LCA를 선형 탐색으로 구하기 : $O(Depth)$
두 노드의 레벨이 같으면 부모 노드를 같은 횟수로 거슬러 올라가면 되지만, 레벨이 다르면 동시에 거슬러 올라가기 전, 두 정점의 깊이를 동일하게 맞춰야 한다.

<br/>

### 구현 - 백준 11438번 문제
![](https://velog.velcdn.com/images/dodo4723/post/f88bd97f-bc15-4b71-9c83-bf434293a149/image.png)
```python
import sys
input = sys.stdin.readline

for _ in range(int(input())):
    N = int(input())
    parents = [0] * (N + 1)
    for _ in range(N-1):
        p, c = map(int, input().split())
        parents[c] = p

    a, b = map(int, input().split())
    a_parent = [a]
    b_parent = [b]
    # 루트까지 탐색하며 경로저장
    while parents[a]:
        a_parent.append(parents[a])
        a = parents[a]

    while parents[b]:
        b_parent.append(parents[b])
        b = parents[b]

    a_level = len(a_parent) - 1
    b_level = len(b_parent) - 1
    # 루트부터 시작하여 공통 조상이 다를때까지 탐색
    while a_parent[a_level] == b_parent[b_level]:
        a_level -= 1
        b_level -= 1
    
    print(a_parent[a_level+1])
 ```
 루트 노드까지 거슬러 올라가며 경로를 저장한 뒤, 루트부터 경로를 확인하며 다른 경로가 나올때까지 탐색하면, 답을 구할 수 있다.
 
하지만, 더 효율적인 방법이 있다.

<br/>
<br/>

### LCA를 이분 탐색으로 구하기 : $O(log(Depth))$

다이나믹 프로그래밍을 이용한 전처리를 통해 LCA를 찾는 시간 복잡도를 **O(log Tree_heigth)** 으로 줄일 수 있다.

**x의 2^k(2의 k승)번째 조상을 parent[x][k]라고 하자.**

**k=0** 이라면, 2^0 = 1으로 x의 부모 노드이다.
**k=1** 이라면, 2^1 = 2으로 x의 두번째 조상, 즉 부모 노드의 부모로 아래와 같이 나타낼 수 있다.
parent[x][1] = parent[parent[x][0]][0]

**k=2** 이라면, x의 4번째 조상으로 x의 두 번째 조상(parent[x][1])의 두 번째 조상이다.
parent[x][2] = parent[parent[x][1]][1]

<br/>

따라서 2^k번째 조상은 **parent[x][k] = parent[parent[x][k-1]][k-1]**로 나타낼 수 있다.

이 2제곱수 부모 정보를 이용하여 LCA를 구할 수 있다.

### 구현 - 백준 11438번 문제
![](https://velog.velcdn.com/images/dodo4723/post/c0731cad-4395-4396-b011-16ef18df5fe9/image.png)

#### 문제를 푸는 과정
1. 트리 간선들을 입력받아 부모 자식관계 연결한다
2. 트리의 높이(최대깊이)를 구한다
3. parent 배열을 초기화한다
4. parent 배열을 활용하여 두 노드의 깊이를 맞춰준다
5. parent 배열과 이분탐색을 활용하여 두 노드의 LCA를 구한다

자세한 설명은 코드 내에 작성했다.

```cpp
#include<iostream>
#include<vector>
#include<cstdio>
#include<string.h>
using namespace std;

int N;
int parent[100001][18]; // [x][y] x의 2^y째 부모
int depth[100001]; // 깊이
int max_height; // 트리의 최대깊이
vector<int> adjection[100001]; // 인접 그래프

void FindParentDfs(int par, int now, int dep){ // 부모 자식 관계를 설정해주는 함수
    if(adjection[now].size() == 0)
        return;
    
    parent[now][0] = par; // 부모노드
    depth[now] = dep;

    for(int i = 0; i < adjection[now].size(); i++){
        if(adjection[now][i] != par) // 부모노드가 아닌건 다 자식노드로 추가함
            FindParentDfs(now, adjection[now][i], dep + 1);
    }
}

int FindLCA(int a, int b){
    if(depth[a] != depth[b]){ // 두 노드의 깊이를 맞춰줌
        if(depth[a] < depth[b]) // 다르면 항상 a가 더 깊게하기
            swap(a, b);
    }
                                    // 13은 이진수로 1101 - 0, 2, 3자리수에 1이있음
    int dif = depth[a] - depth[b]; // 깊이 맞추기 parent[x][13] = parent[parent[parent[x][0]][2]][3]
    for(int i = 0;dif > 0;++i){ // 차이가 7(111)이면 1 + 2 + 4 올라가고 8(1000) 이면 0 + 0 + 0 + 8 올라감
        if(dif % 2 == 1) // 이때 단순히 dif 번 조상으로 올라가는 경우, 최악의 경우 
            a = parent[a][i]; // 치우처진 트리의 리프 노드에서 루트 노드까지 올라가야 할 수 있다.
        dif = dif >> 1; // 따라서 -1 말고 /2로
    }

    if(a != b){ // 만약 parent[a][k] == parent[b][k] 이지만, parent[a][k-1] != parent[b][k-1] 이라면
        for(int k = max_height;k >= 0;k--){ // LCA는 2^(k-1)번째 조상에서 ~ 2^k 번째 조상의 사이에 존재한다.
            if(parent[a][k] != 0 && parent[a][k] != parent[b][k]){ // (LCA 이후의 조상들도 모두 공통될 수 밖에 없기 때문)
                a = parent[a][k]; // 13번(1101)째 조상이 LCA일 경우 각 자리수(3,2,0) 마다 실행
                b = parent[b][k]; // k = 3일때 + 8, k = 2 일떄 + 4 해서 12번째 조상에서 반복을 멈춘다  
            } // 맨 마지막 자리수인 + 1 까지 하면 공통 조상이 되버리기 때문에 +1은 추가안됨 그래서
        } // 맨마지막에 + 1 해주기
        a = parent[a][0]; //
    }
    return a;
}

int main(){
    scanf("%d", &N);
    memset(adjection, 0, sizeof(adjection));
    memset(parent, 0, sizeof(parent));

    int a, b;
    for(int i = 0;i < N-1;++i){
        scanf("%d %d", &a, &b);
        adjection[a].push_back(b);
        adjection[b].push_back(a);
    }

    FindParentDfs(0, 1, 0);

    int temp = N, cnt = 0; // log(N) 을 알아내기
    while(temp > 1){
        temp = temp >> 1;
        ++cnt;
    }
    max_height = cnt;

    for(int k = 1;k <= max_height;++k) // 2의 제곱수마다 초기 설정
        for(int idx = 2;idx <= N;++idx)
            if(parent[idx][k-1] != 0)
                parent[idx][k] = parent[parent[idx][k-1]][k-1];

    int M; 
    scanf("%d", &M);
    while(M--){ // M 입력받기
        scanf("%d %d", &a, &b);
        printf("%d\n", FindLCA(a, b));
    }
    return 0;
}
```

어제 KMP 알고리즘을 공부할때 어느정도 이해해서 풀기까지 5시간 정도가 걸렸는데, 오늘 이 알고리즘도 3~4시간은 걸렸다.

플래티넘 수준의 문제부터는 확실히 이해하기까지 시간이 오래 걸리는 것 같다.

그리고 이제까지 파이썬으로 알고리즘 공부를 했지만, 고급 알고리즘들은 무겁고 시간이 오래걸리는 파이썬으로 푸는데 한계가 느껴진다. 그래서 전역 후 처음으로 근 2년만에 c++을 다뤄봤다.

오랜만에 다뤄보지만 학교다닐때 제일 많이 공부했던 언어가 C시리즈(C, C++, C#)인지라 머릿속에 꽤 남아있어 다행이다.