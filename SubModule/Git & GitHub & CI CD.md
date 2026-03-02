# Git & GitHub & CI/CD

# GitHub Actions

**GitHub Actions** — это встроенная в GitHub система автоматизации CI/CD (непрерывной интеграции и доставки), которая позволяет запускать автоматические процессы (называемые workflow) в ответ на различные события в репозитории.

Actions находятся по пути: `.github/workflows/name.yml` в yml файле. 

```yaml
# Название Action
name: Demo action
# Событие/я на который реагирует Action
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