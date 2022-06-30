## 투 포인터 알고리즘
1차원 배열이 있고, 이 배열에서 각자 다른 원소를 가리키고 있는 2개의 포인터를 조작해가면서 답을 얻는 알고리즘이다.

### 백준 1806번 문제
![](https://velog.velcdn.com/images/dodo4723/post/6ca92553-3fec-4969-b285-47e85e4e556c/image.png)

백준 1806번 문제를 풀면서 알아보자면,


- 포인터 2개를 준비한다. 시작과 끝을 알 수 있도록 start, end 라고 한다.
- 맨 처음에는 start = end = 0이며, 항상 start<=end을 만족해야 한다.
- start=end 일 경우 그건 크기가 0인, 아무것도 포함하지 않는 부분 배열을 뜻한다. 다음의 과정을 start < N인 동안 반복한다.

만약 현재 부분합이 M 이상이거나, 이미 end = N이면 start를 1 늘리고, 그렇지 않다면 end를 늘린다.

```python
N, S = map(int, input().split())
arr = list(map(int, input().split()))

psum = [0] * (N + 1) # 누적합(prefix sum)
for i in range(1, N + 1):
    psum[i] = psum[i-1] + arr[i-1]

start = 0 # 포인터 2개를 준비한다. 시작과 끝을 알 수 있도록 start, end 라고 한다.
end = 0 # 맨 처음에는 start = end = 0이며, 항상 start <= end를 만족해야 한다.
ans = 987654321
while start < N: # 만약 현재 부분합이 S 이상이거나, 이미 end = N이면
    if psum[end] - psum[start] >= S: 
        ans = min(ans, end - start)
        start += 1 # start를 1 늘리고
    else:
        if end == N:
            start += 1
        else:
            end += 1 # 그렇지 않다면 end를 늘린다
if ans == 987654321:
    ans = 0
print(ans)
```
매 루프마다 항상 두 포인터 중 하나는 1씩 증가하고 있고, 각 포인터가 N번 누적 증가해야 끝나므로 투 포인터 알고리즘의 시간복잡도는 $2N = O(N)$이다.