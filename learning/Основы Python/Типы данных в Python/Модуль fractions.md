# Модуль fractions
Модуль fractions предоставляет поддержку рациональных чисел.

class **fractions.Fraction**(numerator=0, denominator=1)
class **fractions.Fraction**(other_fraction)
class **fractions.Fraction**(float)
class **fractions.Fraction**(decimal)
class **fractions.Fraction**(string)

Класс, представляющий собой рациональные числа. Экземпляр класса можно создать из пары чисел (числитель, знаменатель), из другого рационального числа, числа с плавающей точкой, числа типа decimal.Decimal, и из строки, представляющей собой число.

```python
from fractions import Fraction
Fraction(1, 3)
# Fraction(1, 3)
Fraction(2, 6)
# Fraction(1, 3)
Fraction(100)
# Fraction(100, 1)
Fraction()
# Fraction(0, 1)
Fraction('3/7')
# Fraction(3, 7)
Fraction(' 3/7 ')
# Fraction(3, 7)
Fraction('3.1415')
# Fraction(6283, 2000)
Fraction(3.1415)
# Fraction(7074029114692207, 2251799813685248)
```
Необходимо заметить, что, поскольку числа с плавающей точкой не совсем точны, получающееся рациональное число может отличаться от того, что мы хотим получить.

Рациональные числа можно, как int и float, складывать, умножать, делить...
```python
from fractions import Fraction
a = Fraction(1, 7)
b = Fraction(1, 3)
a + b
# Fraction(10, 21)
a - b
# Fraction(-4, 21)
a * b
# Fraction(1, 21)
a / b
# Fraction(3, 7)
a % b
# Fraction(1, 7)
b % a
# Fraction(1, 21)
a ** b
# 0.5227579585747102
abs(a - b)
# Fraction(4, 21)
```

**Fraction.limit_denominator**(max_denominator=1000000) - ближайшее рациональное число со знаменателем не больше данного.
```python
from fractions import Fraction
a = Fraction(3.1415)
a
# Fraction(7074029114692207, 2251799813685248)
a.limit_denominator()
# Fraction(6283, 2000)
```

Также, помимо класса рациональных чисел, модуль fractions предоставляет функцию для нахождения наибольшего общего делителя.

**fractions.gcd**(a, b) - наибольший общий делитель чисел a и b.
```python
from fractions import gcd
gcd(1, 5)
# 1
gcd(1000, 3)
# 1
gcd(4, 6)
# 2
gcd(0, 2)
# 2
gcd(0, 0)
# 0
```
