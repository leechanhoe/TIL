## 스택이란
**스택**은 가장 나중에 들어온 자료가 가장 먼저 처리되는 **LIFO(Last-In-First-Out)** 자료구조이다.
![](https://velog.velcdn.com/images/dodo4723/post/ad244c72-3752-4d31-98f2-d2cb50ef5570/image.png)
백준 17298번 문제는 스택을 이용하여 풀 수 있다.
스택의 생성, 삽입, 삭제, 조회, 응용 등을 알아보자.

![](https://velog.velcdn.com/images/dodo4723/post/77a096db-af3f-4899-8c82-d201b39292a5/image.png)

```python
n = int(input())
seq = list(map(int, input().split()))
stk = [] # 스택 생성
ans = [-1] * n

for i in range(n): # 스택에 큰 수가 밑으로 가게 쌓는다
    while stk and seq[stk[-1]] < seq[i]: # 쌓을 수가 자리에 맞게 아래로 갈 때까지 작은 수들을 pop 해서 스택을 비운다
        ans[stk[-1]] = seq[i] # pop 한 자리는 쌓을 수가 오큰수가 된다.
        stk.pop() # 제거 - 조회만 할때는 stk[-1]
    stk.append(i) # 삽입

print(' '.join(map(str, ans)))
```