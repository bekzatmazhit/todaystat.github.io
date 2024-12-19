Технический регламент
1. Введение

1.1. Термины и определения:

Термин Описание
ПС Платежная система Kaspi.kz
БИН Бизнес-идентификационный номер
API Интерфейс для взаимодействия между программами или системами
QR-токен Содержимое QR-кода (QR - картинки)
MPP Mobile Payment Provider (Участник)
MPP APP Мобильное приложение Участника

MPP Server Программно-аппаратная^ часть,^ отвечающая^ за^ функционирование^ внутренней^ части^
системы Участника

POS

Часть Технологической платформы, программно-аппаратный комплекс посредством
которого с использованием Средств платежа и соединения с информационной
системой Расчетного Банка и Участника совершается Операция

Static QR Неизменяемый QR код без ограничения по сроку жизни
Dynamic QR Динамически меняющийся QR код с ограниченным сроком жизни

IPSec
Безопасное VPN соединение между ПС и MPP. Является основным каналом связи
данных для выполнения операций/транзакций.

Покупатель
Клиент Участника (отправитель денег), приобретающий Товар (услугу) и
использующий Средство платежа

Продавец Юридическое лицо или индивидуальный предприниматель (мерчант и получатель
денег), реализующее Товар

Термины и определения, не указанные выше толкуются и применяются в значении, указанном в Договоре
участия, размещенном на сайте в сети Интернет по адресу: http://www.kaspipay.kz (далее - Договор).

Технической регламент является неотъемлемой частью Договора.
ПС предоставляет набор API для подключения к ПС, позволяющей осуществлять оплату по QR.

1.2. Оплата

    Определение QR-кода ПС на стороне MPP.
    Считывание и отправка QR-токена (содержимое QR-кода) в метод API scan. В ответ ПС
    возвращает детали оплаты для MPP.
    Отрисовка Покупателю экрана выбора способа и деталей оплаты в MPP APP.
    Подтверждение оплаты Покупателем, процессинг и отправка результата в метод API
    notifyPayment.
    1.3. Возврат
    Для предоставления возможности выполнения возврата покупки, MPP необходимо реализовать метод
    refund. При получении запроса на возврат в метод refund от ПС, MPP должен гарантировать возврат
    денег своему Покупателю.
    1.4. Описание API
    ПС предоставляет набор API, работающий по протоколу HTTP.
    ПС и MPP должны придерживаться принципов идемпотентных API. Идемпотентность помогает избежать
    нежелательного дублирования запросов, в случае различных отказов и повторов операций. Например,
    идемпотентность помогает гарантировать, что деньги Покупателя будут списаны только один раз, если
    один и тот же запрос (с одинаковыми аргументами) в API был выполнен несколько раз.
    1.5. Версионность

    Текущая версия V1. Указывается в URL
    https://{host}/api/v1/qr/scan

1.6. Формат передачи данных API

    Формат передачи данных: JSON
    Формат даты: ISO 8601 (YYYY-MM-DDThh:mm:ss±h). Пример: 2023 - 08 - 09T18:31:42+
    Форматы суммы: 2 знака после точки (100.00)
    1.7. Заголовки HTTP запроса
    Content-Type: application/json.
    X-Request-ID – уникальный идентификатор запроса
    ⚠ максимальная длина 64 символа
    X-Caller-Name – уникальный идентификатор MPP(БИН) присваиваемый ПС
    ⚠ максимальная длина 12 символа
    X-Locale – идентификатор используемого языка (ISO 639-1 и ISO 3166-1 alpha- 2 )
    ⚠ ru-RU или kk-KZ

1.8. Метод HTTP запроса

    Для выполнения HTTP запросов используется метод POST.
    1.9. Формат ответа от ПС
    Ответ содержит в себе информацию о выполненном запросе. Набор полей изменяется в зависимости от
    метода. Однако, каждый ответ обязательно содержит объект result, содержащий resultCode и
    resultMessage.

Если resultCode содержит значение SUCCESS – это означает успешное выполнение метода, иначе
данное поле будет содержать код ошибки (коды в описании API), а поле resultMessage описание ошибки.

Наименование Тип Описание

resultCode String

Код результата
⚠ максимальная длина 64 символа

resultMessage String
Сообщение о результате
⚠ максимальная длина 256 символов

1.10. Подключение к ПС
Для подключения к ПС используется IPSec туннель, ниже приведена типовая конфигурация туннеля IPSec
для интеграций.

VPN Gateway Device Information LLP Kaspi Pay Partner
Name / FQDN USG
IP Address x.x.x.x
VPN Device Description
Tunnel Properties LLP Kaspi Pay Partner

Phase 1

Authentication Method Preshared keys (via SMS) Preshared keys (via SMS)
Encryption Scheme IKEv2 KEv
PRF Algorithm SHA 256 SHA 256
Diffie-Hellman Group Group19 Group
Encryption Algorithm AES 256 AES 256
Hashing Algorithm SHA 256 SHA 256
Main or Aggressive Mode main main
Lifetime (for renegotiation) 86400 86400

Phase 2

Encapsulation (ESP or AH) ESP ESP
Encryption Algorithm AES256 AES
Authentication Algorithm Sha256 Sha
Perfect Forward Secrecy YES (Group19) YES (Group19)
Lifetime (for renegotiation) 28800 28800
Encryption networks (Internal) x.x.x.x
Lifesize in KB (for renegotiation) Not use Not use
Presharedkey Send over SMS Send over SMS

2. Процесс оплаты QR

2.1. Процесс оплаты MPP состоит из следующих шагов:

    Шаг 1, 2. Покупатель MPP сканирует QR код, предоставленный Продавцом
    Шаг 3. MPP APP получает QR токен
    Шаг 4, 5. Передача MPP APP, MPP Server - > QR токена + данные Покупателя (customerId) в scan
    Шаг 6,7. Ответ от scan (тип токена: Dynamic/Static/Custom paymentId, наименование торговой
    точки)
    Шаг 8. Если тип Static, требуется отобразить экран для ввода суммы
    Шаг 9. Покупатель вводит сумму
    Шаг 10. Если тип Custom, требуется отобразить экран для ввода дополнительных данных
    Шаг 11. Покупателя заполняет поля
    Шаг 12. Передача данных Покупателя + paymentId
    Шаг 13. Вызов метода checkout (подробнее пункт 4.2)
    Шаг 14. Обработка данных на стороне ПС
    Шаг 15, 16 Ответ от метода checkout (сумма к оплате)
    Шаг 1 7. Требуется отобразить экран выбора способа оплаты и кнопку подтверждения
    Шаг 1 8. Покупатель выбирает способ оплаты и подтверждает покупку
    Шаг 1 9. Подтверждение
    Шаг 20. MPP Server производит списание денег Покупателя
    Шаг 21. Запрос в notifyPayment (paymentId, customerId, +сумма если Static или Custom)
    Шаг 22 , 23. Ответ от notifyPayment (статус подтверждения) и linkReceipt (ссылка на товарный чек,
    необходимо отобразить Покупателю MPP)
    Шаг 24. Отображение страницы “Thank You Page””

3. Процесс возврата

3.1. Процесс возврата MPP состоит из следующих шагов (тип токена Dynamic/ Static):

    Шаг 1. POS генерирует возвратный QR
    Шаг 2, 3. Покупатель MPP сканирует возвратный QR-код
    Шаг 4. MPP APP получает QR-токен
    Шаг 5, 6. Передача MPP APP, MPP Server - > QR-токена + данные Покупателя (customerId) в scan
    Шаг 7. Ответ от scan (тип токена: Refund)
    Шаг 8. Инициация возврата ПС. Команда на отображение в POS списка покупок по данным
    Покупателя
    Шаг 9. Закрытие страницы сканирования
    Шаг 10. Отображение на POS всех покупок Покупателя и выбор покупки кассиром для возврата
    Шаг 11. Запрос от POS к ПС на возврат выбранной покупки
    Шаг 12. Запрос на инициацию возврата средств Покупателю MPP (⚠ при недоступности MPP
    Server API (Refund), на стороне Kaspi Pay реализован механизм retry)
    Шаг 1 3. Ответ MPP Server
    Шаг 14. Отображение страницы на POS “Thank You Page”
    Шаг 15. MPP Server производит гарантированный возврат денег Покупателю
    Шаг 16, 17 Процесс возврата денег Покупателю и его уведомление о возврате ( решение о
    реализации на стороне MPP)

⚠ Если оплата была произведена по типу токена Custom, то Покупатель должен обратится к Продавцу
(поставщику услуги) и запросить отмену платежа. Далее Продавец обращается к ПС для инициации
отмены. В случае успешной обработки отмены на стороне ПС, ПС вызывает метод MPP.Refund для
возврата денег Покупателю. По факту возврата денежных средств, MPP необходимо уведомить
Покупателя о возврате и зачислении денежных средств, посредством Push-уведомления.
4. API методы

4.1. Scan
[POST] /api/v1/qr/scan
Данный метод используется для расшифровки QR кода, когда Покупатель сканирует QR.

⚠ Метод scan возвращает значение payment.canConfirmUntil, содержащее крайнее время (до
наступления которого) вызова notifyPayment.

Параметры запроса :
qrCode string обязательное
Содержимое отсканированного QR-кода.
Максимальная длина 128 символов

customerId string обязательное
Индивидуальный идентификационный номер Покупателя MPP (уникальный номер Покупателя,
присвоенный Участником)
Максимальная длина 12 символов

Параметры ответа
result object обязательное
Результат бизнес-обработки

Содержит поля:
resultCode string обязательное
код результата
максимальная длина 64 символов

resultMessage string обязательное
сообщение о результате
максимальная длина 256 символов

qrCodeType string
Тип QR токена. Может иметь значение Dynamic, Static, Refund, Custom.
Данный параметр возвращается только при успешном resultCode = SUCCESS
Максимальная длина 16 символов

merchant object
Информация о продавце.
Данный параметр возвращается только при успешном resultCode = SUCCESS

Содержит поля:
merchantId string обязательное
идентификатор продавца
максимальная длина 32 символов

merchantName string обязательное
наименование продавца
максимальная длина 128 символов

merchantMCC string обязательное
МСС код продавца
максимальная длина 32 символов

payment object
Информация о платеже
Данный параметр возвращается только при успешном resultCode = SUCCESS и qrCodeType =
Dynamic/Static/Custom

Содержит поля:
paymentId string обязательное
идентификатор платежа в ПС
максимальная длина 64 символов

paymentAmount string
сумма платежа (при qrCodeType = Dynamic)

canConfirmUntil string обязательное
крайнее время для вызова notifyPayment

refund object
Информация о возврате
Данный параметр возвращается только при успешном resultCode = SUCCESS и qrCodeType = Refund

Содержит поля:
refundId string обязательное
идентификатор возврата в ПС
максимальная длина 64 символов

Custom
paymentData object
Содержит описание дополнительных полей

    если parameters.value не заполнены, требуется заполнить
    если в ответе получили parameters.options , требуется отобразить выпадающий список из
    элементов выпадающего списка и предоставить пользователю выбрать необходимый элемент из
    списка.

Содержит поля:
serviceId long обязательное
идентификатор сервиса в ПС

serviceName string обязательное
наименование сервиса в ПС

parameters array обязательное
Содержит массив параметров
Id long обязательное
идентификатор параметра

value string
значение параметра

name string обязательное
описание value на языке согласно локализации

options array
Содержит массив элементов выпадающего списка
key string обязательное
идентификатор элемента

label string обязательное
наименование элемента

description string
описание элемента

Пример (Dynamic)

Метод Запрос Ответ
Scan
(MPP -> KaspiPay)

{

"qrCode": "
592xxxxxxxxxxxxxxxxxxxxxxxxxxx6574344",
"customerId": " 9009 ********"
}

{

"result": {
"resultCode": "SUCCESS",
"resultMessage": "success"
},
"qrCodeType": "Dynamic",
"payment": {
"paymentId": "abc1234567890",
"paymentAmount": "100.00",
"canConfirmUntil":"2024- 02 -
19T15:57:41+06:00"
},
"merchant": {
"merchantId": "123456789",
"merchantName": "Coffee Bar",
"merchantMCC": "5812"
}
}

Пример (Custom)

Метод Запрос Ответ
Scan
(MPP -> KaspiPay)

{

"qrCode":
"https://kaspi.kz/pay/MyTest3Param",
"customerId": "9009********"
}

{

"qrCodeType": "Custom",
"payment": {
"paymentId": "379eb787-1a36- 4096 -
ac39-4f3006e26a19"
},
"paymentData": {
"serviceId": 2862,
"serviceName": "Парковка",
"parameters": [
{
"id": 3932,
"value": "",
"name": "Госномер"
}
]
},
"result": {
"resultCode": "SUCCESS",
"resultMessage": "success"
}
}

Пример ( Refund)

Метод Запрос Ответ
Scan
(MPP -> KaspiPay)

{

"qrCode":
" 71 2xxxxxxxxxxxxxxxxxxxxxxxxxxx6574344"
,
"customerId": "9009********"
}

{

"result": {
"resultCode": "SUCCESS",
"resultMessage": "success"
},
"qrCodeType": "Refund",
"refund": {
"refundId": "QR150895463"
}
}

Коды результата

Код Описание
SUCCESS Уведомление об успешной обработке запроса
CUSTOM_QR_ERROR Общая ошибка по QR типа custom
EXPIRED_CODE Срок действия QR-кода истёк
INVALID_TOKEN QR-токен не валидный
PROCESS_FAIL Произошел общий сбой бизнес-обработки. Не повторять попытку
INVALID_MERCHANT Общая ошибка по продавцу
PARAM_ILLEGAL Некорректный параметр запроса

UNKNOWN_EXCEPTION Непредвиденная ошибка
PAYMENTS_SUSPENDED Платежи Продавца временно приостановлены ПС
UNKNOWN_SOURCE Указан неверный X-Caller-Name

4.2. Checkout
[POST] /api/v1/qr/checkout
Данный метод используется для расчета суммы за услугу если тип токена Custom

⚠ Метод checkout возвращает значение canConfirmUntil, содержащее крайнее время (до наступления
которого) вызова notifyPayment.

⚠ При вызове метода checkout необходимо передать массив parameters с заполненными значениями.
В случае если в ответе от scan parameters.value было заполнено необходимо передать его с этим же
value.
В случае если parameters.value не было заполнено, необходимо передать значение, введенное или
выбранное пользователем из выпадающего списка (в зависимости наличия массива options на элементе)

Параметры запроса
PaymentId string обязательное
идентификатор платежа в ПС
максимальная длина 64 символов

Важно: в случае, если ПС получит несколько последовательных запросов в checkout от MPP с
одинаковым значением PaymentId, то ПС будет придерживаться принципа идемпотентности и будет
возвращать одинаковый ответ с одинаковым объектом result и amount

serviceId long обязательное
идентификатор сервиса в ПС

parameters array обязательное
Содержит массив параметров
Id long обязательное
идентификатор параметра

value string
значение параметра
максимальная длина 64 символов

Параметры ответа
result object обязательное
Результат бизнес-обработки
Содержит поля:
resultCode string обязательное
код результата
максимальная длина 64 символов

resultMessage string обязательное
сообщение о результате
максимальная длина 256 символов

paymentAmount string обязательное

сумма платежа

canConfirmUntil string обязательное
крайнее время для вызова notifyPayment

merchant object
Информация о продавце.
Данный параметр возвращается только при успешном resultCode = SUCCESS

Содержит поля:
merchantId string обязательное
идентификатор продавца
максимальная длина 32 символов

merchantName string обязательное
наименование продавца
максимальная длина 128 символов

merchantMCC string обязательное
МСС код продавца
максимальная длина 32 символов

Пример

Метод Запрос Ответ
Checkout
(MPP -> KaspiPay)

{

"paymentId": "abc1234567890",
"serviceId": 5301,
"parameters": [
{
"id": 8352,
"value":" 547 RBA02"
}
]
}

{

"paymentAmount": 1 00 ,
"canConfirmUntil": "2024- 10 -
31T16:26:29+05:00",
"merchant": {
"merchantName": "Парковка Test",
"merchantId": "2862",
"merchantMCC": "5732"
},
"result": {
"resultCode": "SUCCESS",
"resultMessage": "success"
}
}

Коды результата

Код Описание
SUCCESS Уведомление об успешной обработке запроса
PROCESS_FAIL Произошел общий сбой бизнес-обработки. Не повторять попытку
PARAM_ILLEGAL Некорректный параметр запроса
EXPIRED_CODE Срок действия кода истёк
UNKNOWN_EXCEPTION Непредвиденная ошибка
PAYMENTS_SUSPENDED Платежи Продавца временно приостановлены ПС
UNKNOWN_SOURCE Указан неверный X-Caller-Name

4.3. NotifyPayment
[POST] /api/v1/qr/notifyPayment
Данный метод используется для уведомления Kaspi Pay о результате процессинга покупки со стороны
MPP. В методе присутствует параметр paymentAmount, который должен быть заполнен при обработке
покупок qrCodeType = Static.

При успешном процессинге покупки, MPP должен отправить объект result c resultCode = SUCCESS. При
неуспешном процессинге должен использоваться соответствующий код.

⚠ Важно: MPP при вызове notifyPayment необходимо реализовать механизм retry. Данный механизм
должен автоматически повторять запросы к ПС в случае ее недоступности, до тех пор, пока не будет
получен ответ от ПС. Это позволит обеспечить успешное завершение операции при временных проблемах
с доступом к ПС. Т.к. API придерживается принципа идемпотентности, гарантируется атомарность
операции (повторное проведение операции с теми же реквизитами невозможно).

⚠ Важно: после сканирования QR кода, в случае когда клиент MPP в MPP APP нажал кнопку «Назад» (),
необходимо вызвать notifyPayment в объекте result требуется resultCode = CANCELLED_BY_CLIENT и
resultMessage = cancelled by client

Параметры запроса
result object обязательное
Результат бизнес-обработки

Содержит поля:
resultCode string обязательное
код результата
максимальная длина 64 символов

resultMessage string обязательное
сообщение о результате
максимальная длина 256 символов

payment object
Информация о платеже

Содержит поля:
paymentId string обязательное
идентификатор платежа в ПС, полученный в ответе метода scan
максимальная длина 64 символов

Важно: в случае, если ПС получит несколько последовательных запросов в notifyPayment от MPP, с
одинаковым значением paymentId, то ПС будет придерживаться принципа идемпотентности и будет
возвращать одинаковый ответ с одинаковым объектом result и resultCode

mppPaymentId string обязательное
идентификатор платежа в системе MPP
максимальная длина 64 символов

paymentAmount string
сумма платежа (при qrCodeType = Static/Custom)

customerId string обязательное
Индивидуальный идентификационный номер Покупателя MPP
Максимальная длина 12 символов

Параметры ответа
result object обязательное
Результат бизнес-обработки

Содержит поля:
resultCode string обязательное
код результата
максимальная длина 64 символов

resultMessage string обязательное
сообщение о результате
максимальная длина 256 символов

linkReceipt string (если resultCode SUCCESS)

Ссылка на товарный чек, необходимо отобразить Покупателю MPP (решение о реализации на стороне
MPP)

Пример

Метод Запрос Ответ
NotifyPayment
(MPP -> KaspiPay)

{

"result": {
"resultCode": "SUCCESS",
"resultMessage": "success"
},
"payment": {
"paymentId": "abc1234567890",
"mppPaymentId": "abcd12333",
"paymentAmount": "100.00"
},
"customerId": "9009********",
}

{

"result": {
"resultCode": "SUCCESS",
"resultMessage": "success"
},
"linkReceipt":
"https://receipt.kaspi.kz/ext?TranId=a3ffb
a1a-5add&sale_date=2024- 01 -
10+14:24:09.646000"
}

Коды результата

Код Описание
SUCCESS Уведомление об успешной обработке запроса
PROCESS_FAIL Произошел общий сбой бизнес-обработки. Не повторять попытку
CANCELLED_BY_CLIENT Отмена покупки клиентом MPP
INVALID_MERCHANT Общая ошибка по продавцу
PARAM_ILLEGAL Некорректный параметр запроса
UNKNOWN_EXCEPTION Непредвиденная ошибка
PAYMENTS_SUSPENDED Платежи Продавца временно приостановлены ПС
RISK_REJECT Запрос отклонен по причине контроля рисков
ORDER_NOT_EXIST Транзакция не найдена

PAYMENT_INCORRECT_STATUS
Несоответствующий статус оплаты для выполнения данной
операции
UNKNOWN_SOURCE Указан неверный X-Caller-Name
UNKNOWN_CUSTOMER Неверный customerId (отличается от переданного в scan)

4.4. Refund
[POST] /{host MPP}/refund
*Реализация метода на стороне MPP

ПС использует данный метод для инициирования возврата покупки. ПС поддерживает полный, частичный
и множественный частичный возврат.

Параметры запроса
customerId string обязательное
Индивидуальный идентификационный номер Покупателя MPP
Максимальная длина 12 символов

paymentId string обязательное
Идентификатор платежа в ПС
Максимальная длина 64 символов

refundAmount string обязательное
Сумма к возврату (ПС проверяет, что сумма всех возвратов не должна превышать сумму оригинальной
покупки)

refundId string обязательное
Идентификатор возврата в ПС

Максимальная длина 64 символов

linkReceipt string обязательное
Ссылка на чек, необходимо отобразить Покупателю MPP ( решение о реализации на стороне MPP)

Важно: в случае, если MPP получит несколько последовательных запросов в refund от ПС, с одинаковым
значением refundId, то MPP должен придерживаться принципа идемпотентности, а именно:

    выполнять возврат средств только 1 раз за серию одинаковых запросов
    возвращать одинаковый ответ с одинаковым объектом result и resultCode

Параметры ответа
result object обязательное
Результат бизнес-обработки

Содержит поля:
resultCode string обязательное
код результата
максимальная длина 64 символов

resultMessage string обязательное
сообщение о результате
максимальная длина 256 символов

Пример

Метод Запрос Ответ
Refund
(KaspiPay -> MPP)

{

"refundAmount": 100.00,
"customerId": "9009********",
"paymentId ": "abc1234567890",
"refundId ": "Ref1234567",
"linkReceipt":
"https://receipt.kaspi.kz/ext?TranId=a3ffba1a

    5add&sale_date=2024- 01
    10+14:24:09.646000"
    }

{

"result": {
"resultCode": "SUCCESS",
"resultMessage": "success"
}
}

Коды результата

Код Описание
SUCCESS Уведомление об успешной обработке запроса
PROCESS_FAIL Произошел общий сбой бизнес-обработки. Не повторять попытку
PARAM_ILLEGAL Некорректный параметр запроса
UNKNOWN_EXCEPTION Непредвиденная ошибка
PAYMENTS_SUSPENDED Платежи Продавца временно приостановлены ПС
REFUND_IN_PROCESS Возврат в процессе выполнения

5. Форматы Реестров и порядок передачи.

5.1. Ежедневный Реестр по подтвержденным Платежам содержит информацию по подтвержденным
Платежам и возвратам платежей, в том числе идентификационный номер Покупателя,
идентификационный номер операции MPP, идентификационный номер операции ПС, дату и время, тип
операции, сумму Платежей/возвратов и размер комиссии за Платежную услугу.

Пример:

Ежедневный реестр передается Участнику в формате Excel ежедневно в рабочие дни на электронный
адрес Участника, указанный в Заявлении и/или выгружается Участником самостоятельно через его
личный кабинет.

5.2. Ежемесячный Реестр по подтвержденным Платежам. Содержит информацию по подтвержденным
Платежам и возвратам платежей, в том числе идентификационный номер Покупателя,
идентификационный номер операции MPP, идентификационный номер операции ПС, дату и время, тип
операции, сумму Платежей/возвратов и размер комиссии за услугу «Сервис».

Пример:

Ежемесячный реестр передается Участнику в формате Excel в первый рабочий день, следующий за
отчетным месяцем на электронный адрес Участника, указанный в Заявлении и/или выгружается
Участником самостоятельно через его личный кабинет.