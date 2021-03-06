# Описание протокола Tokenizer

## Ресурсы

- Тестовый: https://devpfront.a-3.ru/token/api/v1
- Продуктовый: https://pfront.a-3.ru/token/api/v1

## Запросы и формат данных

Метод: POST

Тип данных: JSON

Ресурс создания токена принимает POST-запрос и принимает тип содержимого "application/json", где указание типа содержимого запроса является обязательным.

**Данные запроса**

Формат: JSON.

| Название | Тип | Обязательно | Пример  | Описание
| -------- | --- | ----------- | ------- | -------
| systemId    | Строка | Да | TEST             | ID внешней системы
| cardNumber  | Строка | Да | 1234456732160002 | Номер карты – это индивидуальный номер именно вашей карты. У платежных систем Visa и MasterCard номер состоит из 16 цифр разделённых на 4 блока по 4 цифры (4-4-4-4). Иногда номер карты может иметь 18 или 19 цифр.
| cardCVV     | Строка | Да | 123  | Код проверки подлинности карты (CVV2 и CVC2)
| cardExp     | Строка | Да | 1805 | Срок действия карты - указывается последние две цифры года и месяц в цифровом формате - гг/мм (год/месяц).
| cardHolder  | Строка |    | CARD OWNER | Имя и фамилия держателя карты – указываются в латинской транскрипции.
| fingerprint | Строка |    | 
| orderId **  | Строка |    |  | ID операции во внешней системе (уникальное значение).
| userId **   | Строка |    |  | ID пользователя во внешней системе.

\* жирным отмечены обязательные данные

\*\* в запросе необходимо отправить хотя бы один из отмеченых параметров

**Данные ответа**

Формат: JSON.

| Название | Тип | Пример | Описание
| -------- | ---    | ----------- | ------- 
| response |  | | 
| token | Строка | | JWT
| status | | |
| code | Число | 0 | Статус в числовом представлении 
| message | Строка | Success | Описание статуса ответа

**Пример ответа**

```json
{
    "response": {
        "token": "<JWT>"
    },
    "status": {
        "code": 0,
        "message": "Success"
    }
}
```

## Работа с JWT

При создании JWT используется алгоритм HS256 с использованием секретного ключа. Секретный ключ - необходимо получить у A-3.

JWT содержит JSON о следующими полями:

| Название | Тип | Пример | Описание
| --- | --- | --- | ---
| systemId     | Строка | TEST | ID внешней системы
| card         | Строка | 5547********0002  | Маскированный номер карты
| uuid         | Строка | 8025c153-c939-4c81-9531-8aea040bda6d  | Системный UUID запроса
| createdDate  | Строка | 2018-07-23 10:28:36 |  Дата создания JWT
| expiredDate  | Строка | 2018-07-23 11:28:36 |  Срок действия JWT
| data         | Строка |  | Зашифрованные данные
| exp          | Число  | 1532345316 | Срок действия JWT в формате timestamp

## Статусы

| Код | Сообщение | Описание
| --- | --- | --- 
| 0   | Success | Запрос обработан.
| 1   | wrong content type | В запросе не указан Content-Type
| 2   | wrong data | Некорректный json или данные указаны неполностью
| 3   | required userId or orderId | Один из параметров не указан


## Примеры

Пример запроса

```bash
curl -X POST 'https://devpfront.a-3.ru/token/api/v1' \
-H 'content-type: application/json' \
-d '{"systemId": "TEST", "cardNumber": "5547598384220002", "cardCVV": "230", "cardExp":
"1906", "cardHolder": "CARD OWNER", "orderId": "1"}'
```

Ответ

```json
{
    "response": {
        "token": "<JWT>"
    },
    "status": {
        "code": 0,
        "message": "Success"
    }
}
```

Пример содержания JWT

```json
{
    "systemId": "TEST",
    "card": "5547********0002",
    "uuid": "8025c153-c939-4c81-9531-8aea040bda6d",
    "createdDate": "2018-07-23 10:28:36",
    "expiredDate": "2018-07-23 11:28:36",
    "data": "gOT1BnDbK/fx03C7...ST+M4N/udVM+u3s6pzudnnSDdJl+PTwk+ug==",
    "exp": 1532345316
}
```

## Автозаполнение карты

В случае, если пользователь уже совершал успешную оплату, то можно сделать запрос в сервис tokenizer
без указания полных данных карты, только указать параметр cardCVV и userId.

```sh
curl -X POST 'https://devpfront.a-3.ru/token/api/v1' \
-H 'content-type: application/json' \
-d '{"systemId": "TEST", "cardCVV": "230", "userId": "EXT_USER_ID_1", "orderId": "1"}'
```

**Возможные ошибки**

| Код | Описание
| -- | --
| 9  | Не указаны параметры userId и cardCVV
| 10 | Данные о карте для указанного пользователя не найдены, необходимо указать параметры cardNumber, cardExp, cardOwner
