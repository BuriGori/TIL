# Bubble_Sort(버블 정렬)

## 정의
    현재 위치부터 N까지의 값을 모두 탐색한 뒤 최솟값을 Key값으로 "선택"
	현재 위치 값과 교체한다.

## 로직
1. Key값은 처음부터 시작하여 배열의 끝까지 탐색한다.
2. 현재의 Key값 위치에서부터 N번까지 탐색한다.
3. 값이 Key값보다 작다면 최솟값으로 기억한다.
4. 현재의 Key값을 최솟 값의 위치과 변경한뒤 오른쪽으로 이동후 2번으로 돌아간다.

## 코드
```c++
#include<bits/stdc++.h>

using namespace std;

void selection_sort(vector<int> &v) {
	int n = v.size();
	for (int i = 0; i < n; i++) {
		int key = v[i];
		int key_idx = i;
		for (int j = i; j < n; j++) {
			if (key > v[j]) {
				key = v[j];
				key_idx = j;
			}
		}
		swap(v[i], v[key_idx]);
	}
}

int main() {
	vector<int> v = { 8, 5, 3, 1 };
	selection_sort(v);
	for (auto cur : v)cout << cur << "\n";
}

//출력
1
3
5
8
```