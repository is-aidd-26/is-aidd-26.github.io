# is-aidd-26.github.io

Страница курса **Технологии ИИ в разработке ПО**: программа, лекции, лабораторные работы.
Сайт собран на [Jekyll](https://jekyllrb.com) и публикуется через [GitHub Pages](https://pages.github.com).

## Локальный запуск

Сайт откроется на <http://localhost:4000>.

### Вариант 1: Ruby + Bundler

Требуется Ruby ≥ 3.1 и Bundler (`gem install bundler`, если ещё не установлен).

```bash
bundle install          # один раз
bundle exec jekyll serve
```

С live-перезагрузкой при изменениях — это поведение по умолчанию.
Собрать без запуска сервера: `bundle exec jekyll build` (результат в `_site/`).

### Вариант 2: Docker

Без локальной установки Ruby:

```bash
docker run --rm -it -p 4000:4000 -v "$PWD:/usr/src/app" -w /usr/src/app ruby:3.3 \
  sh -c "bundle install && bundle exec jekyll serve --host 0.0.0.0"
```

## Как добавить материал

**Лабораторную** — создайте файл `_labs/lab-N.md`:

```markdown
---
title: "Лабораторная N"
num: N
summary: "Краткое описание для списка"
---

Текст задания в markdown. Формулы: $inline$ и $$block$$.
```

Страница появится по адресу `/labs/lab-N/` и в списке на `/labs/`.

**Лекцию** — положите PDF в `assets/lectures/` и добавьте запись в `_data/lectures.yml`.

**Раздел навигации** — добавьте пункт в `_data/nav.yml`.

## Публикация

Сборка выполняется самим GitHub Pages: достаточно сделать `git push` в `main`.
Настройки репозитория: *Settings → Pages → Source: Deploy from a branch → main / (root)*.

> При деплое GitHub Pages использует собственные версии гемов
> (см. <https://pages.github.com/versions/>), поэтому новые фичи Jekyll и плагины
> вне whitelist на сервере работать не будут — локально сайт собирается Jekyll 4.4.1,
> но используйте только возможности, доступные на GitHub Pages.

## Структура проекта

```
.
├── _config.yml            # настройки сайта (название, collections, markdown)
├── Gemfile                # зависимости Ruby (jekyll)
├── index.md               # главная страница
├── 404.md                 # страница 404
├── rules.md               # правила курса
├── do-donts.md            # Do & Don't
├── _data/
│   ├── nav.yml            # пункты навигации в шапке
│   └── lectures.yml       # список лекций (данные отдельно от вёрстки)
├── _labs/                 # коллекция лабораторных работ
│   └── lab-1.md … lab-5.md
├── labs/index.html        # страница со списком лабораторных
├── lectures/index.html    # страница со списком лекций
├── _layouts/default.html  # базовый шаблон (шапка, контент, футер, MathJax)
├── _includes/             # фрагменты шаблонов (header, footer, mathjax)
└── assets/
    ├── css/style.scss     # стили (компилируются в /assets/css/style.css)
    └── lectures/*.pdf     # PDF-файлы лекций
```
