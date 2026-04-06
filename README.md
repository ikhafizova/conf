# Гайд по созданию HTML-презентации

Полное руководство по созданию слайдовой презентации в формате единого HTML-файла.  
Основано на реальном проекте: конференция Mailfit/Letteros, Ташкент, май 2026.

---

## Структура проекта

```
conf/
├── index.html          # вся презентация — один файл
├── M.png, J.png, I.png # фотографии спикеров
├── img_1.svg           # декоративная иллюстрация (слайд 1)
├── img_2.svg           # декоративная иллюстрация (слайд 11)
├── logo letteros.svg   # логотип Letteros
├── logo mailfit uz.svg # логотип Mailfit
└── presentation.pdf    # экспорт в PDF (генерируется отдельно)
```

---

## Технические параметры

| Параметр | Значение |
|---|---|
| Размер холста | 1920 × 1080 px |
| Шрифт | Manrope (Google Fonts), веса 300–800 |
| Масштабирование | CSS `transform: scale()` под любой экран |
| Навигация | стрелки клавиатуры + клик мышью |
| Переходы | fade (opacity) 0.35s |
| Анимация входа | slideUpFade, cubic-bezier(.22,1,.36,1) |

---

## Ключевые CSS-паттерны

### Масштабирование под экран
```css
html, body { width: 100%; height: 100%; overflow: hidden; background: #000; }
#stage { position: absolute; width: 1920px; height: 1080px; transform-origin: top left; }
```
```js
function scale() {
  const s = Math.min(window.innerWidth / 1920, window.innerHeight / 1080);
  const ml = (window.innerWidth - 1920 * s) / 2;
  const mt = (window.innerHeight - 1080 * s) / 2;
  stage.style.transform = `scale(${s})`;
  stage.style.left = ml + 'px';
  stage.style.top = mt + 'px';
}
window.addEventListener('resize', scale);
scale();
```

### Анимация входа элементов
```css
@keyframes slideUpFade {
  from { opacity: 0; transform: translateY(30px); }
  to   { opacity: 1; transform: translateY(0); }
}
.a {
  animation-duration: .65s;
  animation-timing-function: cubic-bezier(.22, 1, .36, 1);
  animation-fill-mode: both;
  animation-delay: var(--d, 0s);
}
.slide.active .a { animation-name: slideUpFade; }
```
Использование: `<div class="a" style="--d:.2s">` — задержка через CSS-переменную.

### Анимация SVG-декора (вращение колец)
```css
@keyframes spinCW  { to { transform: rotate(360deg); } }
@keyframes spinCCW { to { transform: rotate(-360deg); } }
.spin-cw  { animation: spinCW  var(--sd, 60s) linear infinite; transform-box: fill-box; transform-origin: 50% 50%; }
.spin-ccw { animation: spinCCW var(--sd, 80s) linear infinite; transform-box: fill-box; transform-origin: 50% 50%; }
```
Использование: `<circle ... class="spin-cw" style="--sd:90s"/>`

**Важно:** SVG-элементы с анимацией должны позиционироваться через `cx`/`cy`, а не через `transform="matrix(...)"` — иначе CSS-анимация перезапишет матрицу и круги съедут.

### Навигационные точки
```css
#navdots { position: fixed; bottom: 18px; left: 50%; transform: translateX(-50%); display: flex; gap: 8px; z-index: 999; }
.dot { width: 8px; height: 8px; border-radius: 4px; background: rgba(150,150,150,.45); box-shadow: 0 0 0 1.5px rgba(0,0,0,.12); cursor: pointer; transition: all .3s ease; }
.dot.on { width: 24px; background: #00C8F0; }
```
Серые на светлых слайдах видны за счёт `box-shadow`; активная точка — бирюзовая полоска.

### Footer (стандартный на всех слайдах кроме первого)
```css
.sfoot { position: absolute; bottom: 44px; left: 80px; right: 80px; display: flex; justify-content: space-between; font-size: 14px; font-weight: 500; letter-spacing: .14em; text-transform: uppercase; }
.sfoot-dk { color: rgba(255,255,255,.28); } /* тёмный фон */
.sfoot-lt { color: rgba(0,0,0,.28); }       /* светлый фон */
```

---

## Структура слайда

```html
<div class="slide" id="s01">
  <!-- контент -->
  <div class="sfoot sfoot-dk">
    <span>MAILFIT &nbsp;&nbsp;|&nbsp;&nbsp; LETTEROS</span>
    <span>ТАШКЕНТ &nbsp;&nbsp;|&nbsp;&nbsp; МАЙ 2026</span>
  </div>
</div>
```

Все элементы внутри слайда — `position: absolute` с координатами от краёв холста.  
Базовые отступы: **80px** от левого/правого края, **80px** сверху.

---

## Типографика (проверенные размеры)

| Элемент | Размер | Вес |
|---|---|---|
| Главный заголовок (обложка) | 140px | 400 |
| Заголовок слайда | 100px | 600 |
| Подзаголовок | 50px | 500 |
| Текст карточки | 32px | 400 |
| Формула / акцент | 100px | 600 |
| Имя спикера | 50px | 700 |
| Лейбл (uppercase) | 22px | 700 |
| Footer | 14px | 500 |

**Межстрочное расстояние:** `line-height: 1.2` для заголовков, `line-height: 1.3` для текста.  
**Висячие предлоги:** привязывать к следующему слову через `&nbsp;` — `с&nbsp;клиентами`, `по&nbsp;задачам`.

---

## QR-коды (offline)

Используется библиотека [qrcodejs](https://github.com/davidshimjs/qrcodejs) — генерирует QR прямо в браузере, не нужен интернет для рендеринга.

```html
<script src="https://cdn.jsdelivr.net/npm/qrcodejs@1.0.0/qrcode.min.js"></script>

<div id="qr-site"></div>

<script>
new QRCode(document.getElementById('qr-site'), {
  text: 'https://letteros.com',
  width: 320, height: 320,
  colorDark: '#000', colorLight: '#fff',
  correctLevel: QRCode.CorrectLevel.M
});
</script>
```

---

## Экспорт в PDF

Требует Google Chrome. Запускать из папки с проектом:

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new \
  --disable-gpu \
  --no-pdf-header-footer \
  --print-to-pdf="presentation.pdf" \
  --paper-width=20 \
  --paper-height=11.25 \
  --no-margins \
  "file:///ПОЛНЫЙ/ПУТЬ/ДО/index.html"
```

Размер 20×11.25 дюймов = ровно 16:9 при 96 dpi.

Для корректного PDF в `index.html` должен быть блок:
```css
@media print {
  @page { size: 20in 11.25in; margin: 0; }
  * { animation: none !important; transition: none !important; }
  #stage { position: relative !important; transform: none !important; left: 0 !important; top: 0 !important; }
  .slide { position: relative !important; width: 1920px !important; height: 1080px !important; opacity: 1 !important; page-break-after: always; break-after: page; }
  .a { opacity: 1 !important; transform: translateY(0) !important; }
  #navdots { display: none !important; }
}
```

---

## Публикация на GitHub Pages

1. Создать репозиторий на GitHub (публичный)
2. Загрузить все файлы папки в `main` ветку
3. В настройках репо: **Settings → Pages → Source: main branch, / (root)**
4. Ссылка появится через 1–2 минуты: `https://USERNAME.github.io/REPO/`

Через API (автоматически):
```bash
# Создать репо
curl -H "Authorization: token TOKEN" \
  -d '{"name":"conf","private":false}' \
  https://api.github.com/user/repos

# Включить Pages
curl -X POST -H "Authorization: token TOKEN" \
  -d '{"source":{"branch":"main","path":"/"}}' \
  https://api.github.com/repos/USERNAME/conf/pages
```

---

## Чеклист перед показом

- [ ] Шрифт Manrope загружается (нужен интернет или кэш)
- [ ] Фотографии открываются (`M.png`, `J.png`, `I.png` лежат рядом с `index.html`)
- [ ] QR-коды рендерятся (qrcodejs грузится с CDN)
- [ ] Анимация иллюстрации работает (SVG-кольца вращаются)
- [ ] Навигация: стрелки ← → и клик работают
- [ ] PDF сгенерирован заново после последних правок
- [ ] GitHub Pages обновился (пушнуть изменения)

---

## Частые ошибки

| Проблема | Причина | Решение |
|---|---|---|
| SVG-кольца при анимации прыгают на неправильное место | У элементов был `transform="matrix(...)"` — CSS анимация его перезаписала | Вычислить реальные `cx`/`cy` применив матрицу, убрать `transform` атрибут |
| Фото серые прямоугольники | `background` на `.tphoto` + изображение не покрывает контейнер | `object-fit: cover; object-position: top center; width: 100%; height: 100%; position: absolute; inset: 0` |
| Нижние карточки не на всю ширину | `grid-column: 2/4` вместо `1/4` | Проверить `grid-column` у каждой карточки |
| Точки навигации не видны на светлых слайдах | `rgba(255,255,255,.28)` невидим на белом | Добавить `box-shadow: 0 0 0 1.5px rgba(0,0,0,.12)` |
| QR-коды не загружаются | `api.qrserver.com` недоступен | Заменить на `qrcodejs` (генерация в браузере) |
| Заголовок и подзаголовок слипаются | Слишком маленький `top` у подзаголовка | Увеличить `top` или добавить `margin-top` |
