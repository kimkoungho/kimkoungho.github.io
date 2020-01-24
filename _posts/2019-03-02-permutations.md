---
title: "Permutations(순열) algorithms"
excerpt: ""
categories: algorithms
tags:
  - permutation
  - permutation algorithms
last_modified_at: 2019-03-09
---

# Permutations(순열) algorithms

순열은 일반적으로 서로 다른 n개 원소 중에서 r개를 선택하여 순서를 고려하여 나열한 것.
여기서는 특정 n 개의 원소를 모두 이용하여 조합가능한 **모든 순열 구하기** 에 대해서 설명함.

[A, B, C, D] 라는 원소를 이용하여 가능한 모든 순열은 
- A, B, C, D
- A, B, D, C
- A, C, B, D
- A, C, D, B
- ...

## 순열의 알고리즘의 종류 

순열을 생성하는 알고리즘은 정말 많은 종류가 있으며, 여기서는 그 중 일부만을 설명함.

1. 특정 요소를 1개씩 추가하면서 생성
2. 원본 배열(n)에서 각 요소들을 제거하면서 n-1 개 요소를 이용하여 순열을 생성
3. Heap's algorithm
4. Steinhaus-Johnson-Trotter algorithm
5. Lexicographic algorithm

## 1. 특정 요소를 1개씩 추가하면서 생성

[A, B, C, D] 에 대한 예제 
![No Image](/assets/images/posts/20190302/permutation_01.png)
- 각 요소를 1개씩 선택하여 출력 배열에 추가하면서 순열을 출력하는 방법 
- 가장 처음 칸에는 A, B, C, D 모두 가능
- 두번째 칸에는 가장 처음칸에서 선택한 요소를 제외한 요소를 선택가능
- if A 를 선택한 경우, B, C, D 가능
- 세번째 칸에는 처음, 두번째 칸에서 선택한 요소를 제외한 요소를 선택가능
- if A, B 를 선택한 경우, C, D 가능

예제 코드 
```java
// input 배열 caching
private Object[]inputs;
// 방문 배열 
private boolean[] visited;
// 출력 배열 
private Object[] outputs;

public void printPermutatinAll(Object[] inputs) {
    this.inputs = inputs;
    visited = new boolean[inputs.length];
    Arrays.fill(visited, false);
    outputs = new Object[inputs.length];
    
    selectElement(0);
}

private void selectElement(int selectIndex) {
    // all element select 
    if(inputs.length == selectIndex) {
        System.out.println(Arrays.toString(outputs));
        return;
    }
    
    for(int i=0; i<inputs.length; i++) {
        // 이미 선택한 요소 제외 
        if(visited[i]) {
            continue;
        }
        
        //현재 요소 선택됨 
        visited[i] = true; 
        outputs[selectIndex] = inputs[i];
        // 다음 칸 선택 
        selectElement(selectIndex + 1);
        //현재 요소 해제
        visited[i] = false;
    }
}
```

순열을 계산하는 메소드(selectElement)의 호출 횟수는 1 + 4 * 24(4!) = 65 번 이다.
- 1 : 가장 처음 호출 -> selectElement(0)
- 4 : 가장 최상위 원소를 선택하는 방법의 수 (A, B, C, D => n)
- 24 : 가장 최상위 원소를 선택 후, n-1 개를 이용해서 순열을 만드는데 필요한 횟수

즉, 시간복잡도는 n * n!

[전체 소스](https://github.com/kimkoungho/Algorithm/blob/master/src/permutation/SelectPermutation.java)


## 2. 원본 배열(n)에서 각 요소들을 제거하면서 n-1 개 요소를 이용하여 순열을 생성

[A, B, C, D] 에 대한 예제 
![No Image](/assets/images/posts/20190302/permutation_02.png)
- 선택한 요소를 제외한 부분 배열에서 특정 2개의 요소를 선택해 swap 하여 순열을 생성하는 방법
- 1번 알고리즘은 입력 배열에서 요소를 1개씩 선택하여 추가했다고 한다면, 이 알고리즘은 swap 을 이용함으로써 추가적인 메모리가 필요하지 않음 
- 0, 1, 2, 3 인덱스에서 0번 인덱스를 고정하고 1, 2, 3 부분 배열에서 swap 을 통해서 순열을 생성
- (0,0) (1,1) (2,2) 자기 자신을 스왑하여 A, B, C, D 가 가장 먼저 출력
- (0,0) (1,1) (2,3) 을 스왑하여 A, B, D, C 를 출력
- (0,0) (1,2) (2,2) 을 스왑하고 A, C, B, D 를 출력
- (0,0) (1,2) (2,3) 을 스왑하고 A, C, D, B 를 출력
- ....

```java
private Object[] inputs;

private void swap(int base, int other) {
    if(base == other) {
		return;
	}

    Object baseObj = inputs[base];
    inputs[base] = inputs[other];
    inputs[other] = baseObj;
}

public void printPermutatinAll(Object[] inputs) {
    this.inputs = inputs;
    removePermutation(0);
}

private void removePermutation(int index) {
    if(index == inputs.length) {
        System.out.println(Arrays.toString(inputs));
    }else {
        for(int i = index; i < inputs.length; i++) {
            swap(index, i); // 기존 배열에서 다른 순열을 생성하기 위한 swap
            removePermutation(index+1);
            swap(index, i); // 기존 배열로 돌려 놓기 위한 swap
        }
    }
}
```
- 위 코드에서 기존 배열로 돌려놓기 위해서 swap 을 1번 더 해야함
- 원본 배열로 되지 않을 경우 순열이 제대로 생성되지 않는 문제가 있음 
![No Image](/assets/images/posts/20190302/permutation_03.png)


순열을 계산하는 메소드(removePermutation)의 호출 횟수는 1번 알고리즘과 마찬가지로 65번
즉, 시간복잡도는 n * n!  

[전체 소스](https://github.com/kimkoungho/Algorithm/blob/master/src/permutation/RemovePermutation.java)

## 3. Heap's algorithm

[위키 Heap's_algorithm](https://en.wikipedia.org/wiki/Heap%27s_algorithm) 의 정의  
Heap's Alogrithm 은 n 개의 객체 배열에서 가능한 **모든 순열을 생성하는 알고리즘**  
1963년 B. R. Heap 이 최초로 제안한 알고리즘으로 순열을 계산할 때, 1쌍의 원소를 교환함으로써 이전에 계산한 순열을 이용하여 순열을 계산하는 방법

[A, B, C, D] 에 대한 예제  
![No Image](/assets/images/posts/20190302/permutation_04.png)
- 이 알고리즘은 2번 알고리즘에서 swap 을 2번 수행하는 것을 개선하기 위해서 heap 이 고안한 알고리즘
- 2번 알고리즘은 상위 요소를 고정했다면, 여기서는 마지막 요소를 고정한다는 차이가 있음
- Heap 은 배열의 상태를 원본 상태로 돌리지 않고도 이전에 출력하지 않은 순열을 찾기 위해서 다음과 같은 규칙을 정의함
    - 현재 구하려는 부분 순열의 크기가 홀수라면, 항상 최초 원소와 swap
    - 현재 구하려는 부분 순열의 크기가 짝수라면, i번째 원소와 swap
- 이미지를 보면 알 수 있듯이 이전 상태에서 swap 을 수행하여 새로운 순열을 생성하고 있음 
- Heap's 알고리즘은 recursive 와 iterative 한 구현이 있음 (여기서는 recursive 구현만 다룸)

recursive 구현
```java
public void printPermutatinAll(Object[] inputs) {
    this.inputs = inputs;
    heapsAlgorithm(inputs.length);
}

private void heapsAlgorithm(int n) {
    if(n == 1) {
        System.out.println(Arrays.toString(inputs));
    }else {
        for(int i=0; i<n-1; i++) {	
            heapsAlgorithm(n-1);
            // 다음 순열 출력을 위한 swap 
            if(n % 2 == 0) {
                swap(i, n-1);
            }else {
                swap(0, n-1);
            }
        }
        // loop 종료시 마지막 swap 한 결과 출력을 위한 재귀 
        heapsAlgorithm(n-1);
    }
}
```
순열을 계산하는 메소드(heapsAlgorithm)의 호출 횟수는 41번으로 위 이미지의 트리의 노드의 총 수를 의미함  
- 2 = 3     (2 * 1 + 1)
- 3 = 10    (3 * 3 + 1)
- 4 = 41    (4 * 10 + 1)
- 5 = 206   (5 * 41 + 1)
- 5 = 206   5 * (4 * (3 * (2 * 1)))

시간복잡도는 거의 1.71828 n! 이라고 함..
- 이에 대한 증명은 [stackoverflow 글](https://stackoverflow.com/questions/42877789/heaps-algorithm-time-complexity) 을 참고

[Heap's algorithm 소스](https://github.com/kimkoungho/Algorithm/blob/master/src/permutation/HeapsAlgorithmPermutation.java)

## 4. Steinhaus-Johnson-Trotter algorithm
이 알고리즘은 주어진 n 개의 객체 배열의 시퀀스(인덱스)를 이용하여 시퀀스에 대한 순열을 생성하는 방법  
특정 시퀀스 i가 배치 될수 있는 모든 위치에 i를 배치해 가면서 모든 순열을 구하는 알고리즘  

**알고리즘의 특징**  
- 이 알고리즘은 시퀀스 순열의 결과가 홀수와 짝수가 교대로 등장함  
- 이러한 특징을 이용하여 시퀀스 순열의 결과를 홀수(or 짝수)만 만드는 방법도 가능함  
- 1개의 순열을 만들기 위해 O(n) 의 시간복잡도를 이용하여 순열을 생성함 

**알고리즘 원리**  
- 순열을 생성하는 시퀀스(index)를 순서대로 배치하고 LEFT 방향을 설정 
- mobile 정수를 찾기  (i는 시퀀스 배열의 인덱스)
    * 시퀀스 i 가 자신이 가리키는 방향에서 최대 값이면 mobile 정수  
    * 방향이 LEFT 인 경우, i = 1 or i < i-1 이동 불가이기에 탈락  
    * 방향이 RIGHT 인 경우, i = n or i < i+1 이동 불가이기에 탈락  
- mobile 정수가 없으면, 종료
- mobile 정수가 있으면, mobile 정수 이동 (이는 실제로 mobile 정수와 인접한 값을 swap 하는것을 의미)  
- mobile 정수보다 큰 값들 방향을 전환 
    * mobile 정수보다 큰 값들은 이동이 불가하기 때문에 탈락한 것이므로 방향을 반대로 전환 
- 1개의 순열이 생성됨 , 다시 mobile 정수 찾기 수행 (2번째로 이동)

**알고리즘 수행과정**   
[A, B, C, D] 에 대한 예제  
>
| 번호 |      순열          |                  설명                                            |
|-----|-------------------|-----------------------------------------------------------------|
|  1  |< 1 < 2 < 3 **< 4**|     초기화 이후 순열 출력                                            |
|-----|-------------------|-----------------------------------------------------------------|
|  2  |< 1 < 2 **< 4** < 3|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
|  3  |< 1 **< 4** < 2 < 3|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
|  4  |**< 4** < 1 < 2 < 3|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
|  5  |4 > < 1 **< 3** < 2|     mobile : 3, swap 후 출력, 이동 불가 값(4)들 방향 전환              |  
|-----|-------------------|-----------------------------------------------------------------|
|  6  |< 1 **4 >** < 3 < 2|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
|  7  |< 1 < 3 **4 >** < 2|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
|  8  |< 1 < 3 < 2 **4 >**|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
|  9  |**< 3** < 1 < 2 < 4|     mobile : 3, swap 후 출력, 이동 불가 값(4)들 방향 전환              | 
|-----|-------------------|-----------------------------------------------------------------|
| 10  |< 3 < 1 **< 4** < 2|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
| 11  |< 3 **< 4** < 1 < 2|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
| 12  |**< 4** < 3 < 1 < 2|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
| 13  |4 > 3 > **< 2** < 1|     mobile : 2, swap 후 출력, 이동 불가 값(4,3)들 방향 전환            | 
|-----|-------------------|-----------------------------------------------------------------|
| 14  |3 > **4 >** < 2 < 1|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
| 15  |3 > < 2 **4 >** < 1|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
| 16  |3 > < 2 < 1 **4 >**|     mobile : 4, swap 후 출력                                      |
|-----|-------------------|-----------------------------------------------------------------|
| 17  |**< 2** 3 > < 1 < 4|     mobile : 2, swap 후 출력, 이동 불가 값(4)들 방향 전환              | 
|-----|-------------------|-----------------------------------------------------------------|
| 18  |< 2 3 > **< 4** < 1|     mobile : 4, swap 후 출력                                     | 
|-----|-------------------|-----------------------------------------------------------------|
| 19  |< 2 **< 4** 3 > < 1|     mobile : 4, swap 후 출력                                     | 
|-----|-------------------|-----------------------------------------------------------------|
| 20  |**< 4** < 2 3 > < 1|     mobile : 4, swap 후 출력                                     | 
|-----|-------------------|-----------------------------------------------------------------|
| 21  |4 > < 2 < 1 **3 >**|     mobile : 3, swap 후 출력, 이동 불가 값(4)들 방향 전환              | 
|-----|-------------------|-----------------------------------------------------------------|
| 22  |< 2 **4 >** < 1 < 3|     mobile : 2, swap 후 출력                                     | 
|-----|-------------------|-----------------------------------------------------------------|
| 23  |< 2 < 1 **4 >** < 3|     mobile : 2, swap 후 출력                                     | 
|-----|-------------------|-----------------------------------------------------------------|
| 24  |< 2 < 1 < 3 **4 >**|     mobile : 2, swap 후 출력                                      | 
|-----|-------------------|-----------------------------------------------------------------|

구현 - 구현은 Apache 재단의 [PermutationIterator](https://commons.apache.org/proper/commons-collections/jacoco/org.apache.commons.collections4.iterators/PermutationIterator.java.html) 를 참고  

**초기화**
```java
// index array
private int[] keys;
// direction array : false (left), true (right)
private boolean[] direction;
// 다음 출력 순열
private Object[] nextOutputs;
	
public void printPermutationAll(Object[] inputs) {
    keys = new int[inputs.length];
    // 1 ~ n 까지 값 설정 
    for(int i = 0; i < keys.length; i++) {
        keys[i] = i + 1;
    }
    
    direction = new boolean[inputs.length];
    Arrays.fill(direction, false); // left 로 모두 설정 
    
    // inputs 으로 outputs 초기화 (최초 순열은 입력 값)
    nextOutputs = new Object[inputs.length];
    System.arraycopy(inputs, 0, nextOutputs, 0, inputs.length);
    
    // permutation 이 존재하는 동안 모든 permutation 찾기 
    while(hasNext()) {
        Object[] out = next();
        System.out.println(Arrays.toString(out));
    }
}
```
- keys : 입력 데이터의 인덱스 배열 (1부터 시작)
- direction : 방향 배열
    * false : 왼쪽 방향 <
    * true : 오른쪽 방향 >
- nextOutputs : 다음 출력할 순열 
    * 입력 값을 최초 순열로 그대로 사용하기 위해 매번 다음 순열을 미리 계산함 
- hasNext() : 다음 순열이 존재하는지 여부를 반환 
- next() : 다음 순열을 반환하는 함수 

**hasNext()**
```java
// next permutation 존재하는지 
private boolean hasNext() {
    return nextOutputs != null;
}
```
- next() 에서 계산한 다음 순열이 존재하는지 여부만을 반환 

**1개의 순열 만들기**
```java
private Object[] next() {		
    // 현재 permutation 
    Object[] outputs = new Object[nextOutputs.length];
    System.arraycopy(nextOutputs, 0, outputs, 0, nextOutputs.length);
    
    // find the largest mobile integer index
    int mobileIndex = findMobileIndex();
    
    // mobile integer not found exit
    if(mobileIndex == -1) {
        nextOutputs = null;
        return outputs;
    }
    
    // swap 전에 mobile 값 저장 
    int mobile = keys[mobileIndex];
    // swap 
    swap(mobileIndex);
    
    // not move integer direction toggle
    reverseDirection(mobile);
    
    return outputs;
}	
```
- findMobileIndex() : mobile 정수 찾기
- swap() : mobile 정수와 mobile 정수가 가리키는 방향의 값 swap
- reverseDirection() : mobile 정수보다 큰 값들(즉, 이동이 불가능한 값들) 방향 전환

**findMobileIndex()**
```java
// mobile integer 의 index return
private int findMobileIndex() {
    int mobileIndex = -1;
    int mobile = -1; // 가리키는 방향의 최대 값 
    for(int i = 0; i < keys.length; i++) {
        if(direction[i]) { // right
            if(i < keys.length - 1 && keys[i] > keys[i+1] && mobile < keys[i]) {
                mobileIndex = i;
                mobile = keys[i];
            }
        }else { // left
            if(i > 0 && keys[i] > keys[i-1] && mobile < keys[i]) {
                mobileIndex = i;
                mobile = keys[i];
            }
        }
    }
    
    return mobileIndex;
}
```
- 방향이 right 인 경우
    * i < keys.length - 1 : i 가 last index 라면, 오른쪽으로 이동 불가
    * keys[i] > keys[i+1] : 이동할 뱡향의 원소보다 커야 한다는 mobile 정수 조건 
    * mobile < keys[i] : 가장 큰 mobile 정수를 찾기 위한 조건
- 방향이 left 인 경우
    * i > 0 : i 가 최초 index 라면, 왼쪽으로 이동 불가
    * keys[i] > keys[i-1] : 이동할 뱡향의 원소보다 커야 한다는 mobile 정수 조건 
    * mobile < keys[i] : 가장 큰 mobile 정수를 찾기 위한 조건

**swap()**
```java
private void swap(int mobileIndex) {
    // swap 위치를 지정할 offset
    int offset = direction[mobileIndex] ? 1 : -1;
    
    // index swap 
    int mobile = keys[mobileIndex];
    keys[mobileIndex] = keys[mobileIndex + offset];
    keys[mobileIndex + offset] = mobile;
    
    // real value swap
    Object value = nextOutputs[keys[mobileIndex] - 1];
    nextOutputs[keys[mobileIndex] -1] = nextOutputs[keys[mobileIndex + offset] - 1];
    nextOutputs[keys[mobileIndex + offset] - 1] = value;
    
    // 방향 swap
    boolean mobileDirection = direction[mobileIndex];
    direction[mobileIndex] = direction[mobileIndex + offset];
    direction[mobileIndex + offset] = mobileDirection;
}
```
- keys : 입력 데이터의 인덱스 배열을 swap
- nextOutputs : 실제 출력할 다음 순열의 값을 swap
- direction : key 를 swap 했으므로 방향도 swap 

**reverseDirection()**
```java
private void reverseDirection(int mobile) {
    for(int i = 0; i < keys.length; i++) {
        if(keys[i] > mobile) {
            direction[i] = !direction[i];
        }
    }
}
```
- mobile 정수 보다 큰 값들, 이동이 불가능한 값들 방향 전환 

[Steinhaus-Johnson-Trotter algorithm 소스](https://github.com/kimkoungho/Algorithm/blob/master/src/permutation/SteinhausJohnsonTrotterPermutation.java)

### 5. Lexicographic algorithm

이 알고리즘은 주어진 n 개의 객체 배열에서 사전식 순서로 순열을 생성하는 방법  
c++ stl 라이브러리의 std::next_permutation() 에서 구현한 알고리즘으로 알려져 있음  
이 방법은 14세기 인도의  Narayana Pandita 가 고안한 알고리즘  

**알고리즘 동작방식**
- 입력된 순열에서 증가하지 않는 index(i)를 찾아 **접미사**를 추출 
    * 접미사: 증가하지 않는 index ~ 마지막 index 까지
- 피봇을 설정 (pivot = index - 1) 
- 피봇보다 큰 값을 갖는 가장 오른쪽 index(j)를 추출 (단, j >= i)
- 피봇과 가장 오른쪽 index(j)를 swap 
- 접미사를 뒤집는다
    * 접미사는 사전순으로 가장 큰 값이므로 가장 작은 값을 만들어 순서대로 순열을 출력하기 위함 

**알고리즘의 의사코드**
>
- A[i-1] < A[i] 을 만족하는 가장 큰 인덱스 i 를 찾기 (접미사의 시작 index 찾기)
- A[j] > A[i-1] 인 가장 큰 인덱스 j 를 찾기 (pivot 보다 큰 가장 오른쪽 인덱스 찾기)
- [i-1] 와 [j] 를 swap (pivot과 가장 오른쪽 인덱스 j를 swap)
- 접미사 [i] ~ [length] reverse 


**알고리즘 수행과정**
[A, B, C, D] 에 대한 예제  
- ABCD : 초기화 후 출력
- ABDC 
    * i 찾기 : ABCD 에서 모두 증가하고 있으므로 i = 4 (즉, 접미사가 없음)
    * j 찾기 : [i-1]인 C 보다 큰 값은 D 이므로 j = 4
    * swap [3], [4] = AB**DC**
    * i=4 이므로 reverse 할 수 없음
- ACBD
    * i 찾기 : ABDC 에서 B < D 이지만 D > C 므로 i = 3 (접미사: DC)
    * j 찾기 : [i-1]인 B 보다 큰 가장 오른쪽값은 C 이므로 j = 4
    * swap [2], [4] = A**C**D**B**
    * 접미사 i=3 부터 4까지 reverse => AC**BD**
- ACDB
    * i 찾기 : ACBD 에서 A<C(i=2), B<D(i=4) 이므로 가장 큰 i = 4 (접미사가 없음)
    * j 찾기 : [i-1]인 B 보다 큰 가장 오른쪽값은 D 이므로 j = 4
    * swap [3], [4] = AC**DB**
    * i=4 이므로 reverse 할 수 없음
- ADBC
    * i 찾기 : ACDB 에서 i = 3 (접미사: DB)
    * j 찾기 : [i-1]인 C 보다 큰 가장 오른쪽값은 D 이므로 j = 3
    * swap [2], [3] = A**DC**B
    * 접미사 i=3 부터 4까지 reverse => AD**BC**
- ADCB
    * i 찾기 : ADBC 에서 i = 4 (접미사가 없음)
    * j 찾기 : [i-1]인 B 보다 큰 가장 오른쪽값은 C 이므로 j = 4
    * swap [3], [4] = AD**CB**
    * i=4 이므로 reverse 할 수 없음
- BACD
    * i 찾기 : ADCB 에서 i = 2 (접미사: DCB)
    * j 찾기 : [i-1]인 A 보다 큰 가장 오른쪽값은 B 이므로 j = 4
    * swap [1], [4] = **B**DC**A**
    * 접미사 i=2 부터 4까지 reverse => B**ACD**
- BADC
    * i 찾기 : BACD 에서 i = 4 (접미사가 없음)
    * j 찾기 : [i-1]인 C 보다 큰 가장 오른쪽값은 D 이므로 j = 4
    * swap [3], [4] = BA**DC**
    * i=4 이므로 reverse 할 수 없음
...

**구현**
```java
private boolean nextPermutation(Object[] inputs) {
    // 접미사의 시작 i를 찾기
    // 가장 큰 i 를 찾기 위해서 뒤에서부터 검색 
    int i = inputs.length - 1;
    while(i > 0 && comparator.compare(inputs[i - 1], inputs[i]) >= 0) {
        i--;
    }
    
    // i < 0 경우는 가장 큰 순열을 출력한 경우 
    if(i <= 0) {
        return false;
    }
    
    // pivot = [i-1] 을 만족하는 가장 큰 j 찾기 
    int j = inputs.length - 1;
    while (comparator.compare(inputs[j], inputs[i - 1]) <= 0) {
            j--;
    }
        
    // swap pivot, j
    swap(inputs, i-1, j);
    
    // 접미사 reverse
    int k = inputs.length - 1;
    while(i < k) {
            swap(inputs, i++, k--);
    }
    
    return true;
}
```
[Lexicographic algorithm 소스](https://github.com/kimkoungho/Algorithm/blob/master/src/permutation/LexicographicPermutation.java)


- 코드 
[전체 코드 git 링크](https://github.com/kimkoungho/Algorithm/tree/master/src/permutation)

## Reference
- [topcoder generating-permutations](https://www.topcoder.com/blog/generating-permutations/)
- [위키 Heap's_algorithm](https://en.wikipedia.org/wiki/Heap%27s_algorithm)
- [Ruslan Ledesma-Garza why-does-heap-work](http://ruslanledesma.com/2016/06/17/why-does-heap-work.html)
- [stackoverflow 의 heaps-algorithm-time-complexity](https://stackoverflow.com/questions/42877789/heaps-algorithm-time-complexity)
- [위키 Steinhaus–Johnson–Trotter_algorithm](https://en.wikipedia.org/wiki/Steinhaus%E2%80%93Johnson%E2%80%93Trotter_algorithm)
- [Ragavan N 의 steinhaus-johnson-trotter-permutation-algorithm-explained-and-implemented-in-java](https://tropenhitze.wordpress.com/2010/01/25/steinhaus-johnson-trotter-permutation-algorithm-explained-and-implemented-in-java/)
- [아파치 오픈소스 PermutationIterator.java](https://commons.apache.org/proper/commons-collections/jacoco/org.apache.commons.collections4.iterators/PermutationIterator.java.html)
- [위키 Permutation 중 Generation_in_lexicographic_order](https://en.wikipedia.org/wiki/Permutation#Generation_in_lexicographic_order)
- [nayuki 의 next-lexicographical-permutation-algorithm](https://www.nayuki.io/page/next-lexicographical-permutation-algorithm)
- [quora 의 generates-permutations-using-lexicographic-ordering](https://www.quora.com/How-would-you-explain-an-algorithm-that-generates-permutations-using-lexicographic-ordering)

