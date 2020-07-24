---
title: "Создаем техблог за вечер с CD pipeline и блэкджеком"
date: 2020-07-23T22:47:00+03:00
draft: false
tags: [github, blog]
---

![image alt text](images/tech-blog.png)

Вчера вечером ~~в очередной раз~~ понял, что хочу завести свой технический блог. В прошлом уже были неудачные попытки, когда интерес к этой затее терялся. 
Но мы же оптимисты - и в этот в раз все будет по-другому! 💪 Дабы не растерять запал решил как можно быстрее сделать первый шаг - зарелизить платформу для блога.

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
- [Gatsby](),  [Next.js]() - новомодные на React

В итоге в результате беглого сравнения мой выбор пал на Hugo, понравилась документация, много звезд на github, более производительный так как написан на Go.

Следуя [Quick start guide](https://gohugo.io/getting-started/quick-start/) легко создаем основу блога и первый контент.

### Минималистичный дизайн

### Blog as Code
В моем случае все будет храниться в [репозитории](https://github.com/eugenix/jekurb-blog) на github.

### Простота и бесплатность хостинга
Давно известно, у github есть замечательная фича [Gihub pages](https://pages.github.com/), которая позволяет хостить статический сайт напрямую из репозитория. И да, это бесплатно. Поэтому заведем [второй репозиторий](https://github.com/eugenix/eugenix.github.io) для "релизного билда" блога.

### Доменное имя

### CD pipeline
```yaml
name: Push to GitHub Pages on push to master
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
        
      - name: Deploy the site
        uses: benmatselby/hugo-deploy-gh-pages@master
        env:
          HUGO_VERSION: 0.74.2
          TARGET_REPO: eugenix/eugenix.github.io
          TOKEN: ${{ secrets.PERSONAL_TOKEN }}
          CNAME: jekurb.dev
```

### Выводы
  