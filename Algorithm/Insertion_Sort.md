# Insertion_Sort(삽입 정렬)

## 정의
    현재의 키를 정렬된 배열의 각 원소와 비교하고
    자신의 위치를 찾아 삽입해 정렬을 완성하는 알고리즘이다.

## 로직
1. Key값은 두번째 위치에서부터 시작하여 배열의 끝까지 탐색한다.
2. 현재의 Key값 위치에서부터 왼쪽의 값들은 정렬된 값이므로 Key값 왼쪽부터 시작하여 1번까지 탐색한다.
3. 값이 Key값보다 크다면  탐색하는 위치의 값과 자리를 바꾼 후 왼쪽으로 탐색을 진행한다.
4. 첫 번째 값까지 왔다면 다시 1번으로 돌아간다.

## 코드
```c++
#include<bits/stdc++.h>

using namespace std;

void insert_sort(vector<int> &v) {
	int n = v.size();
	for (int i = 1; i < n; i++) {
		for (int j = i; j > 0; j--) {
			if (v[j] < v[j - 1])swap(v[j], v[j - 1]);
		}
	}
}

int main() {
	vector<int> v = { 8, 5, 3, 1 };
	insert_sort(v);
	for (auto cur : v)cout << cur << "\n";
}

//출력
1
3
5
8
```