# HighLoad сервис музыкального стриминга

## Часть 1. Тема и целевая аудитория

### Тема курсовой работы - **"Проектирование высоконагруженного музыкального стримингового сервиса"**
Примером для разработки послужили одни из главных музыкальных стриминговых сервисов  - [VK Музыка](https://music.vk.com/) и [Яндекс.Музыка](https://music.yandex.com/)

### Ключевой функционал сервиса, необходимый для MVP
- Регистрация и авторизация
- Создание страницы автора
- Загрузка треков и создание плейлистов со стороны пользователя
- Разбиение треков по жанрам и другим вспомогательным категориям
- Стриминг аудио

### Ключевые продуктовые решения
- Доступность части основного функционала сервиса без интернета
- Поддержка высококачественного аудио
- Сбор статистики по прослушиваниям для исполниетелей
- Создание персональных рекомендаций для пользователя

### Целевая аудитория сервиса
- 44 миллиона [**MAU**](https://vk.company/ru/investors/ir_blog/11472/)
- 14 миллионов [**DAU**](https://vk.company/ru/investors/ir_blog/11472/)
- Средний пользователь проводит на сервисе около [4-х часов](https://er10.kz/obzory/statistika-jandeksa-v-2022-godu)
- На возрастную группу 18-24 года приходится [45%](https://marketsplash.com/music-streaming-statistics/#:~:text=age%20group%20represents-,45,-%25%20of%20all%20music) всех пользователей сервисов потоковой музыки

## Часть 2. Расчет нагрузки

### Аудитория и объём данных
- Среднее время прослушивания в месяц - [**27 часов**](https://thegirl.ru/articles/v-yandeks-muzyke-poschitali-chi-treki-v-rossii-slushali-bolshe-vsego-v-2022-godu/), исходя из этого:
- Среднее время прослушивания за день - **54 минуты**
- Количество треков на платформе - [**60 миллионов**](https://re-store.ru/blog/sravneniya/music-services-in-russia/)
- **1 час** музыки в среднем весит примерно [**115 МБайт**](https://yandex.ru/support/music-app-android/offline/save-music.html), следовательно **1 минута** - **2 МБайта**
- **Длительность** среднего трека - **3 минуты**
- Средний **вес аудио** ~ **6КБ**
- Средний **вес обложки** альбома ~150 КБайт
- Песня в плейлисте(маленькая картинка + название) ~ 9 Кб.
- Конкретная песня(большая картинка + название + вес аудио) ~ 6200 Кб
  
### Рассчёт RPS
Представим стандартный кейс использования музыкального сервиса:
- Открытие своего списка песен, подгрузка ~20 названий и картинок (10 раз)
- Открытие конкретного плейлиста, в среднем - 30 песен(3 раза)
- Открытие альбома, в среднем - 12 песен (5 раз)
- Поиск песни, в среднем - выдача 10 треков (5 раз)
- Загрузка трека для прослушивания (54/3 = 18 раз)

Перемножив всё на количество пользоваталей в день, получаем **средний RPS** = (7+3+5+5+18)*14000000/86400 = **6158**.
Возьмём увеличение нагрузки в пиках до значений, в два раза превышающих стандартные, тогда получаем ~ **12300 RPS в пике**

### Рассчёт сетевой нагрузки

**Трафик на выгрузку**
Исходя из приведённых выше данных посчитаем нагрузку на сеть:
- Получение списка песен: 20 * 10 * 9КБ  = 1800 КБайт
- Открытие плейлиста(большая обложка + треки):  3 * (30 * 9КБ + 150КБ) = 1260 КБайт
- Открытие альбома(большая обложка + треки): 5 * (12 * 9КБ + 150КБ) = 1290 КБайт
- Поиск песни: 5 * 10 * 9КБ = 450 КБайт
- Загрузка треков: 18 * 6200КБ = 111 МБайт

Перемножим на ежедневное количество пользователей, получим **нагрузку на сеть**:
(111600+450+1290+1260+1800)*14000000 = **1629600 ГБайт** = **13036800 ГБит** или **150ГБит/секунду**
Соответсвенно, **Пиковая нагрузка** = **300ГБит/секунду**

**Трафик на загрузку**
Ежедневно на платформу будут загружать [20000](https://xmldatafeed.com/spotify-stats-2022-fakty-dannye-polzovateli-ispolzovanie-i-chasy-proslushivaniya/#Skolko_novyh_pesen_ezednevno_zagruzaetsa_na_Spotify) новых треков
Возьмём в рассчёт, что примерно 80% треков имеют разные обложки, тогда получим ежедневный объём данных на загрузку:
20000 * (6КБ + 150КБ * 0.8) = 2520000КБ = **2,52 ГБайта или 29КБайт/секунду**, или **60КБайт/секунду в пике**

Получим таблицу:
### Технические метрики
|          | RPS  |  Пиковый RPS | Нагрузка на сеть | Пиковая нагрузка на сеть |
|----------|------|--------------|------------------|--------------------------|
| Выгрузка | **6158** | **12 316**       | **150ГБит/секунду**  | **300ГБит/секунду**          |
| Загрузка | **0.1** | **0.2**          | **29КБайт/секунду**  | **58КБайт/секунду**         |
| Всего    | **6159** | **12 318**      | **151ГБит/секунду** | **302ГБит/секунду**        |

## Часть 3. Балансировка и расположение ЦОДов

### Расположение целевой аудитории

Целевая аудитория сервиса расположениа в России и странах СНГ. 
Взглянем на карту плотности населения Росиии:
![image](https://github.com/qqq4u/HL_Music/assets/44649392/9b63c08b-a173-47d0-90b4-631da7a7643d)

Исходя из неё, расположим наши ЦОДы в основных крупных городах, которые являются центрами административных округов, а именно:
* **Москва**
* **Санкт-Петербург**
* **Краснодар**
* **Красноярск**
* **Хабаровск**

Для балансировки будем использовать latency-based resolver Amazon Route 53(мб нельзя если не хостишься у них же) ИЛИ аналоги. Поскольку в нашей реализации некоторые ЦОДы находятся в одной DNS зоне, будем использовать BGP Anycast балансировку для того, чтобы резолвить IP-шники.
