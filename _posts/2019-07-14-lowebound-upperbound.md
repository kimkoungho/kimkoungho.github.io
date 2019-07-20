---
title: "Upper Bound 와 Lower Bound"
excerpt: ""
categories: algorithms
tags:
  - upper bound
  - lower bound
last_modified_at: 2019-07-20
---

여기서 설명하는 **Upper Bound** 와 **Lower Bound** 는 이분 탐색(Binary Search)에 Upper Bound 와 Lower Bound 알고리즘을 정리한 것.  
(c++ std 라이브러리)  

[위키](https://ko.wikipedia.org/wiki/%EC%83%81%ED%95%9C%EA%B3%BC_%ED%95%98%ED%95%9C) 에서는 집합에서의 Upper Bound 와 Lower Bound 를 설명하고 있음  


## Upper Bound
- std::upper_bound() 함수는 정렬된 배열 [first, end) range 에서 찾으려는 값(key) 보다 큰 처음 요소의 위치를 반환한다 
- [first, end) : first 를 포함하고 end 는 포함하지 않는 구간을 의미 
- end 를 포함하지 않는 이유 : 모든 요소가 찾으려는 값(key) 보다 작은 경우 end 를 반환 
- [c++ std::upper_bound](http://www.cplusplus.com/reference/algorithm/upper_bound/) 
- 이진 탐색을 이용하여 upper bound 를 찾는 예제  


>  
입력 데이터  
array: 10 10 10 20 20 20 30 30  
key: 20

**1.** first = 0, end = 8  

```
 v - first              v - end 
10 10 10 20 20 20 30 30 x
             ^ - mid

array[mid] = key -> 오른쪽 구간으로 이동  
```

**2.** first = 5, end = 8  

```
        first - v       v - end 
10 10 10 20 20 20 30 30 x
                   ^ - mid

array[mid] > key -> 왼쪽 구간으로 이동 
```

**3.** first = 5, end = 6  

```
        first - v  v - end 
10 10 10 20 20 20 30 30 x
                ^ - mid

array[mid] = key -> 오른쪽 구간으로 이동 
```

**4.** first = end -> 종료 

### 구현 코드 

```java
public static int upperBound(int[] arr, int key) {
    int first = 0;
    int end =  arr.length;
    
    while(first < end) {
        int mid = (first + end) >>> 1;
        
        // if) from + 1 = to, mid = from  
        if(key >= arr[mid]) {
            first = mid + 1;
        }else {
            end = mid;
        }
    }
    
    return first;
}
```


## Lower Bound
- std::lower_bound() 함수는 정렬된 배열 [first, end) range 에서 찾으려는 값(key) 보다 작지 않은 처음 요소의 위치를 반환한다 
- [first, end) : first 를 포함하고 end 는 포함하지 않는 구간을 의미 
- end 를 포함하지 않는 이유 : 모든 요소가 찾으려는 값(key) 보다 작은 경우 end 를 반환 
- [c++ std::lower_bound](http://www.cplusplus.com/reference/algorithm/lower_bound/)


> 
입력 데이터  
array: 10 10 10 20 20 20 30 30  
key: 20

**1.** first = 0, end = 8  

```
 v - first              v - end 
10 10 10 20 20 20 30 30 x
             ^ - mid

array[mid] = key -> 왼쪽 구간으로 이동  
```

**2.** first = 0, end = 4  

```
v - first    v - end 
10 10 10 20 20 20 30 30 x
       ^ - mid

array[mid] < key -> 오른쪽 구간으로 이동 
```

**3.** first = 3, end = 4  

```
first- v  v - end 
10 10 10 20 20 20 30 30 x
       ^ - mid

array[mid] < key -> 오른쪽 구간으로 이동 
```

**4.** first = end -> 종료 

### 구현 코드 

```java
public static int lowerBound(int[] arr, int key) {
    int first = 0;
    int end = arr.length;
    
    while(first < end) {
        int mid = (first + end) >>> 1;
        
        // 
        if(key <= arr[mid]) {
            end = mid;
        }else {
            first = mid + 1;
        }
    }
    
    return end;
}
```

### Binary Search, Upper Bound, Lower Bound  
예제를 통해 Binary Search, Upper Bound, Lower Bound 를 비교  

> 
입력 데이터  
array: 2 3 4 5 5 5 6 7 9

**중복 된 데이터를 찾는 경우** : key = 5  

```
  binary search
        v   v - upper bound
2 3 4 5 5 5 6 7 9
      ^ - lower bound
```
- binary search: 배열의 총 길이에 따라 결과 값이 달라짐 
- upper bound / lower bound : 항상 같은 결과 


**찾으려는 값이 배열의 모든 요소보다 작은 경우** : key = 1

```
v - upper bound
2 3 4 5 5 5 6 7 9
^ - lower bound
```
- binary search: 값이 존재하지 않으므로 -1 반환 (not found)
- lower bound / upper bound : 0 반환, [first, end) 구간에서 first 를 반환함 


**찾으려는 값이 배열의 모든 요소보다 큰 경우** : key = 10

```
                  v - upper bound
2 3 4 5 5 5 6 7 9 x
                  ^ - lower bound
```
- binary search: 값이 존재하지 않으므로 -1 반환 (not found)
- lower bound / upper bound : 10 반환, [first, end) 구간에서 end 를 반환함 


**찾으려는 값이 배열에 없는 경우** : key = 8
```
                v - upper bound
2 3 4 5 5 5 6 7 9
                ^ - lower bound
```
- binary search: 값이 존재하지 않으므로 -1 반환 (not found)
- lower bound / upper bound : 9 반환

> 
binary search 는 배열에서 찾으려는 값(key)가 해당 배열에 있다는 가정이 있음  
따라서 값을 찾지 못했을 경우, 음수 값을 반환
실제로 [c++ std::binary_search](https://en.cppreference.com/w/cpp/algorithm/binary_search) 의 구현에서 내부적으로 std::lower_bound() 호출하여 구현하고 있음  


[git 소스](https://github.com/kimkoungho/Algorithm/blob/master/src/search/BinarySearch.java)

### Reference
- <https://ko.wikipedia.org/wiki/%EC%83%81%ED%95%9C%EA%B3%BC_%ED%95%98%ED%95%9C>
- <http://www.cplusplus.com/reference/algorithm/upper_bound/>
- <http://www.cplusplus.com/reference/algorithm/lower_bound/>
- <https://stackoverflow.com/questions/41958581/difference-between-upper-bound-and-lower-bound-in-stl>
- <https://stackoverflow.com/questions/28389065/difference-between-basic-binary-search-for-upper-bound-and-lower-bound>
- <https://en.cppreference.com/w/cpp/algorithm/binary_search>