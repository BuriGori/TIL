# Merge_Sort(합병 정렬)

## 정의
    1부터 N까지의 배열을 반으로 나누어서 정렬한다.
	1개의 값이 남을때까지 나눈 후 합병하면서 정렬하는 방식

## 로직
1. 현재 배열을 반으로 나누어서 합병정렬를 진행한다.
2. 각각의 값만 남을때까지 배열을 나눈다.
3. 인접한 두 배열의 값을 비교하면서 작은값부터 새로운 배열에 채워넣는다.
4. 모든 나눠진 배열을 비교하여 처음과 같은 크기의 배열이 완성되면 정렬된 값을 얻을 수 있다.

## 코드
```c++
#include<bits/stdc++.h>

using namespace std;

vector<int> merge(vector<int> a, vector<int> b) {
	int a_idx=0, b_idx=0;
	vector<int> ret;
	while (a_idx < a.size() && b_idx < b.size()) {
		if (a[a_idx] < b[b_idx])
			ret.push_back(a[a_idx++]);
		else
			ret.push_back(b[b_idx++]);
	}
	while (a_idx < a.size()) {
		ret.push_back(a[a_idx++]);
	}
	while (b_idx < b.size()) {
		ret.push_back(b[b_idx++]);
	}
	return ret;
}

vector<int> merge_sort(int start, int end, vector<int>& v) {
	vector<int> ret;
	if (start >= end) {
		ret.push_back(v[start]);
	
	}
	else {
		int half = (start + end) / 2;
		vector<int> left = merge_sort(start, half, v);
		vector<int> right = merge_sort(half + 1, end, v);
		ret = merge(left, right);
	}
	return ret;
}


int main() {
	vector<int> v = { 8, 5, 3, 1 };
	int n = v.size();
	vector<int> temp = merge_sort(0, n - 1, v);
	for (auto cur : temp)cout << cur << "\n";
}

//출력
1
3
5
8
```