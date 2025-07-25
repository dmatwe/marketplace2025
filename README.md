**Тема проекта «Проектирование маркетплейса»**

**Оглавление**

[1. Требования](#m1)

[2. System Design](#m2)

[3. Wireframe](#m3)

[4. Er диаграмма](#m4)

[5. Разработка спецификации OpenAPI для маркетплейса](#m5)

***Требования***
<a name="m1"></a>

Функциональные требования:
<br>
<br>
Покупатели:
<br>
1. Просмотр товаров, рекомендаций 
2. Оплачивать товары 
3. Получать уведомления SMS/email/Push
<br>
Продавцы:
<br>
1. Управлять своими товарами (добавлять, удалять, редактировать)
<br>
<br>
Нефункциональные требования:
<br> 
1. Отзывчивость системы (быстрая загрузка страницы)
2. Высокая доступность (площадка всегда доступна)


**System Design**
<a name="m2"></a>

![Image alt](https://github.com/dmatwe/marketplace2025/blob/main/img/systemd.png)


***Wireframe***
<a name="m3"></a>

![Image alt](https://github.com/dmatwe/marketplace2025/blob/main/img/wireframe.png)


***Er диаграмма***
<a name="m4"></a>

![Image alt](https://github.com/dmatwe/marketplace2025/blob/main/img/er.png)

```plantUML

@startuml
skinparam packageStyle rectangle
skinparam package {
    BackgroundColor #f8f9fa
    ArrowColor black
    LineColor black
}

entity Пользователь {
    id : int pk
    username : varchar(100)
    password : varchar(255)
}

entity Категория {
    id : int pk
    название : varchar(255)
}

entity Товар {
    id : int pk
    название : varchar(255)
    цена : decimal(10,2)
    categoryId : int fk
}

entity Заказ {
    id : int pk
    status : varchar(50)
    deliveryMethod : varchar(20)
    paymentMethod : varchar(20)
    orderDate : datetime
}

entity Позиция_Заказа {
    id : int pk
    id_заказа : int fk
    id_товара : int fk
    количество : int
}

entity Токен {
    id : int pk
    access_token : varchar(500)
    id_пользователя : int fk
    дата_создания : datetime
}

Пользователь "1" -- "*" Токен : имеет
Пользователь "1" -- "*" Заказ : делает
Категория "1" -- "*" Товар : содержит
Заказ "1" -- "*" Позиция_Заказа : включает
Товар "1" -- "*" Позиция_Заказа : содержится_в

@enduml

```


***Разработка спецификации OpenAPI для маркетплейса*** 
<a name="m5"></a>

[YAML FILE](https://github.com/dmatwe/marketplace2025/blob/main/file/openapi.yaml)

```openapi
openapi: 3.0.0 # Обязательное поле — версия спецификации OpenAPI, с которой вы работаете.

info: # Обязательный блок с общей информацией об API.
  title: Marketplace Client API # Название API, отображается в документации.
  version: '1.0' # Версия API, чтобы отслеживать изменения и совместимость.
  description: | # Краткое описание назначения API.
    Простая спецификация для клиентского API маркетплейса.
    

servers: # Список серверов, где работает API.
  - url: https://api.marketplace.com/v1 # Адрес, на который отправляются запросы.
  

components: # Переиспользуемые части спецификации — схемы, авторизация, ответы и т.п.

  securitySchemes: # Способы авторизации.
    BearerAuth: # Название схемы авторизации. Так будем ссылаться на неё в других местах.
      type: http # Тип схемы — HTTP (есть еще apiKey, oauth2 и др.).
      scheme: bearer # Указывает, что используется схема Bearer.
      bearerFormat: JWT # Формат токена — JWT (JSON Web Token).
      description:  Используйте JWT токен в заголовке Authorization # Описание, как использовать токен.
      
  schemas: # Описание структур данных, которые используются в запросах и ответах.
  
    AuthRequest: # Схема данных, которые отправляют при авторизации.
      type: object
      properties:
        username:
          type: string
          example: user1
        password:
          type: string
          example: mysecretpassword
      required: # Обязательные поля запроса.
        - username
        - password
        
    AuthResponse: # Схема ответа на успешную авторизацию.
      type: object
      properties:
        access_token:
          type: string
          example: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
          
    Category: # Описание категории товара.
      type: object
      properties:
        id:
          type: integer
          example: 1
        name:
          type: string
          example: Электроника
          
          
    Product: # Описание товара.
      type: object
      properties:
        id:
          type: integer
          example: 42
        name:
          type: string
          example: Смартфон
        price:
          type: number
          example: 19999.99
        categoryId:
          type: integer
          example: 1
          

    Order: # Описание заказа.
      type: object
      properties:
        id:
          type: integer
        items:
          type: array
          items:
            type: integer # ID товаров в заказе
        status:
          type: string
          example: pending # Например, "ожидание"
        deliveryMethod:
          type: string
          description: Способ доставки
          enum: [courier, pickup] # Список допустимых способов доставки
          example: courier
        paymentMethod:
          type: string
          description: Способ оплаты
          enum: [card, on_delivery] # Список допустимых способов оплаты
          example: card
        orderDate:
          type: string
          format: date-time # Специальный формат для даты-времени
          description: Дата и время заказа в формате ISO 8601
          example: '2024-06-27T12:34:56Z'
          
          
security: # Глобальная настройка: все эндпоинты требуют BearerAuth (токен).
  - BearerAuth: []
  
  
  
paths: # Описание всех маршрутов (эндпоинтов) API.


  /auth/login: # Эндпоинт для авторизации пользователя.
    post: # POST-запрос (отправка данных).
      summary: Генерация JWT-токена по логину и паролю # Краткое описание.
      description: |
        Передайте логин и пароль, чтобы получить JWT-токен для авторизации.
      requestBody: # Описание принимаемых данных.
        required: true
        content:
          application/json: # Ожидается JSON.
            schema:
              $ref: '#/components/schemas/AuthRequest' # Схема запроса.
            example:
              username: user1
              password: mysecretpassword
      responses: # Описание возможных ответов сервера.
        '200':
          description: Успешная авторизация, возвращён JWT-токен
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/AuthResponse'
              example:
                access_token: eyJhbGciOiJIUzI1NiIsInT5cCI6IkpXVCJ9...
        '401':
          description: Неверный логин или пароль
          
          
  /categories: # Получение списка категорий товаров.
    get:
      summary: Получить все категории товаров
      security:
        - BearerAuth: [] # Требуется авторизация
      responses:
        '200':
          description: Успешный ответ
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Category'
        '401':
          description: Не авторизован
          
          
  /products: # Получение списка товаров (с возможностью фильтрации по категории).
    get:
      summary: Получить все товары
      security:
        - BearerAuth: []
      parameters:
        - name: categoryId
          in: query # Передаётся через строку запроса: ?categoryId=2
          schema:
            type: integer
          description: Фильтр по категории
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Product'
        '401':
          description: Не авторизован
          
  /products/{id}: # Получение информации о товаре по его ID.
    get:
      summary: Получить товар по ID
      security:
        - BearerAuth: []
      parameters:
        - name: id
          in: path # Переменная пути, например /products/42
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Product'
        '404':
          description: Товар не найден
        '401':
          description: Не авторизован
  
  
  /orders: # Работа с заказами пользователя.
    post: # Создать новый заказ.
      summary: Создать новый заказ
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                items:
                  type: array
                  items:
                    type: integer
                deliveryMethod:
                  type: string
                  enum: [courier, pickup]
                  example: courier
                paymentMethod:
                  type: string
                  enum: [card, on_delivery]
                  example: card
      responses:
        '201':
          description: Заказ создан
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '400':
          description: Ошибка в данных заказа
        '401':
          description: Не авторизован

    get: # Получить список заказов пользователя.
      summary: Получить список заказов пользователя
      security:
        - BearerAuth: []
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Order'
        '401':
          description: Не авторизован
          
          
  /orders/{id}: # Работа с отдельным заказом пользователя.
    get: # Получить заказ по ID.
      summary: Получить заказ по ID
      security:
        - BearerAuth: []
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: ОК
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '404':
          description: Заказ не найден
        '401':
          description: Не авторизован
          
    patch: # Частичное обновление заказа (например, смена способа доставки).
      summary: Изменить способ доставки или оплаты заказа
      security:
        - BearerAuth: []
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                deliveryMethod:
                  type: string
                  enum: [courier, home, pickup]
                paymentMethod:
                  type: string
                  enum: [card, on_delivery]
      responses:
        '200':
          description: Параметры заказа изменены
        '400':
          description: Некорректные данные
        '404':
          description: Заказ не найден
          
          
    delete: # Отмена (удаление) заказа.
      summary: Отменить заказ
      security:
        - BearerAuth: []
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '204':
          description: Заказ отменён
        '404':
          description: Заказ не найден 
```