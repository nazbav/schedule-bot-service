# schedule-service*

![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/nazbav/schedule-bot-service?style=for-the-badge)
![GitHub](https://img.shields.io/github/license/nazbav/schedule-bot-service?style=for-the-badge)
![GitHub issues](https://img.shields.io/github/issues/nazbav/schedule-bot-service?style=for-the-badge)
![PHP](https://img.shields.io/badge/PHP-7.3-green?style=for-the-badge)

Сервис (демон для автоматического уведомления о заменах в расписании)
Работает через стороннее API.
Сервис разделен на 2 сервиса.
1. `bot.php` -- сервис обработки команд пользователей, введенных через сообщения сообщества (+ в беседах)
2. `notifications.php` -- сервис автоматического поиска и уведомления о заменах
# Установка и запуск

Настройка под 18.04.3 (systemd, php-cli 7.3)

-1. Залить папку с сервисами на сервер.

0. Установить и настроить PHP 7.3, php-curl, php-json, php-mysqli, php-mbstring.

1. Заполнить `config.php`

2. Залить залить в папку `/etc/systemd/system/`, настроить `bot_vk.service` (указать корневую папку проекта)
2. Залить в папку `/etc/systemd/system/`, настроить `bot_notify.service` (указать корневую папку проекта)

3. Проверяем сервисы `systemctl -i status bot_vk` и `systemctl -i status bot_notify`

4. Автозапуск сервисов `systemctl enable bot_vk` и `systemctl enable bot_notify`

5. Запускаем сервисы `systemctl start bot_vk` и `systemctl start bot_notify`

6. Перезагрузка сервисов `systemctl daemon-reload` (когда необходимо)

# "Тесты"

``` Проект не тестировался больше 1-2 часов, могут быть ошибки, недоработки, неуместные усложнения и т.д. ```

1. Ubuntu 18.04.3 (тест в качестве сервиса systemd, php-cli 7.3, php-curl, php-json, php-mbstring, внешняя mysql база данных)

2. По приколу на DIR-320 NRU (8 FLASH\32 RAM)(своп+перенос корневой системы на usb-flash) OpenWrt 18.06.6 (тест в рамках ssh сессии, php-cli 7.3, php-curl, php-json, php-mbstring, внешняя mysql база данных)

# Краткий алгоритм работы поиска замен

1. Проверяем API на наличие изменений.

2. Если изменения найдены

2.1. Проверяем актуальность замен по полям *дата_вступления_замены* и *имя_замены* (проверяем что такой замены нет в базе, иначе -- игнорим

2.2. Считываем содержимое *.doc файла замены.

2.3. Ищем в заменах инициалы преподавателей и группы (по регуляркам).

2.4. Записываем все что нашли в базу данных.

3. Авто удаление  замен из базы

3.1. Ищем все устаревшие замены.

3.2. Чистим сохранённые теги для этих замен

3.3. Чистим информацию о заменах

4. Рассылка уведомлений об изменениях в расписании.

4.1. Выбираем по 20 заданий для уведомления (задания формируются во время исполнения этапа `п. 2.4`).

4.2. Определяем тип задания (ответить пользователю в вк, ответить на почту(не реализовано), и т.д.)

4.3. При отправке через ВК, формируем execute запрос. 

4.4. Удаляем задания из списка.

5. Возвращаемся к `п. 4.` и смотрим есть ли еще замены.

6. Определяем сколько времени нам осталось спать до следующего пробуждения.

7. Спим.

8. Просыпаемся, повторяем `п. 1.`
