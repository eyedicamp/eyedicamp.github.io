---
layout: post
title: "[BOJ 14888] 연산자 끼워넣기"
date: 2026-03-03 14:00:00 +0900
excerpt: "백준 문제 풀이"
categories: [Algorithm, BOJ]
tags: [python, backtracking, recursion, implementation]
---

## 문제 요약

- 숫자 `A1..AN`의 순서는 고정
- 연산자 `+ - * /`가 각각 몇 개씩 주어짐(합은 `N-1`)
- 연산자 우선순위 없이 **왼쪽부터** 계산
- 나눗셈은 **정수 나눗셈(몫)**, 음수 처리 방식은 **C++14처럼 0을 향해 절삭(trunc toward 0)**

가능한 모든 식의 결과 중 **최댓값/최솟값**을 구한다.

---

## 중간에 겪었던 이슈 (GPT 도움 받은 부분)

처음에는 아래처럼 연산자 조합(경로)만 전부 만들어서 `out`에 모으는 방식으로 시작했다.

```python
n = int(input())
numbers = list(map(int, input().split()))
operators = list(map(int, input().split()))

path = []
out = []

def dfs(operator_count):
    if len(path) == n - 1:
        out.append(path)
        return
    for x in range(4):
        if operator_count[x] != 0:
            path.append(x)
            operator_count[x] -= 1
            dfs(operator_count)
            path.pop()

dfs(operators)

print(out)
```

여기서 핵심 문제는 2가지였다.

### 1) `out.append(path)`는 “그 시점의 경로 복사”가 아니다
`path`는 리스트(가변 객체)라서, `out`에 `path` 자체를 넣으면 이후 `path`가 바뀔 때 `out` 안의 참조도 같이 바뀐다.  
그래서 보통은 아래처럼 **복사본을 저장**한다.

```python
out.append(path.copy())
```

### 2) `operator_count`는 “복사”가 아니라 “참조”로 전달된다
파이썬에서 리스트를 인자로 넘기면 **같은 리스트 객체를 공유**한다.  
그래서 재귀에서 `operator_count[x] -= 1`을 하면 아래 호출에서도 바뀐 상태가 보인다.

하지만 이건 “문제가 된다”기보다, 백트래킹에서 흔히 쓰는 방식이다.

- 변경하고 내려갔다가
- 돌아오면서 **원복**하면 된다.

```python
operator_count[x] -= 1
dfs(operator_count)
operator_count[x] += 1  # 원복
```

---

## 내가 최종 제출한 코드 (정답)

나는
1) 연산자 수열을 전부 만들고(`out`)
2) 각 수열을 다시 계산해서(`results`)
3) 정렬 후 최댓값/최솟값을 뽑는 방식으로 풀었다.

```python
n = int(input())
numbers = list(map(int, input().split()))
operators = list(map(int, input().split()))

path = []
out = []
results = []

def dfs(operator_count):
    if len(path) == n - 1:
        out.append(path.copy())
        return
    for x in range(4):
        if operator_count[x] != 0:
            path.append(x)
            operator_count[x] -= 1
            dfs(operator_count)
            operator_count[x] += 1
            path.pop()

dfs(operators)

for operation in out:
    numbers_copy = numbers.copy()
    for y in range(n-1):
        if operation[y] == 0:
            numbers_copy[y+1] = numbers_copy[y] + numbers_copy[y+1]
        elif operation[y] == 1:
            numbers_copy[y+1] = numbers_copy[y] - numbers_copy[y+1]
        elif operation[y] == 2:
            numbers_copy[y+1] = numbers_copy[y] * numbers_copy[y+1]
        elif operation[y] == 3:
            if numbers_copy[y] < 0:
                numbers_copy[y+1] = -1 * ((-1 * numbers_copy[y]) // numbers_copy[y+1])
            else:
                numbers_copy[y+1] = numbers_copy[y] // numbers_copy[y+1]
    results.append(numbers_copy[n-1])

results.sort()
print(results[-1])
print(results[0])
```

---

## 개선 포인트

위 풀이는 정답이지만, 성능/메모리 면에서 개선 여지가 있다.

- `out`에 “모든 연산자 수열”을 저장 → 경우의 수가 커지면 메모리가 커짐
- `results`에도 “모든 결과”를 저장 → 사실 최댓값/최솟값만 필요함
- 즉, DFS 중에 바로 계산하면서 `min/max`만 갱신하면 더 깔끔하고 효율적이다.

---

## 개선된 코드 (추천): DFS 하면서 바로 계산하기

아래 코드는 연산자 수열을 저장하지 않고, DFS 과정에서 **현재 계산값을 들고 다니며** 바로 갱신한다.

```python
n = int(input())
nums = list(map(int, input().split()))
ops = list(map(int, input().split()))  # [+,-,*,/]

max_val = -10**18
min_val = 10**18

def div_trunc_zero(a, b):
    # b는 항상 양수(Ai >= 1)
    if a < 0:
        return - (abs(a) // b)
    return a // b

def dfs(idx, cur, plus, minus, mul, div):
    global max_val, min_val

    # idx: 다음에 사용할 nums 인덱스 (현재 cur은 nums[idx-1]까지 반영된 값)
    if idx == n:
        max_val = max(max_val, cur)
        min_val = min(min_val, cur)
        return

    x = nums[idx]

    if plus:
        dfs(idx + 1, cur + x, plus - 1, minus, mul, div)
    if minus:
        dfs(idx + 1, cur - x, plus, minus - 1, mul, div)
    if mul:
        dfs(idx + 1, cur * x, plus, minus, mul - 1, div)
    if div:
        dfs(idx + 1, div_trunc_zero(cur, x), plus, minus, mul, div - 1)

dfs(1, nums[0], ops[0], ops[1], ops[2], ops[3])
print(max_val)
print(min_val)
```

---

## 이 코드에서 배울 수 있는 점

1) **“필요한 값만 유지”하면 메모리를 크게 줄일 수 있다**  
- 모든 경로/결과를 저장하지 않고 `max/min`만 갱신

2) **상태(state)를 인자로 넘기면 백트래킹이 더 깔끔해진다**  
- `operator_count` 리스트를 직접 수정/원복하는 방식도 가능하지만  
- 여기서는 `plus, minus, mul, div`를 각각 정수로 넘겨서 “공유 참조 이슈” 자체가 없다

3) **파이썬의 `//`는 음수에서 C++과 다르다**  
- 파이썬 `-7 // 3 == -3` (내림, floor)
- C++14는 `-7 / 3 == -2` (0을 향해 절삭)
- 그래서 이 문제는 `div_trunc_zero` 같은 처리가 필요하다

4) 재귀/백트래킹의 기본 뼈대  
- “종료 조건”에서 결과 갱신  
- “가능한 선택(연산자)”들을 시도하며 깊게 들어감

그래도 이제 dfs를 이용한 백트래킹에 대한 감이 조금씩 오는 것 같아.

---
