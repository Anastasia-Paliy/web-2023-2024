## 0. Подготовка

```bash
pip install mariadb
```

```python
import mariadb
```

---

## 1. Подключение к MariaDB

```python
conn = mariadb.connect(
    host="localhost",
    port=3306,
    user="app_user",
    password="secret",
    database="school",
    autocommit=False   # важно для UPDATE / DELETE
)

cursor = conn.cursor()
```

👉 `cursor` — объект, через который выполняется SQL  
👉 `conn` — управляет транзакциями (`commit`, `rollback`)

---

## 2. SELECT — получение данных

### 2.1 Одна строка (`fetchone`)

```python

id = "123"
cursor.execute(
    "SELECT id, name, age FROM students WHERE id = ?",
    (id, )
)

row = cursor.fetchone()
print(row)        # (1, 'Alice', 20)
```

```python
if row:
    student_id, name, age = row
```

👉 `?` — защита от SQL-инъекций  
👉 всегда передаём параметры **кортежем**

---

### 2.2 Несколько строк (`fetchall`)

```python
cursor.execute(
    "SELECT id, name FROM students WHERE age >= ?",
    (18,)
)

rows = cursor.fetchall()
print(rows)
```

```python
for student_id, name in rows:
    print(student_id, name)
```

---

### 2.3 Итерация без `fetchall` (для больших таблиц)

```python
cursor.execute("SELECT id, name FROM students")

for row in cursor:
    print(row)
```

👉 экономит память

---

## 3. SELECT → словари (удобно для API)

```python
cursor = conn.cursor(dictionary=True)

cursor.execute("SELECT id, name  FROM students")
rows = cursor.fetchall()

print(rows[0]["name"])
```

👉 часто используется в web-приложениях

---

## 4. INSERT — добавление данных

```python
cursor.execute(
    "INSERT INTO students (name, age) VALUES (?, ?)",
    ("Bob", 22)
)

conn.commit()
```

### Получение `id` новой записи

```python

new_id = cursor.lastrowid
print(new_id)
```

---

## 5. UPDATE — обновление строк

```python
cursor.execute(
    "UPDATE students SET age = ? WHERE id = ?",
    (23, 1)
)

conn.commit()
print(cursor.rowcount)  # сколько строк изменено
```

👉 если `rowcount == 0` — запись не найдена

---

## 6. DELETE — удаление

```python
cursor.execute(
    "DELETE FROM students WHERE id = ?",
    (5,)
)

conn.commit()
```

---

## 7. Транзакции (важно!)

```python
try:
    cursor.execute("UPDATE students SET age = age + 1")
    cursor.execute("DELETE FROM students WHERE age > 30")

    conn.commit()
except Exception as e:
    conn.rollback()
    print("Ошибка:", e)
```

👉 либо всё применяется, либо ничего

---

## 8. Пример: функция репозитория (реальный стиль)

```python
def get_student_by_id(conn, student_id):
    cur = conn.cursor(dictionary=True)
    cur.execute(
        "SELECT * FROM students WHERE id = ?",
        (student_id,)
    )
    return cur.fetchone()
```

```python
student = get_student_by_id(conn, 1)
if student:
    print(student["name"])
```

---

## 9. Частые ошибки

❌ **НЕ ТАК**

```python
cursor.execute(
    f"SELECT * FROM students WHERE name = '{name}'"
)
```

✔ **ТАК**

```python
cursor.execute(
    "SELECT * FROM students WHERE name = ?",
    (name,)
)
```

---

## 10. Закрытие соединения

```python
cursor.close()
conn.close()
```

---

## Мини-шпаргалка

```text
execute()      — выполнить SQL
fetchone()     — одна строка
fetchall()     — список строк
rowcount       — сколько строк затронуто
lastrowid      — id вставленной записи
commit()       — сохранить изменения
rollback()     — отменить
```




# Учебное задание

## Student Registry API (FastAPI + MariaDB)

### Цель

Написать небольшой HTTP-сервер на **FastAPI**, который:

- подключается к **MariaDB**
- читает данные из БД
- добавляет, обновляет и удаляет записи
- использует **реальные SQL-запросы из Python-кода**

---

## Предметная область

### Таблица `students`

```sql
CREATE TABLE students (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    age INT NOT NULL,
    group_name VARCHAR(20)
);
```

---

# Этап 0. Подготовка проекта

### Требования

- MariaDB
- Библиотеки:

```bash
pip install fastapi uvicorn mariadb
```

### Структура проекта

```
project/
├── main.py
├── db.py
```

---

# Этап 1. Подключение к БД

### Задание

Создать файл `db.py`, который:

1. Подключается к MariaDB
2. Предоставляет функцию получения соединения

### Требования

- Использовать `mariadb.connect`

### Критерий готовности

- Сервер запускается
- Подключение к БД создаётся без ошибок

---

# Этап 2. Первый endpoint — получить всех студентов

### Endpoint

```
GET /students
```

### Задание

- Получить **все записи** из таблицы `students`
- Вернуть список JSON-объектов

### Пример ответа

```json
[
  {"id": 1, "name": "Alice", "age": 20, "group_name": "A1"},
  {"id": 2, "name": "Bob", "age": 22, "group_name": "B2"}
]
```

### Требования

- Использовать `SELECT`
- Использовать `fetchall`
- Советую использовать `dictionary=True`

### Критерий готовности

- `curl /students` возвращает список студентов

---

# Этап 3. Получение одной записи по id

### Endpoint

```
GET /students/{student_id}
```

### Задание

- Найти студента по `id`
- Если не найден — вернуть `404`

### Требования

- `fetchone`
- SQL с параметром (`?`)
- Обработка `None`

### Пример ошибок

```json
{"detail": "Student not found"}
```

---

# Этап 4. Добавление нового студента

### Endpoint

```
POST /students
```

### Тело запроса

```json
{
  "name": "Charlie",
  "age": 19,
  "group_name": "A1"
}
```

### Задание

- Добавить запись в БД
- Вернуть `id` созданного студента

### Требования

- `INSERT`
- `commit`
- `lastrowid`

### Пример ответа

```json
{"id": 5}
```

---

# Этап 5. Обновление существующего студента

### Endpoint

```
PUT /students/{student_id}
```

### Тело запроса

```json
{
  "age": 21,
  "group_name": "B1"
}
```

### Задание

- Обновить **только существующую** запись
- Если `id` не найден — `404`

### Требования

- `UPDATE`
- Проверка `rowcount`
- `commit`

---

# Этап 6. Удаление студента

### Endpoint

```
DELETE /students/{student_id}
```

### Задание

- Удалить запись по `id`
- Если запись отсутствует — `404`

### Требования

- `DELETE`
- Проверка `rowcount`

---

# Этап 7. Транзакции и ошибки

### Задание

- Обернуть `INSERT / UPDATE / DELETE` в `try / except`
- При ошибке делать `rollback`

### Критерий готовности

- Ошибка в SQL **не ломает сервер**
- Данные не повреждаются

---

# Этап 8. Мини-рефакторинг (архитектура)

### Задание

- Вынести SQL-запросы в отдельные функции:
    
    - `get_all_students`
    - `get_student_by_id`
    - `create_student`
    - `update_student`
    - `delete_student`

### Цель

Показать:

> FastAPI != SQL  
> SQL живёт в отдельном слое

---

# Этап 9. Бонус (по желанию)

- Фильтр:

```
GET /students?group=A1
```

- Ограничение:

```
GET /students?limit=10
```

- Проверка входных данных через Pydantic