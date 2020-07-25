---
title: "Создаем техблог за вечер с CD pipeline и блэкджеком"
date: 2020-07-23T22:47:00+03:00
draft: false
tags: [github, blog]
---

![image alt text](images/tech-blog.png)

Вчера вечером ~~в очередной раз~~ понял, что хочу завести свой технический блог. В прошлом уже были неудачные попытки,
 когда интерес к этой затее терялся. Но мы же оптимисты - и в этот в раз все будет по-другому! 💪 Дабы не растерять
 запал решил как можно быстрее сделать первый шаг - зарелизить платформу для блога.

Итак, начнем с требований:

- Простота написания контента
- Минималистичный дизайн
- Хранение всего в системе контроля версий, можно это назвать подходом - Blog as Code
- Простота и бесплатность хостинга
- Простое и уникальное доменное имя
- CD pipeline со smoke-тестами и выкладкой на хостинг по коммиту

Думаю, для технического блога программиста самое оно 😉

### Простота написания контента

Для этого удобно использовать старый добрый markdown и статический генератор на основе него. Перечислю основные проекты:

- [Jekyll](http://jekyllrb.com)
- [Hugo](https://gohugo.io/)
- [Gatsby](https://gatsby.js),  [Next.js](https://next.js) - новомодные на React

В итоге в результате беглого сравнения мой выбор пал на Hugo, понравилась документация, много звезд на github,
 более производительный так как написан на Go (как заявляет автор < 1 ms per page).

Следуя [Quick start guide](https://gohugo.io/getting-started/quick-start/) легко создаем основу блога и первый контент.

```bash
brew install hugo
hugo new site jekurb
hugo new posts/blog-kick-off/index.md
hugo server -D
```

### Минималистичный дизайн

Основной фокус блога будет на контенте, поэтому в дизайне и функциональности не должно быть ничего лишнего.
У Hugo есть [отдельный репозиторий](https://themes.gohugo.io/) с темами. Мой топ-3 тем для блога:

1. [Hugo Ink](https://themes.gohugo.io/hugo-ink/)
2. [Hugo Vitae](https://themes.gohugo.io/hugo-vitae/)
3. [LoveIt](https://themes.gohugo.io/LoveIt/)

С большой долей вероятности в теме в будущем будут какие-то кастомные правки, поэтому лучше класть в _/themes_ ее
как не как git module. Для активации темы в _config.toml_ добавляем:

```toml
theme = "hugo-ink"
```  

### Blog as Code

В моем случае все будет храниться в [репозитории](https://github.com/eugenix/jekurb-blog) на github.
Не забываем добавить в .gitignore все, то что генерится Hugo.

```gitignore
public
.idea
resources/_gen
```

Немного о структуре директорий Hugo

```text
.
├── archetypes   | архетипы, на основе который будут создаваться блог посты с помощью команды hugo new
├── config.toml  | основной конфигурационный файл
├── content      | контент в виде md файлов, структура может быть вложенной
├── data         | конфигурационные файлы для генерации
├── layouts      | шаблоны для страниц
├── public       | сборка сайта
├── resources    | кэш
├── static       | статичные файлы, копируются при генерации
└── themes       | установленные темы
```

### Простота и бесплатность хостинга

Давно известно, у github есть замечательная фича [Gihub pages](https://pages.github.com/), которая позволяет хостить
статический сайт напрямую из репозитория. И да, это бесплатно.
Поэтому заведем [второй репозиторий](https://github.com/eugenix/eugenix.github.io) для "релизного билда" блога.

### Доменное имя

После некоторых раздумий, было выбрано доменное имя **jekurb.dev** и успешно зарегистрированно на reg.ru.
Стоимость регистрации в этой зоне - 1392 RUB за год.
Следующим шагом будет custom domains в
[Github pages](https://docs.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site),
 для этого переходим в настройки репозитория.
![image custom domain](images/custom-domain.png)
Альтернативный способ установки поля Custom domain - создание CNAME в репозитории.
Далее идем к регистратору и настаиваем DNS записи, настроим CNAME для сабдомена www, указывающую на eugenix.github.io
и A records для amex домена.
![image dns](images/dns.png)
Примерно через 4 часа DNS обновился и блог стал доступен по домену. На всякий случай проверяем, все ли корректно встало:

```bash
dig www.jekurb.dev +nostats +nocomments +nocmd

; <<>> DiG 9.10.6 <<>> www.jekurb.dev +nostats +nocomments +nocmd
;; global options: +cmd
;www.jekurb.dev.			IN	A
www.jekurb.dev.		21599	IN	CNAME	eugenix.github.io.
eugenix.github.io.	3599	IN	A	185.199.109.153
eugenix.github.io.	3599	IN	A	185.199.108.153
eugenix.github.io.	3599	IN	A	185.199.111.153
eugenix.github.io.	3599	IN	A	185.199.110.153
```

```bash
dig jekurb.dev +noall +answer

; <<>> DiG 9.10.6 <<>> jekurb.dev +noall +answer
;; global options: +cmd
jekurb.dev.		21599	IN	A	185.199.108.153
jekurb.dev.		21599	IN	A	185.199.109.153
jekurb.dev.		21599	IN	A	185.199.110.153
jekurb.dev.		21599	IN	A	185.199.111.153
```

### CD pipeline

- md lint
- e2e test

```yaml
name: Deploy to GitHub Pages on push to master
on:
  push:
    branches:
      - master
jobs:
  build:
    name: Deploy
    runs-on: ubuntu-latest
    steps:
      - name: Checkout master
        uses: actions/checkout@v1

      - name: Setup node
        uses: actions/setup-node@v1
        with:
          node-version: '12.x'

      - name: Check markdown
      - run: npm install -g markdownlint-cli
      - run: markdownlint content

      - name: Deploy the site
        uses: benmatselby/hugo-deploy-gh-pages@master
        env:
          HUGO_VERSION: 0.74.2
          TARGET_REPO: eugenix/eugenix.github.io
          TOKEN: ${{ secrets.PERSONAL_TOKEN }}
          HUGO_ARGS: ''
          CNAME: jekurb.dev

      - name: Run smoke tests
        env:
          BLOG_BASE_URL: https://jekurb.dev
        run: cd smoke-tests && npm install && npm test
```

### Выводы

Во-первых, стоит отметить, что 99% всего что использовалось, начиная движком для блога и заканчивая инфраструктурой
 для pipeline и хостингом есть на github.
Во-вторых, результатом я оказался доволен и смог доказать что поднять свой блог с CD pipeline'ом и блэкджеком не
имея особого опыта в "блогостроении" можно за несколько часов. Главное сделать первый шаг, а дальше процесс вас затянет.
