### 백준 1018번 문제

지민이는 자신의 저택에서 MN개의 단위 정사각형으로 나누어져 있는 M×N 크기의 보드를 찾았다. 어떤 정사각형은 검은색으로 칠해져 있고, 나머지는 흰색으로 칠해져 있다. 지민이는 이 보드를 잘라서 8×8 크기의 체스판으로 만들려고 한다.

체스판은 검은색과 흰색이 번갈아서 칠해져 있어야 한다. 구체적으로, 각 칸이 검은색과 흰색 중 하나로 색칠되어 있고, 변을 공유하는 두 개의 사각형은 다른 색으로 칠해져 있어야 한다. 따라서 이 정의를 따르면 체스판을 색칠하는 경우는 두 가지뿐이다. 하나는 맨 왼쪽 위 칸이 흰색인 경우, 하나는 검은색인 경우이다.

보드가 체스판처럼 칠해져 있다는 보장이 없어서, 지민이는 8×8 크기의 체스판으로 잘라낸 후에 몇 개의 정사각형을 다시 칠해야겠다고 생각했다. 당연히 8*8 크기는 아무데서나 골라도 된다. 지민이가 다시 칠해야 하는 정사각형의 최소 개수를 구하는 프로그램을 작성하시오.
<br/>
**입력**

첫째 줄에 N과 M이 주어진다. N과 M은 8보다 크거나 같고, 50보다 작거나 같은 자연수이다. 둘째 줄부터 N개의 줄에는 보드의 각 행의 상태가 주어진다. B는 검은색이며, W는 흰색이다.
<br/>
**출력**
첫째 줄에 지민이가 다시 칠해야 하는 정사각형 개수의 최솟값을 출력한다.
<br/>
이 문제는 브루트포스 알고리즘으로 풀 수 있는 문제이다. 
```java
public class Main {
    static char[][][] chess = new char[2][8][8];
    public static void main(String[] args) throws java.lang.Exception {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
        StringBuilder sb = new StringBuilder();

        StringTokenizer st = new StringTokenizer(br.readLine());
        int N = Integer.parseInt(st.nextToken());
        int M = Integer.parseInt(st.nextToken());
        char[][] board = new char[N][M];

        loadChess();

        for(int i = 0;i < N;i++){
            String input = br.readLine();
            for(int j = 0;j < M;j++)
                board[i][j] = input.charAt(j);
        }

        int min = Integer.MAX_VALUE;
        for(int i = 0;i < N - 7;i++){
            for(int j = 0;j < M - 7;j++) {
                for(int k = 0;k < 2;k++) {
                    int reDraw = 0;
                    for (int a = 0; a < 8; a++) {
                        for (int b = 0; b < 8; b++) {
                            if (board[i + a][j + b] != chess[k][a][b])
                                reDraw++;
                        }
                    }
                    if (reDraw < min)
                        min = reDraw;
                }
            }
        }

        bw.write(min + "");
        bw.flush();
    }

    static void loadChess(){
        for(int i = 0;i < 8;i++){
            for(int j = 0;j < 8;j++) {
                if((i % 2 == 0 && j % 2 == 0) || (i % 2 == 1 && j % 2 == 1)) {
                    chess[0][i][j] = 'B';
                    chess[1][i][j] = 'W';
                }
                else if((i % 2 == 1 && j % 2 == 0) || (i % 2 == 0 && j % 2 == 1)){
                    chess[0][i][j] = 'W';
                    chess[1][i][j] = 'B';
                }
            }
        }
    }
}
```
이렇게 4중for문을 돌려가면서 완전탐색을 실행한다. N,M이 50일때가 경우의 수가 가장 많은데, 43x43x2x8x8 = 236672번이다.
시간복잡도는(N제곱x2x8제곱) = O(N**2)이다. 하지만 8x8이아니라 800x800칸을 살펴봐야 했다면 비록 계수라 할지라도 총 연산 수는 무시하지 못할 정도로 큰 수이므로 완전탐색으로 풀기엔 시간이 촉박했을 것이다.

## 누적합

만약 이 문제가 완전탐색으로 풀 수 없을 정도로 제한시간 대비 N이 크다면 DP의 일종인 누적합이라는 방법을 생각해 볼 수 있다.
7 , 6 , 3 , 2 , 1 배열이 있으면
7, 13, 16, 18, 19 누적합 배열을 만들 수 있다.

[a, b]구간의 값의 합을 구하는 문제가 있다면 a, b까지 반복문으로 합을 구하는 O(N)의 방법이 있지만, 누적합에서 [b] - [a-1]로 O(1)로 구할 수 있다.

2차원 배열에서도 마찬가지로 원리를 적용할 수 있다
![](https://velog.velcdn.com/images/dodo4723/post/922e80a1-e431-47bd-95a3-3ac94761319e/image.png)

왼쪽 아래 2X2의 누적합을 구하려면, 1행과 1열을 빼준다음, 겹치는 구간은 2번 빠지게 되므로 한번 더해주면 된다.

백준 1018번 문제도 누적합으로 풀면 O(N제곱)의 시간복잡도로 해결할 수 있다.
