# Bubble_Sort(버블 정렬)

## 정의
    피벗(Pivot)이라는 기준점을 가지고 양 옆으로 나누어 정렬을 한다.
	피벗보다 큰 것은 오른쪽, 작은 것은 왼쪽으로 옮기는 정렬

## 로직
1. 배열의 첫번째 값을 피벗으로 잡는다.
2. 처음부터 끝까지 값들을 비교하면서(투 포인터) 피벗을 기준으로 양옆을 나눈다.
3. 양 옆을 나눠진 배열들은 각각 다시 버블정렬을 진행한다.
4. 1~3을 반복하면 정렬된 배열을 얻을 수 있다.

## 코드
```c++
#include<bits/stdc++.h>

using namespace std;

int quick(int left, int right, vector<int>& v) {
	int pivot = v[left];
	int l = left;
	int r = right;
	while (l < r) {
		while (pivot < v[r]) {
			r--;
		}
		while (l < r && pivot >= v[l]) {
			l++;
		}
		swap(v[l], v[r]);
	}
	swap(v[l], v[left]);
	return l;
}

void quick_sort(int left, int right, vector<int> &v) {
	if (left >= right)
		return;
	int pivot = quick(left, right, v);
	quick_sort(left, pivot - 1, v);
	quick_sort(pivot + 1, right, v);
	return;
}

int main() {
	vector<int> v = { 8, 5, 3, 1 };
	int n = v.size();
	quick_sort(0,n-1,v);
	for (auto cur : v)cout << cur << "\n";
}
//출력
1
3
5
8
```