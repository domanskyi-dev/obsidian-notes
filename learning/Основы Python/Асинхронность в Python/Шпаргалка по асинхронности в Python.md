## **Асинхронность**

### **Ключевые концепции:**

#### **Зачем нужна асинхронность?**

Для выполнения задач без блокировки потока (например, обработка множества сетевых запросов).

#### **`asyncio` и `async/await`**

Пример асинхронного чтения файла:
```python
import asyncio

async def read_file_async(filename):
    loop = asyncio.get_event_loop()
    # Выполнение блокирующей операции в отдельном потоке
    content = await loop.run_in_executor(None, open(filename, 'r').read)
    return content

async def main():
    content = await read_file_async('data.txt')
    print(content)

asyncio.run(main())
```

#### **Асинхронные HTTP-запросы с `aiohttp`**

```python
import aiohttp
import asyncio

async def fetch_data(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def main():
    data = await fetch_data('https://api.denet.com/data')
    print(data)

asyncio.run(main())
```
