---
title: "Ошибка проверки оплаты контента"
date: 2023-12-20
draft: true
summary: "Получение доступа к платнону контенту через ошибку обработки ответа платежного сервиса stripe"
tags: ["payments", "stripe"]
categories: ["BugBounty"]
---

## Описание

Сервис предоставляет возможность получать платный медиаконтент за разовую плату. 

Оплата осуществляется через платежную систему [stripe](https://stripe.com), после чего контент становится доступным в пользовательском каталоге.

В процессе тестовой оплаты, был исседован API-запрос инициализации платежа согласно документации.

Фрагмент запроса представлен ниже:

```json
{
  ...
  "sepa_debit_info": null,
  "session_id": "cs_live_n7RSUDnLkxw4vcxfSX6XOrGmPfHorRdYpziXltcPDUziqUVFsqjEtBTzNnUnqf9fLCRou0F",
  "setup_future_usage": null,
  "setup_future_usage_for_payment_method_type": {},
  "setup_intent": null,
  "shipping_address_collection": null,
  "shipping_options": [],
  "shipping_rate": null,
  "shipping_tax_amounts": [],
  "state": "active",
  "status": "open",
  "submit_type": "pay",
  "subscription_data": null,
  "subscription_settings": null,
  "success_url": "https://example.com/api/stripe-success?session_id=cs_live_n7RSUDnLkxw4vcxfSX6XOrGmPfHorRdYpziXltcPDUziqUVFsqjEtBTzNnUnqf9fLCRou0F",
  "tax_context": {
    "automatic_tax_address_source": null,
    "automatic_tax_enabled": false,
    "automatic_tax_error": null,
    "automatic_tax_exempt": "none",
    "customer_tax_country": null,
    "dynamic_tax_enabled": false,
    "has_maximum_tax_ids": false,
    "tax_id_collection_enabled": false
    ...
  }
```

Здесь session_id - это id сессии, который отдает stripe при создании платежа, а success_url - это страница, на которую перебрасывает пользователя, в случае успешной оплаты. 

При переходе по этой ссылке, сайт сообщает нам об успешной оплате и зачисляет контент в пользовальской каталог.

Как оказалось, на стороне сервера не правильно обрабатывался статус paid от stripe.

## Решение

Необходимо загружать контент в каталог пользователя только в том случае, когда есть подтверждение от самого stripe (статус - paid) по соответствующему session_id.