---
title: "카라츠바 알고리즘"
categories: algorithm math
comments: true
---
## 소개
- 카라츠바 알고리즘은 분할정복을 이용하여 곱셈을 최적화시킨 알고리즘이다.
- 기존 우리가 하던 방식으로는 $$O(n^2)$$ 의 시간복잡도를 가지지만, 카라츠바 알고리즘은 이것을 $$O(n^{\log_{2}3})$$ 의 시간복잡도로 바꿔준다.

## math

$$x$$와 $$y$$를 B진수 n자릿수로 표현될수있는 수라고 하자. 그리고 B를 진수, m은 $$ m\le n$$인 양의 정수라고 하자.

그러면 두 x와 y는 다음과 같이 쓸수있다.  
$$
x = x_1B^m + x_2\\
y = y_1B^m + y_2
$$

$$x_2,y_2$$는 $$B^m$$보다 작은 양의 정수이다. 그러면 $$xy$$는 다음과 같이 쓸수있다.  
$$
xy = (x_1B^m + x_2)(y_1B^m + y_2)\\
xy = x_1y_1B^{2m} + B^m(x_1y_2+x_2y_1) + x_2y_2
$$  
여기서  
$$x_1y_2 + x_2y_1 = (x_1+x_2)(y_1+y_2) - x_1y_1 - x_2y_2$$  
이므로 3번의 곱셈과 몇번의 덧셈으로 xy를 구할수있다! 즉, $$x_1y_2,x_2y_2,(x_1+x_2)(y_1+y_2)$$ 만 구하면 된다.

## algorithm

$$x_1x_2, x_2y_2,(x_1+x_2)(y_1+y_2)$$는 모두 재귀적으로 구할수있다.

그리고 저 알고리즘이 최대로 효율을 내려면 두 수중 가장 큰 수의 자릿수 / 2로 m을 택하는것이 가장 좋다.

자릿수가 모두 1자리 수가 될때 일반적인 곱셈으로 구한다.

즉, pseudo 코드는 다음과 같다.(위키에서 가져옴.)

```
procedure karatsuba(num1, num2)
  if (num1 < 10) or (num2 < 10)
    return num1*num2
  /* calculates the size of the numbers */
  m = max(size_base10(num1), size_base10(num2))
  m2 = floor(m/2)
  /* split the digit sequences in the middle */
  high1, low1 = split_at(num1, m2)
  high2, low2 = split_at(num2, m2)
  /* 3 calls made to numbers approximately half the size */
  z0 = karatsuba(low1, low2)
  z1 = karatsuba((low1 + high1), (low2 + high2))
  z2 = karatsuba(high1, high2)
  return (z2 * 10 ^ (m2 * 2)) + ((z1 - z2 - z0) * 10 ^ m2) + z0

```

## To-do

https://algospot.com/judge/problem/read/FANMEETING 풀어보기.