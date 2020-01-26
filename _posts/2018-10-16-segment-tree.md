---
title: "세그먼트 트리 (Segment Tree)"
excerpt: ""
categories: algorithms
tags:
  - Segment Tree
last_modified_at: 2018-10-16
---

## 세그먼트 트리 (Segment Tree) 란 ?

세그먼트 트리는 구간합을 binary tree 를 이용해 저장하는 자료구조입니다.
- 부분합을 가장 단순히 저장하는 것은 배열을 이용하는 방법이죠 .. 
- 구간합은 사실 부분합을 이용해서 구할 수 있습니다. 
- ex) 부분합 배열을 이용하여 2 ~ 4 번 합을 구하는 방법은 sum[4] - sum[1] 입니다.

### 배열을 이용하여 부분합을 저장 Example
```java
/** 누적합을 저장할 배열을 세팅 */
int[] getSumed(int[] input){
  int[] sumed = new int[input.lenght];

  sumed[0] = input[0];
  for(int i=1; i<input.lenght; i++){
    sumed[i] = sumed[i - 1] + input[i];
  }

  return sumed;
}
/** 시작, 종료 인덱스를 받아 부분 합을 찾기 */
int getSum(int[] sumed, int start, int end){
    return sumed[end] - sumed[start];
}
```

## 그럼 세그먼트 트리는 언제 필요할 까요 ?
입력받은 배열인 int[] input 의 값이 계속 변한다면 ...?
- 위 처럼 배열을 이용한 경우
1. 원본 배열의 값을 변경 
2. 누적합을 저장한 배열을 변경

### Example
```java
void changeValue(int[] input, int[] sumed, int targetIndex, int newValue){
  // 이전 값과 새로운 값의 차이를 구함 
  int diff = newValue - input[targetIndex];
  input[targetIndex] = newValue;

  // 차이를 누적합 배열에 적용
  for(int i=0; i<=targetIndex; i++){
    sumed[i] += diff;
  }
}
```

- 이런 경우 O(N)의 시간복잡도가 걸리게 됩니다. 따라서 원본 배열의 값이 계속 바뀌는 상황에서는 너무 오랜 시간이 걸리게 됩니다.
- 세그먼트 트리를 이용하면 이진트리의 특성을 살려 해당 연산을 O(logN) 의 시간복잡도를 보장할 수 있습니다.

ex) N = 10인 경우 세그먼트 트리
![No Image](/assets/images/posts/20181016/segment_tree_01.jpeg)

- 각 노드를 탐색함으로써 특정 구간의 부분합 정보를 알 수 있습니다.


### 세그먼트 트리의 특징
1. binary 트리 : 세그먼트 트리는 binary 트리로 O(logN)의 시간복잡도를 보장
- 원본 배열의 길이 N이 2의 제곱수 인 경우, 세그먼트 트리는 Full Binary Tree 로 leaf 노드를 제외한 모든 노드들은 2개의 자식노드가 존재합니다.
- 원본 배열의 길이 N이 2의 제곱수가 아닌 경우, 세그먼트 트리는 Complete Binary Tree 로 마지막 level 을 제외한 모든 노드들이 2개의 자식노드가 존재합니다.
- 원본배열에 삽입, 삭제 연산은 고려하지 않음

2. 각 노드 : 세그먼트 트리에서 각 노드는 배열의 값 or 구간합 정보를 저장
- leaf 노드 : 트리에서 최하위 노드들은 배열의 값을 저장합니다.
- 부 노드 : leaf 노드를 제외한 자식을 갖는 모든 부 노드들은 자노드들이 갖는 구간합 정보를 저장 (두 자식 노드의 합)

3. 트리의 노드의 수
- 세그먼트 트리가 Full Binary Tree 라면, 총 노드 수 = 2 * N - 1
- 세그먼트 트리가 Complete Binary Tree 라면, 총 노드 수 = 2^((logN) + 1) - 1


### 세그먼트 트리 만들기
위에서 보았듯이 세그먼트 트리는 편향 트리가 생기지 않으므로 배열을 이용해서 충분히 구현할 수 있습니다.
여기서는 배열을 이용해서 구현해 보겠습니다. 

우선 세그먼트 트리의 노드를 정의 합니다.
```java
class Node{
	/** 노드의 구간 시작 정보 */
	private int start;
	/** 노드의 구간 종료 정보 */
	private int end;
	/** 노드에 저장할 부분합 */
	private long value;
	
  // 노드의 구간 정보는 1번 세팅하면 변하지 않으므로 생성자에서 세팅 했습니다.
	public Node(int start, int end) {
		this.start = start;
		this.end = end;
	}
	
	public void setValue(long value) {
		this.value = value;
	}
	
	public int getStart() {
		return start;
	}
	public int getEnd() {
		return end;
	}
	public long getValue() {
		return value;
	}
}
```

다음으로 원본 배열을 이용해서 트리를 만드는 내용입니다.
```java

// 입력 배열
long[] input;
// 세그먼트 트리
Node[] tree;

public void init(long[] input){
  this.input = input;
  // height = log2(N)
  int height = (int) Math.ceil(Math.log10(input.length) / Math.log10(2));
  // 2 ^ (height+1)
	int length = (1 << (height + 1));
  tree = new Node[length];
  // 편의를 위해 1번 노드부터 세팅 
  setUp(0, input.length-1, 1);
}

/** 트리 세팅
  * @param start: 노드의 시작 구간  
  * @param end: 노드의 종료 구간 
  * @param nodeIndex: 세그먼트 트리에서 현재 노드의 인덱스 
  * @return nodeValue: 노드의 값 
  */
private long setUp(int start, int end, int nodeIndex) {
  // 해당 노드의 구간정보를 설정하여 생성 
  tree[nodeIndex] = new Node(start, end);
  
  if(start == end) {// leaf  노드 인 경우 값 복사  
    tree[nodeIndex].setValue(input[start]);
  }else {
    int m = (start + end) / 2;
    // 자노드 저장 
    long left = setUp(start, m, nodeIndex * 2);
    long right = setUp(m+1, end, nodeIndex * 2 + 1);
    tree[nodeIndex].setValue(left + right);
  }
  
  return tree[nodeIndex].getValue();
}
```
외부에서 init() 메소드를 호출하면 세그먼트 트리의 총 노드 수를 계산하고 setUp() 메소드를 재귀적으로 호출하여 세그먼트 트리의 각 노드들에 부분합을 저장하는 소스입니다.
핵심이되는 setUp() 메소드는 크게 2가지로 구현됩니다.
- start = end : 해당 노드가 leaf 노드인 경우로 입력 값 자체를 저장합니다.
- start != end : 해당 노드가 자식 노드가 존재하는 경우로 왼쪽 자노드와 오른쪽 자노드의 합을 저장합니다.

setUp() 메소드를 root 노드부터 수행 시키면 모든 노드를 세팅할 수 있습니다.
setUp() 메소드는 노드의 수 만큼 수행되므로 시간복잡도는 최대 트리의 길이인 O(2^((logN) + 1)) 가 됩니다.


### 검색
2 ~ 4 까지 합을 구하는 경우
![No Image](/assets/images/posts/20181016/segment_tree_02.jpeg)

[left, right] 구간의 합을 검색하려고 합니다. 이때 특정 노드에 방문했을때 크게 4가지 경우로 나뉘게 됩니다.
1. [left, right] 구간이 현재 노드의 구간과 겹치지 않는 경우 
- 현재 노드의 구간이 [0, 1] 이라면 구간이 겹치지 않으므로 탐색을 종료하고 0을 리턴하면 됩니다.
2. [left, right] 구간이 현재 노드의 구간을 포함하는 경우
- 현재 노드의 구간이 [3, 4] 이라면 해당 노드의 구간이 포함되므로 현재 노드의 값을 리턴하면 됩니다.
3. [left, right] 구간이 현재 노드의 구간에 포함되는 경우
- 현재 노드이 구간이 [0, 9] 인 경우 구간 합을 알 방법이 없습니다. 따라서 왼쪽 자노드와 오른쪽 자노드를 탐색하여 합을 구합니다.
4. [left, right] 구간이 현재 노드의 구간과 겹치는 경우
- 현재 노드의 구간이 [0, 2] 인 경우 2부분이 겹치므로 왼쪽 노드와 오른쪽 노드를 탐색하여 값을 더합니다.


```java
/** 트리에서 부분합 찾기
  * @param left: 찾으려는 시작 구간   
  * @param right: 찾으려는 종료 구간  
  * @param nodeIndex: 세그먼트 트리에서 현재 노드의 인덱스 
  * @return nodeValue: 노드의 값 
  */
public long getSum(int left, int right, int nodeIndex) {
  Node node = tree[nodeIndex];
  
  // 찾으려는 범위가 현재 노드의 구간에 포함되지 않은 경우
  // left > 노드의 종료구간  or right < 노드의 시작구간  
  if(left > node.getEnd() || right < node.getStart()) {
    return 0;
  }
  
  // 찾으려는 범위가 노드의 범위를 포함하는 경우 
  if(left <= node.getStart() && right >= node.getEnd()) {
    return node.getValue();
  }
  
  //+ 찾으려는 범위가 노드의 범위에 포함되는 경우 
  //+ 두 범위가 겹쳐지는 경우
  // 왼쪽 자노드 탐색 + 오른쪽 자노드 탐색 
  long leftSum = getSum(left, right, nodeIndex * 2);
  long rightSum = getSum(left, right, nodeIndex * 2 + 1);
  
  return leftSum + rightSum;
}
```

### 업데이트 
값을 변경하는 경우, 변경할 값이 있는 leaf 노드의 모든 부모노드들을 업데이트 해야합니다.
ex) 5 번 인덱스의 값을 변경하는 경우
![No Image](/assets/images/posts/20181016/segment_tree_03.jpeg)

```java
/** 외부에서 호출할 업데이트 메소드 
  * @param index : 값을 변경할 입력 배열의 인덱스
  * @param newValue : 변경할 새로운 값 
  */
public void update(int index, long newValue) {
  long diff = newValue - input[index];
  input[index] = newValue;

  treeUpdate(index, ROOT_INDEX, diff);
}

/** 트리를 순회하면서 업데이트 수행 
  *  @param index : 값을 변경할 입력 배열의 인덱스
  *  @param nodeIndex : 현재 노드의 인덱스 
  *  @param diff : 원래 값과 새로운 값의 차이 (newValue - input[index]) 
  */
private void treeUpdate(int index, int nodeIndex, long diff) {
  Node node = tree[nodeIndex];
  //  해당 노드의 범위에 찾으려는 인덱스가 포함되지 않는 경우 
  if(node.getStart() > index || node.getEnd() <index) {
    return;
  }
  
  node.setValue(node.getValue() + diff);
  // 업데이트 할 노드들이 더 있는 경우 
  if(node.getStart() != node.getEnd()) {
    // 왼쪽 노드 업데이트 
    treeUpdate(index, nodeIndex * 2, diff);
    // 오른쪽 노드 업데이트
    treeUpdate(index, nodeIndex * 2 + 1, diff);
  }
}
```

해당 글은 백준님의 세그먼트 트리 게시물을 읽고 작성하였습니다.
- [백준님의 posting 세그먼트 트리](https://www.acmicpc.net/blog/view/9)

### 전체 소스
- [백준 2042번 문제](https://www.acmicpc.net/problem/2042)
- [세그먼트 트리](https://github.com/kimkoungho/BaekJoonOnlineJudge/blob/master/src/Problem_2042/SegmentTree.java)

## Reference
- [백준님의 posting 세그먼트 트리](https://www.acmicpc.net/blog/view/9)
- [위키 이진트리](https://ko.wikipedia.org/wiki/%EC%9D%B4%EC%A7%84_%ED%8A%B8%EB%A6%AC)