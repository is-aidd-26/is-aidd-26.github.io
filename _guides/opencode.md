---
title: "Настройка OpenCode"
num: 2
summary: "Установка OpenCode и подключение ИИ-моделей"
---

# Установка OpenCode и подключение ИИ-моделей

[OpenCode](https://opencode.ai/) — это ИИ-агент для работы с кодом, один из ведущих агентов в OpenSource.

На этом курсе рекомендуем использовать OpenCode версий 1.x.

## Установка Opencode

Следуйте официальной инструкции: [opencode.ai/download](https://opencode.ai/download)

## Ключи Deepseek-Flash через прокси llm.atdd.ru

Прокси-сервер `llm.atdd.ru` (Bifrost с виртуальными ключами) открывает доступ к модели Deepseek-Flash:

- бюджет: до 2$ в неделю без учёта cache hit;
- этого достаточно, чтобы опробовать в деле AI-Driven разработку в Opencode.

Правила использования:

1. Прокси-сервер предназначен только для использования на нашем курсе AI-Driven Development
2. Используйте его только из OpenCode или иного современного ИИ-агента — это обеспечит хороший cache hit и сбережёт бюджет токенов для других студентов
3. Нельзя использовать ключ в целях, нарушающих закон — это как минимум приведёт к бану на нашем курсе

Инструкция:

1. Перейдите на [https://llm.atdd.ru/keys/](https://llm.atdd.ru/keys/)
2. Авторизуйтесь через Github
3. Следуйте инструкции и используйте ключ, указанный на этой странице

Ключ выдаётся только аккаунтам, зарегистрированным в ASAP. Если вы не смогли получить ключ — обратитесь к преподавателям.

### Добавление ключа вручную

Если преподаватель выдал вам ключ вручную, добавьте его в конфигурацию OpenCode самостоятельно. Настройка выполняется один раз на компьютер.

<details markdown="1">
<summary>Пошаговая инструкция</summary>

1. Установите OpenCode — см. раздел [«Установка OpenCode»](#установка-opencode).
2. Запустите `opencode models` — при первом запуске opencode создаст каталог конфигурации.
3. Откройте в текстовом редакторе файл глобальной конфигурации. Если файла нет — создайте его:
   - macOS / Linux: `~/.config/opencode/opencode.jsonc`
   - Windows: `%USERPROFILE%\.config\opencode\opencode.jsonc`
4. Вставьте в файл конфигурацию, замените `sk-bf-ВАШ-КЛЮЧ` на выданный ключ
   и сохраните файл в кодировке UTF-8:

   ```json
   {
     "$schema": "https://opencode.ai/config.json",
     "model": "aidd-course/deepseek-flash",
     "provider": {
       "aidd-course": {
         "npm": "@ai-sdk/openai-compatible",
         "name": "LLM-шлюз курса",
         "options": {
           "baseURL": "https://llm.atdd.ru/v1",
           "apiKey": "sk-bf-ВАШ-КЛЮЧ"
         },
         "models": {
           "deepseek-flash": {}
         }
       }
     }
   }
   ```

5. Проверьте, что провайдер настроен — команда должна вывести модель `deepseek-flash`:

   ```
   opencode models aidd-course
   ```

6. Проверьте доступность модели — запустите минимальный промпт в каталоге любого проекта:

   ```
   opencode run "Привет! Ответь одним словом."
   ```

Модель по умолчанию уже задана в конфигурации; сменить её можно командой `/models`.

</details>

## OpenCode Zen

> OpenCode Zen недоступен с российских IP-адресов.

OpenCode Zen — провайдер моделей от команды OpenCode. Модели с пометкой Free в названии бесплатны и доступны сразу после регистрации.

1. Войдите через Google- или GitHub-аккаунт на [opencode.ai/auth](https://opencode.ai/auth) и скопируйте API-ключ.
2. Перейдите в каталог проекта и запустите `opencode`.
3. Выполните команду `/connect`, выберите **OpenCode Zen** и вставьте API-ключ.
4. Выполните команду `/models` и выберите модель с пометкой Free.

## Ссылки

1. [Homebrew on Linux](https://docs.brew.sh/Homebrew-on-Linux)
2. [Installation](https://docs.brew.sh/Installation)
