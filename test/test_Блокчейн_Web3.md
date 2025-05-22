## test_Блокчейн_Web3

## **1. Основы Web3**

### **1.1 Что такое Web3?**
- **Web1**: Только чтение (статические сайты).
- **Web2**: Чтение-запись (соцсети, платформы).
- **Web3**: Чтение-запись-владение (децентрализация, криптоэкономика).

**Ключевые особенности:**
- **Блокчейн** — неизменяемый реестр данных.
- **Смарт-контракты** — самоисполняемый код на блокчейне (например, Ethereum).
- **Токены** — цифровые активы (NFT, криптовалюты).
- **DAOs** — децентрализованные автономные организации.

---

## **2. Ключевые компоненты Web3**

### **2.1 Блокчейны**

|Блокчейн|Особенности|Язык смарт-контрактов|
|---|---|---|
|Ethereum|Лидер для dApps, поддержка NFT/Solidity|Solidity/Vyper|
|Solana|Высокая скорость, низкие комиссии|Rust, C, C++|
|Polygon|Сайдчейн для Ethereum|Solidity|
|Binance Smart Chain|Совместимость с Ethereum|Solidity|

### **2.2 Криптокошельки**

- **MetaMask** — браузерный кошелёк для Ethereum.
- **Phantom** — кошелёк для Solana.
- **Типы кошельков**:
    - Горячие (подключены к интернету).
    - Холодные (аппаратные, например, Ledger).

**Пример создания кошелька в коде:**
```python
from web3 import Web3

w3 = Web3()
account = w3.eth.account.create()
print("Private key:", account.key.hex())
print("Address:", account.address)
```

## **3. Работа с Ethereum (Python + web3.py)**

### **3.1 Подключение к сети**
```python
from web3 import Web3

# Подключение к Infura (нода Ethereum)
w3 = Web3(Web3.HTTPProvider("https://mainnet.infura.io/v3/YOUR_KEY"))

if w3.is_connected():
    print("Connected to Ethereum!")
    print("Latest block:", w3.eth.block_number)
```

### **3.2 Чтение данных с блокчейна**

**Пример:** Получение баланса кошелька.
```python
address = "0x742d35Cc6634C0532925a3b844Bc454e4438f44e"  # Пример адреса
balance_wei = w3.eth.get_balance(address)
balance_eth = Web3.from_wei(balance_wei, "ether")
print(f"Balance: {balance_eth} ETH")
```

### **3.3 Взаимодействие со смарт-контрактами**

**Шаг 1:** Получить ABI и адрес контракта (например, USDT на Ethereum).
```python
usdt_address = "0xdAC17F958D2ee523a2206206994597C13D831ec7"
usdt_abi = [...]  # ABI контракта (можно найти на Etherscan)
```
**Шаг 2:** Создать объект контракта.
```python
usdt_contract = w3.eth.contract(address=usdt_address, abi=usdt_abi)
```
**Шаг 3:** Вызвать методы контракта.
```python
name = usdt_contract.functions.name().call()
print("Token name:", name)  # "Tether USD"
```

## **4. Транзакции в Web3**

### **4.1 Отправка ETH**
```python
private_key = "YOUR_PRIVATE_KEY"
from_address = "YOUR_ADDRESS"
to_address = "RECIPIENT_ADDRESS"

# Параметры транзакции
tx = {
    "nonce": w3.eth.get_transaction_count(from_address),
    "to": to_address,
    "value": Web3.to_wei(0.01, "ether"),
    "gas": 21000,
    "gasPrice": w3.eth.gas_price,
    "chainId": 1  # Ethereum Mainnet
}

# Подпись и отправка
signed_tx = w3.eth.account.sign_transaction(tx, private_key)
tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
print("Transaction hash:", tx_hash.hex())
```

### **4.2 Взаимодействие с контрактом (например, отправка USDT)**
```python
# Данные для вызова функции transfer()
transfer_data = usdt_contract.encodeABI(
    fn_name="transfer",
    args=[to_address, Web3.to_wei(10, "ether")]  # 10 USDT
)

# Транзакция
tx = {
    "nonce": w3.eth.get_transaction_count(from_address),
    "to": usdt_address,
    "data": transfer_data,
    "gas": 100000,
    "gasPrice": w3.eth.gas_price,
    "chainId": 1
}

signed_tx = w3.eth.account.sign_transaction(tx, private_key)
tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
```

## **5. Разработка смарт-контрактов (Solidity)**

### **5.1 Простой контракт**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Counter {
    uint256 public count;

    function increment() public {
        count += 1;
    }
}
```
### **5.2 Компиляция и деплой**

**Инструменты:**
- **Remix IDE** — онлайн-компилятор.
- **Hardhat** — фреймворк для разработки.
- **Truffle** — альтернатива Hardhat.

**Деплой через Python:**
```python
from solcx import compile_source

compiled = compile_source("""[код контракта]""")
contract_id, contract_interface = compiled.popitem()

# Деплой
tx_hash = w3.eth.contract(
    abi=contract_interface["abi"],
    bytecode=contract_interface["bin"]
).constructor().transact()
```
## **6. NFT и токены**

### **6.1 Стандарты токенов**

- **ERC-20** — взаимозаменяемые токены (например, USDT).
- **ERC-721** — NFT (уникальные токены).
- **ERC-1155** — гибрид (уникальные и взаимозаменяемые).

### **6.2 Минтинг NFT**

**Пример контракта (ERC-721):**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {
    constructor() ERC721("MyNFT", "MNFT") {}

    function mint(address to, uint256 tokenId) public {
        _mint(to, tokenId);
    }
}
```
**Вызов через Python:**
```python
nft_contract.functions.mint(to_address, 1).transact()
```

## **7. Инструменты и библиотеки**

|Инструмент|Назначение|
|---|---|
|**web3.py**|Работа с Ethereum из Python|
|**Brownie**|Фреймворк для тестирования контрактов|
|**Alchemy**|Нода как сервис (аналог Infura)|
|**OpenZeppelin**|Библиотека безопасных контрактов|
|**IPFS**|Децентрализованное хранилище файлов|

---

## **8. Вопросы с собеседований**

### **Вопрос 1: Чем Web3 отличается от традиционных баз данных?**

**Ответ:**
- Децентрализация (нет единого владельца).
- Неизменяемость данных.
- Прозрачность (все транзакции публичны).

### **Вопрос 2: Что такое gas в Ethereum?**

**Ответ:**  
Комиссия за выполнение операций в сети. Измеряется в **wei** (1 ETH = 10¹⁸ wei).

### **Вопрос 3: Как защититься от reentrancy-атак?**

**Ответ:**  
Использовать паттерн **Checks-Effects-Interactions** или модификатор `nonReentrant` из OpenZeppelin.

---

## **9. Практические задачи**

**Задача 1:** Написать скрипт, который проверяет баланс USDT на Ethereum.  
**Решение:**
```python
usdt_contract.functions.balanceOf(address).call()
```
**Задача 2:** Создать и задеплоить контракт-хранилище строк.  
**Контракт:**
```solidity
contract StringStorage {
    string public data;

    function set(string memory _data) public {
        data = _data;
    }
}
```
## **Итог**

- **Web3** — это новый интернет с децентрализацией и токенизацией.
- **Ethereum** — лидер для dApps, но есть альтернативы (Solana, Polygon).
- **web3.py** — главная библиотека для работы с блокчейном из Python.
- **Смарт-контракты** — основа Web3 (пишутся на Solidity).
