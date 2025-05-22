## test_ООП
## **1. Основные концепции ООП**
### **1.1 Инкапсуляция**
**Что это?** Сокрытие внутренней реализации и данных от внешнего мира.  
**Как в Python?**
- **Публичные атрибуты:** `self.name`
- **Защищённые (protected):** `self._name` (по соглашению, но доступ есть)
- **Приватные (private):** `self.__name` (name mangling: `_ClassName__name`)
**Пример:**
```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance  # Приватный атрибут

    def get_balance(self):        # Геттер
        return self.__balance

account = BankAccount(1000)
print(account.get_balance())      # 1000
print(account._BankAccount__balance)  # 1000 (но так делать не стоит!)
```
**Собеседование:**  
_Зачем нужны геттеры/сеттеры?_  
→ Для валидации данных и контроля доступа.

### **1.2 Наследование**

**Что это?** Создание нового класса на основе существующего.  
**Пример:**
```python
class Animal:
    def speak(self):
        raise NotImplementedError("Subclasses must implement this method")

class Dog(Animal):
    def speak(self):
        return "Woof!"

dog = Dog()
print(dog.speak())  # "Woof!"
```
**Множественное наследование:**
```python
class A:
    pass

class B:
    pass

class C(A, B):  # Порядок важен (Method Resolution Order — MRO)
    pass
```
**Собеседование:**  
_Что такое MRO?_  
→ Алгоритм поиска методов в иерархии наследования (`C.__mro__`).

### **1.3 Полиморфизм**

**Что это?** Возможность использовать один интерфейс для разных типов.  
**Пример:**
```python
def make_sound(animal):
    print(animal.speak())

make_sound(Dog())  # "Woof!"
make_sound(Cat())  # "Meow!" (если класс Cat тоже реализует speak())
```

### **1.4 Абстракция**

**Что это?** Создание абстрактных классов (шаблонов).  
**Как в Python?** Через модуль `abc`:
```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Circle(Shape):
    def area(self):
        return 3.14 * self.radius ** 2
```
**Собеседование:**  
_Зачем нужны абстрактные классы?_  
→ Чтобы гарантировать, что подклассы реализуют определённые методы.

## **2. Магические методы (Dunder Methods)**

Методы, которые начинаются и заканчиваются на `__`. Позволяют переопределять поведение объектов.

### **Основные магические методы**
| Метод      | Описание                  | Пример вызова   |
| ---------- | ------------------------- | --------------- |
| `__init__` | Конструктор               | `obj = Class()` |
| `__str__`  | Строковое представление   | `print(obj)`    |
| `__repr__` | Формальное представление  | `repr(obj)`     |
| `__eq__`   | Сравнение (`==`)          | `obj1 == obj2`  |
| `__len__`  | Длина объекта             | `len(obj)`      |
| `__call__` | Вызов объекта как функции | `obj()`         |
**Пример:**
```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __str__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)  # "Vector(4, 6)"
```
**Собеседование:**  
_В чём разница между `__str__` и `__repr__`?_  
→ `__str__` — для пользователя, `__repr__` — для разработчика (например, в REPL).

## **3. Классовые и статические методы**

### **3.1 `@classmethod`**

Принимает класс (`cls`) в качестве первого аргумента. Используется для фабричных методов.
```python
class Pizza:
    def __init__(self, ingredients):
        self.ingredients = ingredients

    @classmethod
    def margherita(cls):
        return cls(["tomato", "mozzarella"])

pizza = Pizza.margherita()
```

### **3.2 `@staticmethod`**

Не получает ни `self`, ни `cls`. Это обычная функция внутри класса.
```python
class Math:
    @staticmethod
    def add(a, b):
        return a + b

Math.add(2, 3)  # 5
```

**Собеседование:**  
_Когда использовать `@staticmethod` вместо `@classmethod`?_  
→ Когда метод не требует доступа к классу или экземпляру.

## **4. Дескрипторы и property**

### **4.1 `@property`**

Позволяет превратить метод в атрибут.
```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value

circle = Circle(5)
print(circle.radius)  # 5 (вызывается геттер)
circle.radius = 10    # Вызывается сеттер
```

### **4.2 Дескрипторы**

Классы с `__get__`, `__set__`, `__delete__`. Используются, например, в `@property`.
```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, owner):
        return obj.__dict__[self.name]

    def __set__(self, obj, value):
        if value < 0:
            raise ValueError("Value must be positive")
        obj.__dict__[self.name] = value

class Person:
    age = PositiveNumber()

p = Person()
p.age = 30  # ОК
p.age = -5  # ValueError
```
**Собеседование:**  
_Как работает `@property` под капотом?_  
→ Это дескриптор, который управляет доступом к атрибуту.

## **5. Метаклассы**

### **5.1 Что это?**

Классы, которые создают другие классы (например, `type` — базовый метакласс).

### **5.2 Пример**
```python
class Meta(type):
    def __new__(cls, name, bases, dct):
        dct["version"] = 1.0
        return super().__new__(cls, name, bases, dct)

class MyClass(metaclass=Meta):
    pass

print(MyClass.version)  # 1.0
```
**Собеседование:**  
_Где используются метаклассы?_  
→ В ORM (Django), валидации, регистрации классов.

## **6. Слоты (`__slots__`)**

### **6.1 Что это?**

Ограничение атрибутов экземпляра для экономии памяти.
```python
class Point:
    __slots__ = ["x", "y"]  # Запрещает создание других атрибутов

    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
p.z = 3  # AttributeError
```

**Плюсы:**

- Экономит память (не создаёт `__dict__`).
- Ускоряет доступ к атрибутам.

**Минусы:**

- Нельзя динамически добавлять атрибуты.

**Собеседование:**  
_Когда использовать `__slots__`?_  
→ Когда есть много экземпляров с фиксированным набором атрибутов.

## **7. Вопросы с собеседований**

### **Вопрос 1: Что такое `self`?**

**Ответ:**  
`self` — ссылка на экземпляр класса. Это просто соглашение (можно использовать другое имя).

### **Вопрос 2: Как сделать класс неизменяемым?**

**Ответ:**  
Переопределить `__setattr__`:
```python
class Immutable:
    def __setattr__(self, key, value):
        raise AttributeError("Cannot modify immutable object")
```

### **Вопрос 3: Как реализовать синглтон?**

**Ответ:**  
Через метакласс или декоратор:
```python
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Logger(metaclass=Singleton):
    pass
```
## **8. Практика**

**Задача 1:** Написать класс `Counter` с методом `increment()`, который увеличивает счётчик на 1.
```python
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1
        return self.count
```

**Задача 2:** Реализовать класс `Rectangle` с методами `area()` и `perimeter()`.
```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)
```

## **Итог**

- **Инкапсуляция:** Сокрытие данных (`private`, `protected`).
- **Наследование:** Создание иерархий классов.
- **Полиморфизм:** Один интерфейс — разные реализации.
- **Абстракция:** Абстрактные классы (`ABC`).
- **Магические методы:** Переопределение поведения (`__str__`, `__add__`).
- **Метаклассы:** Создание классов на лету.