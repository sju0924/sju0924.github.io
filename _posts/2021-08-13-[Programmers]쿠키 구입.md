---
title: LV4 - 쿠키 구입
categories: [Algorithm]
comments: true
---

## 접근
<p>수의 범위가 얼마 안 되길래 cookies[i][j] 벡터를 만들고, i번부터 j번까지의 부분합을 저장하였다. cookies[j+1] 벡터에 cookies[i][j] 와 같은 값이 있는지 logn 탐색으로 찾고, 만약 있다면 cookies[i][j]를 정답으로 다시 저장하였다. 또한 시간 낭비를 줄이기 위해 cookies[i][j]가 cookies[i][n] 보다 크거나, cookies[i][j]가 기존 정답보다 작다면 탐색을 하지 않았다.</p>

## code
``` c++
#include <string>
#include <vector>
#include<algorithm>
#include<iostream>

using namespace std;

vector<vector<int>> cookies(2001,vector<int>(2001));
int cookie[2001];
int main() {

	int input,n;

	int answer = 0;

	cin >> n;
	for (int i = 1; i <= n;i++) {
		cin >> cookie[i];
	}
	for (int i = 1; i <= n;i++) {
		cookies[i][i] = cookie[i];
		for (int j = i + 1;j <= n;j++) {
			cookies[i][j] = cookies[i][j - 1] + cookie[j];
		}
	}

	for (int i = 1; i <= n;i++) {
		for (int j = i;j < n;j++) {
			if (cookies[i][j] <= answer || cookies[i][j] > cookies[j + 1][n]) continue;

			if (find(cookies[j+1].begin(),cookies[j+1].end(),cookies[i][j]) != cookies[j+1].end()) {
				if (cookies[i][j] > answer) {
					answer = cookies[i][j];
					}
			}
		}
	}
	cout << answer;
}
```