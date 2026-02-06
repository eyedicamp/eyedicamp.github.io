---
layout: post
title: "[BOJ 10798] 세로읽기"
date: 2026-02-05
excerpt: "백준 문제 풀이"
categories: [Algorithm, BOJ]
tags: [python, string, list, join, itertools]
---

## 문제 요약

문자열 5개(각 1~15자)가 주어진다.  
왼쪽에서 오른쪽으로 열(column) 단위로 이동하면서, 같은 위치의 글자를 위에서 아래로 읽되 **해당 위치에 글자가 없으면 건너뛴다.**  
읽은 글자들을 공백 없이 이어서 출력한다.

---

## 내가 작성한 코드 (정답)

```python
import sys

input = sys.stdin.readline

letters = []
result = ''
max_length = 0

for _ in range(5):
    letter = input().strip()
    letters.append(letter)

for letter in letters:
    if len(letter) > max_length:
        max_length = len(letter)

for index in range(len(letters)):
    if len(letters[index]) < max_length:
        letters[index] += ' ' * (max_length - len(letters[index]))

for i in range(max_length):
    for j in range(5):
        result += letters[j][i]

result = result.replace(' ', '')

print(result)
```

---

## 이번 문제로 배운 점 (내가 체감한 포인트)

### 1) `for letter in letters:`에서 `letter += ...` 해도 리스트가 안 바뀐다
문자열은 **불변(immutable)**이고, `for letter in letters:`의 `letter`는 리스트 원소를 가리키는 지역 변수라서,
`letter += ' '`를 해도 **리스트의 원소 자체를 갱신하지 않는다.**

리스트를 실제로 바꾸려면, 아래처럼 **인덱스로 다시 대입**해야 한다.

```python
for idx in range(len(letters)):
    letters[idx] += "..."
```

### 2) `replace()`로 특정 문자를 한 번에 제거할 수 있다
공백 패딩을 한 뒤, 마지막에

```python
result = result.replace(' ', '')
```

처럼 처리하면 내부 공백을 쉽게 제거할 수 있다.

---

## 개선 포인트

위 코드는 정답이지만, 아래를 더 깔끔하게 만들 수 있다.

1. `result += ...`로 문자열을 계속 더하면(누적) 파이썬에서는 비효율적일 수 있다.  
   → 리스트에 모아두고 `''.join(...)`으로 합치는 방식이 정석이다.

2. 애초에 공백으로 패딩했다가 `replace(' ', '')`로 지우지 말고,  
   **해당 인덱스가 문자열 길이 안에 있을 때만** 추가하면 로직이 더 직관적이다.

3. (선택) `itertools.zip_longest`를 쓰면 “열 기준 순회”를 깔끔하게 표현할 수 있다.

---

## 최종 코드 1 (추천): 조건으로 건너뛰기 + join

```python
import sys
input = sys.stdin.readline

words = [input().strip() for _ in range(5)]
max_len = max(map(len, words))

out = []
for c in range(max_len):
    for r in range(5):
        if c < len(words[r]):
            out.append(words[r][c])

print(''.join(out))
```

### 이 코드에서 배울 수 있는 점
- `max_len = max(map(len, words))` 처럼 **길이의 최댓값을 한 줄로** 구할 수 있다.
- 문자열 누적은 `out.append(...)`로 모은 뒤 `''.join(out)`이 안전하고 깔끔하다.
- “없으면 건너뛴다”는 문제 조건이 `if c < len(words[r])`로 바로 드러난다.

---

## 최종 코드 2 (대안): zip_longest로 더 파이써닉하게

```python
import sys
from itertools import zip_longest

input = sys.stdin.readline

words = [input().strip() for _ in range(5)]

# 각 열을 튜플로 묶고, 없는 칸은 ''로 채운 뒤, 그대로 이어붙이기
print(''.join(ch for col in zip_longest(*words, fillvalue='') for ch in col))
```

### 이 코드에서 배울 수 있는 점
- `zip_longest(*words)`는 “행(row) 5개를 열(column) 단위로” 묶어서 순회할 수 있게 해준다.
- `fillvalue=''`로 “없는 글자는 빈 문자열” 처리 → 자연스럽게 건너뛰는 효과가 난다.
- 중첩 루프를 한 줄로 표현할 수 있지만, 익숙하지 않으면 1번 코드가 더 읽기 쉽다.

---
