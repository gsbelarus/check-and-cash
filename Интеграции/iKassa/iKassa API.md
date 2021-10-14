# Документация по API iKassa
## Описание взаимодействия:
Сервис iKassa работает по протоколу TCP. Подключение к сервису осуществляется на порт 1829 (по умолчанию).

Одновременно к сервису iKassa может быть 1 подключение для взаимодействия с СКО (далее - токен).

Структура TCP пакета: `<размер пакета в байтах (int32)><сообщение в виде json (string)>`

Структура JSON сообщения:
```json
{
    "type": "$string",
    "address": "$string",
    "replyAddress": "$string?"
    "data": "$string?",
    "headers": "$map<string,string>"
}
```

Действие (action) указывается в "headers" поле с ключом "action"

Возможные типы сообщения:
- ping
- pong
- send
- publish
- error
- listen

После подключения, сервис iKassa периодически шлёт PING запросы

# Описание типов сообщения
## ping
Шлется сервисов для проверки соединения на случай непредвиденного разрыва. 

Шлется всегда на адресс `topics.default.ping`. В replyAddress передается адресс, на который ожидается сообщение pong

## pong
Должен слаться от клиентской части в случае, если от сервиса iKassa пришло сообщение с типом ping.

В случае, если в течении двух секунд ответ отсутствует, сервис iKassa закрывает данное соединение.

## send
Сообщение, которое может иметь тело. Будет доставлено лишь в один обработчик сообщений внутри сервиса iKassa

## publish
Сообщение, которое может иметь тело. Будет доставлено всем обработчикам событий внутри сервиса iKassa. Не имеет ответа

## error
Сообщение, уведомляющее об ошибке. Шлется на replyAddress сообщения, на которое возникла ошибка, либо на адрес `topics.default.error`, в случае если replyAddress не был указан

## listen
Сообщение, которое шлется от клиентской части, чтобы подписаться на события от сервиса iKassa

# Внутренние сервисы
Внутренний сервис - набор "действий", сгруппированных по назначению

# Список ошибок:
```kotlin
enum class ServiceErrorCode(
    val statusCode: Int,
    val code: Int,
    val msg: String = ""
) {
    CHEQUE_NOT_FOUND(404, 2000),
    TOKEN_NOT_FOUND_ERROR(404, 2010),
    WRONG_INPUT_ERROR(400, 2011),
    SALE_CHEQUE_NOT_FOUND_ERROR(404, 2012),
    SHIFT_CLOSED_ERROR(400, 2013),
    SALE_DISCOUNT_ERROR(400, 2014),
    INDEX_OUT_OF_RANGE_ERROR(400, 2015),
    CHANGE_ERROR(400, 2016),
    EMPTY_SALE_ERROR(400, 2017),
    SALE_NOT_PAYED_ERROR(400, 2018),
    UNAUTHORIZED_ERROR(400, 2019),
    UNSUPPORTED_CURRENCY_ERROR(400, 2022),
    UNKNOWN_ERROR(500, 2023),
    NOT_SECURE_ERROR(400, 2025),
    SALE_WAS_ROLLBACKED_ERROR(400, 2026),
    ZERO_SUM_ERROR(400, 2027),
    ZERO_QUANTITY_ERROR(400, 2028),
    REVOKED_KEY_ERROR(400, 2029),
    PRINTER_NOT_FOUND_ERROR(400, 2030),
    PRINTING_FAILURE_ERROR(500, 2031),
    WRONG_DISCOUNT_ERROR(400, 2032),
    INVALID_CODE_ERROR(400, 2033),
    TOO_MANY_ITEMS_ERROR(400, 2036),
    NOT_PRINTED_CHEQUE_NOT_FOUND_ERROR(404, 2038),
    NOT_ENOUGH_MONEY_IN_CASH_BOX_ERROR(400, 2039),
    EAN_ITEM_NOT_FOUND_ERROR(404, 2040),
    ACTION_ERROR(400, 2042),
    DELAY_ERROR(400, 2043),
    ACTIVE_CHEQUE_ERROR(400, 2044),
    ACTIVE_CHEQUE_NOT_FOUND_ERROR(404, 2045),
    INVALID_PIN_PUK_FORMAT(400, 2047, "Неверный формат pin/puk кода"),
    NAME_LENGTH_ERROR(400, 2048),
    CHANGE_SUM_ERROR(400, 2049),
    WRONG_CHEQUE_SUM_ERROR(400, 2050),
    SALE_ITEM_PRICE_ERROR(400, 2051),

    EMPTY_Z_REPORT(400,1_001, "Пусой Z-отчет. Требуется совершение операции"),
    OPERATION_IS_NOT_ALLOWED(403, 1_004, "Операция не может быть выполнена"),
    EMPTY_CASHIER_NAME(400, 1_005, "Имя кассира не может быть пустым"),

    INVALID_MONEY_FORMAT_ERROR(400, 1_105, "Неверный денежный формат"),
    INVALID_QUANTITY_FORMAT_ERROR(400, 1_106, "Неверный формат кол-ва"),

    AVQFR_INVALID_COMMAND(500, 5, "Подана команда с неизвестным кодом или нарушен порядок подачи команд"),
    AVQFR_UNSUPPORTED_VERSION(500, 6, "Версия протокола взаимодействия с устройством не поддерживается"),
    AVQFR_ACCESS_DENIED(403, 8, "Неверно указана роль при авторизации (отличная от PIN, PUK или REG)"),
    AVQFR_BAD_SERIAL_NUMBER(400, 9, "Неверно задан серийный номер устиойства"),
    AVQFR_BAD_DEVICE_MODE(400, 10, "Команда не может быть выполнена в текущем режиме устройства"),
    AVQFR_INTERNAL_ERROR(500, 11, "Внутренняя ошибка (возможная причина - аппаратный сбой)"),
    AVQFR_TIMEOUT(500, 12, "Превышено время ожидания ответа при синхронизации времени"),
    AVQFR_BAD_FW_UPGRADE_KEY(500, 13, "Отсутствует ключ обновления прошивки или его целостность нарушена"),
    AVQFR_NOT_MOUNTED(500, 101, "Файловая система на смонтирована (внутренняя ошибка, устройство неисправно)"),
    AVQFR_NOT_FORMATED(500, 102, "Носитель на отформатирован (внутренняя ошибка, устройство неисправно)"),
    AVQFR_INSUFFICIENT_SPACE(500, 103, "Недостаточно места для записи данных"),
    AVQFR_FILE_IS_TOO_BIG(500, 104, "Превышен размер файла (внутренняя ошибка, устройство неисправно)"),
    AVQFR_FILE_NOT_EXIST(500, 105, "Файл не найден, необходимые данные отсутствуют"),
    AVQFR_FILE_ALREADY_EXISTS(500, 106, "Файл уже имеется (внутренняя ошибка, устройство неисправно)"),
    AVQFR_INVALID_OFFSET(500, 107, "Неверное смещение (внутренняя ошибка, устройство неисправно)"),
    AVQFR_PROGRAM_ERROR(500, 108, "Ошибка записи во флеш-память (внутренняя ошибка, устройство неисправно)"),
    AVQFR_BAD_ADDRESS(500, 109, "Неверно задан адрес блока памяти (внутренняя ошибка, устройство неисправно)"),
    AVQFR_ERASE_ERROR(500, 110, "Ошибка стирания флеш-памяти (внутренняя ошибка, устройство неисправно)"),
    AVQFR_NO_CIPHER_CALLBACKS(500, 111, "Функции шифрования файловой системы недоступны (внутренняя ошибка, устройство неисправно)"),
    AVQFR_MAC_NOT_FOUND(500, 112, "У файла отсутствует имитовставка (внутренняя ошибка, устройство неисправно)"),
    AVQFR_NOT_ACTUAL_KEY(400, 17, "Cрок действия ключа не наступил или истек"),
    AVQFR_BAD_SIGN(500, 18, "Подпись не верна"),
    AVQFR_INCORRECT_PARAM_SIZE(500, 19, "Неверный размер параметра (один из параметров команды имеет неверную длину)"),
    AVQFR_BAD_SYNC_REQUEST_ID(500, 20, "Неверный идентификатор запроса синхронизации времени"),
    AVQFR_BAD_FILE_INDEX(500, 21, "Недопустимое имя файла (внутренняя ошибка, устройство неисправно)"),
    AVQFR_NO_DATA(404, 22, "Запрашиваемые данные отсутствуют (попытка получить внутренний или отсутствующий документ)"),
    AVQFR_INTEGRITY_ERROR(500, 24, "Нарушение целостности данных, хранящихся в устройстве (устройство неисправно)"),
    AVQFR_PRNG_NOT_INITIALIZED(500, 25, "Генератор СЧП не инициализирован, инициализация устройства не завершена"),
    AVQFR_BAD_SIGN_CTR(500, 26, "Неверное значение счетчика подписей"),
    AVQFR_BAD_STATUS_CODE(500, 27, "Неверное значение кода завершения обработки запроса сервером"),
    AVQFR_INVALID_MODE(500, 28, "Неверно задан режим алгоритма (внутренняя ошибка, устройство неисправно)"),
    AVQFR_INCORRECT_PARAM(500, 29, "Один из переданных параметров имеет недопустимое значение"),
    AVQFR_SELF_TEST_FAILURE(500, 30, "Ошибка самотестирования (устройство неисправно)"),
    AVQFR_SHIFT_IDLE_TIMEOUT(500, 31, "Превышено время бездействия, требуется синхронизация времени"),
    AVQFR_BAD_KEY_AUTH_DATA(403,32, "Неверные данные для авторизации сессии (неверное значение PIN, PUK или REG)"),
    AVQFR_INVALID_ATTR_ID(500,33, "Неверный идентификатор атрибута устройства (атрибут, не поддерживается устройством)"),
    AVQFR_BAD_KEY_TOKEN(500, 34, "Токен ключа сформирован некорректно"),
    AVQFR_FILE_BACKUP_ERROR(500, 35, "Ошибка дублирования файла (внутренняя ошибка, устройство неисправно)"),
    AVQFR_BAD_KEY_ID(500, 36, "Идентификатор ключа отсутствует или имеет неверный размер"),
    AVQFR_BAD_KEY(500, 37, "Ключ отсутствует, имеет неверный размер или неверный формат набора ключей"),
    AVQFR_ASN1_PARSE_ERROR(500, 38, "Невозможно разобрать ASN1-структуру (нарушена структура сертификата или ответа сервера)"),
    AVQFR_INCORRECT_DATA_SIZE(500, 39, "Неверный общий размер данных команды (передана команда неподдерживаемой длины)"),
    AVQFR_SESSIONS_LIMIT_EXCEEDED(500, 40, "Слишком много одновременно открытых сессий"),
    AVQFR_INVALID_SESSION_ID(500, 41, "Неверный идентификатор сессии"),
    AVQFR_SHIFT_IS_PENDING(400,42, "Необходимо закрыть смену"),
    AVQFR_RECEIPT_IS_PENDING(400,43, "В устройстве присутствуют кассовые документы, которые необходимо передать на сервер"),
    AVQFR_RECEIPT_SUM_OVERFLOW(400, 44, "Переполнение счетчиков, необходимо закрыть смену"),
    AVQFR_RECEIPT_NEGATIVE_VALUE(400, 45, "Задано отрицательное значение суммы"),
    AVQFR_NEGATIVE_SHIFT_BALANCE(400, 46, "Получен отрицательный сменный баланс"),
    AVQFR_SESSION_ALREADY_AUTHORIZED(400,47, "Сессия уже авторизована (попытка повторной авторизации уже авторизованной сессии)"),
    AVQFR_SESSION_NOT_AUTHORIZED(403, 48, "Сессия не авторизована (команда требует обязательной авторизации)"),
    AVQFR_SESSION_EXISTS(500, 49, "Имеется открытая сессия (выполнение команды возможно только в монопольном режиме)"),
    AVQFR_SHIFT_IS_OPENED(400, 50, "Смена открыта (команда возможна только при закрытой смене)"),
    AVQFR_SHIFT_IS_CLOSED(400, 51, "Смена закрыта (команда возможна только при открытой смене)"),
    AVQFR_BAD_DATE_TIME(400, 52, "Неверный формат или значение даты или времени (операция выполняется \"задним\" числом)"),
    AVQFR_BAD_CASHREG_NUMBER(400, 53, "Неверный номер кассового аппарата (номер КСА в СККО)"),
    AVQFR_BAD_CURRENCY_NAME(400, 54, "Неверное наименование кода валюты"),
    AVQFR_BAD_RECEIPT_TYPE(500, 55, "Неверный тип кассового документа"),
    AVQFR_BAD_RECEIPT_NUMBER(500, 56, "Номер кассового документа отличается от значения внутренего счетчика кассовых документов"),
    AVQFR_BAD_RECEIPT_COST(400, 57, "Рассогласование по полю \"Итого общая стоимость\""),
    AVQFR_BAD_RECEIPT_DISCOUNT(400, 58, "Рассогласование по полю \"Сумма скидки (надбавки)\""),
    AVQFR_BAD_RECEIPT_TOTAL(400, 59, "Рассогласование по полю \"Итого к оплате\""),
    AVQFR_BAD_RECEIPT_CENTS(500,60, "Неправильное значение дробной части денежного поля"),
    AVQFR_BAD_TAXPAYER_NUMBER(400, 61, "Задан неверный УНП"),
    AVQFR_BAD_CORRECTION_VALUE(500, 62, "Задана ненулевая сумма коррекции при отсутствии коррекций"),
    AVQFR_TOTAL_TRADE_OVERFLOW(400, 63, "Переполнение счетчика суммарного торгового оборота");
}
```

# Примечания
1. Максимальная длина имени кассира не должна превышать 16 символов
2. При добавлении товарной позиции через `EANEventBusService:updateItem` поле code должно соответствовать формату GTIN/EAN

# Перечень внутренних сервисов, а так же их "действий":
# AppEBService (ik.service.app)
## version (получение информации о версии сервиса iKassa)
Входные данные: null

Ответ: 
```json
{
    "version": "$string"
}
```

## printers (получение списка доступных принтеров)
Входные данные: null

Ответ: 
```json
[
    {
        "name": "$string",
        "addr": "$string"
    }
]
```

# PrintersController (ik.service.printers)
## print
Тело запроса представляет из себя набор байт закодированных в base64 строку 
Входные данные:
```json
$string
```

Требуемые заголовки (headers):
- printerAddr

Ответ: 
```json
{
    "result": "Ok"
}
```


# EANEventBusService (ik.service.ean)
## updateItem
Входные данные:
```json
{
    "code": "$long",
    "name": "$string",
    "price": "$decimal"
}
```

Ответ:
```json
{
    "result": "Ok"
}
```
## getItems
Входные данные:
null

Ответ:
```json
[
    {
        "code": "$long",
        "name": "$string",
        "price": "$decimal"
    }
]
```
## getItem
Входные данные:
Long (код товара)

Ответ:
```json
{
    "code": "$long",
    "name": "$string",
    "price": "$decimal"
}
```

# TokenAuthorityEBService (ik.service.token.authority)
## authorize
Входные данные:
```json
{
    "pin": "$string"
}
```

Требуемые заголовки (headers):
- token

Ответ:
```json
{
    "result": "Ok"
}
```

## changePinCode
Входные данные:
```json
{
    "pin": "$string",
    "puk": "$string"
}
```

Требуемые заголовки (headers):
- token

Ответ:
```json
{
    "result": "Ok"
}
```

## closeToken
Входные данные: null

Требуемые заголовки (headers):
- token

Ответ:
```json
{
    "result": "Ok"
}
```

## logout
Входные данные: null

Требуемые заголовки (headers):
- token

Ответ:
```json
{
    "result": "Ok"
}
```

# TokenDepositsEBService (ik.service.token.deposit)
## createDeposit

Входные данные: 
```json
{
    "sum": "$decimal",
    "header": {
        "cashier": "$string",
        "currency": "$string"
    }
}
```

Требуемые заголовки (headers):
- token
- printerAddr

Ответ:
```json
{
    "device": {
        "deviceId": "$long",
        "serialNumber": "$string"
    },
    
    "header": {
        "companyName": "$string",
        "companyUnp": "$int",
        "documentTypeName": "$string",
        "documentTypeId": "$int",
        "documentNumber": "$long",
        "currency": "$string",
        "created": "$datetime",
        "uid": "$string",
        "cashier": "$string",
        "shiftNumber": "$int"
    },
    
    "sum": "$decimal",
    
    "props": "$map<string, any>"
}
```

# TokenInformationEBService (ik.service.token)
## getTokens
Входные данные:
null

Ответ:
```json
[
    {
        "serial": "$string",
        "deviceId": "$string",
        "organization": "$string",
        "unp": "$string",
        "isRegistered": "$boolean",
        "name": "$string",
        "label": "$string",
        "manufacturer": "$string",
        "version": "$string",
        "pinCodeLength": "$int",
        "pukCodeLength": "$int",
        "type": "$string",
        "income": "$decimal?"
    }
]
```
## getTokenBySerial
Входные данные:
null

Требуемые заголовки:
- token

Ответ:
```json
{
    "serial": "$string",
    "deviceId": "$string",
    "organization": "$string",
    "unp": "$string",
    "isRegistered": "$boolean",
    "name": "$string",
    "label": "$string",
    "manufacturer": "$string",
    "version": "$string",
    "pinCodeLength": "$int",
    "pukCodeLength": "$int",
    "type": "$string",
    "income": "$decimal?"
}
```

## getCashInToken
Входные данные:
null

Требуемые заголовки:
- token

Ответ:
```json
[
    {
        "currency": "$string",
        "cash": $decimal
    }
]
```
## nextChequeNumber
Входные данные:
null

Требуемые заголовки:
- token

Ответ:
```json
$long
```
## getStoredDocuments
Входные данные:
null

Требуемые заголовки:
- token

Ответ:
```json
$int
```

## getCertificate
Входные данные:
null

Требуемые заголовки:
- token

Ответ:
```json
{
    "certificate": "$string"
}
```
Примечание: сертификат передается в виде base64 строки

## retrieveNotPrintedCheques
Входные данные:
null

Требуемые заголовки:
- token

Ответ:
```json
[ $string ]
```

## reprint
Входные данные:
uid ($string)

Требуемые заголовки:
- token
- printerAddr

Ответ:
```json
{
    "result": "Ok"
}
```

# TokenRetailMoneyBacksEBService (ik.service.token.moneyback)

## createMoneyBackEx
Входные данные:
```json
{
    "cashSum": "$decimal",
    "cashlessSum": "$decimal",
    
    "header": {
        "cashier": "$string",
        "currency": "$string"
    },
    
    "item": {
        "code": "$long",
        "codeType": "$int",
        "quantity": "$decimal",
        "price": "$decimal",
        "name": "$string"
    }
}
```

Требуемые заголовки (headers):
- token
- printerAddr

Ответ:
```json
{
    "device": {
        "deviceId": "$long",
        "serialNumber": "$string"
    },
    
    "header": {
        "companyName": "$string",
        "companyUnp": "$int",
        "documentTypeName": "$string",
        "documentTypeId": "$int",
        "documentNumber": "$long",
        "currency": "$string",
        "created": "$datetime",
        "uid": "$string",
        "cashier": "$string",
        "shiftNumber": "$int"
    },
    
    "totalCash": "$decimal",
    "totalCashless": "$decimal",
    
    "props": "$map<string, any>"
}
```

## createMoneyBack
Входные данные: 
```json
{
    "cashSum": "$decimal",
    "cashlessSum": "$decimal",
    
    "header": {
        "cashier": "$string",
        "currency": "$string"
    },
    
    "item": {
        "code": "$long",
        "quantity": "$decimal"
    }
}
```

Требуемые заголовки (headers):
- token
- printerAddr

Ответ:
```json
{
    "device": {
        "deviceId": "$long",
        "serialNumber": "$string"
    },
    
    "header": {
        "companyName": "$string",
        "companyUnp": "$int",
        "documentTypeName": "$string",
        "documentTypeId": "$int",
        "documentNumber": "$long",
        "currency": "$string",
        "created": "$datetime",
        "uid": "$string",
        "cashier": "$string",
        "shiftNumber": "$int"
    },
    
    "totalCash": "$decimal",
    "totalCashless": "$decimal",
    
    "props": "$map<string, any>"
}
```

# TokenRollbacksEBService (ik.service.token.rollback)
## createRollback
Входные данные: 
```json
{
    "header": {
        "cashier": "$string",
        "currency": "$string"
    },
    
    "chequeNumber": "$long"
}
```

Требуемые заголовки (headers):
- token
- printerAddr

Ответ:
```json
{
    "device": {
        "deviceId": "$long",
        "serialNumber": "$string"
    },
    
    "header": {
        "companyName": "$string",
        "companyUnp": "$int",
        "documentTypeName": "$string",
        "documentTypeId": "$int",
        "documentNumber": "$long",
        "currency": "$string",
        "created": "$datetime",
        "uid": "$string",
        "cashier": "$string",
        "shiftNumber": "$int"
    },
    
    "target": {
        "device": {
            "deviceId": "$long",
            "serialNumber": "$string"
        },
        
        "header": {
            "companyName": "$string",
            "companyUnp": "$int",
            "documentTypeName": "$string",
            "documentTypeId": "$int",
            "documentNumber": "$long",
            "currency": "$string",
            "created": "$datetime",
            "uid": "$string",
            "cashier": "$string",
            "shiftNumber": "$int"
        },
        
        "items": [
            {
                "codeType": "$int",
                "code": "$long",
                "name": "$string",
                "quantity": "$decimal",
                "discount": "$decimal",
                "itemPrice": "$decimal",
                "totalPrice": "$decimal",
                "resultPrice": "$decimal"
            }
        ],
        
        "subTotals": {
            "sum": "$decimal",
            "positionsDiscount": "$decimal",
            "chequeDiscount": "$decimal"
        },
        
        "totals": {
            "discount": "$decimal",
            "sum": "$decimal",
            "cash": "$decimal",
            "cashless": "$decimal",
            "other": "$decimal",
            "change": "$decimal"
        },
        
        "removedItems": [
            {
                "codeType": "$int",
                "code": "$long",
                "name": "$string",
                "quantity": "$decimal",
                "discount": "$decimal",
                "itemPrice": "$decimal",
                "totalPrice": "$decimal",
                "resultPrice": "$decimal"
            }
        ],
        
        "isClosed": "$boolean",
        "wasRollbacked": "$boolean",
        "props": "$map<string, any>"
    }
    
    "props": "$map<string, any>"
}
```

# TokenShiftEBService (ik.service.token.shift)
## getXReport
Входные данные:
null

Требуемые заголовки:
- token

Ответ:
```json
{
    "companyName": "$string",
    "companyUnp": "$long",
    "documentTypeName": "$string",
    "documentTypeId": "$string",
    "shiftNumber": "$long",
    "openDate": "$datetime",
    "closed": "$datetime?",
    "firstSaleDocumentNumber": "$long",
    "lastSaleDocumentNumber": "$long",
    "currencies": [ $string ],
    "saleDocumentsCount": "$long",
    "baseCurrencySum": "$decimal",
    "currencyReports": [
        {
            "currency": "$string",
            "salesCount": "$long",
            "salesCash": "$decimal",
            "salesCashless": "$decimal",
            "salesOther": "$decimal",
            "salesSum": "$decimal",
            "moneyBacksCount": "$long",
            "moneyBacksSum": "$decimal",
            "depositsCount": "$long",
            "depositsSum": "$decimal",
            "withdrawsCount": "$long",
            "withdrawsSum": "$decimal",
            "rollbacksCount": "$long",
            "rollbacksSum": "$decimal",
            "cancelationsCount": "$long",
            "cancelationsSum": "$decimal",
            "correctionsCount": "$long",
            "correctionsSum": "$decimal",
        }
    ],
    "deviceProperties": {
        "deviceId": "$long",
        "serial": "$string"
    },
    "props": "$map<string, any>"
}
```

## openShift
Входные данные:
null

Требуемые заголовки:
- token

Ответ:
```json
{
    "result": "Ok"
}
```

## closeShift
Входные данные:
null

Требуемые заголовки:
- token
- printerAddr


Ответ:
```json
{
    "uid": "$string"
    "companyName": "$string",
    "companyUnp": "$long",
    "documentTypeName": "$string",
    "documentTypeId": "$string",
    "shiftNumber": "$long",
    "openDate": "$datetime",
    "closed": "$datetime?",
    "firstSaleDocumentNumber": "$long",
    "lastSaleDocumentNumber": "$long",
    "currencies": [ $string ],
    "saleDocumentsCount": "$long",
    "baseCurrencySum": "$decimal",
    "currencyReports": [
        {
            "currency": "$string",
            "salesCount": "$long",
            "salesCash": "$decimal",
            "salesCashless": "$decimal",
            "salesOther": "$decimal",
            "salesSum": "$decimal",
            "moneyBacksCount": "$long",
            "moneyBacksSum": "$decimal",
            "depositsCount": "$long",
            "depositsSum": "$decimal",
            "withdrawsCount": "$long",
            "withdrawsSum": "$decimal",
            "rollbacksCount": "$long",
            "rollbacksSum": "$decimal",
            "cancelationsCount": "$long",
            "cancelationsSum": "$decimal",
            "correctionsCount": "$long",
            "correctionsSum": "$decimal",
        }
    ],
    "deviceProperties": {
        "deviceId": "$long",
        "serial": "$string"
    },
    "props": "$map<string, any>"
}
```

# TokenWithdrawEBService (ik.service.token.withdraw)
## createWithdraw
Входные данные: 
```json
{
    "sum": "$decimal",
    "header": {
        "cashier": "$string",
        "currency": "$string"
    }
}
```

Требуемые заголовки (headers):
- token
- printerAddr

Ответ:
```json
{
    "device": {
        "deviceId": "$long",
        "serialNumber": "$string"
    },
    
    "header": {
        "companyName": "$string",
        "companyUnp": "$int",
        "documentTypeName": "$string",
        "documentTypeId": "$int",
        "documentNumber": "$long",
        "currency": "$string",
        "created": "$datetime",
        "uid": "$string",
        "cashier": "$string",
        "shiftNumber": "$int"
    },
    
    "sum": "$decimal",
    
    "props": "$map<string, any>"
}
```

# TokenRetailSalesEBService (ik.service.token.sales.retail)

## getActiveSale
Входящие данные:
null

Требуемые заголовки:
- token

Ответ:
```json
{
    "saleId": "$long",
    "cheque": {
        "device": {
            "deviceId": "$long",
            "serialNumber": "$string"
        },
        
        "header": {
            "companyName": "$string",
            "companyUnp": "$int",
            "documentTypeName": "$string",
            "documentTypeId": "$int",
            "documentNumber": "$long",
            "currency": "$string",
            "created": "$datetime",
            "uid": "$string",
            "cashier": "$string",
            "shiftNumber": "$int"
        },
        
        "items": [
            {
                "codeType": "$int",
                "code": "$long",
                "name": "$string",
                "quantity": "$decimal",
                "discount": "$decimal",
                "itemPrice": "$decimal",
                "totalPrice": "$decimal",
                "resultPrice": "$decimal"
            }
        ],
        
        "subTotals": {
            "sum": "$decimal",
            "positionsDiscount": "$decimal",
            "chequeDiscount": "$decimal"
        },
        
        "totals": {
            "discount": "$decimal",
            "sum": "$decimal",
            "cash": "$decimal",
            "cashless": "$decimal",
            "other": "$decimal",
            "change": "$decimal"
        },
        
        "removedItems": [
            {
                "codeType": "$int",
                "code": "$long",
                "name": "$string",
                "quantity": "$decimal",
                "discount": "$decimal",
                "itemPrice": "$decimal",
                "totalPrice": "$decimal",
                "resultPrice": "$decimal"
            }
        ],
        
        "isClosed": "$boolean",
        "wasRollbacked": "$boolean",
        "props": "$map<string, any>"
    }
}
```

Примечание: ответ может быть null, если активная продажа отсутствует

## createSale
Входящие данные:
```json
{
    "cashier": "$string",
    "cheque_discount": "$decimal",
    "cash": "$decimal",
    "cashless": "$decimal",
    "other": "$decimal",
    "items": [
        {
            "code": "$long",
            "codeType": "$int",
            "quantity": "$decimal",
            "discount": "$decimal",
            "price": "$decimal",
            "name": "$string"
        }
    ]
}
```

Требуемые заголовки:
- token
- printerAddr

Ответ:
```json
{
    "device": {
        "deviceId": "$long",
        "serialNumber": "$string"
    },
    
    "header": {
        "companyName": "$string",
        "companyUnp": "$int",
        "documentTypeName": "$string",
        "documentTypeId": "$int",
        "documentNumber": "$long",
        "currency": "$string",
        "created": "$datetime",
        "uid": "$string",
        "cashier": "$string",
        "shiftNumber": "$int"
    },
    
    "items": [
        {
            "codeType": "$int",
            "code": "$long",
            "name": "$string",
            "quantity": "$decimal",
            "discount": "$decimal",
            "itemPrice": "$decimal",
            "totalPrice": "$decimal",
            "resultPrice": "$decimal"
        }
    ],
    
    "subTotals": {
        "sum": "$decimal",
        "positionsDiscount": "$decimal",
        "chequeDiscount": "$decimal"
    },
    
    "totals": {
        "discount": "$decimal",
        "sum": "$decimal",
        "cash": "$decimal",
        "cashless": "$decimal",
        "other": "$decimal",
        "change": "$decimal"
    },
    
    "removedItems": [
        {
            "codeType": "$int",
            "code": "$long",
            "name": "$string",
            "quantity": "$decimal",
            "discount": "$decimal",
            "itemPrice": "$decimal",
            "totalPrice": "$decimal",
            "resultPrice": "$decimal"
        }
    ],
    
    "isClosed": "$boolean",
    "wasRollbacked": "$boolean",
    "props": "$map<string, any>"
}
```

## startSale
Входящие данные:
```json
{
    "cashier": "$string",
    "currency": "$string"
}
```

Требуемые заголовки:
- token

Ответ:
```json
{
    "saleId": "$long",
    "cheque": {
        "device": {
            "deviceId": "$long",
            "serialNumber": "$string"
        },
        
        "header": {
            "companyName": "$string",
            "companyUnp": "$int",
            "documentTypeName": "$string",
            "documentTypeId": "$int",
            "documentNumber": "$long",
            "currency": "$string",
            "created": "$datetime",
            "uid": "$string",
            "cashier": "$string",
            "shiftNumber": "$int"
        },
        
        "items": [
            {
                "codeType": "$int",
                "code": "$long",
                "name": "$string",
                "quantity": "$decimal",
                "discount": "$decimal",
                "itemPrice": "$decimal",
                "totalPrice": "$decimal",
                "resultPrice": "$decimal"
            }
        ],
        
        "subTotals": {
            "sum": "$decimal",
            "positionsDiscount": "$decimal",
            "chequeDiscount": "$decimal"
        },
        
        "totals": {
            "discount": "$decimal",
            "sum": "$decimal",
            "cash": "$decimal",
            "cashless": "$decimal",
            "other": "$decimal",
            "change": "$decimal"
        },
        
        "removedItems": [
            {
                "codeType": "$int",
                "code": "$long",
                "name": "$string",
                "quantity": "$decimal",
                "discount": "$decimal",
                "itemPrice": "$decimal",
                "totalPrice": "$decimal",
                "resultPrice": "$decimal"
            }
        ],
        
        "isClosed": "$boolean",
        "wasRollbacked": "$boolean",
        "props": "$map<string, any>"
    }
}
```

## addItem
Входящие данные:
```json
{
    "code": "$long",
    "quantity": "$decimal",
    "discount": "$decimal"
}
```

Требуемые заголовки:
- token

Ответ:
```json
{
    "saleId": "$long",
    "cheque": {
        "device": {
            "deviceId": "$long",
            "serialNumber": "$string"
        },
        
        "header": {
            "companyName": "$string",
            "companyUnp": "$int",
            "documentTypeName": "$string",
            "documentTypeId": "$int",
            "documentNumber": "$long",
            "currency": "$string",
            "created": "$datetime",
            "uid": "$string",
            "cashier": "$string",
            "shiftNumber": "$int"
        },
        
        "items": [
            {
                "codeType": "$int",
                "code": "$long",
                "name": "$string",
                "quantity": "$decimal",
                "discount": "$decimal",
                "itemPrice": "$decimal",
                "totalPrice": "$decimal",
                "resultPrice": "$decimal"
            }
        ],
        
        "subTotals": {
            "sum": "$decimal",
            "positionsDiscount": "$decimal",
            "chequeDiscount": "$decimal"
        },
        
        "totals": {
            "discount": "$decimal",
            "sum": "$decimal",
            "cash": "$decimal",
            "cashless": "$decimal",
            "other": "$decimal",
            "change": "$decimal"
        },
        
        "removedItems": [
            {
                "codeType": "$int",
                "code": "$long",
                "name": "$string",
                "quantity": "$decimal",
                "discount": "$decimal",
                "itemPrice": "$decimal",
                "totalPrice": "$decimal",
                "resultPrice": "$decimal"
            }
        ],
        
        "isClosed": "$boolean",
        "wasRollbacked": "$boolean",
        "props": "$map<string, any>"
    }
}
```

## removeItem
Входящие данные:
```json
$int
```

Требуемые заголовки:
- token

Ответ:
```json
{
    "saleId": "$long",
    "cheque": {
        "device": {
            "deviceId": "$long",
            "serialNumber": "$string"
        },
        
        "header": {
            "companyName": "$string",
            "companyUnp": "$int",
            "documentTypeName": "$string",
            "documentTypeId": "$int",
            "documentNumber": "$long",
            "currency": "$string",
            "created": "$datetime",
            "uid": "$string",
            "cashier": "$string",
            "shiftNumber": "$int"
        },
        
        "items": [
            {
                "codeType": "$int",
                "code": "$long",
                "name": "$string",
                "quantity": "$decimal",
                "discount": "$decimal",
                "itemPrice": "$decimal",
                "totalPrice": "$decimal",
                "resultPrice": "$decimal"
            }
        ],
        
        "subTotals": {
            "sum": "$decimal",
            "positionsDiscount": "$decimal",
            "chequeDiscount": "$decimal"
        },
        
        "totals": {
            "discount": "$decimal",
            "sum": "$decimal",
            "cash": "$decimal",
            "cashless": "$decimal",
            "other": "$decimal",
            "change": "$decimal"
        },
        
        "removedItems": [
            {
                "codeType": "$int",
                "code": "$long",
                "name": "$string",
                "quantity": "$decimal",
                "discount": "$decimal",
                "itemPrice": "$decimal",
                "totalPrice": "$decimal",
                "resultPrice": "$decimal"
            }
        ],
        
        "isClosed": "$boolean",
        "wasRollbacked": "$boolean",
        "props": "$map<string, any>"
    }
}
```

## cancelSale
Входящие данные:
null

Требуемые заголовки:
- token

Ответ:
```json
{
    "result": "Ok"
}
```

## setChequeDiscount
Входящие данные:
```json
$decimal
```

Требуемые заголовки:
- token

Ответ:
```json
{
    "device": {
        "deviceId": "$long",
        "serialNumber": "$string"
    },
    
    "header": {
        "companyName": "$string",
        "companyUnp": "$int",
        "documentTypeName": "$string",
        "documentTypeId": "$int",
        "documentNumber": "$long",
        "currency": "$string",
        "created": "$datetime",
        "uid": "$string",
        "cashier": "$string",
        "shiftNumber": "$int"
    },
    
    "items": [
        {
            "codeType": "$int",
            "code": "$long",
            "name": "$string",
            "quantity": "$decimal",
            "discount": "$decimal",
            "itemPrice": "$decimal",
            "totalPrice": "$decimal",
            "resultPrice": "$decimal"
        }
    ],
    
    "subTotals": {
        "sum": "$decimal",
        "positionsDiscount": "$decimal",
        "chequeDiscount": "$decimal"
    },
    
    "totals": {
        "discount": "$decimal",
        "sum": "$decimal",
        "cash": "$decimal",
        "cashless": "$decimal",
        "other": "$decimal",
        "change": "$decimal"
    },
    
    "removedItems": [
        {
            "codeType": "$int",
            "code": "$long",
            "name": "$string",
            "quantity": "$decimal",
            "discount": "$decimal",
            "itemPrice": "$decimal",
            "totalPrice": "$decimal",
            "resultPrice": "$decimal"
        }
    ],
    
    "isClosed": "$boolean",
    "wasRollbacked": "$boolean",
    "props": "$map<string, any>"
}
```
## finalizeSale
Входящие данные:
```json
{
    "cash": "$decimal",
    "cashless": "$decimal",
    "other": "$decimal"
}
```

Требуемые заголовки:
- token
- printerAddr

Ответ:
```json
{
    "device": {
        "deviceId": "$long",
        "serialNumber": "$string"
    },
    
    "header": {
        "companyName": "$string",
        "companyUnp": "$int",
        "documentTypeName": "$string",
        "documentTypeId": "$int",
        "documentNumber": "$long",
        "currency": "$string",
        "created": "$datetime",
        "uid": "$string",
        "cashier": "$string",
        "shiftNumber": "$int"
    },
    
    "items": [
        {
            "codeType": "$int",
            "code": "$long",
            "name": "$string",
            "quantity": "$decimal",
            "discount": "$decimal",
            "itemPrice": "$decimal",
            "totalPrice": "$decimal",
            "resultPrice": "$decimal"
        }
    ],
    
    "subTotals": {
        "sum": "$decimal",
        "positionsDiscount": "$decimal",
        "chequeDiscount": "$decimal"
    },
    
    "totals": {
        "discount": "$decimal",
        "sum": "$decimal",
        "cash": "$decimal",
        "cashless": "$decimal",
        "other": "$decimal",
        "change": "$decimal"
    },
    
    "removedItems": [
        {
            "codeType": "$int",
            "code": "$long",
            "name": "$string",
            "quantity": "$decimal",
            "discount": "$decimal",
            "itemPrice": "$decimal",
            "totalPrice": "$decimal",
            "resultPrice": "$decimal"
        }
    ],
    
    "isClosed": "$boolean",
    "wasRollbacked": "$boolean",
    "props": "$map<string, any>"
}
```

## copy
Входящие данные:
```json
$long
```

Требуемые заголовки:
- token
- printerAddr

Ответ:
```json
{
    "device": {
        "deviceId": "$long",
        "serialNumber": "$string"
    },
    
    "header": {
        "companyName": "$string",
        "companyUnp": "$int",
        "documentTypeName": "$string",
        "documentTypeId": "$int",
        "documentNumber": "$long",
        "currency": "$string",
        "created": "$datetime",
        "uid": "$string",
        "cashier": "$string",
        "shiftNumber": "$int"
    },
    
    "items": [
        {
            "codeType": "$int",
            "code": "$long",
            "name": "$string",
            "quantity": "$decimal",
            "discount": "$decimal",
            "itemPrice": "$decimal",
            "totalPrice": "$decimal",
            "resultPrice": "$decimal"
        }
    ],
    
    "subTotals": {
        "sum": "$decimal",
        "positionsDiscount": "$decimal",
        "chequeDiscount": "$decimal"
    },
    
    "totals": {
        "discount": "$decimal",
        "sum": "$decimal",
        "cash": "$decimal",
        "cashless": "$decimal",
        "other": "$decimal",
        "change": "$decimal"
    },
    
    "removedItems": [
        {
            "codeType": "$int",
            "code": "$long",
            "name": "$string",
            "quantity": "$decimal",
            "discount": "$decimal",
            "itemPrice": "$decimal",
            "totalPrice": "$decimal",
            "resultPrice": "$decimal"
        }
    ],
    
    "isClosed": "$boolean",
    "wasRollbacked": "$boolean",
    "props": "$map<string, any>"
}
```
