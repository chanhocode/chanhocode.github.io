---
title: "[Algorithm] 6가지 정렬 알고리즘 한번에 끝내기"
writer: chanho
date: 2025-04-07 22:00:00 +0900
categories: [CoreCS, Algorithm]
tags: [cs, algorithm, theory]
---

> 업무를 하다보니 생각보다 자료구조나 알고리즘을 고민하게 되는 일이 많아졌다. 이에 머리도 풀겸 일주일에 한번 정도는 알고리즘을 다시 공부하는 시간을 가지는게 좋을 것 같다는 생각이 들어 공부를 다시 시작해보고자 한다. 먼저, 기억을 되살리기 위해 정렬 부터 탐색, 재귀, 그리드, DP, 그래프 정도 한 주에 하나씩 이론 부터 정리하고자 하며, 그 이후에는 백준 문제를 가볍게 풀어보면 어떨까 생각 중이다.

---

## Background

### 시간 복잡도 간단한 정리

| 이름 | 표기 | 간단 설명 |
| --- | --- | --- |
| 상수 시간 | O(1) | 한번에 바로 찾음 |
| 로그 시간 | O(log n) | 절반 씩 줄여가며 찾음 |
| 선형 시간 | O(n) | 한번 씩 다 확인 |
| 선형 로그 | O(n log n) | 한번 씩 보지만, 정리할 때는 절반씩 |
| 제곱 시간 | O(n^2) | 두 번 반복 |

### 정렬 알고리즘 비교 표

| 알고리즘 | 평균 시간 복잡도 | 공간 복잡도 | 안정성 | 특징 |
| --- | --- | --- | --- | --- |
| 버블 정렬 | O(n^2) | O(1) | O | 구현이 쉬움 |
| 선택 정렬 | O(n^2) | O(1) | X | 교환 적음 |
| 삽입 정렬 | O(n^2) | O(1) | O | 거의 정렬된 경우에 좋음 |
| 병합 정렬 | O(n log n) | O(n) | O | 안정적, 메모리 사용 |
| 퀵 정렬 | O(n log n) | O(log n) | X | 매우 빠름 |
| 힙 정렬 | O(n log n) | O(1) | X | 힙 사용 |
| 계수 정렬 | O(n + k) | O(k) | O | 정수에 만 사용 가능 |

---

## 버블 정렬

> 인접한 두 값을 비교해서 큰 값을 뒤로 보내는 방식의 정렬 알고리즘이다. 이 과정을 반복하면 가장 큰 값이 맨 뒤로, 두 번째 큰 값이 그앞에 이런 식으로 정렬이 된다. 마치 비누방울이 위로 올라가는 것처럼, 큰 수가 뒤로 이동한다고 해서 버블 정렬이라고 불린다고 한다. 효율이 ‘O(n^2) 안좋다 보니, 실무에서 사용될 일은 거의 없을 것으로 보인다.

```cpp
#include <iostream>
#include <vector>
using namespace std;

void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    bool swapped;

    for (int i = 0; i < n - 1; i++) {
        swapped = false;
        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                // swap
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = true;
            }
        }
        if (!swapped) break;
    }
}
```

## 선택 정렬

> 가장 작은 값을 선택해서 앞으로 보내는 정렬 방법이다. 배열 전체에서 최소값을 찾은 후에 맨 앞의 값과 위치를 교환하고, 그 다음 위치부터 반복하면서 정렬 하는 방식이다. 이 방식도 버블 정렬 처럼 ‘O(n^2)’의 시간 복잡도를 가지며 단순하지만 비효율적 이여서 마찬가지로 학습용 외에는 실무에서 사용할 일은 없을 것 같다.

```cpp
void selectionSort(vector<int>& arr) {
    int n = arr.size();

    for (int i = 0; i < n - 1; i++) {
        int minIndex = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIndex]) {
                minIndex = j;
            }
        }
        // swap
        int temp = arr[i];
        arr[i] = arr[minIndex];
        arr[minIndex] = temp;
    }
}
```

## 삽입 정렬

> 삽입 정렬은 마치 카드 정렬처럼 손에 든 카드를 하나씩 왼쪽에 정렬된 위치에 ‘삽입’하는 방식이다. 즉, 리스트의 왼쪽부터 차례로 값을 하나씩 가져와서 이미 정렬된 부분(왼쪽)에 적절한 위치를 찾아 앞으로 밀면서 삽입해 나간다. 평균 적으로는 ‘O(n^2)’을 가지지만 이미 정렬된 경우 ‘O(n)’을 가진다.

- swap이 아닌 이동(shift), 값을  덮어쓰면서 shift하고 나중에 한 번만 삽입

```cpp
#include <iostream>
#include <vector>
using namespace std;

void insertionSort(vector<int>& arr) {
    int n = arr.size();

    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

## 병합 정렬

> 병합 정렬은 ‘Devide and Conquer: 분할 정복)’ 기법을 사용하는 정렬이다. 배열을 절반으로 재귀적으로 분할하고 나누어진 배열들을 정렬된 상태로 병합한다. 즉, 정렬된 작은 조각들이 합쳐지면서 정렬된 큰 배열을 만들어 낸다. 시간 복잡도는 ‘O(n log n)’으로 항상 일정하다. 다만 공간 복잡도가 O(n)으로 메모리를 추가로 사용하는 점이 있다. 실무에서는 주로 안정 정렬이 필요할 때나 메모리 외부 정렬시 사용할 수 있는 것으로 알고 있다.

- 병합 하는 과정에서 모든 요소를 정확히 한 번씩만 본다. ‘O(n)’
- 분할 단계에서는 배열을 절반씩 쪼개면 깊이 ‘log n’만큼 재귀 호출이 생긴다.
- 즉, O(n log n)

```cpp
#include <iostream>
#include <vector>
using namespace std;

void merge(vector<int>& arr, int left, int mid, int right) {
    vector<int> L(arr.begin() + left, arr.begin() + mid + 1);
    vector<int> R(arr.begin() + mid + 1, arr.begin() + right + 1);

    int i = 0, j = 0, k = left;
    while (i < L.size() && j < R.size()) {
        if (L[i] <= R[j])
            arr[k++] = L[i++];
        else
            arr[k++] = R[j++];
    }
    while (i < L.size()) arr[k++] = L[i++];
    while (j < R.size()) arr[k++] = R[j++];
}

void mergeSort(vector<int>& arr, int left, int right) {
    if (left >= right) return;

    int mid = (left + right) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}
```

## 퀵 정렬

> 퀵 정렬은 `피벗(pivot)`이라는 기준값을 정해서, 작은 값은 왼쪽으로 큰 값은 오른 쪽으로 분할하고 그 후, 왼쪽과 오른쪽을 다시 퀵 정렬로 정렬하는 방식이다. 퀵정렬은 이름처럼 빠른 속도로 인해 실무에서도 많이 사용하는 정렬 방식이다.

- 데이터가 많고 빠르게 정렬해야 할 때 사용
- C/C++의 ‘std::sort()’기본 구현
- 공간복잡도는 o(log n)
- 같은 값의 순서가 바뀔 수 있는 불안정 정렬이다.

```cpp
#include <iostream>
#include <vector>
using namespace std;

int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[high]; // 마지막 요소
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
        }
    }
    int temp = arr[i + 1];
    arr[i + 1] = arr[high];
    arr[high] = temp;
    return i + 1;
}

void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pivotIndex = partition(arr, low, high);
        quickSort(arr, low, pivotIndex - 1);
        quickSort(arr, pivotIndex + 1, high);
    }
}
```

## 힙 정렬

> 힙(Heap) 정렬은 힙 자료구조를 기반으로 하는 정렬이다. 여기서 힙 자료구조란, 완전 이진 트리 형태를 가진다. 메모리 제한이 심한 임베디드 시스템에서 주로 사용되며, 우선순위 큐로 정렬된 데이터를 출력할 때 사용한다.

- 최대 힙(Max Heap): 부모가 항상 자식 보다 크다.
- 최소 힙(Min Heap): 부모가 항상 자식 보다 작다.
- 시간 복잡도: O(n log n)
- 공간 복잡도: O(1)
    - 불안정 정렬

`정렬 방식`

1. 주어진 배열을 최대 힙으로 만든다.
2. 가장 큰 값(루트)을 배열 끝으로 보내고,
3. 남은 힙을 다시 재정렬을 반복 한다.

```cpp
#include <iostream>
#include <vector>
using namespace std;

void heapify(vector<int>& arr, int n, int i) {
    int largest = i;
    int l = 2 * i + 1;
    int r = 2 * i + 2;

    if (l < n && arr[l] > arr[largest])
        largest = l;
    if (r < n && arr[r] > arr[largest])
        largest = r;
    if (largest != i) {
        int temp = arr[i];
        arr[i] = arr[largest];
        arr[largest] = temp;
        heapify(arr, n, largest);
    }
}

void heapSort(vector<int>& arr) {
    int n = arr.size();
    
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);

    for (int i = n - 1; i >= 0; i--) {
        int temp = arr[0];
        arr[0] = arr[i];
        arr[i] = temp;
        heapify(arr, i, 0);
    }
}
```

- [4, 10, 3, 5, 1]
1. 최대 힙으로 만들기 → [10, 5, 3, 4, 1]
2. 10과 마지막 요소 1을 교환 → [1, 5, 3, 4, 1]
3. 다시 힙 재정렬 → [5, 4, 3, 1, 10]
4. 5와 1 교환 → [1, 4, 3, 5, 10]
5. (반복)

## 계수 정렬

> 마지막으로 조금 특별한 정렬을 알아보고 마무리 하고자 한다. 계수 정렬은 비교를 하지 않고, 데이터의 ‘개수’를 세어서 정렬하는 방식이다. 모든 데이터를 직접 비교하는 대신 ‘몇개 있는지’를 세는 배열을 만들어 정렬한다. 이러한 방식 때문에 반드시 데이터가 정수여야 하며, 범위가 작고 양의 정수여야 효율적이다. 점수나 등급, 나이를 정렬하는 경우 사용할 수 있을 것 같다. 메모리를 더 쓰고 범위 제약이 존재하지만 매우 빠른 정렬 방식이다.

- 시간 복잡도: O(n + k), k는 최대 값
- 공간 복잡도: O(k)

```cpp
#include <iostream>
#include <vector>
using namespace std;

vector<int> countingSort(vector<int>& arr) {
    if (arr.empty()) return {};

    int maxVal = *max_element(arr.begin(), arr.end());
    vector<int> count(maxVal + 1, 0);
    for (int num : arr)
        count[num]++;
    vector<int> result;
    for (int i = 0; i <= maxVal; i++) {
        for (int j = 0; j < count[i]; j++)
            result.push_back(i);
    }

    return result;
}
```
