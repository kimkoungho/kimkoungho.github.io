---
title: "팬윅트리 (fenwick tree)"
excerpt: ""
categories: algorithms
tags:
  - Fenwick Tree
last_modified_at: 2018-10-20
---


# 팬윅트리 (BIT : Binary Indexed Tree)

팬윅트리는 부분합을 이진트리에 저장하고 update 연산을 효율적으로 계산할 수 있는 자료구조입니다.
- 1994년 Peter Fenwick이 산술 코딩 압축 알고리즘을 개선하기 위해서 제안했습니다.


## 펜윅 트리 vs 세그먼트 트리

### 기존의 세그먼트 트리 
![No Image](/assets/images/posts/20181020/segment_tree.jpeg)
위 그림에서 처럼 세그먼트 트리는 [2, 8] 구간의 합을 구하기 위해 [2] + [3, 4] + [5, 8] 이용해서 부분합을 효율적으로 구할수 있었습니다.
하지만 세그먼트 트리의 노드 수는 2^((logN) + 1) - 1 이기 때문에 입력 데이터 배열의 길이에 영향을 받습니다.

사실 세그먼트 트리에서는 연산을 이용하여 유추할 수 있는 값들을 모두 저장하고 있습니다.
- 부분합 배열을 이용해서 구간합을 구하는 방법을 이용하는 것을 생각하면 쉽습니다. 
- [2] = [1, 2] - [1]
- [3, 4] = [1, 4] - [1, 2]
- [5, 8] = [1, 8] - [1, 4]

따라서 펜윅 트리를 이용하면 메모리를 절약하는 효과를 볼 수 있습니다.


### 펜윅 트리로 변형
![No Image](/assets/images/posts/20181020/fenwick_tree_01.jpeg)
위 그림 처럼 세그먼트 트리에서 각 부모노드의 오른쪽 자노드를 제거하여 펜윅 트리를 표현할 수 있습니다.

### 펜윅 트리의 특징 
1. 펜윅 트리는 입력 배열의 각 인덱스에 부분합 or 원본 값을 저장합니다.
- 입력 배열의 길이와 동일한 길이의 배열을 이용함으로 공간 복잡도에서 세그먼트 트리보다 효율적입니다.
2. 비트 연산을 이용
- 각 인덱스를 2진수로 변형했을때 마지막 1의 위치에 따라서 해당 노드의 구간 정보를 유추할 수 있습니다.
- 1(0001), 3(0011), 5(0101), 7(0111) : 입력 배열의 원본 값
- 2(0010), 6(0110) : 2개의 원소의 합 
- 4(0100) : 4개의 원소의 합
- 위 처럼 1의 위치에서 따라서 해당 노드들이 몇개의 원소의 합을 저장하는지 알 수 있습니다.

### 펜윅 트리에서 사용할 배열 초기화
```java
/** 팬윅 트리를 저장할 1차원 배열 */
private long[] tree;

public FenwickTree(int size) {
    // 제약 인덱스 1부터 시작 
    tree = new long[size];
}
```

### 검색
1 ~ 7 까지 합을 구하는 경우
![No Image](/assets/images/posts/20181020/fenwick_tree_02.jpeg)
[1, 7] 합은 [1, 4] + [5, 6] + [7] 로 표현할 수 있습니다.
이것은 실제 펜윅 트리가 저장된 배열(T)에서 T[4] + T[6] + T[7] 로 표현할 수 있습니다.
- 위 그림처럼 1이 있는 가장 오른쪽 비트를 빼서 다음 인덱스를 유추할 수 있습니다.
- 7(0111) => 6(0110) => 4(0100) 
- 이제 가장 오른쪽 비트를 제거 하는 방법을 봅시다.
- 여기서 힌트는 보수 인데요. 먼저 7(0111)의 1의 보수는 1000 이죠. 2의 보수는 1을 더한 -7(1001) 입니다.
- 7(0111) & -7(1001) = 1(0001) 이 됩니다.
- 따라서 특정 값의 2의 보수를 AND 연산하여 빼주면 ... 7 - (7 & -7) => 6 - (6 & -6) => 4 - (4 & -4) => 0

2 ~ 7 까지 합은 어떻게 구할까요 ?
[2, 7] 합은 [1, 4] + [5, 6] + [7] - [1] 로 표현할 수 있습니다.
위에서 계산했던 T[4] + T[6] + T[7] - T[1] 입니다.
- 배열에 부분합을 계산했던 것 처럼 [start, end] 구간의 합은 [1, end] - [1, start] 로 표현가능합니다.
- 대충 감이 오시겠지만 1이 있는 가장 오른쪽 비트를 빼는 규칙은 1 ~ n 까지 합을 구할때에만 이용할 수 있습니다.
- 따라서 [1, n] 까지의 구간합은 T[n] + T[n & -n] + ... + T[m] 으로 표현할 수 있습니다. (m 은 0이 아닌 임의의 자연수)

```java
/** 1 ~ index 까지의 합을 구하는 메소드
 * @param index : 합을 구할 마지막 인덱스 
 * @return sum : 1 ~ index 까지의 합  
 */
private long getSum(int index) {
    long sum = 0;
    while(index > 0) {
        sum += tree[index];
        // index 의 2진수화 후에 가장 오른쪽 1의 위치를 찾아서
        // index 에서 가장 오른쪽 1의 위치를 지운다 
        index -= (index & -index);
    }
    return sum;
}

/** [start, end] 구간의 합을 구하는 메소드 
 * @param start : 구간의 시작 인덱스 
 * @param end : 구간의 종료 인덱스
 * @return sum : [start, end] 구간의 합  
 */
public long getSum(int start, int end) {
    // 1 ~ end
    long totalSum = getSum(end);
    // 1 ~ (start-1)
    long subSum = getSum(start-1);
    
    return totalSum - subSum;
}
```


### 수정
5번째 원소 값을 수정하는 경우
![No Image](/assets/images/posts/20181020/fenwick_tree_03.jpeg)
5번째 원소를 수정할 경우, 함께 변경되어야 하는 값은 T[5], T[6], T[8] 입니다.
- 5(0101) => 6(0110) => 8(1000) 
- 부분합을 검색할 때와는 반대로 1이 있는 가장 오른쪽 비트에 1을 더해주면 됩니다.
- 5 + (0101 & 1011) => 6 + (0110 & 1010) => 8 

```java
/** 트리에서 index 포함한 구간의 값을 모두 업데이트   
 * @param index : 원본 배열의 인덱스
 * @param diff : 원래 값과의 차이 
 */
public void update(int index, long diff) {
    while(index < tree.length) {
        tree[index] += diff;
        // index 에서 가장 오른쪽 1의 위치에 1을 더함  
        index += (index & -index);
    }
}
```

해당 글은 백준님의 세그먼트 트리 게시물을 읽고 작성하였습니다.
- [백준님의 posting 펜윅트리](https://www.acmicpc.net/blog/view/21)

### 전체 소스
- [백준 2042번 문제](https://www.acmicpc.net/problem/2042)
- [팬윅 트리](https://github.com/kimkoungho/BaekJoonOnlineJudge/blob/master/src/Problem_2042/FenwickTree.java)


## Reference
- [백준님의 posting 펜윅트리](https://www.acmicpc.net/blog/view/21)
- [crocus 블로그 팬윅트리](https://www.crocus.co.kr/666)
- [위키 팬윅트리](https://en.wikipedia.org/wiki/Fenwick_tree)
- [삼성 소프트웨어 맴버쉽 binary index tree](http://secmem.tistory.com/486)

