## test_Тестирование (unittest, pytest)

## **1. Основы тестирования**

### **1.1 Зачем нужно тестирование?**

- **Проверка корректности** кода.
- **Рефакторинг без страха** (тесты ловят регрессии).
- **Документирование** поведения кода.

### **1.2 Типы тестов**

|Тип|Описание|
|---|---|
|**Unit-тесты**|Тестируют отдельные функции/классы (изолированно).|
|**Интеграционные**|Проверяют взаимодействие компонентов.|
|**E2E (End-to-End)**|Тестируют систему целиком (например, через API или UI).|

---

## **2. Модуль `unittest` (стандартная библиотека)**

### **2.1 Базовый пример**
```python
import unittest

def add(a, b):
    return a + b

class TestAdd(unittest.TestCase):
    def test_add_integers(self):
        self.assertEqual(add(2, 3), 5)  # Проверка равенства

    def test_add_strings(self):
        self.assertEqual(add("hello", " world"), "hello world")

if __name__ == "__main__":
    unittest.main()  # Запуск тестов
```

**Запуск:**
```python
python -m unittest test_module.py
```

### **2.2 Основные методы `TestCase`**

|Метод|Проверяет|
|---|---|
|`assertEqual(a, b)`|`a == b`|
|`assertTrue(x)`|`bool(x) is True`|
|`assertFalse(x)`|`bool(x) is False`|
|`assertRaises(Error, func, *args)`|`func(*args)` вызывает `Error`|
|`assertIn(a, b)`|`a in b`|

---

### **2.3 Фикстуры (setUp/tearDown)**

**`setUp`** — подготовка данных перед каждым тестом.  
**`tearDown`** — очистка после каждого теста.
```python
class TestDatabase(unittest.TestCase):
    def setUp(self):
        self.conn = create_test_connection()  # Инициализация

    def tearDown(self):
        self.conn.close()  # Очистка

    def test_query(self):
        result = self.conn.execute("SELECT 1")
        self.assertEqual(result, 1)
```
**А также:**

- `setUpClass` / `tearDownClass` — один раз для всего класса.
- `setUpModule` / `tearDownModule` — один раз для модуля.

---

## **3. Pytest (популярный фреймворк)**

### **3.1 Базовый пример**

**Установка:**
```python
pip install pytest
```

**Тест:**
```python
# test_sample.py
def multiply(a, b):
    return a * b

def test_multiply():
    assert multiply(2, 3) == 6  # Простой assert вместо методов unittest
```

**Запуск:**
```python
pytest test_sample.py -v  # -v для подробного вывода
```
**Плюсы `pytest`:**

- Лаконичный синтаксис.
- Автообнаружение тестов (ищет файлы `test_*.py` и функции `test_*`).
- Поддержка фикстур и параметризации.

---

### **3.2 Фикстуры (fixtures)**

Фикстуры — это зависимости, которые можно переиспользовать.

**Пример:**
```python
import pytest

@pytest.fixture
def database():
    conn = create_test_connection()
    yield conn  # Возвращает объект
    conn.close()  # Выполнится после теста

def test_query(database):  # Фикстура передаётся как аргумент
    result = database.execute("SELECT 1")
    assert result == 1
```
**Области видимости фикстур:**

- `@pytest.fixture(scope="function")` — по умолчанию (для каждого теста).
- `scope="class"` / `"module"` / `"session"` — для экономии ресурсов.

---

### **3.3 Параметризация тестов**

**Запуск одного теста с разными входными данными:**
```python
@pytest.mark.parametrize("a, b, expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
])
def test_add(a, b, expected):
    assert add(a, b) == expected
```
Вывод:
```python
test_add[2-3-5] PASSED
test_add[0-0-0] PASSED
test_add[-1-1-0] PASSED
```

### **3.4 Моки (`unittest.mock` или `pytest-mock`)**

**Зачем?** Чтобы изолировать тест от внешних зависимостей (API, БД).

**Пример с `unittest.mock`:**
```python
from unittest.mock import Mock, patch

def get_user_name(user_id):
    response = requests.get(f"https://api.com/users/{user_id}")
    return response.json()["name"]

def test_get_user_name():
    mock_response = Mock()
    mock_response.json.return_value = {"name": "Alice"}
    
    with patch("requests.get", return_value=mock_response):
        name = get_user_name(1)
        assert name == "Alice"
```

**С `pytest-mock`:**
```python
def test_get_user_name(mocker):  # Фикстура mocker
    mock_response = mocker.Mock()
    mock_response.json.return_value = {"name": "Alice"}
    mocker.patch("requests.get", return_value=mock_response)
    
    name = get_user_name(1)
    assert name == "Alice"
```
## **4. Продвинутые техники**

### **4.1 Пропуск тестов**
```python
@pytest.mark.skip(reason="Not implemented yet")
def test_something():
    assert False

@pytest.mark.skipif(sys.version_info < (3, 8), reason="Requires Python 3.8+")
def test_new_feature():
    assert True
```

### **4.2 Проверка исключений**
```python
def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        1 / 0
```

### **4.3 Генерация тестов динамически**
```python
def generate_tests():
    return [(1, 1, 2), (2, 2, 4)]

@pytest.mark.parametrize("a,b,expected", generate_tests())
def test_dynamic(a, b, expected):
    assert a + b == expected
```

## **5. Вопросы с собеседований**

### **Вопрос 1: Чем `pytest` лучше `unittest`?**

**Ответ:**

- Лаконичный синтаксис (просто `assert`).
- Автообнаружение тестов.
- Фикстуры и параметризация из коробки.
- Богатые плагины (например, `pytest-cov` для покрытия кода).

---

### **Вопрос 2: Как тестировать приватные методы?**

**Ответ:**  
В Python нет настоящей приватности. Тестировать через:

1. Публичный интерфейс (рекомендуется).
2. Обращение к `_ClassName__method` (нежелательно).
3. Использовать `@pytest.mark.parametrize` для косвенного тестирования.

---

### **Вопрос 3: Что такое TDD?**

**Ответ:**  
**Test-Driven Development** — подход, когда:

1. Пишем **тест** до реализации.
2. Пишем **минимальный код**, чтобы тест прошёл.
3. **Рефакторим** код, сохраняя зелёные тесты.

**Плюсы:**

- Лучшая покрываемость тестами.
- Чистый дизайн кода.

---

### **Вопрос 4: Как измерить покрытие кода тестами?**

**Ответ:**  
Использовать `pytest-cov`:
```bash
pytest --cov=my_project tests/  # Покажет покрытие в %
pytest --cov-report=html --cov=my_project tests/  # Генерация HTML-отчёта
```
## **6. Практические задачи**

**Задача 1:** Написать тест для функции `is_even`.
```python
def is_even(n):
    return n % 2 == 0

# Решение:
@pytest.mark.parametrize("n, expected", [(2, True), (3, False), (0, True)])
def test_is_even(n, expected):
    assert is_even(n) == expected
```

**Задача 2:** Протестировать класс `Calculator` с моком для логгера.
```python
class Calculator:
    def __init__(self, logger):
        self.logger = logger

    def add(self, a, b):
        self.logger.log(f"Adding {a} and {b}")
        return a + b

# Решение:
def test_calculator(mocker):
    mock_logger = mocker.Mock()
    calc = Calculator(mock_logger)
    assert calc.add(2, 3) == 5
    mock_logger.log.assert_called_with("Adding 2 and 3")
```

## **Итог**

- **`unittest`** — стандартный модуль (удобен для ООП-стиля).
- **`pytest`** — мощный и лаконичный (рекомендуется для новых проектов).
- **Фикстуры** — для переиспользуемых зависимостей.
- **Моки** — для изоляции тестов.
- **Параметризация** — для DRY-тестирования.

**Совет:** Начни с простых unit-тестов, затем добавляй интеграционные.