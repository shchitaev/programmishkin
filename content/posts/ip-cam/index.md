---
title: "Получение доступа к IP-камерам"
date: 2024-05-10
description: "Уязвимость частных IP-камер регионального провайдера на основе продукта Flussonic"
tags: ["Flussonic", "IP-CAM", "rtsp"]
categories: ["BugBounty"]
draft: false
cover: "/img/Flussonic-admin1.jpg"
slug: "ip-cam"

---

# {{< param "title" >}}

Была обнаружена уязвимость, которая позволяет получить доступ к административной панели систем Flussonic (видеостриминовый сервер).

Провайдер оказывает услугу доступа к городским и частным видеокамерам. В ходе исследования API, был обнаружен запрос фрагмент которого, представлен ниже:

```json
{
    ...
    "opened_at": 1714103635526,
    "dvr_storage_io_read": 0,
    "dvr_storage_io_write": 1352243,
    "dvr_played_bytes": 0,
    "out_bandwidth": 2024,
    "dvr_replication_running": false,
    "status": "running",
    "url": "rtsp://admin:Rp8aXaUTlsZSb@10.77 ... /ch02/0",
    "running_transcoder": false,
    "playback_bytes": 19214439479,
    "push_duration": 0,
    "playback_duration": 63010,
    "last_access_at": 1716072174866,
    "dvr_recorded_duration": 1968542,
    "alive": true
    ...
}
```
Flussonic обеспечивает доставку от оборудования DVR по каналу RTSP до конечного сервера. 

И в данном случае в поле `url` мы видим внутренний, локальный путь до DVR-камеры с авторизацией администратора:
```
admin:Rp8aXaUTlsZSb
```
Далее происходит авторизация в админ-панели Flussonic и осуществляется полный контроль за кластером, уже приватных камер:

![Flussonic-admin](img/Flussonic-admin1.jpg "Целый кластер камер Flussonic")

## Решение

Необходимо использовать внутренний, безопасный механизм доступа к публичным камерам [embed.html](https://flussonic.com/doc/insert-video-embed-on-website/). Он обеспечивает односторонний механизм внедрения на сервис (через iframe).



