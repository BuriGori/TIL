# 2342번 Dance Dance Revolution

> 문제

https://www.acmicpc.net/problem/2342

> 조건

DDR판이 있을때, 처음에는 가운데에 두 발이 있고 위 아래 좌 우로 발을 옮기면서 게임을 진행해야한다. 중앙에서 이동할 때는 2의 힘을 사용하고 인접한 방향으로 이동할 때는 3의 힘을 반대편으로 이동할 때는 4의 힘을 사용한다고 하였을 떄 게임을 완료하는 최소한의 힘을 구하여라

> 접근법

왼발과 오른발이라는 선택지와 위 아래 좌 우의 길이 존재한다. 여러가지 존재를 비교한 후 최소의 값을 구해야하기 때문에 겹치는 상황을 대비해 **DP**의 **메모라이제이션**을 이용하면 쉽게 답을 구할 수 있을 것이다.

> 코드

 ``` c++
#include<bits/stdc++.h>

using namespace std;

int dp[101010][5][5];
int step = 0;
vector<int> vec;
// 0 == 가운데, 1 == 위, 2 == 좌, 3 == 아래, 4 == 우
//이동하기 전과 후의 위치를 비교하여 점수를 측정
int score(int n, int a) {
	if (n == a)return 1;
	if (n == 0)return 2;
	if (abs(n - a) == 2)return 4;
	return 3;
}
// DDR dfs
int ddr(int s, int l, int r) {
    //마지막 스텝이라면 끝
	if (s == step)return 0;

    //현재 위치에 대한 기록이 있으면 끝
	if (dp[s][l][r])return dp[s][l][r];

    //왼쪽과 오른쪽으로 갔을 때의 값을 계산
	int Rscore = score(r, vec[s]) + ddr(s + 1, l, vec[s]);
	int Lscore = score(l, vec[s]) + ddr(s + 1, vec[s], r);

    //작은 값을 return
	return dp[s][l][r] = min(Rscore, Lscore);
}

int main() {
	while (1) {
		int a; cin >> a;
		if (a == 0)break;
		vec.push_back(a);
	}
	step = vec.size();
	int ans =ddr(0, 0, 0 );
	cout << ans;
}
```

> 배운 점

DDR이라고 해서 상하좌우, 양발을 사용하는 문제여서 어려워 보일 수 있지만 코드를 보면 알 수 있듯이 어느정도의 로직을 생각하면 DP로 쉽게 풀 수 있다. 따라서, 로직을 생각하고 DP에 저장할 인자들을 어떻게 할지 고민해야한다.