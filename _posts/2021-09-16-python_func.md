---
title: Python 함수 및 속성 정리
date: 2021-09-16 13:02 +0900
categories: [python]
tags: [python, function]
lastmod: 2021-09-30 13:21 +0900
mermaid: true
math: true
---

> python, tensorflow에서 사용되는 함수들을 정리한 것입니다. 이 포스트는 계속 업데이트 할 예정입니다.

# numpy.unique(arr, return_index, return_inverse, return_counts)

- 중복되지 않는 고유한 요소들의 배열을 리턴
- 고유 값들의 배열의 튜플 또는 연관된 인덱스를 리턴할 수 있다
- 인덱스의 특성은 함수 호출에서 매개 변수 유형에 달려있다

---

# np.where(condition[, x, y])

- 조건에 맞는 값의 색인 위치를 찾는다

```python
a = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9])

print(np.where(a >= 5))
# (array([4, 5, 6, 7, 8]),)
```

---

# 비트 연산자

### ~: 보수 연산

- a = 60 이면 ~a = -61

---

# np.setdiff1d(x, y)

- 첫 번째 배열 x로 부터 두 번째 배열 y를 뺀 차집합을 반환

---

# argparse

- 명령행 옵션, 인자와 부속 명령을 위한 파서

## 사용법

```python
import argparse

parser = argparse.ArgumentParser(description='Main script for training VAE')

    # Data params:
    parser.add_argument('--len_x', type=int, default=6, help='Sequence length (previous-trajectory)')
    parser.add_argument('--len_y', type=int, default=12, help='Sequence length (posterior-trajectory)')
    parser.add_argument('--sp_x', type=int, default=0, help='Use sparse x (previous-trajectory)')
    parser.add_argument('--sp_y', type=int, default=0, help='Use sparse y (posterior-trajectory)')

    # Experiment params:
    parser.add_argument('--dim_z', type=int, default=16,
                        help='Dimension of (Gaussian) latent vector z. Recommend 32, 64 or 128.')
    parser.add_argument('--kl_weight', type=float, default=1.0, help='KL weight of loss equation.')
    parser.add_argument('--batch_size', type=int, default=64, help='Minibatch size. (32, 64, 128, 256)')

    args = parser.parse_args()
```

- add_argument에 '—' 이 붙으면 positional (필수)고 '-' 이 붙으면 optional 이다.

---

# np.random.permutaion

- 무작위로 섞인 배열을 만든다

---

# keras.optimizers.schedules.ExponentialDecay

```python
tf.keras.optimizers.schedules.ExponentialDecay(
    initial_learning_rate, decay_steps, decay_rate, staircase=False, name=None
)
```

---

# axis의 기능

```python
import numpy as np

arr = np.random.randint(10, size=(5, 3))
arr2 = np.random.randint(10, size=(4, 3))
arr3 = np.concatenate([arr, arr2], axis=0)

print(arr.shape)                # (5, 3)
print(arr2.shape)               # (4, 3)
print('--------------------')
print(arr3.shape)               # (9, 3)
```

- concatenate연산에서 axis의 값이 0일 경우 배열 shape의 0번째인 5와 4를 concatenate 연산을 진행한다.
- 위 코드의 결과를 보면 (5, 3) 배열과 (4, 3) 배열의 concatenate 연산 결과는 (9, 3)으로 나오는 것을 볼 수 있다.
- numpy의 sum, mean, min, max 등의 연산에서 axis 속성이 사용된다.

---

# eval(str)

- `eval()`함수는 괄호 안 문자열을 연산해준다

```python
eval("1+2")
3
```

- 괄호 안에 "1 + 2"를 입력하면 3을 반환한다

---

# list extend()

```python
a = [1, 2, 3]
b = [4, 5]

a.extend(b)
print(a)
----------------
[1, 2, 3, 4, 5]
```

- list append() 대신 extend()를 사용하면 삽입 대상의 리스트를 풀어서 각각의 엘리먼트로 확장해 삽입한다
