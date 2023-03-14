# ШБР Лето 2023. Вступительное задание

Вы устроились в Яндекс, и вам нужно запустить сервис Яндекс Лавка.
Ваша первая задача — разработать новый REST API-сервис, который будет регистрировать курьеров, добавлять новые заказы и 
распределять их по курьерам.

## Требования к сервису

В сервисе должны быть реализованы:

1) REST API сервиса — подробнее в Задании 1;
2) расчет рейтинга курьеров — подробнее в Задании 2;
3) rate limiter для сервиса — подробнее в Задании 3;
4) алгоритм распределения заказов по курьерам — подробнее в Задании 4.

## Правила оценки работы

Задание 1 обязательно для оценки продуктовой задачи — если вы не выполните это задание, ваша работа будет считаться 
невыполненной и не будет проверяться.
Задания 2, 3 и 4 могут быть выполнены независимо и в любых вариациях — чем больше дополнительных заданий вы выполните, 
тем лучше может быть ваш результат.


## Задание 1. REST API

В качестве базовой функциональности сервиса необходимо реализовать 7 базовых методов.

Для всех методов в случае корректного ответа ожидается ответ `HTTP 200 OK`.

### POST /couriers
Для загрузки списка курьеров в систему запланирован описанный ниже интерфейс.

Обработчик принимает на вход список в формате json с данными о курьерах и графиком их работы.

Курьеры работают только в заранее определенных районах, а также различаются по типу: пеший, велокурьер и 
курьер на автомобиле. От типа зависит объем заказов, которые перевозит курьер.
Районы задаются целыми положительными числами. График работы задается списком строк формата `HH:MM-HH:MM`.

### GET /couriers/{courier_id}

Возвращает информацию о курьере.

### GET /couriers

Возвращает информацию о всех курьерах.

У метода есть параметры `offset` и `limit`, чтобы обеспечить постраничную выдачу.
Если:
* `offset` или `limit` не передаются, по умолчанию нужно считать, что `offset = 0`, `limit = 1`;
* офферов по заданным `offset` и `limit` не найдено, нужно возвращать пустой список `couriers`.

### POST /orders

Принимает на вход список с данными о заказах в формате json. У заказа отображаются характеристики — вес, район, 
время доставки и цена.

### GET /orders/{order_id}

Возвращает информацию о заказе по его идентификатору, а также дополнительную информацию: вес заказа, район доставки, 
промежутки времени, в которые удобно принять заказ.

### GET /orders

Возвращает информацию о всех заказах, а также их дополнительную информацию: вес заказа, район доставки, промежутки времени, в которые удобно принять заказ.

У метода есть параметры `offset` и `limit`, чтобы обеспечить постраничную выдачу.
Если:
* `offset` или `limit` не передаются, по умолчанию нужно считать, что `offset = 0`, `limit = 1`;
* офферов по заданным `offset` и `limit` не найдено, нужно возвращать пустой список `orders`.

### POST /orders/complete

Принимает три параметра: id курьера, id заказа и время выполнения заказа, после отмечает, что заказ выполнен.

Если заказ:
* не найден, был назначен на другого курьера или не назначен совсем — следует вернуть ошибку `HTTP 400 Bad Request`.
* выполнен успешно — следует выводить `HTTP 200 OK` и идентификатор завершенного заказа.

Обработчик должен быть идемпотентным.

## Задание 2. Рейтинг курьеров

Команда сервиса решила начать учет заработной платы и рейтинго курьеров.
Для этого необходимо реализовать новый метод `GET /couriers/meta-info/{courier_id}`.

Параметры метода:
* `start_date` - дата начала отсчета рейтинга
* `end_date` - дата конца отсчета рейтинга.

Примером значения параметров может быть `2023-01-20`. В задании можно полагаться на то, что все заказы и даты для 
расчетов имеют одну и ту же фиксированную временную зону - UTC.

Метод должен возвращать заработанные курьером деньги за заказы и его рейтинг.

**Заработок рассчитывается по формуле:**

Заработок рассчитывается как сумма оплаты за каждый завершенный развоз в период с `start_date` (включая) до 
`end_date` (исключая):

`sum = ∑(cost * C)`

`C`  — коэффициент, зависящий от типа курьера:
* пеший — 2
* велокурьер — 3
* авто — 4

Если курьер не завершил ни одного развоза, то рассчитывать и возвращать заработок не нужно.

**Рейтинг рассчитывается по формуле:**

Рейтинг рассчитывается следующим образом:
((число всех выполненных заказов с `start_date` по `end_date`) / (Количество часов между `start_date` и `end_date`)) * C
C - коэффициент, зависящий от типа курьера:
* пеший = 3
* велокурьер = 2
* авто - 1

Если курьер не завершил ни одного развоза, то рассчитывать и возвращать рейтинг не нужно.

## Задание 3. Rate limiter

Каждый большой сервис с API, открытым из интернета, должен ограничивать количество входящих запросов.
Для этого используется rate limiter. Вам нужно реализовать такое решение для разрабатываемого сервиса.

Для решения задачи можно написать собственную реализацию или использовать известное готовое решение.
Сервис должен ограничивать нагрузку в 10 RPS на каждую ручку. Если допустимое количество запросов превышено, сервис должен отвечать кодом 429.

## Задание 4. Распределение заказов

Сейчас сервис Яндекс Лавка для каждого курьера выбирает один заказ.
Это приводит к том, что курьер может быть не загружен или будет доставлять заказ по удаленному адресу.
Перед началом рабочей смены сервис распределяет заказы между курьерами для минимизации стоимости доставки.

Для этого нам понадобятся реализовать:
* метод распределения заказов `POST /orders/assign`. В общем виде он будет выглядеть так: перед началом рабочего дня 
берем список заказов и распределяем их по доступным курьерам
* метод получения уже распределенных заказов `GET /couriers/assignments`

Для распределения заказов между курьерами учитываются следующие параметры:
* вес заказа
* регион доставки
* стоимость доставки

**Вес заказов**

У каждой из категорий курьеров есть ограничение по весу перевозимого заказа и количества заказов.

| Тип курьера | Максимальный вес | Максимальное количество |
|---|---|---|
| пеший | 10 | 2 |
| велокурьер | 20 | 4 |
| авто | 40 | 7 |

**Регион доставки**

Тип используемого транспорта влияет на количество регионов, которые может посетить курьер при доставке заказов.

| Тип курьера | Количество регионов | Комментарий |
|---|---|---|
| пеший | 1 | Доставка осуществляется только в одном регионе |
| велокурьер | 2 | Доставка будет в двух регионах |
| авто | 3 | Можно выбрать 3 региона |

**Время доставки**

Время доставки складывается из посещения всех точек для вручения заказа в регионе и времени ожидания для вручения заказа.

Время посещения всех точек в одном регионе:

| Тип курьера | 1й заказ | Следующие заказы |
|---|---|---|
| пеший | 25 | 10 |
| велокурьер | 12 | 8 |
| авто | 8 | 4 |

При доставке товара в другом регионе, время рассчитывается также:

| Тип курьера | 1й заказ | Следующие заказы |
|---|---|---|
| велокурьер | 12 | 8 |
| авто | 8 | 4 |

Время доставки ограничено рабочим интервалов. Например, если курьер работает с 10-00 до 12-00 без использования 
транспорта, он может доставить 4 заказа без объединения:

| Время | Номер заказа |
|---|---|
| 10:00 | 1 |
| 10:25 | 2 |
| 10:50 | 3 |
| 11:15 | 4 |

И с объединением заказов, получится доставить больше заказов за то же время:

| Время | Номер заказа |
|---|---|
| 10:00 | [1, 2] |
| 10:35 | [3, 4] |
| 11:10 | [5, 6] |
| 11:45 | [7, 8] |

**Стоимость доставки**

Стоимость доставки при группировке заказов, расчитывается следующим образом:

| Тип курьера | 1й заказ | Следующие заказы |
|---|---|---|
| пеший | 100% | 80% |
| велокурьер | 100% | 80% |
| авто | 100% | 80% |

## Приложение. Как выполнить задание?

1. Сгенерировать ssh-ключ
2. Перейти в [профиль](https://school.yandex.ru/profile/)
3. Нажать кнопку "Редактировать профиль"
4. Ввести Публичный ключ и нажать кнопку "Сохранить"
5. Вернуться в задачу в LMS, в комментариях появятся ссылки на 2 репозитория:
    * с тестовым заданием
    * с шаблоном решения

6. Выполнить `git pull` репозитория с текстом задания, в нем можно найти более подробные инструкции по выполнению
7. Выполнить `git pull` репозитория с шаблоном решения.
В данном репозитории нужно выполнить задание, сделать `git commit` и отправить результат на сервер `git push`.
После того, как выполнен git push, нужно вернуться в LMS на страницу с заданием.
и нажать кнопку **Отправить ответ**
Обязательно напишите любой текст в поле ответа (например, "ОК"), иначе кнопка отправки будет недоступна.

## Приложение. Требования к решению.

1. Решение должно быть представлено в виде Dockerfile в котором происходит сборка, настройка и запуск решения;
2. Сервис должен обрабатывать входящие запросы на порту 8080;
3. На порту 5432 будет доступна БД PostgreSQL 15.2. Авторизация происходит по логину и паролю: user=postgres password=password;
4. В Dockerfile в репозитории с решением базовый образ строго зафиксирован. Изменять образ в директиве FROM нельзя. При этом можно добавлять в контейнер необходимые пакеты если это требуется.
