# Проект «Сайгак: сохранить степь»

Некоммерческий просветительский сайт о сохранении сайгака в Калмыкии.
Учебный проект по дисциплине «Веб-аналитика».

## Жёсткие правила

- Проект НЕКОММЕРЧЕСКИЙ: никаких цен, продаж, платных услуг.
- Счётчики аналитики: Яндекс.Метрика, Рамблер/Топ100, Top.Mail.ru (MyTracker),
  LiveInternet. Другие системы не добавлять, даже «на всякий случай».
- Вёрстка: чистый HTML/CSS/JS без фреймворков и сборки.
- Продакшн: статика лежит в nginx (Dockerfile), снаружи — Caddy как
  реверс-прокси с автоматическим TLS (docker-compose.yml),
  домен wa-saiga.duckdns.org.

## Структура

- 6 страниц: index, saiga, decline, rescue, faq, help.
- Общие блоки разметки — в site/partials/ и подключаются через SSI
  (Server Side Includes): head, counters, statusbar, header-nav,
  disclaimer, footer, analytics, saiga-symbol. Переменные страницы
  задаются в начале файла: PAGE, TITLE, DESC, NOINDEX, ISSUE,
  STATUS_TEXT, PAGE_PATH.
- SSI включён в nginx.conf (`ssi on`), который копируется в образ
  (Dockerfile) и монтируется в dev (docker-compose.dev.yml).
  Каталог /partials/ закрыт (`internal`).
- Общие стили: css/style.css (шрифты: Unbounded / IBM Plex Sans / IBM Plex Mono;
  палитра: песок --paper, чернила --ink, красный акцент --red).
- У каждой страницы свои стили в <style> в конце файла.
- Все страницы отдают `<meta name="robots" content="noindex">`
  (NOINDEX=true) — проект учебный.
- Изображения: только локальные, site/img/*.jpg — внешние фотостоки не используем
  (picsum.photos без ВПН отдаёт 403).
- Единый слой аналитики: функция track(action, params) в скрипте каждой
  страницы; YM_ID — номер счётчика Метрики.

## События аналитики (идентификаторы = цели Метрики, тип «JavaScript-событие»)

- help_submit — ГЛАВНАЯ цель (формы на help.html)
- faq_open, faq_filter — вопросы
- anatomy_open, season_open — страница «О сайгаке»
- era_open — страница «Почему исчезает»
- program_open — страница «Как спасают»
- share_copy, share_tg, share_vk — блок «Рассказать»
- form_error — ошибка валидации формы
- page_view — просмотр страницы (на каждой странице, с параметром page)
- навигационные клики через data-track: cta_*, nav_*, faq_link_* (по страницам)

## Счётчики (идентификаторы стоят на всех 6 страницах)

- Яндекс.Метрика: YM_ID = 112433520; в футере каждой страницы стоит
  информер Метрики (cid 112433520)
- Рамблер/Топ100: pid = 7752474
- Top.Mail.ru (MyTracker): id = 3793381
- LiveInternet

## Сервер и деплой (CI/CD)

- Продакшн-ветка — main. Пуш в main запускает GitHub Actions
  (.github/workflows/deploy.yml): по SSH на VPS выполняется
  `git pull origin main` → `docker compose up -d --build` →
  `docker compose restart caddy`. Отдельно заходить на сервер не нужно.
- Секреты репозитория: VPS_HOST, VPS_USER, VPS_SSH_KEY, SITE_PATH.
- На том же Caddy размещён второй, отдельный проект:
  kursograd.duckdns.org → cite-flask:8080 — его конфиг в Caddyfile не трогать
  без необходимости.

## Договорённости по контенту

- Заповедник «Чёрные земли», с 1990, 121,9 тыс. га.
- Калмыцкая популяция: минимум ~300 особей в 2002, ~7 000 в 2024.
- Числа в графиках — упрощённые, подпись об этом обязательна.

## Запуск локально

Dev (для правок, файлы монтируются — перезапуск не нужен):
docker compose -f docker-compose.dev.yml up -d → http://localhost:8080

Прод-конфиг локально (обычно не нужен): docker compose up -d --build —
поднимет nginx + Caddy на портах 80/443.
