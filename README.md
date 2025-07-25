**Тема проекта «Проектирование маркетплейса»**

***Требования***

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

![Image alt](https://github.com/dmatwe/marketplace2025/blob/main/img/systemd.png)


***Wireframe***

![Image alt](https://github.com/dmatwe/marketplace2025/blob/main/img/wireframe.png)


***Er диаграмма***

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