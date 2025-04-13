---
title: "[Algorithm] 탐색 알고리즘 한번에 끝내기"
writer: chanho
date: 2025-04-12 21:00:00 +0900
categories: [CoreCS, Algorithm]
tags: [cs, algorithm, theory]
---

> 오늘은 탐색 알고리즘을 정리해보고자 한다. 오늘 정리할 알고리즘은 “이진 탐색” 알고리즘 부터 그래프를 사용하는 “DFS & BFS” 까지 정리하겠다.


## 이진 탐색

> 이진탐색 알고리즘(Binary Search)은 탐색 알고리즘의 가장 기초 알고리즘으로, 정렬된 데이터에서 빠르게 원하는 값을 찾을 수 있는 알고리즘이다. 동작 방식은 리스트의 가운데 값을 기준으로 찾는 값을 왼쪽/오른쪽 중 어디에 있는지를 판단하며 반씩 줄여가며 탐색을 수행한다.
- 정렬된 리스트에서만 사용 가능
- 시간 복잡도: O(log n)

```cpp
#include <iostream>
#include <vector>
using namespace std;

int binarySearch(vector<int>& arr, int target) {
    int left = 0, right = arr.size() - 1;

    while (left <= right) {
        int mid = (left + right) / 2;

        if (arr[mid] == target)
            return mid;
        else if (arr[mid] < target)
            left = mid + 1;
        else
            right = mid - 1;
    }
    return -1;
}

int main() {
    vector<int> data = { 1, 3, 5, 7, 9, 11 };
    cout << binarySearch(data, 7);
}
```

1. 가운데 값은 5로 타겟(7)은 오른쪽에 있다.
2. 오른쪽 절반 [7, 9, 11] 가운데 는 9로 왼쪽에 타겟이 있음
3. [7] 로 단, 3번 만에 타겟을 찾음.

## 그래프

> 그래프(Graph)는 노드(node)와 간선(edge)으로 이루어진 자료구조이다.

![image](https://github.com/user-attachments/assets/397f6a72-b5f5-4244-b883-bced92bd891f)

- Node: 1, 2, 3, 4, 5, 6, 7
- Edge
    - 1 - 2
    - 1 - 3
    - 2 -4
    - 2- 5
    - (중략)
    - 6 - 7

### 구현

| **방식** | **공간 복잡도** | **간선 여부 확인** | **인접 정점 탐색** | **장점** | **단점** |
| --- | --- | --- | --- | --- | --- |
| 인접 리스트 | O(V + E) | O(degree) | 빠름 | 희소 그래프에 적합 | 연결 여부 확인이 느릴 수 있음 |
| 인접 행렬 | O(V^2) | O(1) | O(V) | 연결 여부 확인 빠름 | 공간 낭비 가능 |
| 간선 리스트 | O(E) | O(E) | 느림 | MST, 정렬이 필요한 알고리즘 | 직접 탐색 비효율적 |

1. 인접 리스트(Adjacency List)

> 각 정점에 연결된 정점들의 리스트를 저장

```python
graph = {
    'A': ['B', 'C'],
    'B': ['A', 'D'],
    'C': ['A'],
    'D': ['B']
}
```

1. 인접 행렬(Adjacency Matrix)

> 정점을 행과 열로 나타내고, 간선이 있으면 1 또는 가중치를 표시, 없으면 0또는 무한
> 

```python
graph = [
    [0, 1, 1],
    [1, 0, 0],
    [1, 0, 0]
]
```

1. 간선 리스트(Edge List)

> 모든 간선을 (시작, 끝) 또는 (시작, 끝, 가중치) 형태로 나열
> 

```python
edges = [
    ('A', 'B'),
    ('A', 'C'),
    ('B', 'D')
]
```

## DFS: Depth-First Search

> DFS는  한 방향으로 끝까지 파고드는 탐색 방식이다. 그래프 또는 트리를 탐색할 때, 이어진 부분을 계속 진입하며 탐색을 하다가 끝이 나면 다시 분기 되었던 부분으로 돌아가 다시 탐색을 이어서 하는 방식이다. 스택을 기반으로 동작하며 일반적으로 재귀함수로 구현할 수 있다.

![image](https://github.com/user-attachments/assets/9425d06f-78eb-4842-acdc-531e7bf10175)

위 같은 그림에서, DFS를 수행하면 “1 → 2 → 4 → 3 → 5”순서로 탐색을 한다.

- 시간 복잡도: O(V + E)
- 공간 복잡도: O(V)

```cpp
#include <iostream>
#include <vector>
using namespace std;

vector<vector<int>> graph = {
    {}, // 0은 사용하지 않음
    {2,3},
    {4},
    {5},
    {},
    {}
};

vector<bool> visited(6, false);

void dfs(int v) {
    visited[v] = true;
    cout << v << " ";

    for (int neighbor : graph[v]) {
        if (!visited[neighbor])
            dfs(neighbor);
    }
}
```

## BFS: Breadth-First Search

> 가까운 곳부터 하나씩 넓게 퍼지듯 탐색하는 방식으로, DFS가 깊이를 우선 했다면 BFS는 너비를 우선으로 하여 가까운 노드부터 차례 차례 탐색을 수행한다. 큐(Queue) 자료구조를 사용하여 구현한다.

![image](https://github.com/user-attachments/assets/9425d06f-78eb-4842-acdc-531e7bf10175)

위 같은 그림에서, BFS를 수행하면 “1 → 2 → 3 → 4 → 5”순서로 탐색을 한다.

- 시간복잡도: O(V + E)
- 공간복잡도: O(V)

```cpp
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

void bfs(vector<vector<int>>& graph, int start) {
    vector<bool> visited(graph.size(), false);
    queue<int> q;

    q.push(start);
    visited[start] = true;

    while (!q.empty()) {
        int v = q.front();
        q.pop();
        cout << v << " ";
        
        for (int neighbor : graph[v]) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                q.push(neighbor);
            }
        }
    }
}
```

## **DFS vs BFS 핵심 비교**

| 항목 | DFS | BFS |
| --- | --- | --- |
| 방식 | 한 방향으로 끝까지 | 가까운 노드부터 차례대로 |
| 자료구조 | 스택 (재귀) | 큐 |
| 경로 탐색 | 전체 탐색에 유리 | **최단 거리 탐색**에 유리 |
| 순서 예시 | 1 → 2 → 4 → 3 → 5 | 1 → 2 → 3 → 4 → 5 |

## **DFS / BFS 사용하기 좋은 상황**

| 상황 | 추천 탐색 | 이유 |
| --- | --- | --- |
| **최단 거리 필요 (미로, 게임 맵)** | BFS | **거리 1, 2, 3… 순서대로 탐색** |
| **경로의 존재만 판단** | 둘 다 가능 | 빠르게 구현 가능한 DFS도 무방 |
| **모든 경로/조합 탐색** | DFS | **백트래킹**에 유리 |
| **메모리 적고 깊은 노드까지 필요** | DFS | BFS는 큐가 커질 수 있음 |

> BFS는 시작점에서부터 1칸 거리, 2칸 거리… 순으로 탐색하기 때문에 가장 먼저 도착한 노드는 “최단 거리”에 있는 노드임이 보장되어 최단 거리를 찾을때 유리 하다.
