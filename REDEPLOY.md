# Инструкция по передеплою RAG Habr API

## Когда нужно передеплоить

Когда закончатся бесплатные $5 кредита на Railway.app, нужно передеплоить сервис на другой бесплатный хостинг или пополнить баланс Railway.

## Вариант 1: Передеплой на другой бесплатный хостинг

### Шаг 1: Выбор платформы

Доступные бесплатные варианты:
- **Render.com** - требует кредитную карту, но бесплатный тир доступен
- **Fly.io** - бесплатный тир с ограничениями
- **Koyeb** - бесплатный тир без карты
- **PythonAnywhere** - бесплатный аккаунт с ограничениями

### Шаг 2: Подготовка к деплою

1. Убедись что код в GitHub актуален:
```bash
cd /path/to/rag_habr/server
git status
git push origin main
```

2. Проверь что все файлы на месте:
```bash
ls -la
# Должны быть: Dockerfile, requirements.txt, app/, .gitignore
```

### Шаг 3: Деплой на Render.com

1. Зайди на https://render.com
2. Зарегистрируйся через GitHub
3. Нажми "New" → "Web Service"
4. Подключи репозиторий `mariklolik/habr_rag`
5. Настройки:
   - **Name**: `rag-habr-api`
   - **Runtime**: `Docker`
   - **Root Directory**: оставь пустым
   - **Build Command**: оставь пустым (используется Dockerfile)
   - **Start Command**: оставь пустым (используется CMD из Dockerfile)
6. Добавь Environment Variables:
   ```
   API_KEY=твой_ключ
   BASE_URL=https://gpt.mwsapis.ru/projects/mws-ai-automation/openai/v1/
   LLM_MODEL=qwen3-235b-instruct
   EMBED_BASE_URL=https://gpt.mwsapis.ru/projects/mws-ai-automation/openai/v1/
   EMBED_API_KEY=твой_ключ
   EMBED_MODEL=bge-m3
   DEFAULT_TEMPERATURE=0.7
   MAX_TOKENS=5000
   DEBUG=False
   QDRANT_API_URL=http://51.250.99.222:6333
   QDRANT_API_KEY=API_KEY
   ```
7. Нажми "Create Web Service"
8. Дождись завершения деплоя (~5 минут)
9. Проверь работу: `curl https://your-service.onrender.com/health`

### Шаг 4: Деплой на Fly.io

1. Установи Fly CLI:
```bash
curl -L https://fly.io/install.sh | sh
```

2. Авторизуйся:
```bash
fly auth login
```

3. Создай приложение:
```bash
cd /path/to/rag_habr/server
fly launch
# Выбери регион, имя приложения
```

4. Добавь переменные окружения:
```bash
fly secrets set API_KEY=твой_ключ
fly secrets set BASE_URL=https://gpt.mwsapis.ru/projects/mws-ai-automation/openai/v1/
fly secrets set LLM_MODEL=qwen3-235b-instruct
fly secrets set EMBED_BASE_URL=https://gpt.mwsapis.ru/projects/mws-ai-automation/openai/v1/
fly secrets set EMBED_API_KEY=твой_ключ
fly secrets set EMBED_MODEL=bge-m3
fly secrets set DEFAULT_TEMPERATURE=0.7
fly secrets set MAX_TOKENS=5000
fly secrets set DEBUG=False
fly secrets set QDRANT_API_URL=http://51.250.99.222:6333
fly secrets set QDRANT_API_KEY=API_KEY
```

5. Деплой:
```bash
fly deploy
```

6. Проверь работу: `curl https://your-app.fly.dev/health`

### Шаг 5: Деплой на Koyeb

1. Зайди на https://www.koyeb.com
2. Зарегистрируйся через GitHub
3. Нажми "Create App"
4. Выбери "GitHub" → репозиторий `mariklolik/habr_rag`
5. Настройки:
   - **Buildpack**: Docker
   - **Root Directory**: оставь пустым
6. Добавь Environment Variables (те же что выше)
7. Нажми "Deploy"
8. Дождись завершения деплоя
9. Проверь работу

## Вариант 2: Пополнение баланса Railway

### Шаг 1: Добавление карты

1. Зайди на https://railway.com
2. Открой проект `diligent-adaptation`
3. Перейди в Settings → Billing
4. Добавь кредитную карту
5. Railway автоматически пополнит баланс при необходимости

### Шаг 2: Проверка баланса

1. В Dashboard проверь баланс
2. Railway показывает "X days or $Y.XX left"
3. При пополнении баланса сервис продолжит работать автоматически

## Вариант 3: Передеплой на Railway с новым аккаунтом

Если не хочешь добавлять карту, можно создать новый аккаунт Railway:

1. Создай новый аккаунт на Railway (другой email/GitHub)
2. Получишь новые $5 бесплатного кредита
3. Повтори шаги деплоя из основной инструкции

## Проверка работоспособности после деплоя

После любого деплоя проверь:

```bash
# 1. Health check
curl https://your-service-url/health
# Ожидаемый ответ: {"status":"ok"}

# 2. Swagger UI
curl https://your-service-url/docs
# Должен вернуть HTML страницу

# 3. Тест RAG API
curl -X POST "https://your-service-url/api/v1/rag" \
  -H "Content-Type: application/json" \
  -d '{"query": [{"role": "user", "content": "Что такое Python?"}]}'
# Должен вернуть JSON с ответом и источниками
```

## Важные моменты

1. **Environment Variables**: Всегда проверяй что все переменные окружения установлены правильно
2. **Root Directory**: Должен быть пустым (репозиторий уже в папке server/)
3. **Custom Start Command**: Должен быть пустым (используется CMD из Dockerfile)
4. **Dockerfile**: Убедись что Dockerfile использует правильный PORT:
   ```dockerfile
   CMD ["sh", "-c", "uvicorn app.main:app --host 0.0.0.0 --port ${PORT:-8000}"]
   ```

## Мониторинг

После деплоя следи за:
- Логами сервиса (в панели хостинга)
- Использованием ресурсов
- Балансом (если платный хостинг)
- Доступностью API (можно настроить uptime monitoring)

## Резервное копирование

Перед передеплоем убедись что:
- Код в GitHub актуален
- Environment Variables сохранены в безопасном месте
- Конфигурация Dockerfile правильная

## Контакты и поддержка

Если возникли проблемы:
- Проверь логи деплоя в панели хостинга
- Убедись что все Environment Variables установлены
- Проверь что Dockerfile собирается без ошибок
- Проверь доступность внешних сервисов (Qdrant, LLM API)

