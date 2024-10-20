# Проектирование высоконагруженных систем
## 1. Тема и целевая аудитория
### Тема: X

Информация о сервисе и нагрузке:
- DAU 217 миллионов пользователей. (в лекции было 436, взято из [2](https://www.meltwater.com/en/blog/twitter-stats-marketers-need-to-know))

| Гендер  | Процент (%) |
| --- | ------ |
| Мужчина | 56 |
| Женщина | 44 |



| Возраст (лет) | Процент пользователей (%) |
| --- | ------ |
| 13-17 |  6.6   |
| 18-24 |  17.1   |
| 25-34 |  38.5  |
| 35-49 |  20.7   |
| 50+  | 17.1   |

MVP функционал:
- Регистрация   
- Аккаунт: профиль, информация о профиле
- Фото и видео для профиля
- твиты, фото и видео в твитах, репосты твитов
- комментарии к твитам и комментарии к комментариям(в виде дерева)
- рекомендательная система твитов
- Настройки профиля
- Поиск: поиск по хэштегам
- Уведомления

Ключевой функционал - посты и поиск по ним.
Ключевые продуктовые решения:
- хэштеги: для поиска и группировки твитов по темам
- ретвиты

## 2. Расчет нагрузки

### Продуктовые метрики

| метрика | значение |
| --- | ------ |
| MAU |  550MM |
| DAU |  217MM |

| память на 1 | значение |
| --- | ------ |
| твит |  300 байт |
| фото | 16 кбайт |
| видео | 600 кБайт |


| действия пользователя (среднее) | значение | память |
| ---  | ---- |  --- |
| чтение твитов | 50 | - |
| лайк (твита и комментария) | 10 | 8 байт |
| поиск твита | 20 | - |
| написание комментария | 4 | 100 байт |
| написания твита | 2.4 | (расчитано ниже) |
| репост твита | 1.7 | - |

возьмём, что каждый 3 твит с видео, и в каждом твите в среднем есть 2 фотографии.

32 КБ + 200 Байт = 32.2 КБ (твит с 2 фото)
200 байт + 600 КБайт = 0.6002 МБ(твит с видео)
100 байт * 10 = 1000 байт ~ 1КБ (комментарии)

Средний размер хранилища пользователя - 532.176 КБ - пользователь/ сутки

### Технические метрики

хранения в разбивке по типам данных

| Данные | Кол-во в сутки | память в сутки (ГБ) |
| ---   |  -- |  ---              |
| твиты | 518.4 M |     145       |
| фото | 1 B |          15820     |
| видео | 172 M |       98420     |
| комментарии | 1.3 B | 81 |

| Сетевой трафик | среднее потребление | пиковое потребление |
| --- | ---- | --- |
| твиты | 0.001678  ГБ/c |  0.004195 ГБ/c  | 
| фото |  0.1831018 ГБ/c   |  0.4577545 ГБ/c  |
| видео | 1.13912037 ГБ/c   | 2.847800925  ГБ/c |

Суммарный суточный  трафик - 114466 ГБ/cутки

| RPS | средний | пик |
| ---  | ---- | --- |
| получение твита | 125787  | 314467 |
| лайк твита | 25157  | 62892 |
| поиск твита | 25574  | 62835 |
| написание комментария | 15000  | 37500 |
| написания твита | 6000 | 15000  | 
| репост твита | 4276  | 10691  |
| проверка авторизации + авторизация | 200000 | 250000 |
| изменение настроек профиля | 200 | 450 |
| подписка на / отписка от пользователя | 600 | 1500 |

## Глобальная балансировка нагрузки.

Разбиение по доменам.

x.com.static - домен для статического контента 
x.com.auth - домен для проверки регистрации/ авторизации
x.com.tweet - домен для получения твита
x.com.user - домен для получения странички пользователя - настроек



| Регион | % пользователей |
| --- | -- |
| Северная Америка | 35.62% |
| Азия | 46.20% |
| Европа | 11.90% |
| Южная Америка | 6.28% |


| Расположение ДЦ | назначение |
| --- | -- |
| Атланта (Джорджия) | основной (США) |
| Портленде (Орегон) | основной (США) |
| Даллас (Техас) |  основной (США) |
| Мумбаи (Индия) | основной (Азия) |
| Сингнапур | основной (Азия) |
| Дублин (Ирландия) | основной (Европа) |
| Нью-Йорк | CDN (США) |
| Сеул (Южная Корея) | CDN (Азия) |
| Токио (Япония) | CDN (Азия) |
| Франкфурт (Германия) | CDN (Европа) |
| Сан-Паулу (Бразилия) | CDN (Южная америка) |
| Сидней (Австралия) | CDN (Океания) | 


Такое расположение основных датацентров, потому что наибольшая часть аудитории "X" находится в США и Азии, после идёт Европа и Южная Америка. Такое расположениец ДЦ позволит ускорить доступ к сервису для большей части аудитории, а CDN ускорит отдачу статического контента в удаленных регионах и уменьшит нагрузку на основные ДЦ.

| Датацентр                              | % нагрузки    | 
| ---                                    | ----          |
| Атланта(Джорджия)                      | 14            | 
| Портленд (Орегон)                      | 14	         |
| Даллас(Техас)                          | 14	         |
| Сингнапур                              | 23.1          |
| Мумбаи (Индия)                         | 23.1          |
| Дублин (Ирландия)                      | 11.8          |


DNS балансировка - Latency-based DNS.

Latency-based DNS направляет запросы на сервер с наименьшей задержкой, постоянно измеряя время отклика.
Хорошо подходит, так как сеть глобально распределённая и сконцентрирована в некоторых странах особенно.
DNS сервера переодически проходятся по списку доступных серверов и определяют сервер с наименьшей задержкой и направляют запросы на него.

Anycast балансировка - BGP Anycast.
Отдельные IP для: США, Японии, Сингапура, Индии, Европы.
Будем использовать для устойчивости к сбоям, чтобы в случае если сервер выходит из строя запросы перенаправлялись на ближайший работающий. 

### Локальная балансировка нагрузки

Для входящего трафика будем использовать L4 балансировку - Virtual Server via Direct Routing. Есть ограничение, что сервера должны быть в одной физической сети, т.е. в рамках одного ДЦ. В каждом основном ДЦ
будет стоять балансировщик который будет распределять запросы в рамках датацентра. Балансировщики будут настроены с резервированием по протоколу CARP (Common Address Redundancy Protocol), что обеспечит отказоустойчивость и автоматическое переключение на резервный балансировщик в случае отказа основного.


С L4 балансировщика трафик будет попадать на L7 балансировщик. 


Будем использовать в качестве L7 балансировщика Nginx, чтобы предотвратить блокировку серверных ресурсов от "медленных пользователей", также на уровне Nginx добавим проверку авторизации, что позволит отсеивать неавторизованные запросы ещё до их попадания в бекенд.
В случае если бекенд упадет во время запроса, который не меняет БД, Nginx это поймет и перенаправит запрос на другой сервер, пользователь ничего не заметит, ещё уменьшаем нагрузку на бекенд, так как держим постоянные  соединения(upstreams) с бекендом.

Межсервисные запросы: используем Envoy, Least Connections для L7 балансировки межсервисных запросов в рамках ДЦ.

Отказоустойчивость: Будем использовать 2 кластера в Kubernetes с Autoscaling, что позволит в случае сбоя одного из кластеров обеспечить доступность к сервису, а autoscaling позволит увеличивать/уменьшать количество реплик в зависимости от нагрузки.



### Логическая схема БД

```mermaid
erDiagram    
    User {
        BIGINT user_id PK
        TEXT username
        TEXT password_hash
        VARCHAR(32) password_salt
        TEXT email UK
        TEXT path_to_avatar
        TEXT bio
        JSON settings
        TIMESTAMPZ updated_at
        TIMESTAMPZ created_at
        DATE birthdate
        ENUM status
    }

    Tweet {
        BIGINT tweet_id PK,FK
        BIGINT user_id FK
        BIGINT parent_id
        TEXT content
        JSON media
        VARCHAR(30)[] hashtag
        TIMESTAMPZ updated_at
        TIMESTAMPZ created_at
        ENUM status
    }

    Tweet |{--|| User : user_tweets

    TweetStat {
        BIGINT tweet_id PK,FK
        INT view_count  
        INT like_count
        INT retweet_count
        INT status
    }

    Tweet ||--|| TweetStat : tweet_statistics

    Comment {
        BIGINT comment_id PK
        BIGINT tweet_id FK
        BIGINT user_id FK
        TEXT content
        JSON media
        BIGINT parent_comment_id
        TIMESTAMPZ updated_at
        TIMESTAMPZ created_at
        ENUM status
    }
    Comment ||--|| Tweet : tweet_comments
    Comment ||--|| User : user_comments

    TweetHashtag {
        BIGINT hashtag_id FK
        BIGINT tweet_id
    }
    TweetHashtag ||--|| Tweet : tweet_hashtags

    Session {
        BIGINT session_id PK
        BIGINT user_id FK
        TIMESTAMPZ created_at
        TIMESTAMPZ expire_time
        ENUM status
    }

    MediaContent {
        BIGINT user_id PK
        binary data
    }
    
    MediaContent |{--|| Tweet : media_content
    MediaContent |{--|| Comment : media_content

    Session ||--|| User : user_sessions
```

### Физическая схема БД

![Схема бд](/DB.jpg)



### Индексы и Шардирование




### Шардирование
Таблица User:

• Индексы:
  * user_id (PRIMARY KEY) - обязательный индекс.

• Шардирование: 
  *  User по user_id. Например, разделить пользователей по диапазонам.

Таблица Tweet:

• Индексы:
  * tweet_id (PRIMARY KEY) - обязательный индекс.
  * user_id - для быстрой выборки твитов пользователя.
  * created_at - для эффективной сортировки и фильтрации по дате создания.
  * parent_id - для быстрого поиска ответов на твиты.

• Шардирование:
  * По tweet_id.

Таблица TweetStat:

• Индексы:
  * tweet_id (PRIMARY KEY) - обязательный индекс.

• Шардирование:
  * По tweet_id.

Таблица Comment:

• Индексы:
  * comment_id (PRIMARY KEY) - обязательный индекс.
  * tweet_id - для быстрого поиска комментариев к твиту.
  * parent_comment_id - для быстрого поиска ответов на комментарии.
• Шардирование:
  * Можно шардировать по comment_id по диапазонам.

Таблица TweetHashtag:

• Индексы:
  * hashtag_id, tweet_id - для быстрого поиска твитов по хештегу. 

• Шардирование:
  * По hashtag_id. 

Таблица Session:

• Индексы:
  * session_id (PRIMARY KEY) - обязательный индекс.
  * user_id - для быстрого поиска сессии пользователя.

• Шардирование:
  * По session_id. 

Выбор базы данных:

• User, Tweet, TweetStat, Comment, TweetHashtag: 
  * SQL: 
    * PostgreSQL: Хорошо подходит для структурированных данных, поддерживает JSONB для settings и media в User, Tweet и Comment.
    * MySQL: Простой в использовании, поддерживает JSON, но менее функциональный, чем PostgreSQL.
  * NoSQL: 
    * MongoDB: Хорошо подходит для хранения данных с непредсказуемой структурой, например, settings, media.
    * Cassandra: Хорош для масштабируемости, но не очень удобен для сложных запросов.

• Session: 
  * Redis: Хороший выбор для хранения сессий, так как он предоставляет высокую скорость чтения и записи.

## Хранение в JSON файлах

• User.settings:
  * theme: "light" | "dark"
  * language: "en" | "ru" | ...
  * notifications: { "email": true, "push": false } 
• Tweet.media:
  * images: [ "path/to/image1.jpg", "path/to/image2.jpg" ] 
  * videos: [ { "url": "https://...", "thumbnail": "https://..." } ]
• Comment.media: 
  * images: ... (аналогично Tweet.media) 

## Дополнительные рекомендации

• Используйте индексы с умом. Слишком много индексов может замедлить вставку данных. 
• Для ускорения запросов с JSON данными используйте JSONB (в PostgreSQL) или JSON_EXTRACT (в MySQL). 
• Тестируйте разные варианты шардирования и выбирайте тот, который лучше всего подходит для вашего приложения. 
• Используйте кеширование, чтобы снизить нагрузку на базу данных.

### Список источников:
1. [X](https://x.com/ "сам твиттер")
2. [Statista](https://www.statista.com/statistics/242606/number-of-active-twitter-users-in-selected-countries/)
3. [Tridenstechnology](https://tridenstechnology.com/ru/c%D1%82%D0%B0%D1%82%D0%B8%D1%81%D1%82%D0%B8%D0%BA%D0%B0-%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D0%B5%D0%B9-twitter/)
4. [MeltWater](https://www.meltwater.com/en/blog/twitter-stats-marketers-need-to-know)
5. [wifitalents](https://wifitalents.com/statistic/twitter/ "немного отличается от предыдущих статистик")
6. [quora](https://www.quora.com/Does-Twitter-store-its-data-and-if-so-where-and-how-is-it-organized#:~:text=By%20means%20of%20Hadoop%2C%20Twitter,Twitter%20is%20over%2010K%20nodes "немного инфы про RPS твиттера")
7. [oberlo](https://www.oberlo.com/statistics/number-of-twitter-users-by-country "распределение по странам")
8. [siliconangle](https://siliconangle.com/2022/12/30/twitter-reportedly-closes-sacramento-data-center-part-cost-cutting-initiative/ "где главные датацентры твиттера")
9. [servernews](https://servernews.ru/1077720 "где главные датацентры твиттера")
