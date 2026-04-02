# Freegate -> MAX уведомления (Max Notify) 📹💬

Blueprint отправляет события **Frigate** в мессенджер **MAX** через интеграцию **[max-notify-ha](https://github.com/kai-zer-ru/max-notify-ha)**.

К уведомлениям можно прикреплять `thumbnail / snapshot / GIF / video` (в зависимости от выбранных параметров) и опционально отправлять сообщения при обновлениях события.

## Импорт в Home Assistant

[Import blueprint](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/kai-zer-ru/freegate_camera_notify_blueprint/blob/master/blueprint.yaml)

## Что нужно заранее ✅

1. Home Assistant с установленной интеграцией **Frigate**.
2. MQTT-обмен (в blueprint используется MQTT-триггер с топиком `frigate/reviews` по умолчанию).
3. Интеграция **Max Notify**:
  - установи `max-notify-ha`
  - настрой `notify.max_...` сущности для нужного чата/группы

## Быстрый старт 🔧

1. Создай автоматизацию по blueprint (лучше делать по одной автоматизации на набор камер).
2. В разделе `camera` выбери нужные камеры из Frigate.
3. Заполни `base_url` (обычно нужен, если ты используешь внешние URL для вложений).
4. Если нужна отправка клипа видео в MAX:
  - в параметре `video` выбери вариант `**Clip mp4`** (важно: для mp4 будет вызван `max_notify.send_video`).

## Настройка Max Notify (самое важное) 🎯

В blueprint есть блок `max_notify` с параметрами:

- `max_notify_entity` — выбери сущность формата `notify.max_...` из Max Notify.
- `max_send_initial` — отправлять уведомление в MAX на **первом** срабатывании события.
- `max_send_updates` — отправлять в MAX в цикле **обновлений** события (может быть полезно для “дозагрузки”/замены вложений в рамках длинного события).
- `max_send_genai` — отправлять в MAX для ветки `GenAI Summary`.
- `max_send_keyboard` — передаёт `send_keyboard` в Max Notify (обычно лучше `false`, т.к. кнопки blueprint не мапятся 1-в-1).

### `MAX count_requests` (чтобы видео успевало обработаться) 🎥

Параметр: `max_count_requests` (по умолчанию `25`).

Что делает:

- blueprint передаёт его как `count_requests` в сервисы:
  - `max_notify.send_video`
  - `max_notify.send_photo`

Зачем:

- интеграция Max Notify делает несколько попыток отправки, пока вложение не станет готовым (`attachment.not.ready`).
- если “в MAX не приходит видео”, чаще всего это решается увеличением `max_count_requests`.

Рекомендация:

- начни с `25`
- если всё равно не приходит, попробуй `30-40`

## Troubleshooting 🧯

1. В trace не видно вызовов `max_notify.send_video/send_photo`
  - проверь, что событие проходит фильтры blueprint: совпадает камера из Frigate и выбранные `Frigate Cameras`, severity/object/zone фильтры.
2. В trace видно `max_notify.send_video`, но видео не приходит
  - увеличь `max_count_requests` (например, до `30-40`)
  - убедись, что выбран именно `Clip mp4`, а не вариант `m3u8 (IOS)`
3. В Max приходит фото/сообщение, но не видео
  - это может быть из-за выбора типа вложения: если клип не считается `mp4`, blueprint отправит через `max_notify.send_photo`.

