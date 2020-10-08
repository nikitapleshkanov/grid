
**Пример с сервисом Топливо(Турбо):**

Для создания заказа необходима витрина с типом KIT, каталог которой заранее был загружен и опубликован. Также в качестве precondition 
создания заказа требуется выполнить следующие запросы:

**1. Запросить список доступных товаров**

```
POST http://marketplace-uapi-kit-test.tcsbank.ru/kit/application/{appId}/goods

header:
{
   'content-type': 'application/json'
}

body:
{
  "commonProperties": {
    "interfaceType": "string",
    "platform": "string",
    "appName": "string",
    "appVersion": "string"
  },
  "adult": true,
  "clientId": "9-REGRESS107",
  "pageNumber": 0,
  "pageSize": 6
}


где:
appId - applicationId витрины с типом KIT
"commonProperties": Общие параметры запроса
"adult": Флаг: отображать или нет товары для взрослых
"clientId": Идентификатор клиента
"pageNumber": Номер запрошенной страницы
"pageSize": Количество элементов на одной странице
```
Из ответа на данный запрос необходимо из массива goods взять i-ый товар и сохранить его id = goods[i].id


**2. Сохранить параметры клиента (корзину) - этот шаг необязательный и его можно пропустить, но если есть потребность повторять полное флоу заказа - то этот шаг лучше оставить.**
```
POST http://marketplace-uapi-kit-test.tcsbank.ru/kit/application/{appId}/clientInfo/set

header:
{
   'content-type': 'application/json'
}

body:
{
  "commonProperties": {
    "interfaceType": "string",
    "platform": "string",
    "appName": "string",
    "appVersion": "string"
  },
  "clientId": "9-REGRESS107",
  "clientInfo": {
    "goodId": 34
  }
}


где:
appId - applicationId витрины с типом KIT
"commonProperties": Общие параметры запроса
"adult": Флаг: отображать или нет товары для взрослых
"clientId": Идентификатор клиента
"clientInfo": Данные клиента
"goodId": Идентификатор товара, полученный на шаге 1.
```
**3. Получаем список places для данной витрины**
```
POST http://marketplace-uapi-kit-test.tcsbank.ru/kit/application/{appId}/places
header:
{
   'content-type': 'application/json'
}

body:
{
  "commonProperties": {
    "interfaceType": "string",
    "platform": "string",
    "appName": "string",
    "appVersion": "string"
  },
  "adult": true,
  "clientId": "9-REGRESS107",
  "goods": [
    {
      "id": 34
    }
  ],
  "location": {
    "latitude": 33,
    "longitude": 55
  },
  "pageNumber": 0,
  "pageSize": 10
}

где:
appId - applicationId витрины с типом KIT
"commonProperties": Общие параметры запроса
"adult": Флаг: отображать или нет товары для взрослых
"clientId": Идентификатор клиента
"clientInfo": Данные клиента
"goodId": Идентификатор товара, полученный на шаге 1
"location": Геолокация клиента  
"location.latitude": Широта клиента
"location.longitude": Долгота клиента                                                            
"pageNumber": Номер запрошенной страницы
"pageSize": Количество элементов на одной странице
```

В ответе на данный запрос из массива places выбрать i-ое место, для которого places[i].aviable = true и массив places[i].goods содержит выбранный на 1-м шаге товар. 
Сохранить places[i].info.id

**4. Получаем информацию о выбранной АЗС**
```
POST http://marketplace-uapi-kit-test.tcsbank.ru/kit/application/{appId}/places/{placeId}/gasStationDetails
header:
{
   'content-type': 'application/json'
}

body:
{
  "commonProperties": {
    "interfaceType": "string",
    "platform": "string",
    "appName": "string",
    "appVersion": "string"
  },
  "clientId": "9-REGRESS107"
}

где:
appId - applicationId витрины с типом KIT
placeId- id места, полученный на 3-м шаге 
"commonProperties": Общие параметры запроса
"clientId": Идентификатор клиента
```

В ответе на данный запрос находится оставшаяся нужная для создания заказа информация. В массиве goods необходимо найти i-ый товар, для котого goods[i].id = id товара из шага 1.
Тогда можно сохранить:
"goods[i].priceValue": цена за 1л.
"goods[i].limits.min": Минимальный объем
"goods[i].limits.min": Максимальный объем
"extraGoods[i].id": id дополнительного товара, использующегося в заказе
"extraGoods[i].priceValue": цена доп. товара 

Получив все необходимые данные, можно создать заказ.

**5. Создание заказа**
```
POST http://marketplace-uapi-kit-test.tcsbank.ru/kit/application/{appId}/createOrder
header:
{
   'content-type': 'application/json'
}
body:
{
  "commonProperties": {
    "interfaceType": "string"
  },
  "clientId": "9-REGRESS107",
  "clientName": "AutoTestPostman",
  "clientPhone": "87778989900",
  "clientEmail": "qa@mail.com",
  "goods": [
    {
      "id": 34,
      "count": 10,
      "priceValue": 82.56
    }
  ],
  "sum": 825.6,
  "place": {
    "id": 1267
  }
}

где:
appId - applicationId витрины с типом KIT
"commonProperties": Общие параметры запроса
"clientId": Идентификатор клиента
"clientName": Имя клиента
"clientPhone": Телефон клиента
"clientEmail": 	E-mail клиента (необязательный)
"goods[i].id": id товара(ов), полученное на шаге 1
"goods[i].count": любое количетсво литриов с учетом limits.min и limits.max для данного товара 
"goods[i].priceValue": значение цены данного товара priceValue, полученные на шаге 4
"sum": Сумма заказа (рассчитывается как сумма goods[i].priceValue * goods[i].count )
"place": id места, полученное на шаге 3
```
При создании заказ переходит в статус created, если тип витрины PRE_PAYMENT или POST_PAYMENT, и в статус created_dynamic, если тип витрины NO_PAYMENT
В ответе на запрос создания заказа можно получить его id и foreignId для дальнейшей работы с ним.



**Подтверждение заказа:**
Оплату и потверждение тестового заказа можно сделать либо через PG(обращаться к Погонину Кириллу, либо через тестовое приложение). После подверждения заказа его статус принимает значение в зависимости от ответа
партнера: если значение parameter.val не определено или = '', то CREATED_DYNAMIC, иначе меняем статус на указанный в parameter.val (Должен быть один из CREATED_DYNAMIC, 
PROCESSING, DONE, иначе ошибка 500 Unknown internal error).





**Изменение статуса заказа:**
```
POST http://marketplace-uapi-kit-test.tcsbank.ru/kit/application/external/{appExtId}/order/foreign/{orderForeignId}/changeOrderStatus
header:
{
   'content-type': 'application/json'
}

{
  "commonProperties": {
    "interfaceType": "string",
    "platform": "string",
    "appName": "string",
    "appVersion": "string"
  },
  "comment": "string",
  "processingStage": "pending",
  "status": "processing",
  "newOrderDetails": {
    "goods": [
      {
        "count": 0,
        "foreignId": "string",
        "priceValue": 0
      }
    ],
    "sum": 0
  }
}

где:
appExtId - external id витрины с типом KIT
"commonProperties": Общие параметры запроса
"status": статус заказа (proccesing - заказ выполняется/done- заказ успешно выполнен) 
"processingStage": стадия выполнения заказа в статусе proccesing (pending - в ожидании начала исполнения/ in_progress - в процессе исполнения)
"comment": Комментарий (необязательное поле)
"newOrderDetails": Новые детали заказа (необязательно). С помощью этого параметра можно изменить количество товаров (добавлять новые нельзя) и общую сумму заказа, указав только те товары, которые надо оставить
```

**Отмена  заказа:**
```
POST http://marketplace-uapi-kit-test.tcsbank.ru/kit/application/{appId}/cancelOrder
{
   'content-type': 'application/json'
}
{
  "commonProperties": {
    "interfaceType": "string",
    "platform": "string",
    "appName": "string",
    "appVersion": "string"
  },
  "clientId": "9-REGRESS107",
  "orderId": 2000002769  ,
  "comment": "string",
  "type": "by_client",
  "amount": 688.6
}

где:
appId - applicationId витрины с типом KIT
"commonProperties": Общие параметры запроса
"clientId": Идентификатор клиента
"orderId": id отменяющегося заказа
"comment": Комментарий (необязательно)
"type": by_client/by_supervisor/by_supervisor_partially/by_partner/by_partner_partially (переводят заказ в статусы cancelled/canceled_by_supervisor/partially_canceled_by_supervisor/canceled_by_organization/partially_canceled соответственно)
"amount": сумма возврата 
```


















