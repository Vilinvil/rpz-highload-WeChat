**Ильинский Владислав**

[Телеграм для связи](https://t.me/Vilin0)

[Методические указания](https://github.com/init/highload/blob/main/homework_architecture.md)

## 1. Тема, аудитория, функционал

### Тема: мессенджер Whatsapp

### MVP функционал

1. Отправка сообщений в чате.
2. Просмотр чата.
3. Вложения фото, видео, файлы, голосовые.
4. Поиск сообщений по чатам.
5. Группировка чатов по папкам.
6. Групповой чат с несколькими пользователями.
7. Статус сообщения(отправлено, доставлено, прочитано)

**Ключевые особенности**

* Эмоции на сообщения в виде смайликов.

* Голосовые звонки внутри мессенджера

* Проектировать будем с учетом отсутствия постоянного хранения сообщений на серверах.
Т.е. сообщения могут храниться только пока адресат не доступен. 
После доставки сообщения удаляются с серверов.
Можно хранить метаданные о статусе доставке, времени отправки, получения и т.д.

### Продуктовые метрики

| Метрика                        | Значение |
|--------------------------------|:--------:|
| Аудитория                      | Весь мир |
| MAU                            | 482  млн |
| DAU                            | 193 млн  |
| Сообщений в день               | 25 млрд  |
| Голосовых сообщений в день     | 1.4 млрд |
| Голосовые звонки минут в день  | 200 млн  |

Я взял monthly active users(MAU) и количество сообщений в день
Whatsapp за 2022 с сайта [1] и поделил их на 5.

Daily active users(DAU) я взял с учетом сайтов [2] и [3]
приблизительно как 40% от MAU и поделил также на 5.

Т.к. распределение по количеству пользователей довольно равномерное по странам [4] и регионам [6]
не будем концентрироваться на определенный регион.

Количество голосовых сообщений в день взято с официального поста whatsapp и поделено на 5 [8]

Количество голосовых и видео звонков есть здесь [9]. Но т.к. там упоминается и видео,
и голосовых звонков поделим на 2 и как и везде еще поделил на 5. 

В WhatsApp 76% чатов между двумя людьми(групповых чатов 24%).
Но в то же время в групповых чатах генерируется 50% общего объема сообщений [5].
Примерно равномерно 75% всех сообщений отправлены с 12 по 24ч
Около 18% всех отправлено сообщений с 8ч до 12ч см. рисунок 1. [5]

![Распределение сообщений в течении дня](destribution_messages_per_time_of_day.png)

**Рисунок 1. Распределение сообщений в течении дня**

На рисунке 2 можно увидеть распределение аудитории по условным регионам, основанные на данных [4].
Сами расчеты посмотреть здесь [6].

![MAU по регионам](MAU_by_region.png)

**Рисунок 2.  Распределение MAU по условным регионам**


## 2. Расчет нагрузки

Согласно данным:

* Примерно 86% всех сообщений до 10 слов длинной [5].
* Только 2% сообщений используют вложения [5].
* 1 слово в английской в среднем состоит из 5 символов [7].
* 4 байта максимально занимает символ в UTF8
* Средний размер минуты голосового звонка 0.72 Мбайт [10].

Пусть средний размер текстового сообщения равным:
10 слов * 5 букв * 4 байта = 200 Байт

С учетом необходимости метаданных: времени отправки, статуса, id-ков и т.д.
примем размер одного сообщения в 1 Кбайт

Примем средний размер голосового в 1 мин и 1 Мбайт.

Особо данных по средним размерам вложений нет. Примем средний размер вложения 10 Мбайт.

### Рассчитаем объем трафика и необходимое хранилище. 

Из 25 млрд сообщений в день 2% c вложениями(фото, видео, документы, файлы).
И 1.4 млрд голосовых сообщений в день. 

25 млрд в день / 100 * 2 = 500 млн сообщений с вложениями в день.

Трафик сообщений с вложениями в день: 500 млн в день * 10 Мбайт = 4768 Тбайт в день 

Трафик сообщений с голосовыми в день: 1.4 млрд в день * 1 Мбайт = 1335 Тбайт в день

Трафик текстовых сообщений в день: (25 - 1.4 - 0.5) млрд в день * 1 Кбайт = 
22 Тбайт в день

Трафик голосовых звонков в день: 200 млн в день * 0.72 Мбайт = 137 Тбайт в день

Суммарно: 4768 + 1335 + 22 + 137 = 6262 Тбайт в день

С учетом того, что примерно 32% всех сообщений прочитывается за минуту.
50% сообщений прочитывается за час [5]. Будем считать, что хранить нам необходимо
только 60% общего объема в день.

Объем хранилища: (4768 + 1335 + 22) Тбайт в день * 0.6 = 3675 Тбайт в день.

| Метрика                       |     Значение     |
|-------------------------------|:----------------:|
| Трафик текстовых сообщений    |  22 Тбайт\день   |
| Трафик голосовых сообщений    | 1335 Тбайт\день  |
| Трафик сообщений с вложениями | 4768 Тбайт\день  |
| Трафик голосовых звонков      |  137 Тбайт\день  |
| Суммарный трафик              | 6262  Тбайт\день |
| Объем хранилища в день        | 3675 Тбайт\день  |


### Рассчитаем RPS

Кол-во запросов на отправку текстовых сообщений: (25 - 1.4 - 0.5) млрд в день /
/ (24 д * 60 мин * 60 с) = 267361 запросов/сек 

Кол-во запросов на отправку голосовых сообщений: 1.4 млрд в день / (24 д * 60 мин * 60 с) =
= 16204 запросов/сек

Кол-во запросов на отправку сообщений с вложениями: 1.4 млрд в день /
/ (24 д * 60 мин * 60 с) = 5787 запросов/сек

Примем среднее время звонка 10 мин согласно [11].
Тогда кол-во запросов на создание звонка: 200 млн мин в день \ 10 мин = 20 млн в день = 
= 20 млн в день / (24 д * 60 мин * 60 с) = 231 запрос в секунду


| Метрика                                |       Значение        |
|----------------------------------------|:---------------------:|
| Отправка текстовых сообщений           |  267361 запросов/сек  |
| Отправка голосовых сообщений           |  16204 запросов/сек   |
| Отправка  сообщений с вложениями       |   5787 запросов/сек   |
| Запросы на создание голосового звонка  |  231 запрос в секунду |


## Список литературы

1. Общая статистика Whatsapp https://www.bankmycell.com/blog/number-of-whatsapp-users/#1613581803631-96bfa-c1678a7f-e18553c4-2e8a2161-0689e002-e6d075e8-7b5a
2. Частота использования ежедневно в США https://www.statista.com/statistics/814813/frequency-with-which-us-internet-users-visit-whatsapp/
3. Количество ежедневных пользователей https://www.statista.com/statistics/730306/whatsapp-status-dau/
4. Распределение MAU по странам https://worldpopulationreview.com/country-rankings/whatsapp-users-by-country
5. Исследование 4 млн сообщений от 100 людей 2016г https://www.researchgate.net/publication/299487660_WhatsApp_Usage_Patterns_and_Prediction_Models
6. Расчеты MAU по условным регионам https://docs.google.com/spreadsheets/d/1bwwJy5Y3Objel3J5x4IXkPFhqaDwpeiLdYi_NVZZPw0/edit?usp=sharing
7. Исследование английских слов https://norvig.com/mayzner.html
8. Официальный пост Whatsapp о количестве голосовых сообщений https://blog.whatsapp.com/making-voice-messages-better
9. Количество голосовых звонков https://dataprot.net/statistics/whatsapp-statistics/#:~:text=3.-,Over%202%20billion%20minutes%20of%20voice%20and,are%20sent%20daily%20via%20WhatsApp.&text=There's%20more%20to%20WhatsApp%20than,the%20latest%20WhatsApp%20personal%20statistics.
10. Калькулятор голосовых звонков WhatsApp https://3roam.com/whatsapp-data-and-bandwidth-usage-calculator/
11. Среднее кол-во времени на один голосовой звонок https://www.wharftt.com/how-long-can-a-whatsapp-call-last/
