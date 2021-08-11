---
title: boj9025 - Widest path
categories: [Algorithm]
comments: true
---

## 접근
<p>다익스트라를 변형해서 풀었다. 노드 수를 n이라 할 때, d[n]에는 노드 n까지 오는 경로의 가장 넓은 경로를 저장하고, 인접 노드를 갱신할 때에는 원래 노드의 경로의 넓이와 원래 노드와 인접 노드 사이의 엣지의 넓이를 고려하여서, 더 작은 값을 갱신하였다. </p>

## code
``` c++
#include <iostream>
#include<vector>
#include<algorithm>
#include<queue>
using namespace std;
int d[1001];
int P[1001][1001];
bool searched[1001];
int n, m, s, t, T;
int cur;
vector<vector<int>>g;
void cleard() { // 배열들 초기화
	for (int i = 0; i <= 1000;i++) {
		d[i] = -1;
		searched[i] = 0;
		for (int j = 0; j <= 1000;j++) {
			P[i][j] = 0;
		}
	}
}
int main()
{
	ios::sync_with_stdio(false);
	cin.tie(NULL);
	cout.tie(NULL);
	cin >> T;

	int start, end, width;
	vector<int>blank;
	
	for (int i = 0; i < T; i++) {
		cin >> n >> m >> s >> t;
		d[s] = 0;
		g = vector<vector<int>>(n + 1);

    //트리 만들기, P[i][j] 입력
		for (int j = 0; j < m; j++) {
			cin >> start >> end >> width;
			
			g[start].push_back(end);
			g[end].push_back(start);
			P[start][end] = width;
			P[end][start] = width;
			if (start == s) {
				d[end] = width;
			}
			else if (end == s) {
				d[start] = width;
			}
		}

    //제일 넓은 경로 찾기
		int curNode = s, nextNode = 0;
		
		for (int src = 0; src < n; src++) {
			searched[curNode] = 1;
				
			for (int n = 0; n < g[curNode].size();n++) { 
		
				if (g[curNode][n] == s) continue;
        //전 노드까지의 경로보다 현재 추가하는 노드로 가는 간선이 더 넓은 경우, 전 노드까지의 경로의 넓이를 추가 노드까지의 경로의 넓이로 설정.
				if (d[curNode] < P[curNode][g[curNode][n]]) { 
					if (d[g[curNode][n]] < d[curNode]) {
						d[g[curNode][n]] = d[curNode];
					}
				}
        //전 노드까지의 경로보다 현재 추가하는 노드로 가는 간선이 더 좁은 경우, 현재 추가하는 노드로의 간선의 넓이를 추가 노드까지의 경로의 넓이로 설정.
				else if (d[curNode] >= P[curNode][g[curNode][n]]) {
					if(d[g[curNode][n]] < P[curNode][g[curNode][n]]) {
						d[g[curNode][n]] = P[curNode][g[curNode][n]];
					}
				}
			}
			
      //다음으로 갈 노드 찾기 : 탐색하지 않은 노드들 중 경로가 제일 넓은 노드
			nextNode = 1;
			while (searched[nextNode]) {
				nextNode++;
			}

			for (int next = nextNode; next <= n; next++) {
				if (searched[next] || d[nextNode] > d[next]) continue;
				nextNode = next;
			}
			curNode = nextNode;
				
		}

		cout << d[t] << "\n";
		g.clear();
		}
		
		
}
```