# DOM

- После загрузки HTML браузер создаёт объектную модель документа — **DOM (Document Object Model)**.    
- Каждый HTML-тег превращается в объект, которым можно управлять через JavaScript.
- Для доступа к элементам используются методы: `document.querySelector`, `document.getElementById`, `document.getElementsByClassName` и другие.
- Весь JavaScript-код размещается внутри тега `<script>`:

```html
<script>
  // ваш js
</script>
```

Пример:

```js
console.log(document.querySelector('h1'));
```

Обратите внимание: HTML выполняется сверху вниз.  
Код в `<script>` имеет доступ только к тем элементам, которые уже созданы.

```html
<script>
  const el = document.getElementById("test");
  el.textContent = "changed";
</script>

<p id="test">text</p>
```

В этом примере текст **не изменится**, потому что `<p>` ещё нет в DOM на момент выполнения скрипта.  
Чтобы избежать подобных ошибок, часто `<script>` помещают в конец `<body>`.

---

# Поиск и изменение элементов

```js
// Поиск элементов:
let title = document.getElementById("my-id");           // возвращает один элемент
let header = document.querySelector('h1');              // возвращает первый подходящий элемент
let items = document.getElementsByClassName("item");    // HTMLCollection — массивоподобная коллекция
```

Изменение атрибутов:

```js
title.textContent = "Новый заголовок";
block.innerHTML = "<strong>Жирный текст</strong>";
```

Некоторые элементы имеют собственные свойства:

```js
img.src = "cat.jpg";
button.disabled = true;
```

---

# Работа с классами и стилями

Часто нужно менять внешний вид элементов (CSS) в зависимости от действий пользователя.  
Существуют два основных способа:

### Изменение стилей напрямую:

```js
const box = document.getElementById("box");

box.style.backgroundColor = "red";
box.style.width = "200px";
box.style.height = "200px";
box.style.border = "2px solid red";
```

### Изменение классов:

```js
const box = document.getElementById("box");
box.classList.add("big-red");
// также есть методы: add, remove, toggle
```

Изменение классов предпочтительнее - логика внешнего вида остаётся в CSS, а JS переключает состояния.

---

# Обработчики событий

```js
document.querySelector('#btn').onclick = () => {
  document.body.classList.toggle('dark');
};
```

---

# Создание и удаление элементов

Через JavaScript можно создавать новые узлы:

```js
const list = document.getElementById("list");

const li = document.createElement('li');
li.textContent = "Новый пункт";

list.append(li); // также есть prepend, before, after, remove
```

---

# Больше событий

```js
list.addEventListener("click", (event) => {
  // event — объект, содержащий сведения о событии
  if (event.target.tagName === "LI") {
    event.target.classList.toggle("done");
  }
});

list.addEventListener("dblclick", (event) => {
  if (event.target.tagName === "LI") {
    event.target.remove();
  }
});
```

В браузерах существует множество событий:  
[https://developer.mozilla.org/ru/docs/Web/API/Document_Object_Model/Events](https://developer.mozilla.org/ru/docs/Web/API/Document_Object_Model/Events)

Некоторые события могут конфликтовать.  
Например, перед `dblclick` всегда срабатывают два `click`. Это поведение нормальное, его нужно учитывать.

При этом не стоит решать всё через JS. Многие эффекты лучше делать через CSS:

```html
<button id="mybtn">Кнопка</button>
```

```css
.mybtn:hover {
  background-color: yellow;
}
```

Аналогичная логика на js
```js
const btn = document.getElementById("mybtn");

// при наведении мыши
btn.addEventListener("mouseenter", () => {
  btn.style.backgroundColor = "yellow";
});

// когда мышь ушла
btn.addEventListener("mouseleave", () => {
  btn.style.backgroundColor = "";
});

```

---

# Задания

### Задание 1 (DOM)

На странице есть `<h1>`, `<p>` и `<img>`.  
Через 3 секунды (используйте setTiemeout) скрипт должен изменить:

- текст заголовка,
- текст параграфа,
- изображение.

---

### Задание 2 (Поиск и изменение элементов)

Сделайте кнопку «Показать подробнее».  
При нажатии она должна менять текст в `<p>` на длинный вариант, а при повторном — на короткий.

---

### Задание 3 (Работа с классами и стилями)

Сделайте квадрат, который при клике становится красным и увеличивается, а при повторном клике возвращается в исходное состояние.

---

### Задание 4 (Создание элементов)

На странице есть пустой `<ul>`, поле `<input>` и кнопка «Добавить».  
При нажатии на кнопку создаётся новый `<li>` с введённым текстом.

---

### Задание 5 (События: click и dblclick)

Создайте список задач, в котором:

- при клике задача помечается как выполненная,
- при двойном клике — удаляется.

---
## Задание 6.

**Показать топ-10 самых популярных статей за выбранный день прямо на странице**

На странице должны быть:

- `<input type="date">`
- кнопка «Загрузить»
- пустой `<ul id="articles">`

После нажатия на кнопку:

1. Дата берётся из `<input>` и преобразуется в формат `YYYY/MM/DD`, как требует API Wiki.
2. Выполняется `fetch(...)` на:

```
https://wikimedia.org/api/rest_v1/metrics/pageviews/top/ru.wikipedia/all-access/YYYY/MM/DD
```

(не забываем про заголовок `User-Agent`)  
3. Полученные 10 самых популярных статей вставляются в список `<ul>` как `<li>Название — просмотры</li>`.  
4. Если дата неверная или API вернул ошибку — выводится сообщение об ошибке.

Пример результата:

```
1. Моско́вский Кремль — 114523 просмотра  
2. Россия — 91201 просмотр  
...
```

**Памятка по работе с инпутами**
```js
const input = document.querySelector("#userInput");
const value = input.value;
console.log(value);
```

---

## Задание 7.

**Показать график просмотров одной статьи за диапазон дат**

На странице должны быть:

- `<input type="text" placeholder="Название статьи">`
- `<input type="date" id="start">`
- `<input type="date" id="end">`
- кнопка «Показать статистику»
- `<div id="result"></div>`

После нажатия:
1. Берётся название статьи  
    — важно заменить пробелы на `_` и закодировать строку (`encodeURIComponent`).
2. Для выбранного периода по каждой дате выполняется запрос:

```
https://wikimedia.org/api/rest_v1/metrics/pageviews/per-article/ru.wikipedia/all-access/user/Статья/daily/YYYYMMDD/YYYYMMDD
```

Формат дат здесь другой — без `/`.  
3. На странице появляется список:

```
2024-10-01 — 3200 просмотров
2024-10-02 — 4100 просмотров
...
```

4. Внизу выводится итоговая сумма просмотров за период:

```
Всего: 52 400 просмотров
```