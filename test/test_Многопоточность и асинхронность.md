## test_Многопоточность и асинхронность

## **1. Многопоточность (`threading`)**

### **1.1 Базовые понятия**

- **Поток (Thread)** — легковесный процесс, разделяющий память с другими потоками.
- **GIL (Global Interpreter Lock)** — блокировка, которая позволяет выполняться только одному потоку Python за раз (мешает параллельным CPU-операциям).

**Когда использовать:**

- I/O-задачи (сеть, файлы, ожидание).
- Не подходит для CPU-интенсивных операций.

### **1.2 Создание потоков**

**Способ 1:** Через `Thread` из модуля `threading`.
```python
import threading

def task():
    print("Выполняется в потоке")

thread = threading.Thread(target=task)
thread.start()  # Запуск потока
thread.join()   # Ожидание завершения
```

**Способ 2:** Наследование от `Thread`.
```python
class MyThread(threading.Thread):
    def run(self):
        print("Поток запущен")

thread = MyThread()
thread.start()
```

### **1.3 Синхронизация потоков**

#### **Блокировки (`Lock`)**

Предотвращают состояние гонки (race condition).
```python
lock = threading.Lock()
counter = 0

def increment():
    global counter
    with lock:  # Автоматически освобождается после блока
        counter += 1

threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()
print(counter)  # Гарантированно 10
```

#### **Другие примитивы:**

- `RLock` — реентерабельная блокировка.
- `Semaphore` — ограничивает количество потоков.
- `Event` — уведомления между потоками.

### **1.4 Очереди (`Queue`)**

Безопасный обмен данными между потоками.
```python
from queue import Queue

q = Queue()

def worker():
    while True:
        item = q.get()
        print(f"Обработано: {item}")
        q.task_done()

threading.Thread(target=worker, daemon=True).start()

q.put("Задача 1")
q.put("Задача 2")
q.join()  # Ожидает завершения всех задач
```

## **2. Многопроцессорность (`multiprocessing`)**

### **2.1 Зачем использовать?**

- Обходит GIL — настоящая параллельность для CPU-задач.
- Каждый процесс имеет свою память.

### **2.2 Базовый пример**
```python
from multiprocessing import Process

def task():
    print("Процесс запущен")

process = Process(target=task)
process.start()
process.join()
```

### **2.3 Обмен данными**

#### **Очереди (`multiprocessing.Queue`)**
```python
from multiprocessing import Queue

q = Queue()
q.put("Данные")
print(q.get())  # "Данные"
```

#### **Общая память (`Value`, `Array`)**
```python
from multiprocessing import Value, Array

counter = Value("i", 0)  # 'i' — тип int
arr = Array("d", [1.0, 2.0])  # 'd' — double
```

## **3. Асинхронность (`asyncio`)**

### **3.1 Корутины и Event Loop**

- **Корутина (coroutine)** — функция, которая может приостанавливаться (`async def`).
- **Event Loop** — планировщик выполнения корутин.

**Базовый пример:**
```python
import asyncio

async def hello():
    print("Привет")
    await asyncio.sleep(1)  # Неблокирующая задержка
    print("Мир")

asyncio.run(hello())  # Запуск event loop
```

### **3.2 Параллельное выполнение**

#### **`gather` — запуск нескольких корутин**

```python
async def task1():
    await asyncio.sleep(1)
    return "Результат 1"

async def task2():
    await asyncio.sleep(2)
    return "Результат 2"

async def main():
    results = await asyncio.gather(task1(), task2())
    print(results)  # ["Результат 1", "Результат 2"]

asyncio.run(main())
```

#### **`create_task` — фоновые задачи**
```python
async def background_task():
    await asyncio.sleep(2)
    print("Фоновая задача завершена")

async def main():
    task = asyncio.create_task(background_task())
    print("Основной поток продолжает работу")
    await task  # Ожидание завершения

asyncio.run(main())
```

### **3.3 Асинхронные HTTP-запросы**

Используем `aiohttp` вместо `requests`:
```python
import aiohttp

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def main():
    html = await fetch("https://example.com")
    print(html[:100])  # Первые 100 символов

asyncio.run(main())
```

## **4. Сравнение подходов**

|Характеристика|`threading`|`multiprocessing`|`asyncio`|
|---|---|---|---|
|**Параллельность**|Нет (GIL)|Да|Нет (кооперативная)|
|**Память**|Общая|Раздельная|Общая|
|**Подходит для**|I/O-задачи|CPU-задачи|I/O-задачи|
|**Сложность**|Средняя (блокировки)|Высокая|Низкая (если освоено)|

---

## **5. Вопросы с собеседований**

### **Вопрос 1: Что такое GIL? Как он влияет на потоки?**

**Ответ:**  
GIL (Global Interpreter Lock) — это мьютекс, который позволяет выполняться только одному потоку Python за раз. Он мешает использовать многопоточность для CPU-задач, но не ограничивает I/O-операции.

---

### **Вопрос 2: Как ускорить CPU-интенсивный код?**

**Ответ:**  
Использовать `multiprocessing` или вынести вычисления в C-расширения (например, через `numba` или `cython`).

---

### **Вопрос 3: Чем `asyncio` лучше `threading` для I/O?**

**Ответ:**

- Потоки потребляют больше памяти.
- `asyncio` избегает накладных расходов на переключение потоков.
- Нет риска race conditions (однопоточная модель).

---

### **Вопрос 4: Как обработать 1000 URL параллельно?**

**Решение на `asyncio`:**
```python
async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [session.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [await resp.text() for resp in responses]
```

## **6. Практические задачи**

**Задача 1:** Написать многопоточный сканер портов.
```python
import socket
from concurrent.futures import ThreadPoolExecutor

def scan_port(host, port):
    with socket.socket() as s:
        s.settimeout(1)
        if s.connect_ex((host, port)) == 0:
            print(f"Порт {port} открыт")

with ThreadPoolExecutor(max_workers=100) as executor:
    for port in range(1, 1025):
        executor.submit(scan_port, "localhost", port)
```

**Задача 2:** Асинхронный парсер новостей.
```python
import asyncio
from bs4 import BeautifulSoup

async def parse_news(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            soup = BeautifulSoup(await response.text(), "html.parser")
            titles = [h2.text for h2 in soup.find_all("h2")]
            return titles

async def main():
    urls = ["https://news.com/page1", "https://news.com/page2"]
    results = await asyncio.gather(*[parse_news(url) for url in urls])
    print(results)

asyncio.run(main())
```

## **Итог**

- **Потоки (`threading`)** — для I/O-задач, но есть GIL.
- **Процессы (`multiprocessing`)** — для CPU-задач.
- **Асинхронность (`asyncio`)** — для масштабируемых I/O-приложений.

**Совет:**

- Для сетевых задач выбирай `asyncio`.
- Для вычислений — `multiprocessing`.
- Всегда тестируй на реальных данных!