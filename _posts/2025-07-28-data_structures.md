
# 📌 Python 주요 자료형: List, Tuple, Dictionary, Set

## 1. 📋 List (리스트)
- **정의**: 순서가 있고, 변경 가능한 자료형.
- **형식**: `[]` 대괄호 사용
- **특징**
  - 인덱싱, 슬라이싱 가능
  - 값 변경, 삭제, 추가 가능
  - 중복 허용

```python
fruits = ["apple", "banana", "cherry"]
fruits[1] = "grape"     # 값 변경
fruits.append("kiwi")   # 값 추가
```

---

## 2. 🔒 Tuple (튜플)
- **정의**: 순서가 있고, 변경이 불가능한 자료형.
- **형식**: `()` 소괄호 사용
- **특징**
  - 인덱싱, 슬라이싱 가능
  - **불변** 자료형 → 값 변경 불가
  - 중복 허용

```python
person = ("John", 25, "Engineer")
print(person[0])  # John
```

---

## 3. 📚 Dictionary (딕셔너리)
- **정의**: 키(key)와 값(value) 쌍으로 이루어진 자료형.
- **형식**: `{}` 중괄호 사용
- **특징**
  - 순서 있음 (Python 3.7+)
  - 키는 유일해야 함 (중복 불가)
  - 값은 변경 가능
  - 인덱싱 대신 키로 접근

```python
student = {
    "name": "Alice",
    "age": 23,
    "major": "Computer Science"
}
print(student["name"])  # Alice
student["age"] = 24      # 값 수정
```

---

## 4. 🧩 Set (셋)
- **정의**: 중복을 허용하지 않는, 순서 없는 집합 자료형.
- **형식**: `{}` 중괄호 사용 또는 `set()` 함수
- **특징**
  - 중복 자동 제거
  - 순서 없음 → 인덱싱 불가
  - 집합 연산 가능 (합집합, 교집합 등)

```python
numbers = {1, 2, 3, 3, 4}
print(numbers)  # {1, 2, 3, 4}

a = {1, 2, 3}
b = {3, 4, 5}
print(a | b)  # 합집합: {1, 2, 3, 4, 5}
print(a & b)  # 교집합: {3}
```

---

## 📎 요약 비교표

| 자료형     | 순서 | 변경 가능 | 중복 허용 | 인덱싱 | 키-값 구조 |
|------------|------|------------|-------------|----------|-------------|
| List       | O    | O          | O           | O        | X           |
| Tuple      | O    | X          | O           | O        | X           |
| Dictionary | O    | O          | 키: X, 값: O| 키로 접근 | O          |
| Set        | X    | O          | X           | X        | X           |

---

