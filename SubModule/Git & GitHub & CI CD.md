# Git & GitHub & CI/CD

# GitHub Actions

**GitHub Actions** — это встроенная в GitHub система автоматизации CI/CD (непрерывной интеграции и доставки), которая позволяет запускать автоматические процессы (называемые workflow) в ответ на различные события в репозитории.

Actions находятся по пути: `.github/workflows/name.yml` в yml файле. 

```yaml
# Название Action
name: Demo action
# Событие/я на который реагирует Action
# workflow_dispatch будет срабатывать только когда этот пайп в main ветке
on: 
  push:
    branches: 
      - master
# Можно указать переменные среды
env:
  DB_NAME: db
  # Использование сикретов из github
  DB_PASSWORD: ${{ secrets.DB_PWD }}
  
# Список работ которые должны выполниться
jobs:
  # Название работы
  test:
    # Среда выполнения
    runs-on: ubuntu-latest
    # Шаги выполнения
    steps:
      # Название шага
      - name: Checkout repo
        # Дополнительный Action из GitHub
        uses: actions/checkout@v3
      - name: Echo env
	      run: echo ${{ env.DB_NAME }}

  deploy:
    runs-on: ubuntu-latest
    # Указывается что следует после test
    needs: test
```

Указывается название Action, триггер на который реагирует action, `env` (при необходимости), и jobs с работами, которые состоят из шагов. Одна job состоит из шагов каждый из которых содержит имя.

Триггеры можно ввести как массив [push, pull-request] так и более детально

Можно использовать дополнительные actions из github marketplace, подключается с помощью `uses: название-action`

По умолчанию все работы выполняются параллельно, но можно этим управлять используя `needs: работа` , где текущая работа дожидается завершения другой.

Также можно использовать secrets из нашего github settings, добавляется с помощью:   `${{ secrets.НАЗВАНИЕ }}`

---

# GitHub Runner

В pipelines можно использовать не только исполняющую машину от GitHub (Ubuntu, и тд.), можно разместить свой Runner самостоятельно на своём сервере.

**Создать runner в GitHub**

В: `Settings → Actions → Runners → New self-hosted runner` GitHub покажет точные команды с токеном регистрации для вашей ОС.

**Запуск**

Для запуска можно использовать команду `./run.sh` но при закрытии терминала - runner остановится. Для запуска runner как сервис надо использовать:

```basic
sudo ./svc.sh install    # создать systemd unit
sudo ./svc.sh start      # запустить сервис
# Дополнительные команды
sudo ./svc.sh stop       # остановить
sudo ./svc.sh status     # проверить состояние
sudo ./svc.sh uninstall  # удалить сервис
```

**Использование**

Далее можно использовать self-hosted runner в своём пайплайне.

```yaml
jobs:
  build:
    runs-on: self-hosted          # любой self-hosted runner
    # или по labels:
    runs-on: [self-hosted, linux, x64, production]
```

### Важные нюансы

**Безопасность.** Не используйте self-hosted runners для публичных репозиториев — форкнутый PR может выполнить произвольный код на вашей машине. Для приватных репозиториев это нормально.

**Ephemeral runners.** Для CI в контейнерах рекомендуется запускать runner с флагом `--ephemeral` — он автоматически снимается с регистрации после одной задачи, что безопаснее и чище.

**Группы и labels.** Через `Settings → Actions → Runner groups` можно ограничить, каким репозиториям доступен runner. Labels позволяют точно адресовать нужную машину в `runs-on`.

**Зависимости.** Runner сам не устанавливает Node.js, Docker и прочее — это должно уже быть на машине. Для унификации среды удобно использовать Docker-контейнеры внутри шагов через `container:` или action `docker/build-push-action`.