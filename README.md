# Сайт-резюме от HTML до микросервисов: пошаговый практикум

> **Среда:** Arch Linux · IntelliJ IDEA Ultimate · HTML · CSS · JavaScript · Java 21 · Maven Wrapper · Spring Boot · PostgreSQL · Flyway · Docker Compose · Nginx · Unity Web / Brotli  
> **Принцип обучения:** каждая новая технология появляется только тогда, когда для неё возникает реальная задача. Сначала делаем работающий фронтенд, потом сервер, затем отдельный сервис данных и только после этого базу данных и контейнеризацию.

## Содержание

1. [Что создаём и зачем](#что-создаём-и-зачем)
2. [Часть I. Фронтенд без Java и базы данных (шаги 1–74)](#часть-i-фронтенд-без-java-и-базы-данных-шаги-174)
3. [Часть II. Первый backend: Java, Maven, Spring Boot (шаги 75–91)](#часть-ii-первый-backend-java-maven-spring-boot-шаги-7591)
4. [Часть III. Два сервиса, но ещё без PostgreSQL (шаги 92–108)](#часть-iii-два-сервиса-но-ещё-без-postgresql-шаги-92108)
5. [Часть IV. PostgreSQL, миграции, SQL и сохранение данных (шаги 109–155)](#часть-iv-postgresql-миграции-sql-и-сохранение-данных-шаги-109155)
6. [Часть V. Docker для приложений (шаги 156–166)](#часть-v-docker-для-приложений-шаги-156166)
7. [Часть VI. Unity, Brotli и Nginx (шаги 167–180)](#часть-vi-unity-brotli-и-nginx-шаги-167180)
8. [Часть VII. Тестирование и публикация (шаги 181–200)](#часть-vii-тестирование-и-публикация-шаги-181200)
9. [Итоговая структура и чек-лист](#итоговая-структура-и-чек-лист)
10. [Официальная документация](#официальная-документация)

---

## Что создаём и зачем

Это **личный сайт-резюме Java-разработчика**. На главной странице размещаются сведения о тебе, навыки, опыт, контакты и ссылки на работающие мини-проекты. Демо не заменяют резюме, а доказывают заявленные навыки.

| Страница | Что демонстрирует | Технологии |
|---|---|---|
| `/` | Имя, роль, «Обо мне», образование, проекты, навыки, контакты | HTML, CSS, JS |
| `/catalog.html` | Поиск с подсказками, пагинация по страницам, бесконечная лента, 12–50 тыс. элементов, счётчики категорий | JS → Spring MVC → SQL |
| `/board.html` | Kanban-доска, изменение порядка и перетаскивание карточек | Drag and Drop, REST |
| `/upload.html` | Drag and drop файлов, предпросмотр, позже серверная загрузка | File API, multipart |
| `/gallery.html` | Прокрутка и карусель фотографий | CSS Scroll Snap, JS |
| `/ui.html` | Модальные окна, кастомный dropdown, анимации | HTML, CSS, DOM, ARIA |
| `/unity.html` | Встраивание Unity Web | WebAssembly, Brotli, Nginx |

**Эволюция архитектуры** — именно в таком порядке:

```text
A. Браузер
   HTML + CSS + JavaScript
   Демо-данные: массивы JS и localStorage

B. Один backend
   Браузер ─HTTP─> web-service (Spring Boot)
   Данные: временно Java-объекты в памяти

C. Два backend-сервиса
   Браузер ─HTTP─> web-service ─HTTP─> repository-service
   Данные: временно Java-объекты в памяти repository-service

D. Постоянное хранилище
   Браузер ─HTTP─> web-service ─HTTP─> repository-service ─JDBC─> PostgreSQL

E. Развёртывание
   Браузер ─> Nginx ─> web-service ─> repository-service ─> PostgreSQL
                 └──> Unity Web (.br)
   Все серверные компоненты работают в контейнерах
```

В итоговой системе **только `repository-service` соединяется с PostgreSQL**. `web-service` отдаёт статические страницы и публичное API; к внутреннему сервису он обращается по HTTP. У сервисов разные JAR и Docker-контейнеры. Это учебная архитектура: в коммерческих системах границы микросервисов чаще выбирают по бизнес-доменам, а не только по техническому слою хранилища.

| Технология | Впервые нужна | Зачем |
|---|---|---|
| HTML | Шаг 4 | Структура резюме |
| CSS | Шаг 5 | Оформление, адаптивность, анимации |
| JavaScript | Шаг 12 и далее | Интерактивность, состояние, перетаскивание |
| Java 21 | Шаг 75 | Серверная логика |
| Maven Wrapper | Шаг 76 | Сборка Spring Boot-приложения |
| Spring Boot | Шаг 76 | HTTP API и раздача сайта |
| `repository-service` | Шаг 92 | Отделение ответственности за данные |
| Docker | Шаг 109 | Сначала удобный запуск PostgreSQL |
| PostgreSQL | Шаг 109 | Сохранение и SQL-запросы |
| Flyway | Шаг 113 | Контролируемые версии схемы БД |
| Docker для сервисов | Шаг 158 | Воспроизводимый запуск всей системы |
| Nginx, Unity, Brotli | Шаг 167+ | Публикация Web-сборки с правильными заголовками |

**Ритм работы.** Сделай один небольшой шаг, открой страницу или вызови API, проверь результат, затем сохрани изменения в Git. Каждый раздел заканчивается контрольной точкой. Не добавляй новую технологию, пока не работает предыдущая.

---

# Часть I. Фронтенд без Java и базы данных (шаги 1–74)

## Этап 1. Пустой проект и первая страница

### Шаг 1. Создай пустой проект в IntelliJ IDEA Ultimate

**File → New → Project → Empty Project**, имя `portfolio-lab`. Например, директория `~/Projects/portfolio-lab`. Пока **не** выбирай Spring Boot или Maven. Для работы со статическими страницами они не нужны.

### Шаг 2. Создай структуру фронтенда

Через **Project** (`Alt+1`) → правой кнопкой → **New → Directory / File** создай:

```text
portfolio-lab/
└── frontend/
    ├── index.html
    ├── catalog.html
    ├── board.html
    ├── upload.html
    ├── gallery.html
    ├── ui.html
    ├── unity.html
    └── assets/
        ├── css/
        │   └── style.css
        ├── js/
        │   ├── main.js
        │   ├── catalog.js
        │   ├── board.js
        │   ├── upload.js
        │   ├── gallery.js
        │   └── ui.js
        └── images/
```

HTML задаёт структуру, CSS — оформление, JavaScript — поведение. Позднее всё это перенесём в ресурсы Spring Boot почти без изменений.

### Шаг 3. Инициализируй Git

В терминале IDEA (`Alt+F12`):

```bash
sudo pacman -S --needed git
git init
```

Создай `.gitignore` в корне:

```gitignore
.idea/
*.iml
target/
.env
.DS_Store
```

```bash
git add .
git commit -m "Initial frontend project"
```

Если Git попросит имя и email, настрой их с помощью `git config --global user.name ...` и `git config --global user.email ...`.

### Шаг 4. Напиши HTML главной страницы

`frontend/index.html`:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Моё портфолио | Java Developer</title>
    <link rel="stylesheet" href="assets/css/style.css">
    <script src="assets/js/main.js" defer></script>
</head>
<body>
<header class="site-header">
    <a class="logo" href="index.html">Portfolio</a>
    <nav aria-label="Основная навигация">
        <a href="index.html">Обо мне</a>
        <a href="catalog.html">Каталог</a>
        <a href="board.html">Доска</a>
        <a href="upload.html">Файлы</a>
        <a href="gallery.html">Галерея</a>
        <a href="ui.html">UI</a>
        <a href="unity.html">Unity</a>
    </nav>
</header>
<main>
    <section class="hero">
        <p>Привет! Меня зовут [Имя]</p>
        <h1>Java Backend Developer</h1>
        <p>Изучаю Spring Boot, PostgreSQL, Docker и микросервисы.</p>
        <a class="button" href="#projects">Смотреть проекты</a>
    </section>
    <section id="about">
        <h2>Обо мне</h2>
        <p>Здесь будет краткая информация обо мне.</p>
    </section>
    <section id="skills">
        <h2>Мои навыки</h2>
        <p>Java, HTML, CSS, JavaScript</p>
    </section>
    <section id="projects">
        <h2>Мои проекты</h2>
        <div class="project-grid">
            <article class="project-card">
                <h3>Каталог товаров</h3>
                <p>Пагинация, поиск и фильтрация.</p>
                <a href="catalog.html">Открыть демо</a>
            </article>
        </div>
    </section>
    <section id="contacts">
        <h2>Контакты</h2>
        <p>GitHub, email и другие ссылки.</p>
    </section>
</main>
<footer><p>Моё портфолио</p></footer>
</body>
</html>
```

Здесь `header` — шапка, `nav` — навигация, `main` — основное содержимое, `section` — раздел, `article` — самостоятельная карточка, `footer` — нижняя часть сайта. Выбирай элементы по смыслу, а не по внешнему виду. [MDN: HTML](https://developer.mozilla.org/ru/docs/Learn_web_development/Core/Structuring_content).

### Шаг 5. Добавь CSS

`frontend/assets/css/style.css`:

```css
:root {
    --background: #101827;
    --surface: #1d293b;
    --text: #f1f5f9;
    --accent: #60a5fa;
}
* { box-sizing: border-box; }
body {
    margin: 0;
    font-family: system-ui, sans-serif;
    background: var(--background);
    color: var(--text);
    line-height: 1.6;
}
.site-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 24px;
    flex-wrap: wrap;
    padding: 20px 6%;
    background: var(--surface);
}
.site-header nav { display: flex; flex-wrap: wrap; gap: 16px; }
a { color: var(--accent); }
.hero { padding: 90px 6%; }
section:not(.hero) { padding: 40px 6%; }
.project-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 24px;
}
.project-card {
    padding: 24px;
    border-radius: 12px;
    background: var(--surface);
}
.button {
    display: inline-block;
    padding: 12px 20px;
    border-radius: 8px;
    background: var(--accent);
    color: #101827;
    text-decoration: none;
    transition: transform 200ms, background 200ms;
}
.button:hover { transform: translateY(-3px); background: #93c5fd; }
```

Изучи CSS-переменные, Flexbox, Grid, `box-sizing`, отступы, `transition`. [MDN: CSS](https://developer.mozilla.org/ru/docs/Learn_web_development/Core/Styling_basics).

### Шаг 6. Открой сайт в браузере

В IDEA открой `index.html` и выбери **View → Open in Browser** (или `Alt+F2`). Если используешь предпросмотр, открывай сайт по `http://...`, а не через `file://`: позже `fetch()` будет работать с HTTP API. [JetBrains: открытие в браузере](https://www.jetbrains.com/help/idea/editing-html-files.html).

**Контрольная точка №1:** главная страница открывается; видны карточка проекта и навигация; изменение CSS сразу заметно. Пока ни Java, ни Maven, ни БД.

## Этап 2. Преврати скелет в реальное резюме

Выполняй отдельно, сохраняя Git-коммит после законченного блока.

| Шаг | Задача | Что изучить |
|---:|---|---|
| 7 | Заполни блок «Обо мне»: специализация, цели, образование, опыт | Семантический HTML |
| 8 | Создай карточки навыков: Java, Spring, Docker, SQL, JS | CSS Grid и повторное использование классов |
| 9 | Добавь карточки всех будущих демо | `article`, ссылки |
| 10 | Добавь рабочие ссылки GitHub и email | `<a>`, `mailto:` |
| 11 | Сделай адаптивное меню | Flexbox, media queries |
| 12 | Добавь переключатель светлой и тёмной тем | JS, CSS-переменные, `localStorage` |
| 13 | Сделай анимацию наведения кнопок и изменения цвета | `:hover`, `transform`, `transition` |
| 14 | Добавь анимацию появления блоков при прокрутке | `@keyframes`, `IntersectionObserver` |
| 15 | Проверь доступность с клавиатуры и ограничение анимации | `:focus-visible`, `prefers-reduced-motion` |

Для шага 12 добавь кнопку с `id="theme-button"`, CSS-правило `.light-theme` и обработчик в `main.js`:

```javascript
const themeButton = document.querySelector('#theme-button');
themeButton?.addEventListener('click', () => {
    document.body.classList.toggle('light-theme');
    localStorage.setItem('theme',
        document.body.classList.contains('light-theme') ? 'light' : 'dark');
});
// При старте прочитай localStorage.getItem('theme')
// и восстанови нужный класс на body.
```

Анимации должны уважать системную настройку пользователя:

```css
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        transition-duration: 0.01ms !important;
    }
}
```

**Контрольная точка №2:** сайт уже выглядит как резюме; карточки ведут на будущие демо; мобильная версия и переключатель темы работают. [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage), [MDN: media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries).

## Этап 3. Модальное окно, dropdown и CSS-анимации

### Шаг 16. Создай модальное окно

В `ui.html`:

```html
<button id="open-modal">Открыть окно</button>
<dialog id="demo-modal">
    <h2>Пример модального окна</h2>
    <p>Создано на HTML и JavaScript.</p>
    <form method="dialog"><button>Закрыть</button></form>
</dialog>
<script src="assets/js/ui.js" defer></script>
```

В `ui.js`:

```javascript
const modal = document.querySelector('#demo-modal');
document.querySelector('#open-modal')?.addEventListener('click', () => {
    modal.showModal();
});
```

`<dialog>` уже обеспечивает модальную семантику и стандартное закрытие по Escape. [MDN: dialog](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog).

| Шаг | Задача |
|---:|---|
| 17 | Стилизуй окно и `dialog::backdrop` |
| 18 | Добавь плавную анимацию открытия |
| 19 | Проверь закрытие по Escape и возврат фокуса |
| 20 | Добавь второе модальное окно — форму обратной связи без сервера |
| 21 | Создай обычный `<select>` и изучи его поведение |
| 22 | Замени его кнопкой «Выбрать категорию» и списком вариантов |
| 23 | Реализуй скрытие и раскрытие списка |
| 24 | Добавь переключение по нажатию |
| 25 | При выборе варианта изменяй текст кнопки и выбранное значение |
| 26 | Закрывай список по клику вне компонента |
| 27 | Добавь стрелки, Enter, Escape, ARIA-атрибуты и видимый фокус |

Кастомный dropdown сложнее стандартного `<select>`. Важно сначала добиться доступного управления с клавиатуры, потом заниматься анимацией. [WAI-ARIA Listbox](https://www.w3.org/WAI/ARIA/apg/patterns/listbox/).

**Контрольная точка №3:** модальное окно и селектор работают мышью и клавиатурой.

## Этап 4. Галерея фотографий

### Шаг 28. Добавь изображения

Помести фотографии в `frontend/assets/images/`, затем в `gallery.html`:

```html
<div class="gallery-track" id="gallery-track">
    <img src="assets/images/photo1.jpg" alt="Фото 1">
    <img src="assets/images/photo2.jpg" alt="Фото 2">
    <img src="assets/images/photo3.jpg" alt="Фото 3">
</div>
```

### Шаг 29. Сделай горизонтальную прокрутку с CSS Scroll Snap

```css
.gallery-track {
    display: flex;
    gap: 16px;
    overflow-x: auto;
    scroll-snap-type: x mandatory;
}
.gallery-track img {
    flex: 0 0 100%;
    width: 100%;
    max-height: 500px;
    object-fit: cover;
    scroll-snap-align: start;
}
```

| Шаг | Задача |
|---:|---|
| 30 | Добавь кнопки «Назад» и «Вперёд» |
| 31 | В `gallery.js` добавь прокрутку через `scrollBy()` |
| 32 | Покажи индикатор текущего слайда |
| 33 | Проверь мобильную вёрстку и свайпы |
| 34 | Добавь просмотр фотографии в `<dialog>` |
| 35 | Для изображений ниже первого экрана используй `loading="lazy"` |

**Контрольная точка №4:** галерея прокручивается свайпом и кнопками; фото открываются крупнее. [MDN: CSS Scroll Snap](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll_snap).

## Этап 5. Каталог: тысячи элементов, поиск, счётчики, два вида пагинации

### Шаг 36. Подготовь HTML

`catalog.html` (включи структуру `html`, `head` и общие стили, как на главной):

```html
<h1>Демонстрационный каталог</h1>
<label for="search">Поиск товаров</label>
<input id="search" type="search" placeholder="Например, монитор" autocomplete="off">
<ul id="suggestions" aria-label="Подсказки поиска"></ul>
<p id="counter"></p>
<div id="categories"></div>
<div id="products" class="project-grid"></div>
<nav aria-label="Пагинация">
    <button id="prev-page">Назад</button>
    <span id="page-info"></span>
    <button id="next-page">Вперёд</button>
</nav>
<script src="assets/js/catalog.js" defer></script>
```

### Шаг 37. Сгенерируй 12 000 товаров на клиенте

`catalog.js`:

```javascript
const categories = ['Монитор', 'Клавиатура', 'Мышь', 'Ноутбук'];
const products = Array.from({ length: 12000 }, (_, index) => {
    const category = categories[index % categories.length];
    return {
        id: index + 1,
        name: `${category} модель ${index + 1}`,
        category
    };
});
const state = { page: 1, pageSize: 24, query: '', category: null };
```

Сначала всё в браузере: так легко понять поведение каталога независимо от сервера.

### Шаг 38. Реализуй пагинацию

Продолжение `catalog.js`:

```javascript
const productsContainer = document.querySelector('#products');
const counter = document.querySelector('#counter');
const pageInfo = document.querySelector('#page-info');

function getFilteredProducts() {
    return products.filter(product =>
        product.name.toLowerCase().includes(state.query) &&
        (!state.category || product.category === state.category)
    );
}

function renderCatalog() {
    const filtered = getFilteredProducts();
    const totalPages = Math.max(1, Math.ceil(filtered.length / state.pageSize));
    state.page = Math.min(state.page, totalPages);
    const start = (state.page - 1) * state.pageSize;
    const visible = filtered.slice(start, start + state.pageSize);

    productsContainer.replaceChildren();
    for (const product of visible) {
        const card = document.createElement('article');
        card.className = 'project-card';
        const title = document.createElement('h3');
        title.textContent = product.name;
        card.append(title);
        productsContainer.append(card);
    }
    counter.textContent = `Найдено: ${filtered.length}`;
    pageInfo.textContent = `${state.page} / ${totalPages}`;
    document.querySelector('#prev-page').disabled = state.page === 1;
    document.querySelector('#next-page').disabled = state.page === totalPages;
}

document.querySelector('#prev-page').addEventListener('click', () => {
    state.page--;
    renderCatalog();
});
document.querySelector('#next-page').addEventListener('click', () => {
    state.page++;
    renderCatalog();
});
renderCatalog();
```

`filter()` выбирает подходящие объекты, `slice()` — только одну страницу, `replaceChildren()` очищает старую разметку. Мы **не создаём 12 000 DOM-карточек**, а выводим 24.

### Шаг 39. Добавь текстовый поиск

```javascript
const search = document.querySelector('#search');
search.addEventListener('input', () => {
    state.query = search.value.trim().toLowerCase();
    state.page = 1;
    renderCatalog();
});
```

### Шаг 40. Реализуй подсказки

Напиши `renderSuggestions()` по алгоритму:

1. Получи текст запроса.
2. При пустом запросе очисти и спрячь подсказки.
3. Отфильтруй товары или уникальные названия.
4. Покажи не более 5 вариантов.
5. Для каждого варианта создай кнопку, а не неинтерактивный текст.
6. При выборе заполни поле и обнови каталог.
7. Добавь debounce в 200–300 мс с `setTimeout` и `clearTimeout`.

### Шаг 41. Подсчитай элементы по категориям

```javascript
const counts = products.reduce((result, product) => {
    result[product.category] = (result[product.category] ?? 0) + 1;
    return result;
}, {});
console.log(counts);
```

В этом наборе по 3000 товаров каждой категории. Отобрази значения в `#categories`.

### Шаг 42. Добавь фильтр по категориям

Сделай кнопки категорий. По нажатию обновляй `state.category`, сбрасывай `state.page` и запускай `renderCatalog()`. Убедись, что поиск и категория действуют совместно.

### Шаг 43. Подготовь режим бесконечной ленты

Добавь в разметку `<div id="scroll-sentinel"></div>` и переключатель режимов «Страницы / Лента».

### Шаг 44. Изучи `IntersectionObserver`

```javascript
const sentinel = document.querySelector('#scroll-sentinel');
const observer = new IntersectionObserver(entries => {
    if (!entries[0].isIntersecting) return;
    // Только в режиме ленты:
    // если не идёт загрузка и есть следующая порция,
    // увеличь страницу и ДОБАВЬ новые карточки в конец списка.
}, { rootMargin: '200px' });
observer.observe(sentinel);
```

Это каркас, а не вся реализация: надо учитывать режим, состояние загрузки и конец списка. Для ленты **не** очищай уже выведенные карточки при каждой новой порции. [MDN: IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API).

| Шаг | Задача |
|---:|---|
| 45 | Добавь индикатор загрузки следующей порции |
| 46 | Покажи «Товары закончились» в конце ленты |
| 47 | При новом поиске сбрасывай страницу или накопленную ленту |
| 48 | Сохраняй запрос, категорию и номер страницы в URL-параметрах |
| 49 | Сравни работу на 12 000 и 50 000 объектов; проверь вкладку Performance |

**Контрольная точка №5:** есть страничная пагинация и бесконечный скролл, подсказки, фильтры, счётчики и большие тестовые наборы **без backend**.

## Этап 6. Kanban-доска с drag and drop

На `board.html` сделай три колонки: **To Do**, **In Progress**, **Done**. У задачи должны быть `id`, `title`, `status`, `position`.

| Шаг | Задача |
|---:|---|
| 50 | Нарисуй три колонки |
| 51 | Создай массив объектов задач |
| 52 | Отрисуй карточки через JS |
| 53 | Добавь карточкам `draggable="true"` |
| 54 | В `dragstart` сохрани ID через `dataTransfer.setData()` |
| 55 | В `dragover` целевой колонки вызови `event.preventDefault()` |
| 56 | В `drop` прочитай ID и измени `status` |
| 57 | Перерисуй карточки из массива |
| 58 | Реализуй перестановку внутри колонки, обновляя `position` |
| 59 | Сохраняй массив в `localStorage` |
| 60 | Восстанавливай задачи после F5 |
| 61 | Добавь создание и удаление задачи через модальное окно |
| 62 | Добавь перемещение с клавиатуры и альтернативу для сенсорных экранов |

Не путай **перенос между колонками** с **изменением порядка внутри колонки** — это два разных сценария. [MDN: Drag and Drop API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API).

**Контрольная точка №6:** можно создавать, удалять и перемещать задачи. После обновления страницы состояние остаётся в этом браузере.

## Этап 7. Drag and drop файлов (пока без сервера)

| Шаг | Задача |
|---:|---|
| 63 | Создай зону с текстом «Перетащи файл сюда» |
| 64 | Добавь обычный `<input type="file">` |
| 65 | Обработай `dragover` и `drop` |
| 66 | Прочитай `event.dataTransfer.files` |
| 67 | Покажи имя, размер и тип каждого файла |
| 68 | Для изображений создай превью через `URL.createObjectURL()`; освобождай URL после использования |
| 69 | Покажи сообщение об ошибке для слишком больших файлов, например больше 10 МБ |
| 70 | Сделай кнопку «Загрузить», но без отправки — сервер ещё не нужен |

Пример в `upload.js`:

```javascript
const dropZone = document.querySelector('#drop-zone');
dropZone.addEventListener('dragover', event => event.preventDefault());
dropZone.addEventListener('drop', event => {
    event.preventDefault();
    const files = Array.from(event.dataTransfer.files);
    files.forEach(file => console.log(file.name, file.size, file.type));
});
```

HTML-элемент `#drop-zone` сначала нужно создать в `upload.html`. [MDN: File API](https://developer.mozilla.org/en-US/docs/Web/API/File_API).

**Контрольная точка №7:** перетаскиваемые и выбранные файлы видны с их свойствами и превью; ничего не отправляется.

## Этап 8. Заготовка Unity-страницы

| Шаг | Задача |
|---:|---|
| 71 | Создай `unity.html` с описанием Unity-проекта |
| 72 | Добавь контейнер для будущего Web-приложения |
| 73 | Размести описание управления, размера и технологий |
| 74 | Добавь заглушку «Unity Web Demo появится здесь» |

**Итог части I:** зафиксируй работающий фронтенд:

```bash
git add .
git commit -m "Complete interactive frontend prototype"
```

Не устанавливай Spring, PostgreSQL или Docker, пока не закончены необходимые для тебя фронтенд-демо.

---
# Часть II. Первый backend: Java, Maven, Spring Boot (шаги 75–91)

## Этап 9. Устанавливаем Java только после готового фронтенда

### Шаг 75. Установи JDK 21 на Arch Linux

```bash
sudo pacman -Syu
sudo pacman -S --needed jdk21-openjdk
archlinux-java status
java -version
javac -version
```

При необходимости выбери JDK 21 в **File → Project Structure → Project SDK**. Системную версию Java менять необязательно: в IDEA можно выбрать JDK для конкретного проекта. [ArchWiki: Java](https://wiki.archlinux.org/title/Java).

### Шаг 76. Создай `web-service` через Spring Initializr

В IDEA открой **File → New → Module → Spring Boot** (расположение зависит от версии). Альтернатива: сгенерируй проект на [start.spring.io](https://start.spring.io/) и распакуй в `portfolio-lab/web-service`.

| Параметр | Значение |
|---|---|
| Name / Artifact | `web-service` |
| Group | `dev.portfolio` |
| Language / Java | Java / 21 |
| Build | Maven |
| Packaging | Jar |
| Spring Boot | Стабильная версия линии **3.5.x**, одинаковая в обоих сервисах |
| Dependencies | Spring Web, Spring Boot Starter Test |

Не используй Snapshot/Milestone. Код этого руководства ориентирован на Spring Boot 3.5.x и Spring Framework 6.2.x; если сознательно выбираешь Spring Boot 4.x, проверяй изменения модулей и starter-зависимостей по официальной документации.

### Шаг 77. Изучи файлы Maven

```text
web-service/
├── pom.xml                      # зависимости и сборка
├── mvnw                         # Maven Wrapper для Linux
├── mvnw.cmd                     # Wrapper для Windows
├── .mvn/wrapper/                # конфигурация Wrapper
└── src/
    ├── main/java/              # Java-классы
    ├── main/resources/         # настройки и статические файлы
    └── test/                   # тесты
```

`pom.xml` указывает зависимости и параметры сборки. Wrapper фиксирует версию Maven для команды и IDE.

### Шаг 78. Проверь Wrapper отдельно от импорта IDEA

```bash
cd web-service
chmod +x mvnw
./mvnw -v
./mvnw validate
```

В IDEA открой **Settings → Build, Execution, Deployment → Build Tools → Maven → Maven home path** и выбери Maven Wrapper. Если при переключении появляется `No implementation for ArtifactHandlerManager was bound`, сначала проверь `./mvnw validate` в терминале. При исправной командной сборке и ошибке только в IDEA обнови IDE либо временно используй Bundled Maven 3. Не добавляй Guice в `pom.xml` ради такой ошибки. [Maven Wrapper](https://maven.apache.org/tools/wrapper/), [Maven in IntelliJ](https://www.jetbrains.com/help/idea/maven-support.html).

### Шаги 79–82. Запусти пустой Spring Boot

79. Найди сгенерированный класс `WebServiceApplication`.
80. Запусти его зелёной кнопкой у метода `main()`.
81. В окне **Run** убедись, что сервер стартовал без исключений.
82. Открой `http://localhost:8080`. До добавления страницы/контроллера ответ 404 нормален.

### Шаг 83. Первый HTTP-контроллер

В `src/main/java/dev/portfolio/web/HelloController.java`:

```java
package dev.portfolio.web;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/api/hello")
    public String hello() {
        return "Hello from Spring Boot!";
    }
}
```

Перезапусти приложение и открой `http://localhost:8080/api/hello`.

**Разберись:** `@RestController` помечает обработчик HTTP-запросов, `@GetMapping` связывает URL и Java-метод. [Spring MVC controllers](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html).

### Шаг 84. Возвращай JSON

Создай `Profile.java`:

```java
package dev.portfolio.web;

public record Profile(String name, String role) {}
```

Добавь метод в `HelloController`:

```java
@GetMapping("/api/profile")
public Profile profile() {
    return new Profile("Твоё имя", "Java Developer");
}
```

Результат `http://localhost:8080/api/profile`:

```json
{
  "name": "Твоё имя",
  "role": "Java Developer"
}
```

Spring Boot автоматически преобразует объект в JSON через настроенный механизм сериализации.

### Шаг 85. Перенеси HTML, CSS и JS в Spring Boot

Скопируй **содержимое** `frontend/`, а не вложенную папку, в:

```text
web-service/src/main/resources/static/
├── index.html
├── catalog.html
├── board.html
├── upload.html
├── gallery.html
├── ui.html
├── unity.html
└── assets/
```

Далее редактируй одну рабочую копию в `static/`. Предыдущую версию можно восстановить из Git. Spring Boot умеет раздавать эту директорию без дополнительных контроллеров. [Spring Boot static content](https://docs.spring.io/spring-boot/3.5/reference/web/servlet.html).

### Шаг 86. Настрой порт web-service

`web-service/src/main/resources/application.yaml`:

```yaml
server:
  address: ${SERVER_ADDRESS:127.0.0.1}
  port: 8080
```

`SERVER_ADDRESS` — переменная окружения; если не задана, используется `127.0.0.1`. Локально это безопасный адрес, а в Docker позднее установим `0.0.0.0`.

### Шаг 87. Первый `fetch()` из браузера в Java

В `assets/js/main.js`:

```javascript
async function loadProfile() {
    const response = await fetch('/api/profile');
    if (!response.ok) throw new Error('Не удалось получить профиль');
    const profile = await response.json();
    console.log(profile);
}
loadProfile().catch(console.error);
```

Запусти `web-service`, открой `http://localhost:8080`, найди результат в **Browser DevTools → Console**. Запрос больше не идёт к файлу: он вызывает Java-контроллер.

### Шаги 88–91. Заверши первый backend

88. Выведи имя и роль на главной странице через `textContent`.
89. Добавь состояния «Загрузка...» и «Ошибка загрузки профиля».
90. Напиши тест `/api/profile` с Spring Boot Test + MockMvc.
91. Выполни `./mvnw test` и `./mvnw package`; найди JAR в `target/`.

**Контрольная точка №8:** сайт открывается с `localhost:8080`; часть содержимого приходит из Java-контроллера. PostgreSQL всё ещё нет.

---

# Часть III. Два сервиса, но ещё без PostgreSQL (шаги 92–108)

## Этап 10. Независимый `repository-service`

### Шаг 92. Создай второе приложение

Через Spring Initializr в корне `portfolio-lab` создай `repository-service` со следующими параметрами:

- Group `dev.portfolio`, Java 21, Maven, Jar.
- Та же версия Spring Boot 3.5.x, что у `web-service`.
- Зависимость **Spring Web**.
- **Пока без PostgreSQL Driver, JDBC, JPA, Flyway**.

Это самостоятельное приложение с отдельным `pom.xml`, Wrapper и классом `RepositoryServiceApplication`.

### Шаг 93. Настрой внутренний HTTP-порт

`repository-service/src/main/resources/application.yaml`:

```yaml
server:
  address: ${SERVER_ADDRESS:127.0.0.1}
  port: 8082
```

### Шаг 94. Начни с данных в памяти Java

Создай `ProductController.java` в `dev.portfolio.repository`:

```java
package dev.portfolio.repository;

import java.util.List;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/internal/products")
public class ProductController {
    public record Product(long id, String name, String category) {}

    @GetMapping
    public List<Product> getProducts() {
        return List.of(
            new Product(1, "Монитор 27", "Мониторы"),
            new Product(2, "Клавиатура RGB", "Клавиатуры"),
            new Product(3, "Игровая мышь", "Мыши")
        );
    }
}
```

Сейчас сервис отвечает за данные, но не умеет хранить изменения. Это нормально на учебном этапе.

### Шаг 95. Проверь API напрямую

Запусти `RepositoryServiceApplication`. Открой:

```text
http://127.0.0.1:8082/internal/products
```

Должен прийти JSON с тремя товарами. Оба Spring Boot-приложения работают как отдельные процессы.

### Шаг 96. Подключи web-service к repository-service

В **существующий** `web-service/src/main/resources/application.yaml` добавь:

```yaml
repository:
  base-url: ${REPOSITORY_URL:http://127.0.0.1:8082}
```

Создай `ProductsApiController.java`:

```java
package dev.portfolio.web;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestClient;

@RestController
public class ProductsApiController {
    private final RestClient repositoryClient;

    public ProductsApiController(RestClient.Builder builder,
                                 @Value("${repository.base-url}") String baseUrl) {
        this.repositoryClient = builder.baseUrl(baseUrl).build();
    }

    public record Product(long id, String name, String category) {}

    @GetMapping("/api/products")
    public Product[] getProducts() {
        return repositoryClient.get()
            .uri("/internal/products")
            .retrieve()
            .body(Product[].class);
    }
}
```

`RestClient` — HTTP-клиент для связи между приложениями. На более позднем этапе вынеси вызовы во внутренний `RepositoryClient`, чтобы контроллер не содержал сетевую логику. [Spring: RestClient](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html#rest-restclient).

### Шаг 97. Проверь цепочку двух сервисов

Запусти **оба** приложения в IDEA и открой:

```text
http://127.0.0.1:8080/api/products
```

Тот же JSON теперь проходит по цепочке: браузер → `web-service` → `repository-service` → ответ.

### Шаг 98. Проверь отказ второго сервиса

Останови только `repository-service`, затем снова вызови `/api/products`. Запрос завершится ошибкой связи: так ты увидишь реальное последствие распределённой архитектуры. Запусти сервис обратно.

### Шаг 99. Обработай ошибки связи

Создай обработчик исключений через `@RestControllerAdvice`, чтобы `web-service` отдавал понятный HTTP `503 Service Unavailable` и JSON с сообщением «Каталог временно недоступен». Позже добавь таймауты HTTP-вызовов и логирование.

### Шаги 100–103. Перенеси каталог с JS-массива в Java

100. В `catalog.js` вместо `products = Array.from(...)` начни получать данные через `fetch('/api/products')`.
101. В `repository-service` создай 12 000 Java-объектов при старте приложения.
102. Реализуй на сервере фильтрацию и порционную выдачу с параметрами `page`, `size`, `q`.
103. В `web-service` передавай те же параметры в `repository-service`.

Пример внешнего запроса:

```http
GET /api/products?page=0&size=24&q=монитор
```

Нумерация в API начинается **с нуля**, а пользователь видит страницу **1**. Ответ удобно оформить как DTO:

```json
{
  "items": [
    { "id": 1, "name": "Монитор 27", "category": "Мониторы" }
  ],
  "page": 0,
  "size": 24,
  "total": 3000
}
```

Сделай отдельный Java-record `ProductPage` для этого формата и в обоих сервисах согласуй контракт. **Не передавай все 12 000 товаров браузеру**, когда он просит только одну страницу.

### Шаги 104–108. Остальные функции каталога

104. Реализуй `/internal/suggestions?q=мон` — максимум 5 подсказок.
105. Добавь соответствующий внешний `/api/suggestions`.
106. Реализуй `/internal/categories/counts`.
107. Подключи серверные счётчики категорий к HTML.
108. Переключи бесконечную ленту на тот же серверный endpoint пагинации, увеличивая `page`.

**Контрольная точка №9:** каталог и счётчики работают через **два независимых сервиса**, но в проекте всё ещё нет базы данных. Понятно, зачем нужен HTTP, какие данные передавать в JSON и что произойдёт при падении одного сервиса.

---

# Часть IV. PostgreSQL, миграции, SQL и сохранение данных (шаги 109–155)

## Почему PostgreSQL появляется только теперь

Хранение списка в памяти было достаточным для чтения. Теперь нужны реальные изменения: создание товаров и задач, общий список между устройствами, сохранение после перезапуска, SQL-поиск и агрегаты. Для этих требований памяти одного Java-процесса недостаточно.

## Этап 11–12. Запусти PostgreSQL отдельно через Docker

### Шаг 109. Установи Docker на Arch Linux

```bash
sudo pacman -S --needed docker docker-compose
sudo systemctl enable --now docker
sudo docker version
sudo docker compose version
```

Для обучения можно вызывать Docker через `sudo`. Добавление пользователя в группу `docker` необязательно и даёт права, практически равные root. [ArchWiki: Docker](https://wiki.archlinux.org/title/Docker).

### Шаг 110. Создай `compose.yaml` в корне проекта

```yaml
services:
  db:
    image: postgres:17-alpine
    environment:
      POSTGRES_DB: portfolio
      POSTGRES_USER: portfolio
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-portfolio_dev}
    ports:
      - "127.0.0.1:5433:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U portfolio -d portfolio"]
      interval: 5s
      timeout: 3s
      retries: 10

volumes:
  pgdata:
```

`5432` — порт внутри контейнера; `5433` — порт для Java и IDEA на твоём компьютере. `pgdata` — именованный Docker volume: данные сохраняются при пересоздании контейнера. `portfolio_dev` допустим только для локального обучения.

### Шаг 111. Запусти только БД

Из корня проекта:

```bash
sudo docker compose up -d db
sudo docker compose ps
sudo docker compose exec db psql -U portfolio -d portfolio
```

В SQL-консоли:

```sql
select version();
```

Выход — `\q`.

### Шаг 112. Подключи базу в IntelliJ IDEA Ultimate

**View → Tool Windows → Database → + → Data Source → PostgreSQL**. Укажи:

| Поле | Значение |
|---|---|
| Host | `127.0.0.1` |
| Port | `5433` |
| Database | `portfolio` |
| User | `portfolio` |
| Password | `portfolio_dev` |

Нажми **Test Connection**, скачай драйвер при запросе IDE. 

**Контрольная точка №10:** БД доступна по `127.0.0.1:5433`, IntelliJ может выполнять запросы. [Docker Compose](https://docs.docker.com/compose/gettingstarted/), [JetBrains: Database Tools](https://www.jetbrains.com/help/idea/database-tool-window.html).

## Этап 13. JDBC + Flyway в repository-service

### Шаг 113. Добавь зависимости в `repository-service/pom.xml`

В **существующий** блок `<dependencies>`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

**JDBC** позволяет посылать SQL-запросы из Java. **Flyway** следит за версионированием схемы. Сначала используем явный SQL, чтобы разобраться, что делает база; Hibernate/JPA добавляй уже после изучения JDBC.

### Шаг 114. Настрой подключение в `repository-service`

`repository-service/src/main/resources/application.yaml`:

```yaml
server:
  address: ${SERVER_ADDRESS:127.0.0.1}
  port: 8082

spring:
  datasource:
    url: ${DB_URL:jdbc:postgresql://127.0.0.1:5433/portfolio}
    username: ${DB_USER:portfolio}
    password: ${DB_PASSWORD:portfolio_dev}
```

Это тот самый `application.yaml`, про который ты спрашивал ранее: Spring Boot автоматически читает его при запуске. Выражение `${DB_USER:portfolio}` означает «взять переменную `DB_USER`, иначе использовать `portfolio`».

### Шаг 115. Создай SQL-файлы миграций

```text
repository-service/src/main/resources/
├── application.yaml
└── db/
    └── migration/
        ├── V1__schema.sql
        └── V2__demo_data.sql
```

`V1` и `V2` — **части имён файлов**, это два отдельных `.sql`-файла. После `V1` и `V2` используются **два подчёркивания**. Flyway выполнит их по возрастанию версии.

### Шаг 116. Создай структуру таблиц

`V1__schema.sql`:

```sql
create table category (
    id bigint generated always as identity primary key,
    name varchar(100) not null unique
);

create table product (
    id bigint generated always as identity primary key,
    category_id bigint not null references category(id),
    name varchar(200) not null
);

create index product_category_idx on product (category_id);
```

Сначала **без** `pg_trgm`: обычные таблицы, ключи и индекс по категории. Так проще проверить, что сам JDBC и Flyway работают.

### Шаг 117. Заполни базу тестовыми данными

`V2__demo_data.sql`:

```sql
insert into category (name) values
    ('Мониторы'),
    ('Клавиатуры'),
    ('Мыши'),
    ('Ноутбуки');

insert into product (category_id, name)
select c.id, c.name || ' модель ' || g.n
from generate_series(1, 12000) as g(n)
join category c on c.name = case (g.n % 4)
    when 0 then 'Мониторы'
    when 1 then 'Клавиатуры'
    when 2 then 'Мыши'
    else 'Ноутбуки'
end;
```

`generate_series` создаёт числовые значения прямо в PostgreSQL, что удобно для тестового каталога.

### Шаг 118. Перезапусти repository-service

Flyway должен автоматически выполнить `V1` и `V2`. В IDEA Database Console:

```sql
select count(*) from product;
```

Ожидаемый результат — `12000`.

**Внимание:** этот пример рассчитан на **новую пустую учебную БД**. Если `V1` и `V2` уже применялись, не переписывай их: Flyway хранит контрольные суммы миграций. Новые изменения вноси через `V3`, `V4` и далее. Не удаляй том с ценными данными ради повторного запуска примеров.

### Шаги 119–121. Научись читать SQL

119. Выполни `SELECT` с `WHERE`, `ORDER BY`, `LIMIT` и `OFFSET`.
120. Выполни `JOIN` между `product` и `category`.
121. Выполни `GROUP BY`, чтобы подсчитать товары по категориям:

```sql
select c.name, count(p.id) as total
from category c
left join product p on p.category_id = c.id
group by c.id, c.name
order by c.name;
```

**Контрольная точка №11:** 12 000 строк появились посредством Flyway; SQL-запросы работают в IDE. [Flyway](https://documentation.red-gate.com/fd), [PostgreSQL Tutorial](https://www.postgresql.org/docs/current/tutorial.html).

## Этап 14. Заменяем Java-списки запросами к PostgreSQL

### Шаг 122. Создай класс репозитория

`repository-service/src/main/java/dev/portfolio/repository/repository/ProductRepository.java`:

```java
package dev.portfolio.repository.repository;

import org.springframework.jdbc.core.simple.JdbcClient;
import org.springframework.stereotype.Repository;

@Repository
public class ProductRepository {
    private final JdbcClient jdbc;

    public ProductRepository(JdbcClient jdbc) {
        this.jdbc = jdbc;
    }

    public long countAll() {
        return jdbc.sql("select count(*) from product")
                .query(Long.class)
                .single();
    }
}
```

Аннотация `@Repository` — обозначение Java-компонента, работающего с данными; **не путай её** с именем отдельного приложения `repository-service`. [Spring JdbcClient](https://docs.spring.io/spring-framework/reference/data-access/jdbc/core.html#jdbc-JdbcClient).

### Шаг 123. Напиши SQL для одной страницы

```sql
select p.id, p.name, c.name as category
from product p
join category c on c.id = p.category_id
where p.name ilike '%' || :query || '%'
order by p.id
limit :size offset :offset;
```

`:query`, `:size`, `:offset` здесь — **именованные параметры JdbcClient**, которые Java передаёт безопасно. В SQL-консоли IDEA их потребуется подставить значениями. Не вставляй пользовательский текст непосредственно в SQL-строку конкатенацией Java.

| Шаг | Задача |
|---:|---|
| 124 | Добавь метод `findPage(query, page, size)` |
| 125 | Ограничь страницу, например максимум 100 объектов; валидируй `page >= 0` |
| 126 | Создай `ProductService`, использующий `ProductRepository` |
| 127 | В `ProductController` замени список в памяти вызовом сервиса |
| 128 | Проверь `/internal/products` напрямую |
| 129 | Проверь `/api/products` через web-service |

Важно возвращать **и страницу, и общее число совпадений**. `COUNT(*)` для общего числа и SQL с `LIMIT/OFFSET` для текущих элементов — два разных запроса.

### Шаг 130. Перенеси счётчики в SQL

Создай метод SQL-агрегации по категориям на основе запроса из шага 121. Явно различай *количество всех товаров* и *количество подходящих под текущий поиск*.

### Шаг 131. Перенеси подсказки в SQL

```sql
select name
from product
where name ilike '%' || :query || '%'
order by name
limit 5;
```

В браузере используй debounce и `AbortController`: если пользователь уже ввёл новый запрос, старый ответ не должен перезаписать новые подсказки.

### Шаг 132. Исследуй план запроса

В IntelliJ Database Console:

```sql
explain (analyze, buffers)
select * from product where name ilike '%монитор%';
```

Запомни время, тип сканирования и количество прочитанных страниц.

### Шаг 133. Подключи `pg_trgm` только теперь

Создай **новую** миграцию `V3__product_search_index.sql`:

```sql
create extension if not exists pg_trgm;

create index product_name_trgm_idx
    on product using gin (name gin_trgm_ops);
```

`pg_trgm` — расширение PostgreSQL для операций по тройкам символов, полезное для поиска по части строки. `gin_trgm_ops` — индексный класс операторов, **не имя столбца**. Правильно: `(name gin_trgm_ops)`, без запятой; неверно: `(name, gin_trgm_ops)` и `gin_tgrm_ops`.

Если IDEA подчёркивает `gin_trgm_ops`, но миграция успешно выполняется, проверь диалект PostgreSQL и синхронизацию Data Source. Само подчёркивание не доказывает ошибку в SQL. Сравни `EXPLAIN ANALYZE` до и после. На маленьких таблицах PostgreSQL не обязан использовать индекс. [PostgreSQL: pg_trgm](https://www.postgresql.org/docs/current/pgtrgm.html).

**Контрольная точка №12:** каталог работает через PostgreSQL, результаты подсчитываются и сортируются в SQL, поиск можно исследовать через план выполнения.

## Этап 15. Сохраняем Kanban в PostgreSQL

| Шаг | Задача |
|---:|---|
| 134 | Создай миграцию `V4__board.sql` |
| 135 | Создай таблицу `task(id, title, status, position)` |
| 136 | Добавь несколько демонстрационных задач |
| 137 | Создай `TaskRepository` с методом получения задач |
| 138 | Создай `TaskService` |
| 139 | Добавь `GET /internal/tasks` в repository-service |
| 140 | Добавь `GET /api/tasks` в web-service |
| 141 | Замени чтение `localStorage` на `fetch('/api/tasks')` |
| 142 | Создай `POST` для добавления задачи |
| 143 | Создай `DELETE` для удаления задачи |
| 144 | Добавь `PUT`/`PATCH` для изменения статуса и позиции |
| 145 | Сохраняй перестановку карточек в транзакции `@Transactional` |

Пример `V4__board.sql` для начального варианта:

```sql
create table task (
    id bigint generated always as identity primary key,
    title varchar(250) not null,
    status varchar(30) not null,
    position integer not null,
    constraint task_status_check
        check (status in ('TODO', 'IN_PROGRESS', 'DONE'))
);

create index task_status_position_idx on task(status, position);
```

Один перенос может менять позиции нескольких карточек — сохраняй эти изменения атомарно в одной транзакции. Проверяй корректность входящих данных и защищай mutating API перед публичной публикацией (авторизация, CSRF/токены в зависимости от схемы аутентификации).

## Этап 16. Загрузка файлов на сервер

| Шаг | Задача |
|---:|---|
| 146 | Создай `FormData` в `upload.js` |
| 147 | Добавь выбранные файлы в запрос |
| 148 | Отправь `POST /api/files` с помощью `fetch()` |
| 149 | В web-service принимай файлы через `MultipartFile` |
| 150 | Настрой ограничение загрузки, например 10 МБ |
| 151 | Передай файл из web-service в repository-service |
| 152 | Сохраняй файл вне `src/main/resources`, в выделенной папке |
| 153 | Создай таблицу метаданных: ID, имя, размер, тип, дата, путь |
| 154 | Добавь endpoint списка загруженных файлов |
| 155 | Реализуй удаление файла и метаданных |

Пример браузерного запроса:

```javascript
const form = new FormData();
form.append('file', selectedFile);
const response = await fetch('/api/files', {
    method: 'POST',
    body: form
});
if (!response.ok) throw new Error('Ошибка загрузки');
```

Не устанавливай вручную `Content-Type` для `FormData`: браузер сам сформирует `multipart/form-data` с boundary.

Для Spring Boot в `web-service/src/main/resources/application.yaml` можно добавить:

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB
```

Файлы храни под сгенерированным безопасным именем, например UUID, не используй напрямую путь или имя, присланные пользователем. Проверяй ограничения **на сервере**, а не только в JS. Метаданные храни в PostgreSQL, сами файлы — в выделенном хранилище, которое позднее получит Docker volume. [Spring: Uploading Files](https://spring.io/guides/gs/uploading-files).

**Контрольная точка №13:** задачи и файлы сохраняются после перезапуска сервисов, а каталог и поиск работают через PostgreSQL. 

---
# Часть V. Docker для приложений (шаги 156–166)

## Этап 17. Собираем JAR и создаём Dockerfile

### Шаг 156. Собери `web-service`

Из корня проекта:

```bash
cd web-service
./mvnw clean package
```

Проверь, что в `target/` появился исполняемый JAR.

### Шаг 157. Собери `repository-service`

```bash
cd ../repository-service
./mvnw clean package
```

Ожидай `BUILD SUCCESS`. Для начала собираем JAR **на хост-машине**, а контейнер будет только запускать его; позже можно изучить multi-stage builds.

### Шаг 158. Создай `web-service/Dockerfile`

```dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

- `FROM` — базовая среда Java Runtime.
- `WORKDIR` — рабочая директория приложения.
- `COPY` — перенос собранного JAR.
- `EXPOSE` — документирование внутреннего порта (не публикация наружу).
- `ENTRYPOINT` — команда запуска.

### Шаг 159. Создай `repository-service/Dockerfile`

```dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8082
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

> Эти Dockerfile предполагают, что в `target/` один подходящий исполняемый JAR. Если там несколько JAR, задай более конкретное имя файла или скопируй нужный артефакт в стандартное место перед сборкой образа.

## Этап 18. Запускаем сервисы с Docker Compose

### Шаг 160. Дополни существующий `compose.yaml`

Внутри `services:` добавь **не вместо** существующего `db`, а рядом с ним:

```yaml
  repository-service:
    build: ./repository-service
    environment:
      SERVER_ADDRESS: 0.0.0.0
      DB_URL: jdbc:postgresql://db:5432/portfolio
      DB_USER: portfolio
      DB_PASSWORD: ${POSTGRES_PASSWORD:-portfolio_dev}
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - uploads:/app/uploads

  web-service:
    build: ./web-service
    environment:
      SERVER_ADDRESS: 0.0.0.0
      REPOSITORY_URL: http://repository-service:8082
    ports:
      - "127.0.0.1:8080:8080"
    depends_on:
      - repository-service
```

В существующем блоке верхнего уровня `volumes:` оставь `pgdata:` и добавь:

```yaml
  uploads:
```

`repository-service` — DNS-имя сервиса внутри сети Compose. `127.0.0.1` внутри контейнера означал бы **сам контейнер**, поэтому приложения соединяются по именам `db` и `repository-service`.

Для сохранения файлов настрой `repository-service` так, чтобы использовался путь `/app/uploads` в контейнере. В локальной IDEA можно использовать другую директорию через параметр приложения.

### Шаг 161. Запусти всю систему

Вернись в корень проекта:

```bash
cd ..
sudo docker compose up --build -d
sudo docker compose ps
sudo docker compose logs -f repository-service
```

Открой `http://localhost:8080`. Для выхода из просмотра логов используй `Ctrl+C`: контейнеры продолжат работу.

### Шаги 162–166. Проверь жизненный цикл

162. Проверь, что главная страница открывается из контейнера.
163. Вызови `/api/products`, поиск и счётчики.
164. Перезапусти только `repository-service`: `sudo docker compose restart repository-service`.
165. Проверь, что задачи и загруженные файлы сохранились.
166. Изучи `docker compose ps`, `logs`, `stop`, `start`, `down` и имена сервисов.

**Осторожно:** `docker compose down -v` **удаляет именованные тома** и может уничтожить данные БД. Не выполняй эту команду, если данные нужны. `depends_on` задаёт порядок старта; для готовности приложений добавь healthcheck и обработку временной недоступности.

**Контрольная точка №14:** `web-service`, `repository-service` и PostgreSQL работают раздельно в контейнерах, но доступны как одна система. [Dockerfile](https://docs.docker.com/reference/dockerfile/), [Compose Specification](https://docs.docker.com/reference/compose-file/).

---

# Часть VI. Unity, Brotli и Nginx (шаги 167–180)

## Этап 19. Подготовка Web-сборки Unity

| Шаг | Задача |
|---:|---|
| 167 | Создай или открой свой проект в Unity |
| 168 | Установи модуль **Web Build Support** |
| 169 | Выбери платформу **Web** в настройках сборки |
| 170 | В Player/Publishing Settings выбери **Compression Format: Brotli**; настрой отключение Decompression Fallback, если сервер будет выдавать корректные заголовки |
| 171 | Экспортируй Web-сборку в `portfolio-lab/unity-build/` |

Ожидаемая структура:

```text
portfolio-lab/
└── unity-build/
    ├── index.html
    ├── Build/
    └── TemplateData/
```

Точный состав зависит от версии Unity. Ограничение **100 МБ** проверяется для реального экспортированного проекта — Brotli само по себе не гарантирует нужного размера. При необходимости оптимизируй текстуры, аудио и ресурсы. [Unity Web Deployment](https://docs.unity.com/en-us/engine/6000.3/manual/platform-specific/webgl/building-distribution/deploying).

## Этап 20. Создаём Nginx gateway

### Шаг 172. Создай `infra/nginx.conf`

```nginx
server {
    listen 80;
    server_name localhost;

    location /unity/ {
        root /usr/share/nginx/html;
        try_files $uri $uri/ =404;
    }

    location ~ ^/unity/.*\.wasm\.br$ {
        root /usr/share/nginx/html;
        gzip off;
        default_type application/wasm;
        add_header Content-Encoding br;
        add_header Vary Accept-Encoding;
    }

    location ~ ^/unity/.*\.js\.br$ {
        root /usr/share/nginx/html;
        gzip off;
        default_type application/javascript;
        add_header Content-Encoding br;
        add_header Vary Accept-Encoding;
    }

    location ~ ^/unity/.*\.data\.br$ {
        root /usr/share/nginx/html;
        gzip off;
        default_type application/octet-stream;
        add_header Content-Encoding br;
        add_header Vary Accept-Encoding;
    }

    location ~ ^/unity/.*\.symbols\.json\.br$ {
        root /usr/share/nginx/html;
        gzip off;
        default_type application/octet-stream;
        add_header Content-Encoding br;
        add_header Vary Accept-Encoding;
    }

    location / {
        proxy_pass http://web-service:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Nginx выдаёт **уже сжатые файлы `.br`** и сообщает браузеру с помощью `Content-Encoding: br`, что ответ закодирован Brotli. `Content-Type` для `.wasm.br` должен быть `application/wasm`, для `.js.br` — JavaScript. Это базовая учебная конфигурация; если Unity экспортирует другие имена или расширения, дополни конфигурацию по фактическим файлам и документации версии Unity.

### Шаг 173. Добавь контейнер gateway в `compose.yaml`

В раздел `services:`:

```yaml
  gateway:
    image: nginx:stable-alpine
    ports:
      - "8088:80"
    volumes:
      - ./infra/nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./unity-build:/usr/share/nginx/html/unity:ro
    depends_on:
      - web-service
```

### Шаг 174. Запусти Nginx

```bash
sudo docker compose up --build -d
```

Проверь:

```text
http://localhost:8088
http://localhost:8088/unity/index.html
```

### Шаг 175. Встрой Unity в резюме

В `web-service/src/main/resources/static/unity.html` добавь:

```html
<iframe
    src="/unity/index.html"
    title="Unity Web Demo"
    loading="lazy"
    style="width: 100%; height: 650px; border: 0">
</iframe>
```

Путь `/unity/index.html` рассчитан на открытие сайта через Nginx (`:8088`), а не напрямую через `web-service` (`:8080`).

### Шаг 176. Проверь HTTP-заголовки Brotli

Открой DevTools → **Network**, найди запросы к `.wasm.br`, `.js.br`, `.data.br`. Проверь `Content-Encoding: br` и соответствующий `Content-Type`.

Либо:

```bash
curl -I http://localhost:8088/unity/Build/ИМЯ_ФАЙЛА.wasm.br
```

Замени `ИМЯ_ФАЙЛА` на реальное имя из `unity-build/Build`.

### Шаг 177. Проверь лимит 100 МБ

```bash
du -sh unity-build
du -sb unity-build
```

Для строгой проверки считай **все** файлы, требующиеся для работы сборки: `index.html`, `Build`, `TemplateData` и другие ресурсы. Уточни, что имеется в виду под лимитом — 100 МБ на диске или 100 МБ суммарной передачи — это разные показатели.

### Шаги 178–180

178. Оптимизируй и пересобери Unity, если размер слишком велик.
179. Протестируй Unity как минимум в Firefox и Chromium.
180. Для публичного сайта включи HTTPS и проверь требования безопасности WebAssembly/потоков, если используешь многопоточность.

**Контрольная точка №15:** сайт идёт через Nginx, Unity загружается с корректными заголовками, размер контролируется. [Unity: Nginx server configuration](https://docs.unity.com/en-us/engine/6000.3/manual/platform-specific/webgl/building-distribution/server-configuration-code-samples/web-server-config-nginx).

---

# Часть VII. Тестирование и публикация (шаги 181–200)

## Этап 21. Качество и тесты

| Шаг | Задача |
|---:|---|
| 181 | Напиши интеграционный тест `ProductRepository` с PostgreSQL |
| 182 | Проверь тестами первую, последнюю и пустую страницу |
| 183 | Проверь поиск по кириллице, разному регистру и пустой строке |
| 184 | Проверь подсказки при быстром вводе и отмену старых запросов |
| 185 | Проверь ограничения размера и типа загружаемых файлов |
| 186 | Проверь транзакционное сохранение порядка Kanban |
| 187 | Добавь Spring Boot Actuator и health endpoints |
| 188 | Сделай единый формат ошибок API и логирование |
| 189 | Проверь мобильную вёрстку, контрастность и навигацию с клавиатуры |
| 190 | Проверь Lighthouse, Browser Performance и `EXPLAIN ANALYZE` |

Когда начнёшь писать интеграционные тесты, изучи Testcontainers: он помогает запускать изолированную тестовую PostgreSQL, не завися от локальной рабочей базы. [Testcontainers Java](https://java.testcontainers.org/), [Spring Boot testing](https://docs.spring.io/spring-boot/3.5/reference/testing/index.html).

## Этап 22. Публикация портфолио

| Шаг | Задача |
|---:|---|
| 191 | Создай репозиторий на GitHub |
| 192 | Напиши `README.md`: архитектура, требования, сборка, запуск |
| 193 | Добавь скриншоты каждого демо |
| 194 | Приложи схему взаимодействия сервисов |
| 195 | Для каждой технологии опиши **зачем** она использована |
| 196 | Вынеси продакшен-настройки и секреты в окружение |
| 197 | Убедись, что в Git нет `.env` с паролями и других секретов |
| 198 | Настрой HTTPS, резервные копии PostgreSQL и закрытый доступ к внутренним сервисам |
| 199 | Разверни Docker Compose на сервере |
| 200 | Замени заглушки реальными данными о себе, ссылками и проектами |

Для публичной публикации также важно ограничить доступ к операциям изменения: без аутентификации неизвестные пользователи смогут создавать/удалять задачи или загружать файлы. Учебные демо лучше сделать доступными только для просмотра, а изменение разрешать в изолированной демо-сессии либо после входа.

---

# Итоговая структура и чек-лист

## Финальная структура репозитория

```text
portfolio-lab/
├── web-service/
│   ├── pom.xml
│   ├── mvnw
│   ├── .mvn/
│   ├── Dockerfile
│   └── src/main/
│       ├── java/dev/portfolio/web/
│       │   ├── WebServiceApplication.java
│       │   ├── controller/
│       │   ├── client/
│       │   └── dto/
│       └── resources/
│           ├── application.yaml
│           └── static/
│               ├── index.html
│               ├── catalog.html
│               ├── board.html
│               ├── upload.html
│               ├── gallery.html
│               ├── ui.html
│               ├── unity.html
│               └── assets/
├── repository-service/
│   ├── pom.xml
│   ├── mvnw
│   ├── .mvn/
│   ├── Dockerfile
│   └── src/main/
│       ├── java/dev/portfolio/repository/
│       │   ├── RepositoryServiceApplication.java
│       │   ├── controller/
│       │   ├── service/
│       │   ├── repository/
│       │   └── dto/
│       └── resources/
│           ├── application.yaml
│           └── db/migration/
│               ├── V1__schema.sql
│               ├── V2__demo_data.sql
│               ├── V3__product_search_index.sql
│               └── V4__board.sql
├── unity-build/
│   ├── index.html
│   ├── Build/
│   └── TemplateData/
├── infra/
│   └── nginx.conf
├── compose.yaml
├── .gitignore
└── README.md
```

Это **два независимых Maven-проекта**, а не один multi-module Maven: на первом этапе так проще запускать и отлаживать их отдельно. Когда освоишь Maven, можешь изучить общий родительский `pom.xml` и reactor build.

## Чек-лист контрольных точек

- [ ] **№1.** Главная HTML/CSS-страница открывается локально.
- [ ] **№2.** Резюме адаптивно, темы и навигация работают.
- [ ] **№3.** Модальные окна и dropdown доступны мышью и клавиатурой.
- [ ] **№4.** Галерея прокручивается и открывает фото.
- [ ] **№5.** Каталог умеет поиск, подсказки, фильтры, счётчики и два типа пагинации.
- [ ] **№6.** Kanban с drag and drop сохраняется в `localStorage`.
- [ ] **№7.** Зона загрузки показывает выбранные файлы и превью.
- [ ] **№8.** Сайт работает на Spring Boot и вызывает первый API.
- [ ] **№9.** `web-service` вызывает отдельный `repository-service` по HTTP.
- [ ] **№10.** PostgreSQL запущена в Docker и доступна из IDEA.
- [ ] **№11.** Flyway создал таблицы и 12 000 тестовых строк.
- [ ] **№12.** Каталог и счётчики идут через SQL, изучен индекс `pg_trgm`.
- [ ] **№13.** Задачи и файлы сохраняются на сервере после перезапуска.
- [ ] **№14.** Оба сервиса и PostgreSQL запускаются в Docker Compose.
- [ ] **№15.** Nginx раздаёт сайт и Unity Web с Brotli.

После каждого завершённого этапа:

```bash
git status
git add .
git commit -m "Describe completed milestone"
```

**Правило обучения:** перед переходом к следующему шагу ответь на три вопроса: какую проблему мы решали, почему прежнего способа недостаточно, как проверить, что новый способ работает.

## Если ты уже начал проект по прошлому руководству

- Не удаляй `repository-service`, миграции или Docker-том с нужными данными только ради другого порядка изучения.
- Сделай Git-коммит текущего рабочего состояния; затем можешь отдельно реализовать фронтенд.
- Если `V1__schema.sql` и `V2__demo_data.sql` уже применялись, **не изменяй их задним числом**. Создавай новые версии миграций либо используй совершенно новую *пустую* учебную базу для прохождения варианта с начала.
- Если IntelliJ подсвечивает корректный `gin_trgm_ops` красным, проверь диалект SQL, Data Source и фактическую ошибку в логах Flyway.
- Не пытайся одновременно исправлять Maven Wrapper, настраивать базу и писать сложный фронтенд. Проверяй по одному компоненту.

---

# Официальная документация

Ниже — ссылки на первоисточники. Документацию всегда полезно открывать рядом с IDEA; некоторые разделы в новых версиях продуктов могут называться иначе.

**Arch Linux, IntelliJ, Git**

- [ArchWiki: Java](https://wiki.archlinux.org/title/Java)
- [ArchWiki: Docker](https://wiki.archlinux.org/title/Docker)
- [IntelliJ IDEA: Spring Initializr](https://www.jetbrains.com/help/idea/spring-initializr-project-wizard.html)
- [IntelliJ IDEA: Maven](https://www.jetbrains.com/help/idea/maven-support.html)
- [IntelliJ IDEA: Database Tool Window](https://www.jetbrains.com/help/idea/database-tool-window.html)
- [Git: книга Pro Git на русском](https://git-scm.com/book/ru/v2)

**Frontend**

- [MDN: HTML](https://developer.mozilla.org/ru/docs/Learn_web_development/Core/Structuring_content)
- [MDN: CSS](https://developer.mozilla.org/ru/docs/Learn_web_development/Core/Styling_basics)
- [MDN: JavaScript](https://developer.mozilla.org/ru/docs/Web/JavaScript)
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN: IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
- [MDN: Drag and Drop](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API)
- [MDN: File API](https://developer.mozilla.org/en-US/docs/Web/API/File_API)
- [MDN: HTML dialog](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog)
- [MDN: CSS Scroll Snap](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll_snap)
- [W3C: WAI-ARIA Listbox](https://www.w3.org/WAI/ARIA/apg/patterns/listbox/)

**Java, Maven, Spring, PostgreSQL**

- [Spring Initializr](https://start.spring.io/)
- [Apache Maven Wrapper](https://maven.apache.org/tools/wrapper/)
- [Spring Boot 3.5 Reference](https://docs.spring.io/spring-boot/3.5/reference/)
- [Spring MVC controllers](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html)
- [Spring RestClient](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html#rest-restclient)
- [Spring JdbcClient](https://docs.spring.io/spring-framework/reference/data-access/jdbc/core.html#jdbc-JdbcClient)
- [Spring: Uploading Files](https://spring.io/guides/gs/uploading-files)
- [Flyway documentation](https://documentation.red-gate.com/fd)
- [PostgreSQL: SQL Tutorial](https://www.postgresql.org/docs/current/tutorial.html)
- [PostgreSQL: pg_trgm](https://www.postgresql.org/docs/current/pgtrgm.html)
- [Testcontainers Java](https://java.testcontainers.org/)

**Docker, Unity, Nginx**

- [Docker Compose: Getting Started](https://docs.docker.com/compose/gettingstarted/)
- [Docker Compose Specification](https://docs.docker.com/reference/compose-file/)
- [Dockerfile Reference](https://docs.docker.com/reference/dockerfile/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Unity Web: Deployment](https://docs.unity.com/en-us/engine/6000.3/manual/platform-specific/webgl/building-distribution/deploying)
- [Unity Web: Nginx server configuration](https://docs.unity.com/en-us/engine/6000.3/manual/platform-specific/webgl/building-distribution/server-configuration-code-samples/web-server-config-nginx)

---

*Конец практикума. Выполняй контрольные точки по очереди: в каждый момент времени у тебя должна оставаться рабочая версия проекта.*
