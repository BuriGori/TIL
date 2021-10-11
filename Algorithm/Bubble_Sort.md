# Bubble_Sort(버블 정렬)

## 정의
    key값에 인접한 값을 비교하여 정렬하는 알고리즘이다.
	크기 순서대로 되어있지 않다면 교환.

## 로직
1. Key값은 처음부터 시작하여 배열의 끝까지 탐색한다.
2. 현재의 Key값 위치에서부터 남은 번호수만큼 탐색한다.
3. 값이 Key값보다 다음 값이 작다면 교환
4. 끝까지 탐색을 했다면 현재 값을 끝 위치에 저장하고 남은 번호수를 하나 줄여서 2번으로 돌아간다.

## 코드
```c++
#include<bits/stdc++.h>

using namespace std;

void bubble_sort(vector<int> &v) {
	int n = v.size();
	for (int i = n-1; i > 0; i--) {
		for (int j = 0; j < i; j++) {
			if (v[j] > v[j + 1])swap(v[j], v[j + 1]);
		}
	}
}

int main() {
	vector<int> v = { 8, 5, 3, 1 };
	bubble_sort(v);
	for (auto cur : v)cout << cur << "\n";
}

//출력
1
3
5
8
```