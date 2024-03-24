## 해시
- WHAT) 해시 함수를 사용해서 변환한 값을 인덱스로 삼아 키와 값을 저장해서 빠른 데이터 탐색을 제공하는 자료구조
- HOW) 키와 데이터를 일대일 대응하여 저장하기 때문에 키를 통해 데이터에 바로 접근할 수 있다.
- WHY) 어떤 데이터를 찾을 때 처음부터 끝까지 순차 탐색하는 방법은 최악의 경우 탐색을 할 때마다 모든 데이터를 살펴봐야 할 수 있으므로 효율적이지 않다.
- WHEN)
  - 비밀번호 관리 : 사용자의 비밀번호를 그대로 노출해 저장하는 것은 위험하므로 해시 함수를 활용해 해싱한 비밀번호를 저장한다.
  - 데이터베이스 인덱싱 : 데이터베이스에 저장된 데이터를 효율적으로 검색할 때
  - 블록체인 : 각 블록은 이전 블록의 해시 값을 포함하고 있으며, 이를 통해 데이터 무결성을 확인할 수 있다.

## 해시의 특징
- 단방향으로 동작한다.
  - 키를 통해 값을 찾을 수 있지만, 값을 통해 키를 찾을 순 없다.
- 찾고자 하는 값을 O(1)에서 바로 찾을 수 있다.
  - 키 자체가 해시 함수에 의해 값이 있는 인덱스가 되므로 값을 찾기 위한 탐색 과정이 필요 없다.
```python
def solution(phone_book):
answer = True

    hash_map = {}
    for phone_number in phone_book:
        hash_map[phone_number] = 1

    for phone_number in phone_book:
        temp = ""
        for number in phone_number:
            temp += number
            if temp in hash_map and temp != phone_number:
                answer = False

    return answer


print(solution(["119", "97674223", "1195524421"])) # False
```
```python
def solution(participant, completion):
    hash_dic = {}
    sum_hash = 0

    for part in participant:
        hash_dic[hash(part)] = part
        sum_hash += hash(part)

    for comp in completion:
        sum_hash -= hash(comp)

    return hash_dic[sum_hash]


print(solution(["leo", "kiki", "eden"], ["eden", "kiki"])) # leo
```

## 키워드
- 해시 테이블 : 키와 대응한 값이 저장되어 있는 공간
```python
# 계수 정렬
def count_sort(arr, k):
    hashtable = [0] * (k + 1)

    for num in arr:
        if num <= k:
            hashtable[num] = 1
            
    return hashtable

def solution(arr, target):
    hashtable = count_sort(arr, target)

    for num in arr:
        complement = target - num
        
        if (
            complement != num
            and complement >= 0
            and complement <= target
            and hashtable[complement] == 1
        ):
            return True

    return False


print(solution([1, 2, 3, 4, 8], 6)) # True
print(solution([2, 3, 5, 9], 10)) # False
```
- 버킷 : 해시 테이블의 각 데이터

## 나눗셈법
- h(x) = x mod m
- 키를 소수로 나눈 나머지를 활용 = 모듈러 연산 = 나머지를 취하는 연산
- 소수를 사용하는 이유 : 1과 자신 빼고는 약수가 없기 때문에 다른 수를 사용할 때보다 충돌이 적다.
- 해시 테이블 크기 = K (0 ~ (K - 1))

## 곱셈법
- h(x) = ((x * A) mod 1) * m
  - m = 최대 버킷의 개수
  - A = 황금비(무한소수, 0.6183...)
    - 수학적으로 임의의 길이를 두 부분으로 나누었을 때, 전체와 긴 부분의 비율이 긴 부분과 짧은 부분의 비율과 같은 비율을 뜻한다
- WHY) 나눗셈법은 때에 따라 큰 소수를 사용해야 하는데 큰 소수를 구하기가 쉽지 않다.

## 문자열 해싱
- hash(s) = (s[0] + s[1] * p + s[2] * p² .... s[n - 1] * pⁿ⁻ٰ¹) mod m
  - p = 31
    - 메르센 소수 : 2ⁿ - 1으로 표시할 수 있는 숫자 중 소수인 수
  - m = 테이블 최대 크기
- 문자열의 문자를 숫자로 변환하고 이 숫자들을 다항식의 값으로 변환해서 해싱한다.

### 문자열 해시 함수 수정하기
- 해시 함수를 적용한 값이 해시 테이블 크기에 비해 너무 클 수 있기 때문에, 중간 중간 모듈러 연산을 해 더한 값을 모듈러 연산하면 오버플로를 최대한 방지할 수 있다.
- (a + b) % c = (a % c + b % c) % c
- hash(s) = (s[0] % m + s[1] * p % m + s[2] * p² % m .... s[n - 1] * pⁿ⁻ٰ¹ % m) mod m
```python
def polynomial_hash(str):
    p = 31
    m = 1_000_000_007
    hash_value = 0
    for char in str:
        hash_value = (hash_value * p + ord(char)) % m
    return hash_value

def solution(string_list, query_list):
    hash_list = [polynomial_hash(str) for str in string_list]

    result = []
    for query in query_list:
        query_hash = polynomial_hash(query)

        if query_hash in hash_list:
            result.append(True)
        else:
            result.append(False)

    return result


print(solution(["apple", "banana", "cherry"], ["banana", "kiwi", "melon", "apple"])) # [True, False, False, True]
```

## 충돌 처리
- 충돌 : 서로 다른 두 키에 대해 해싱 함수를 적용한 결과가 동일한 것

### 체이닝으로 처리하기
- 체이닝 : 충돌이 발생하면 해당 버킷에 링크드리스트로 같은 해시값을 가지는 데이터를 연결한다.
  - 링크드리스트 = 데이터 요소들을 연결하여 구성한 선형 데이터 구조
- 단점
  - 해시 테이블 공간 활용성 떨어짐 : 충돌이 많아지면 그만큼 링크드리스트의 길이가 길어지고, 다른 해시 테이블의 공간은 덜 사용하므로 공간 활용성이 떨어진다.
  - 검색 성능이 떨어짐 : 링크드리스트로 연결한 값을 찾으려면 링크드리스트의 맨 앞 데이터부터 검색해야 한다.

### 개방 주소법으로 처리하기
- 개방 주소법 : 빈 버킷을 찾아 충돌값을 삽입한다.
- 장점 : 해시 테이블을 최대한 활용하므로 체이닝보다 메모리를 더 효율적으로 사용한다.

#### 1. 선형 탐사 방식
- h(k, i) = (h(k) + i) mod m
  - m = 수용할 수 있는 최대 버킷
    - 선형 탐사 시 테이블의 범위를 넘으면 안되므로 모듈러 연산을 적용
- 충돌이 발생하면 다른 빈 버킷을 찾을 때까지 일정한 간격으로 이동한다.

#### 2. 이중 해싱 방식
- h(k, i) = (h₁(k) + i * h₂(k)) mod m
- 해시 함수를 2개 사용한다. (때에 따라 해시 함수를 N개로 늘리기도 한다)
- 두 번째 해시 함수는 첫 번째 해시 함수로 충돌이 발생하면 해당 위치를 기준으로 어떻게 위치를 정할지 결정하는 역할을 한다.
- 선형 탐사와 비슷하게 더하는 방식으로 데이터의 위치를 정하지만 클러스터를 줄이기 위해 m을 제곱수로 하거나 소수로 한다.
  - 클러스터 : 선형 탐사 방식에서 충돌이 발생 시 1칸씩 이동하며 해시 테이블 빈 곳에 값을 넣으면 해시 충돌이 발생한 값끼리 모이는 영역
