---
title: "파이썬 이터레이터와 제너레이터 완전 정리"
date: 2025-07-25
categories: [Python]
tags: [Iterator, Generator, Python, Iterable, Lazy Evaluation]
---

# 🌀 파이썬 이터레이터와 제너레이터 완전 정리

파이썬에서 **이터러블(Iterable)**, **이터레이터(Iterator)**, **제너레이터(Generator)**는 반복문과 데이터를 효율적으로 다루기 위한 핵심 개념입니다. 이 문서에서는 세 개념을 정리하고, 차이점과 실제 사용 예제를 통해 완전히 이해할 수 있도록 설명합니다.

---

## 📌 1. 이터러블(Iterable)이란?

이터러블은 반복할 수 있는 객체입니다.  
`for` 문에서 사용할 수 있는 모든 객체는 **이터러블**입니다.

### ✔️ 특징

- `__iter__()` 메서드를 가지고 있음
- `list`, `tuple`, `dict`, `set`, `str` 등이 이에 해당

```python
lst = [1, 2, 3]
it = iter(lst)  # list는 이터러블, iter()로 이터레이터 반환
print(next(it))  # 1
```

---

## 🔁 2. 이터레이터(Iterator)란?

이터레이터는 **다음 값을 차례로 꺼낼 수 있는 객체**입니다.

### ✔️ 조건

- `__iter__()`와 `__next__()`를 모두 가지고 있음
- `iter()` 함수로 생성 가능

```python
class MyIterator:
    def __init__(self):
        self.num = 0
    def __iter__(self):
        return self
    def __next__(self):
        if self.num < 3:
            self.num += 1
            return self.num
        else:
            raise StopIteration

it = MyIterator()
for i in it:
    print(i)  # 1, 2, 3
```

---

## ⚡ 3. 제너레이터(Generator)란?

**제너레이터는 이터레이터를 생성하는 간편한 방법**입니다.  
`yield` 키워드를 사용하여 값을 하나씩 반환합니다.

### ✔️ 장점

- 메모리 효율적 (lazy evaluation)
- 코드가 간결함

```python
def my_gen():
    for i in range(3):
        yield i

gen = my_gen()
print(next(gen))  # 0
print(next(gen))  # 1
```

---

## 📊 4. 이터러블 vs 이터레이터 vs 제너레이터

| 구분          | 이터러블 (Iterable) | 이터레이터 (Iterator) | 제너레이터 (Generator) |
|---------------|---------------------|------------------------|-------------------------|
| 사용 목적     | 반복 가능            | 값을 하나씩 꺼냄        | 이터레이터 생성 방법     |
| 메서드        | `__iter__()`         | `__iter__()`, `__next__()` | `__iter__()`, `__next__()` |
| 예시 객체     | list, tuple, str     | iter(list), file       | 함수에서 `yield` 사용    |
| 메모리 효율   | 낮음                 | 보통                   | **높음 (lazy)**         |
| 사용 방식     | `for`문 등 반복문    | `next()` 호출           | `next()` 호출            |

---

## 🧠 언제 사용하면 좋을까?

- **이터레이터**: 순차적으로 요소를 꺼내야 할 때
- **제너레이터**: 
  - 무한 시퀀스를 다룰 때 (`while True`)
  - 데이터가 많아서 **한 번에 다 메모리에 못 올릴 때**
  - **한 번만 순회하면 되는 상황**

---

## ✅ 마무리

- 이터러블은 `for`문에 사용될 수 있는 객체
- 이터레이터는 `next()`로 값을 꺼낼 수 있는 객체
- 제너레이터는 `yield`로 이터레이터를 쉽게 만드는 방법

> 반복과 데이터 처리를 더 유연하고 메모리 효율적으로 만들고 싶다면, 제너레이터를 꼭 활용해보세요!

---

✍️ 작성자: 지돌멩이  
📎 GitHub: [@yourgithub](https://github.com/yourgithub)
