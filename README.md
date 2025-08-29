# Mirroring GitLab to GitHub

Этот документ описывает процесс автоматического зеркалирования (mirroring) репозитория из **GitLab** в **GitHub** с использованием **CI/CD GitLab**.

---

## Описание процесса

Скрипт в `.gitlab-ci.yml` автоматически отправляет изменения из ветки `master` в GitLab в ветку `main` репозитория на GitHub. Это полезно для:
- Синхронизации кода между платформами.
- Резервного копирования.
- Удобства работы с разными сервисами.

---

## Настройка

### 1. Создание токена на GitHub
1. Необходимо перейти в **Settings** → **Developer settings** → **Personal access tokens**.
2. Создать новый токен с правами:
   - `repo` (полный доступ к репозиториям).
3. Сохранить токен в безопасном месте.

### 2. Настройка переменных в GitLab
1. В GitLab перейти в **Settings** → **CI/CD** → **Variables**.
2. Добавить переменную:
   - **Key**: `GITHUB_TOKEN`
   - **Value**: `<твой_токен_из_GitHub>`
   - **Type**: `Variable` (замаскированная).
3. Сохранить изменения.

---

## Конфигурация `.gitlab-ci.yml`

```yaml
stages:
  - mirror

mirror_to_github:
  stage: mirror
  image: alpine:latest  # Лёгкий образ для выполнения команд
  before_script:
    - apk update
    - apk add --no-cache git bash openssh-client  # Установка необходимых пакетов
    - git config --global user.email "vvilkov@lachestry.com"  # Настройка Git
    - git config --global user.name "cevochka-js"
  script:
    - git remote remove github || true  # Удаляем старую ссылку на удалённый репозиторий (если есть)
    - git remote add github "https://${GITHUB_TOKEN}@github.com/cevochka-js/cevochka-js.github.io.git"  # Добавляем новую ссылку с токеном
    - git push github HEAD:main --force  # Принудительно пушим изменения в ветку `main` на GitHub
  only:
    - master  # Запускаем только для ветки `master`
```

---

## Пояснения

- **`image: alpine:latest`**: Используется минималистичный образ для быстрого выполнения команд.
- **`before_script`**: Устанавливаем необходимые пакеты (`git`, `bash`, `openssh-client`) и настраиваем Git.
- **`script`**:
  - Удаляем старую ссылку на удалённый репозиторий (если она существует).
  - Добавляем новую ссылку с использованием токена для аутентификации.
  - Принудительно пушим изменения в ветку `main` на GitHub.
- **`only: - master`**: Скрипт выполняется только для коммитов в ветку `master`.

